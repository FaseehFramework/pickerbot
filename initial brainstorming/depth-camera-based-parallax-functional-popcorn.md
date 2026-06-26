# Depth-Camera Based Parallax Elimination, Obstacle Detection, and Grip-Strength Reporting

## Context

The existing picker-bot system uses a 2D webcam + YOLOv8-OBB + a one-plane homography to detect and pick microelectronic modules (Arduino, ESP32, LCD) with an Epson VT6-A901S over TCP socket. It assumes every pickable object's top surface lies at a fixed `robot_z = 360 mm`, which is wrong for items with non-trivial height (causes parallax-induced XY error and crashes for tall items). It also has no awareness of obstacles in the workspace, no per-object grip-force information, and no model of the end-effector geometry.

This change adds an **Intel RealSense D435i**-driven depth pipeline that runs alongside YOLOv8 to:

1. Eliminate parallax by computing the true 3D top-surface centroid of every detected object and using that as the pick target (instead of the homography-projected 2D centroid).
2. Detect obstacles two ways: (a) any depth cluster that no YOLO detection corresponds to, and (b) any intrusion into the approach cone above each planned pick.
3. Report a per-label grip-strength index from a JSON lookup table (logged only — not yet wired to the gripper).
4. Account for end-effector geometry (axis-symmetric cylinder, all dims in config) when sizing the approach cone and computing the TCP offset.
5. Deliver a dynamic clearance height to the robot via an extended `PICK x y z u clearance` TCP command.

The new pipeline is delivered as a parallel entry-point (`pickerbot_depth.py`) so the existing webcam workflow keeps working untouched. Hardware is not yet available, so the depth source is abstracted to work against live `pyrealsense2`, `.bag` playback, or saved `.npy` frame pairs.

---

## Approach

### Critical design decisions (locked with user)

| # | Decision | Choice |
|---|---|---|
| 1 | Camera mount | Fixed overhead (eye-to-hand) |
| 2 | Vision strategy | YOLOv8 + depth segmentation in parallel, cross-checked |
| 3 | Grip-strength source | Per-label JSON lookup, logged only |
| 4 | End-effector model | Generic axis-symmetric cylinder, dims in `config.json` |
| 5 | Obstacle definition | Unknown depth cluster OR intrusion into approach cone |
| 6 | Obstacle response | Compute dynamic clearance, lift over |
| 7 | Robot protocol | Extend `PICK` to take optional 5th field `clearance` |
| 8 | Camera↔robot calibration | Full extrinsic via charuco + `cv2.calibrateHandEye` (eye-to-hand) |
| 9 | Hardware status | Develop against `.bag` / `.npy`, abstraction allows live swap |
| 10 | Packaging | New `pickerbot_depth.py` + new module(s); existing pipeline untouched |

### Architectural defaults (chosen with rationale, override in config)

- **Pick ordering**: tallest-first. Sorted by descending `z_top` before dispatch. Reason: picking a short item next to a tall neighbour risks the gripper colliding with the neighbour during approach.
- **Clearance failure**: if even the configured `max_clearance` (default 200 mm) cannot clear the approach cone, that pick is **skipped with a logged warning**; batch continues. Mirrors the conservative "skip and continue" pattern.
- **Detection completeness**: drop a YOLO detection if fewer than 70% of pixels inside its OBB return valid depth (handles partial occlusion).
- **Top-surface segmentation**: inside each OBB, only points within `top_band_mm` (default 8 mm) of the **minimum** depth in that crop count as "top surface." This is the depth-percentile filter, not generic outlier rejection.
- **Z convention**: confirmed from `Main.prg` (`Jump3 Here :X(x) :Y(y) :Z(z + 50)` then `... :Z(z)` is descent), Epson base frame uses **+Z up**. Camera-frame Z (depth, positive into scene) must be flipped during extrinsic transform.
- **Grip-strength schema**: `{"_meta": {"unit": "grams", "default": 30}, "labels": {"lcd": 40, "arduino": 25, "esp32": 20}}`. Unknown labels fall back to `_meta.default`.

---

## Files

### New files

| Path | Purpose |
|---|---|
| `pickerbot_depth.py` | Main entry point. Mirrors `pickerbot.py` structure but uses depth pipeline. |
| `pickerbot_lib/depth.py` | RealSense source abstraction. Class `DepthSource` with three subclasses: `LiveDepthSource` (pyrealsense2 device), `BagDepthSource` (recorded .bag with `set_real_time(False)` for determinism), `NpyDepthSource` (paired `color.npy` / `depth.npy` frames for unit tests). Returns `(color_bgr, depth_mm, intrinsics)`. Applies the rs filter chain (decimation → disparity → spatial → temporal → disparity-back) when source is live or bag. |
| `pickerbot_lib/parallax.py` | `compute_top_surface_centroid(depth_crop, top_band_mm)` returns 3D centroid in camera frame using the depth-percentile filter. `correct_pick_target(detection, depth, intrinsics, extrinsics, ee_config)` returns `(x_base, y_base, z_top_base, angle, completeness_ratio)`. Uses the extrinsic transform directly — does NOT route through the legacy homography. |
| `pickerbot_lib/obstacle.py` | `segment_plane(point_cloud)` RANSAC plane fit (table); `cluster_above_plane(points, eps_mm, min_pts)` DBSCAN. `cross_check_clusters_vs_detections(clusters, detections, iou_thresh)` returns `(matched_detections, unknown_obstacles)`. `approach_cone_clearance(target_xyz_base, ee_radius, safety_margin, depth, intrinsics, extrinsics, full_obstacle_cloud, max_clearance)` returns required clearance in mm, or `None` if blocked. Samples cone in 3D base frame (cylinder surface points at varying heights), projects each to pixel, looks up depth — handles elliptical perspective footprint correctly. |
| `pickerbot_lib/extrinsics.py` | `load_extrinsics(path) -> T_cam2base (4×4)`; `transform_camera_to_base(points_cam, T)`; `transform_base_to_camera(points_base, T)`. Strict about Z-axis sign convention (camera +Z = forward into scene; base +Z = up; transform handles flip). |
| `pickerbot_lib/grip.py` | `load_grip_table(path)`, `get_grip_strength(label) -> int` (returns `_meta.default` for unknown labels). |
| `tools/extrinsic_calibrator.py` | Capture loop: for each robot pose, grab RGB, detect charuco corners, estimate `T_target2cam` via `cv2.aruco.estimatePoseCharucoBoard`. Collect ≥15 poses with rotational diversity. Invert each `T_gripper2base` from the robot's reported pose (must be entered manually, or read via a new `WHERE` SPEL+ command — see Open Questions). Call `cv2.calibrateHandEye(R_base2gripper, t_base2gripper, R_target2cam, t_target2cam, method=cv2.CALIB_HAND_EYE_DANIILIDIS)`. Output is `T_cam2base`. Save to `data/calibration/extrinsics.json`. Compute reprojection residual on held-out pose; refuse to save if > 5 mm. |
| `tools/record_bag.py` | Connect to live RealSense, record N seconds to a `.bag` file in `data/recordings/`. Used to capture footage once hardware arrives. |
| `grip_strengths.json` | The lookup table at project root. |
| `data/calibration/extrinsics.json` | Saved 4×4 extrinsic matrix + metadata (date, robot poses used, residual). |
| `data/recordings/` | Directory for `.bag` files (gitignored). |
| `tests/test_parallax.py` | Synthetic depth crop → top-surface centroid; round-trip with known extrinsic. |
| `tests/test_obstacle.py` | Synthetic point cloud with plane + 2 clusters; verify segmentation + cross-check. |
| `tests/test_grip.py` | Lookup hits + default fallback. |

### Modified files

| Path | Change |
|---|---|
| `pickerbot_lib/sender.py` | `epsonPick(x, y, z, u, clearance=None)` — only emits the 5th token if `clearance is not None`, preserving backward compatibility. `epsonPickAll(locations)` updated to accept either 4-tuple or 5-tuple (and matching 4-/5-field strings). Detected by token count after split. |
| `pickerbot_lib/__init__.py` | Export the new modules. |
| `Epson/Pickerbot_Receiver/Pickerbot_Receiver/Main.prg` | PICK handler: declare `Double clearance` initialised to 50; if `UBound(indata$()) >= 5` and `Val(Trim$(indata$(5))) > 0`, use that value; pass to `Jump3 Here +Z(clearance), Here :X(x) :Y(y) :Z(z + clearance) :U(u), Here :X(x) :Y(y) :Z(z) :U(u)`. Falling back to 50 on missing or invalid value preserves legacy behaviour. **No other commands touched.** |
| `config.json` | Add three blocks: <br>`"end_effector": {"tcp_z_mm": 120, "radius_mm": 25, "length_mm": 80, "safety_margin_mm": 8}` <br>`"realsense": {"source": "bag", "bag_path": "data/recordings/dev.bag", "width": 1280, "height": 720, "fps": 30}` <br>`"depth_pipeline": {"top_band_mm": 8, "ransac_distance_mm": 5, "dbscan_eps_mm": 15, "dbscan_min_pts": 50, "iou_match_thresh": 0.3, "completeness_thresh": 0.7, "approach_clearance_default_mm": 50, "max_clearance_mm": 200, "pick_order": "tallest_first"}` <br>`"extrinsics_file": "data/calibration/extrinsics.json"` <br>`"grip_strengths_file": "grip_strengths.json"` |

---

## Pipeline (in `pickerbot_depth.py`)

1. Load config, `grip_strengths.json`, `extrinsics.json`, YOLOv8 model, BOQ.
2. Open `DepthSource` per `config.realsense.source` — capture one aligned `(color_bgr, depth_mm, intrinsics)` frame (for live mode, a key press triggers capture, identical to existing `pickerbot.py` UX).
3. Run YOLOv8-OBB on the colour frame → list of `(cx, cy, angle, label, conf, obb_polygon)`.
4. Build full point cloud from depth + intrinsics. RANSAC fit table plane. Cluster points above plane (DBSCAN).
5. **Cross-check**:
   - For each YOLO detection: compute completeness (valid-depth ratio inside OBB); drop if < threshold. Match to nearest depth cluster by IoU of 2D bbox. Unmatched YOLO detections → dropped with log.
   - For each depth cluster: if no YOLO match → tagged as `unknown_obstacle`.
6. For each surviving (matched) detection:
   - Extract depth points inside OBB; keep only those within `top_band_mm` of the minimum depth (the top-surface filter the Plan agent flagged).
   - Compute their 3D centroid in camera frame; transform to base via `T_cam2base`. This is `(x_base, y_base, z_top_base)`.
   - Subtract `ee_config.tcp_z_mm` from `z_top_base` to get the commanded robot Z (so the tool **tip** lands on the top surface, not the flange).
7. **Pick ordering**: sort by descending `z_top_base` (tallest first).
8. For each pick in sorted order:
   - Approach-cone check: cylinder centred on `(x_base, y_base)`, radius `ee_config.radius_mm + safety_margin_mm`, from `z_top_base` up to `z_top_base + approach_clearance_default_mm`. Sample N=64 points on the cylinder surface in base frame, transform each to camera frame, project to pixel, sample depth. Compare against the **full obstacle cloud minus the current target's own cluster** (the Plan agent's "neighbour as obstacle" fix). If intrusion: increase clearance in 10 mm steps up to `max_clearance_mm`. If still blocked: log + skip pick.
   - Look up grip strength for label; print report line: `<label> at (x, y, z) angle <u>°, grip=<g>g, clearance=<c>mm`.
9. If `enable_epson_tcp` is true: open socket, dispatch `PICK x y z u clearance` for each. Abort batch on non-OK reply (existing behaviour).

---

## Existing functions / utilities being reused

- `pickerbot_lib.detection.detect_and_annotate` — unchanged, called for YOLO step.
- `pickerbot_lib.config.CONFIG`, `resolve` — extended with new blocks, same loader.
- `pickerbot_lib.sender.connect`, `disconnect`, `epsonStandby` — unchanged. `epsonPick` gets a new optional parameter.
- `pickerbot_lib.calibration.load_calibration_data`, `pixel_to_world` — kept as **legacy fallback only** for users running `pickerbot.py`. Not used in the depth pipeline.
- `boq.txt` loader + `filter_by_boq` from `pickerbot.py` — duplicated/shared into `pickerbot_lib/boq.py` (small refactor to avoid copy-paste); both entry points import it.

---

## Verification

### Unit tests (CI-able, no hardware)

- `pytest tests/test_parallax.py` — synthetic top-of-box depth crop returns the geometric centroid within 1 mm; round-trip `camera→base→camera` with known extrinsic is identity.
- `pytest tests/test_obstacle.py` — synthetic point cloud (plane + 2 clusters, one with matching "YOLO" bbox, one without) yields exactly one obstacle.
- `pytest tests/test_grip.py` — known labels return correct grams, unknown returns `_meta.default`.

### Offline integration test

- Record (or place by hand) a `.bag` containing a flat surface with one Arduino, one LCD, and a tall coffee mug (the "obstacle"):
  - Run `python pickerbot_depth.py --src bag --bag data/recordings/dev.bag --no-robot`.
  - Expect: console reports two picks (Arduino, LCD), each with depth-corrected XY and a non-zero `z_top`. The mug is flagged as `unknown_obstacle`. Approach cones print clearances ≥ 50 mm. Picks sorted tallest-first (LCD before Arduino if LCD is taller).
- Repeat with the mug placed directly in the LCD's approach cone: expect LCD's clearance to grow above mug height; if mug exceeds `max_clearance_mm`, expect LCD to be skipped with `BLOCKED` log.

### SPEL+ test (simulator)

- Boot the Epson RC8 simulator at `127.0.0.1:2001` with the modified `Main.prg`.
- From a Python REPL: `connect(); epsonPick(0, 470, 360, 0)` → expect legacy behaviour (50 mm clearance). `epsonPick(0, 470, 360, 0, clearance=120)` → expect taller jump. `epsonPick(0, 470, 360, 0, clearance=0)` → expect fallback to 50 (guards against malformed value).

### Hardware demonstration (once camera arrives)

1. Run `python tools/extrinsic_calibrator.py` with the charuco board mounted on the gripper; jog the robot to ≥15 diverse poses. Save `extrinsics.json`. Reprojection residual must be < 5 mm.
2. Run `python tools/record_bag.py --seconds 5` to capture a live frame of the populated workspace; commit to verify the .bag pipeline works against fresh data.
3. Switch `config.realsense.source` to `live`. Run `python pickerbot_depth.py --src camera` and confirm the same flagged results appear with live frames.
4. Enable `enable_epson_tcp` and dispatch a real pick batch with the robot on, verifying the dynamic clearance behaves as expected for a deliberately tall obstacle.

---

## Open questions / deferred items

These are known incompletenesses, called out so they don't get lost — none block this plan:

- **Robot pose ingestion for hand-eye calibration**: the calibrator needs `T_gripper2base` per capture. Two paths: (a) the user types in the robot's `Here` readout for each pose (works now), or (b) add a `WHERE` SPEL+ command that prints the current pose over TCP. (b) is cleaner but adds robot-side code; defer to a follow-up unless the user wants it in scope now.
- **Calibration drift detection**: not in this iteration. Future: at startup, the robot moves to a known fiducial in the workspace, the camera locates it, and the residual is logged.
- **Grip-strength → robot wiring**: this iteration logs only. When the gripper gains force feedback, extend the PICK protocol again with a `grip_g` field.
- **`epsonPickAll` legacy callers**: existing usage in `pickerbot.py` passes 4-field strings. The updated parser must accept 4 or 5; a unit test in `tests/test_sender.py` should cover both shapes to prevent regression.