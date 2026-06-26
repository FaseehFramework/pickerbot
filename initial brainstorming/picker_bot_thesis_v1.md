# Height-Aware Pick Sequencing for Robust Pick-and-Place in Unstructured, Mixed-Height Workspaces: A Depth-Driven Approach on an EPSON VT6 Manipulator

| | |
|---|---|
| **Student** | Faseeh Mohammed (M01088120) |
| **Module** | PDE4445 — Robotics Dissertation Project |
| **Programme** | MSc Robotics — Middlesex University Dubai |
| **Supervisor** | JP |
| **Platform** | EPSON VT6-A901S 6-axis manipulator · Intel RealSense **D435** · YOLOv8-OBB |
| **Companion thesis** | Thesis 2 (buddy) — custom end-effector design & pick verification |
| **Document** | Thesis design document — v1 (consolidated) |
| **Date** | June 2026 |

> **Note on scope vs. earlier proposal.** Earlier proposal drafts (v3/v4) framed this as a ROS2 + MoveIt2 sim-to-real project and quoted a ~15,000-word dissertation. This document supersedes that. The spine is now a **direct (non-ROS) depth pipeline** — lower integration risk for a 12-week window — and the deliverable, per the PDE4445 handbook, is a **research article of ~8 pages** plus a **blog (40%)** and **presentation (20%)**, *not* a 15k-word dissertation. ROS2/MoveIt2 is retained only as documented future work.

---

## Abstract

Vision-guided pick-and-place on industrial manipulators is mature for flat, uniform-height parts but remains brittle when components rest at arbitrary heights on an unstructured surface — the setting produced when parts are tipped from a bucket onto a table. The existing *Picker-Bot* system (YOLOv8-OBB detection + a single-plane homography on an EPSON VT6-A901S) assumes every pickable object's top surface lies at a fixed height; this assumption produces parallax-induced XY error and collision risk for any part not lying flat at the calibration height, and it carries no model of obstacles or of the order in which parts should be picked.

This work replaces the planar pipeline with a depth-driven one built on an Intel RealSense D435 in a fixed eye-to-hand configuration, and investigates a single central hypothesis: **that sequencing picks by descending object height ("tallest-first") measurably improves pick success and reduces collisions in unstructured, mixed-height scenes, relative to an unordered baseline — at negligible compute and sensing cost, using only the depth map.** A supporting mechanism, dynamic per-pick approach-cone clearance, is justified independently. The two mechanisms are evaluated together in a 2×2 factorial experiment (sequencing × clearance) on the physical arm, across controlled variation in lighting, height spread, and occlusion, against the prior 2D baseline. The contribution is a lightweight, depth-native sequencing heuristic — validated empirically on real hardware — positioned against the heavier reachability- and learning-based sequencing methods in the literature, together with a clean perception-side interface to a companion end-effector/verification subsystem.

---

## 1. Problem Definition

*Picker-Bot* is a computer-vision-guided pick-and-place system built around an EPSON VT6-A901S 6-axis manipulator. Its perception front-end — a YOLOv8 Oriented Bounding Box (OBB) detector — has been demonstrated on real camera frames against three microelectronic component classes (Arduino, ESP32, LCD module). Everything downstream of detection is fragile or untested on hardware:

1. **The planar-height assumption breaks under real conditions.** Coordinate resolution uses a 77-point homography grid collected once at a fixed camera height, and the robot descends to a fixed `robot_z = 360 mm`. Any component not lying flat at the calibration height yields (a) a wrong descent depth and (b) **parallax error**: an overhead camera projects the *top* of a tall object to an image position offset from its true XY footprint, so the homography returns a laterally shifted pick point. The error grows with object height.

2. **There is no awareness of obstacles or of pick order.** The system has no model of the workspace beyond the detected targets. In a cluttered, mixed-height scene a top-down approach to a short part can collide with a taller neighbour, and the system has no mechanism to avoid this, nor any principled order in which to clear the surface.

3. **There is no end-effector geometry model and no verification.** Picks are open-loop: detect → move → close → assume success. The tool's physical envelope is not accounted for when planning the approach, and nothing checks whether a pick actually succeeded.

This dissertation addresses (1) and (2) directly, and (3) on the perception side only — the gripper hardware and its proprioceptive grasp signal belong to the **companion Thesis 2**. The central scientific question concerns *pick ordering*; the depth pipeline is the instrument that makes ordering, parallax correction, and obstacle clearance possible.

### Scope boundaries (stated up front)

- **In scope:** opaque microelectronic modules resting (possibly stacked) on a flat table; 4-DoF picking `(x, y, z, yaw)` under a planar-rest assumption; fixed overhead camera.
- **Out of scope (declared limitations, not failures):** full 6-DoF pose of arbitrarily tilted parts; **specular/transparent** depth recovery (an open research problem — see §3); the end-effector mechanism and its force sensing (Thesis 2).

---

## 2. Research Questions & Hypotheses

**Central hypothesis (H1).**
> In an unstructured, mixed-height workspace, sequencing picks by **descending object height** improves end-to-end pick success rate and reduces gripper–neighbour collisions, relative to an unordered (random/fixed) pick order.

**Supporting hypothesis (H2).**
> **Dynamic** per-pick approach-cone clearance (lifting the approach height to clear intruding obstacles) reduces collisions and skipped picks relative to a fixed clearance — and is *complementary* to H1: sequencing removes tall obstacles that clearance alone cannot lift over within the arm's limits, while clearance handles residual obstacles that cannot be reordered away.

**Perception question (Q3).**
> Does depth-based 3D pose estimation (parallax-corrected top-surface centroid) reduce pick-point error versus the 2D homography baseline as object height departs from the calibration plane, and by how much (MAE in X/Y/Z)?

**Robustness question (Q4).**
> How do detection and pick performance degrade as a function of lighting, height spread, and occlusion (low/med/high)? Occlusion is **characterised, not solved**.

**Integration question (Q5).**
> Can a stable perception-side interface (object poses, pick/verify signals) cleanly decouple this subsystem from the companion end-effector/verification subsystem?

---

## 3. Existing Solutions & The Gap

Five clusters frame the related work. For each: the state of the art, why it is insufficient *for this problem*, and what this work does instead. (This section is the spine of the Week 1–2 literature review JP asked for — "2–3+ papers, existing solutions, why not sufficient, how is mine different.")

### 3.1 RGBD 6-DoF pose estimation & grasp planning for bin-picking
**SOTA:** Deep 6-DoF grasp/pose estimation from RGBD is mature — comprehensive surveys (Du et al., *Robotic Grasping from Classical to Modern*, 2022; *Deep Learning-Based Object Pose Estimation: A Survey*, IJCV 2025) and industrial systems (e.g. *Pickalo*, 2026) demonstrate reliable cluttered-scene picking.
**Why insufficient here:** these methods target generic objects and typically assume a capable sensor; almost none address **small, thin, low-texture, glossy microelectronic modules** on a budget close-range sensor, and they rarely isolate **pick *order*** as a controlled variable.
**What I do:** a deliberately lightweight depth-only pipeline tuned for small modules, with ordering as the object of study rather than a by-product.

### 3.2 Pick sequencing / clutter removal (the home of H1)
**SOTA:** Sequencing in clutter is a studied problem — target retrieval in clutter (Nam et al., 2020), online re-planning under clutter (2018), optimal clutter removal (2019), and recent cost-map sequencing (*ClutterNav*, 2025). Production bin-pickers rank targets by occlusion count, fail-counters and confidence.
**Why insufficient here — and the trap to avoid:** these methods explicitly do **not** "just pick the tallest/topmost first" — they score reachability, occlusion and stability, and often require full object pose, learned policies, or expensive combinatorial planning. **I therefore do *not* claim tallest-first is the optimal policy** (that argument is lost to this literature).
**What I do — the defensible claim:** a **cheap, depth-only height ordering**, computable directly from the depth map with zero extra modelling, recovers a *statistically significant share* of the collision-avoidance benefit on real hardware at negligible cost. The contribution is "sufficient and cheap," not "optimal."

### 3.3 Closed-loop grasp verification & failure recovery (Q5 / H-stretch)
**SOTA:** Most deployed industrial PnP is **open-loop**. Closed-loop verification is recent and not yet standard — e.g. multimodal failure handling fusing vision + gripper proprioception (Frontiers in Neurorobotics, 2021), and VLM-driven closed-loop manipulation (2024).
**Why insufficient here:** these assume control of the gripper's sensing; in this project the gripper is Thesis 2's.
**What I do:** own the **exteroceptive (vision) modality** — re-scan the ROI after a pick to confirm the object left the scene — and the **recovery re-plan**; fuse with Thesis 2's proprioceptive boolean across a defined interface. Vision verification is core; automated recovery is a stretch goal.

### 3.4 Sim-to-real & robustness of vision-guided grasping (Q4)
**SOTA:** Robust visual sim-to-real and domain randomisation (2023–2025; *ClutterDexGrasp*, 2025; ICRA 2025 Sim2Real PnP).
**Why insufficient / terminology caution:** "sim-to-real" in the literature means transferring *learned policies* from simulation via randomisation. This project's "sim-to-real" means **moving from the RC8 virtual controller to the physical arm** — i.e. deployment/commissioning, not policy transfer. The term is used carefully to avoid overclaiming.
**What I do:** empirically characterise robustness to lighting/height/occlusion on real hardware rather than via randomised training.

### 3.5 ROS2/MoveIt2 integration with proprietary arms (deferred context)
**SOTA:** `ros2_control` hardware-interface plugins and vendor gateways exist (Intel MC Gateway; Flexiv ROS2 bridge; community `ros2-igus-rebel`), but **no community ROS2 driver exists for EPSON SPEL+ arms.**
**Why deferred:** the EPSON RC8 controller does its own trajectory planning in SPEL+; feeding it MoveIt2 trajectories means either fighting the controller or down-sampling to waypoints it re-plans anyway — high risk for 12 weeks.
**What I do:** retain the working SPEL+ TCP link and treat ROS2/MoveIt2 integration as **documented future work**, noting the bridge as a known gap others could fill.

---

## 4. Contributions

1. **C1 (primary):** An empirical demonstration, on a physical industrial manipulator, that a lightweight depth-only **height-ordered pick sequence** significantly improves pick success / reduces collisions in unstructured mixed-height scenes — with effect sizes and significance, against an unordered baseline.
2. **C2:** A **dynamic approach-cone clearance** mechanism, justified independently and shown to be complementary to C1 via a 2×2 factorial.
3. **C3:** A depth-driven **parallax-correction** pipeline quantitatively compared to the prior 2D homography baseline (pose MAE vs height).
4. **C4:** A **vision-based pick-verification** check and a **published interface contract** decoupling this subsystem from the companion end-effector/verification thesis.
5. **C5:** A characterised **failure envelope** (lighting / height / occlusion) and a full **error budget** for the pick point — engineering rigor suitable for the report and viva.

---

## 5. Twelve-Week Timeline & Milestones

**Official PDE4445 deliverables (from the module handbook, Table 1):**

| # | Deliverable | Weight | Due |
|---|---|---|---|
| 1 | **Project blog** — weekly updates, multimedia, final video; evidence of planning/time-management | **40%** | **26 Jul 2026** |
| 2 | **Project report** — *research article of ~8 pages*; clarity, organisation, referencing, figures/tables, design-choice documentation, technical quality | **40%** | **25 Sep 2026** |
| 3 | **Final presentation** — oral, to supervisor + second marker | **20%** | **Oct 2026 (26 Oct)** |

> ⚠️ The report is **~8 pages (research-article format)**, not 15,000 words. Plan for density and selectivity. The **blog is worth as much as the report** — treat continuous documentation as a recurring deliverable, not an afterthought.

**12-week plan (anchored to the handbook's own week schedule).** Blog upkeep is implicit every week.

| Week | Dates (w/c) | Focus | Output / milestone |
|---|---|---|---|
| **1** | 30 Jun | **Literature review** across the 5 clusters (§3); finalise objectives + Gantt; set up blog | Blog live; objectives; §3 draft (JP: "lit review first 2 weeks") |
| **2** | 07 Jul | Literature review cont'd; **experiment design** — what to test & how (§7); scaffold depth pipeline + offline `.npy`/`.bag` harness | Experiment plan; reproducible test harness |
| **3** | 14 Jul | **D435 bring-up**: intrinsics + **eye-to-hand hand-eye calibration**; report reprojection residual; report-writing template | Calibration residual logged (<5 mm target) |
| **4** | 21 Jul | Parallax / top-surface-centroid pose pipeline; fuse with YOLOv8-OBB | **📌 BLOG DUE 26 Jul (40%)** — problem, lit review, plan, early results |
| **5** | 28 Jul | Obstacle detection (RANSAC plane + DBSCAN) + **dynamic approach-cone clearance (H2)**; **height-sort ordering (H1)** | Both mechanisms implemented |
| **6** | 04 Aug | **Vision pick-verification** + Thesis 2 **interface contract** (signed off with buddy + JP); build **ground-truth rig** | Interface spec v1; GT method fixed |
| **7** | 11 Aug | Pilot trials; finalise **scene designs** (mixed-height, density levels); metric logging | Pilot data; scenes frozen |
| **8** | 18 Aug | **Main experiment — block 1:** 2×2 factorial (sequencing × clearance), baseline lighting | Core dataset (H1/H2) |
| **9** | 25 Aug | **Main experiment — block 2:** lighting × height × occlusion sweeps; **2D-vs-3D** pose-MAE comparison | Robustness + baseline dataset |
| **10** | 01 Sep | **Analysis:** significance tests, effect sizes, **error budget**, **failure envelope**, ablations | Results finalised |
| **11** | 08 Sep | **Write the ~8-page research article** (draft); figures/tables; supervisor review | Report draft (handbook: 14 Sep submission review) |
| **12** | 15 Sep | Finalise report; polish blog + final video; buffer | **📌 REPORT DUE 25 Sep (40%)** |
| **post** | Oct | Presentation prep & rehearsal | **📌 PRESENTATION 26 Oct (20%)** |

**Critical-path risk:** every experiment week depends on the D435 and arm being available. Mitigation in §8; the offline `.bag`/`.npy` harness (Week 2) lets perception development and unit tests proceed *without* hardware.

---

## 6. System Architecture & Technical Design

### 6.1 Platform & data flow
Fixed **eye-to-hand** D435 mounted overhead above the work surface. A single aligned capture per cycle yields `(color_bgr, depth_mm, intrinsics)`. The pipeline runs alongside the existing webcam workflow as a **parallel entry point** (`pickerbot_depth.py`) so the legacy path keeps working untouched.

```
D435 → aligned (RGB, depth) ─┬─ YOLOv8-OBB ───────────► detections (cx,cy,yaw,label,conf,obb)
                             └─ point cloud ─ RANSAC plane ─ DBSCAN ─► clusters
                                          │
            cross-check (IoU, completeness) ─► matched targets + unknown obstacles
                                          │
   per target: top-surface centroid (parallax fix) → (x,y,z_top) in base frame
                                          │
            HEIGHT-SORT (H1) → for each: dynamic approach-cone clearance (H2)
                                          │
         grip-strength lookup (log) → PICK x y z u clearance → EPSON RC8 (SPEL+)
                                          │
              after pick: vision verification → success? → (stretch) recovery re-plan
                                          │
                         ↕ interface contract ↔ Thesis 2 (end-effector + proprioceptive verify)
```

### 6.2 Sensor selection & limitations (D435 — honest treatment)
The D435 is the chosen and available sensor. It is tuned for ~0.3–3 m robotics, so **small, near, low-relief modules sit near its depth-noise floor**, and glossy surfaces (LCD glass, shiny IC packages) cause depth dropouts. This is treated as an **evaluated design decision**, not hidden:

- **Mitigation 1 — RGB for XY, depth for Z.** YOLOv8-OBB (sharp RGB) supplies lateral position and yaw; depth supplies *height and parallax correction only*. The small-object problem is thus reduced to a **Z-accuracy** problem, not a detect/localise problem.
- **Mitigation 2 — many-point averaging.** The pick height is the centroid of *all* valid top-surface points in the OBB, not a single pixel; depth noise falls ≈ 1/√N.
- **Mitigation 3 — filter chain.** RealSense decimation → disparity → spatial → temporal → disparity-back on live/bag sources.
- **Mitigation 4 — top-band percentile filter.** Inside each OBB, only points within `top_band_mm` (default 8 mm) of the *minimum* depth count as "top surface."
- **Mitigation 5 — working distance.** Mount as close as the FOV allows to improve depth precision.
- **Scoped out:** specular/transparent depth recovery (ClearGrasp / ASGrasp / GraspNeRF territory — cited as the field boundary, §3.1). Glossy-part breakdown is *characterised* in the failure envelope (§7.9).
- **Documented alternative (future work):** the RealSense **D405** (close-range, sub-mm at 7 cm, 7–50 cm range) would directly raise Z accuracy at the cost of FOV; named so the examiner sees the trade-off was understood.

### 6.3 Coordinate frames & hand-eye calibration
- **Z convention:** EPSON base frame is **+Z up** (confirmed from `Main.prg`: `Jump3 ... :Z(z + 50)` then `:Z(z)` is descent). Camera-frame Z is positive *into* the scene; the extrinsic transform flips it.
- **Extrinsic:** full `T_cam2base` (4×4) via a charuco board + `cv2.calibrateHandEye` (eye-to-hand, `CALIB_HAND_EYE_DANIILIDIS`). ≥15 robot poses with rotational diversity. **Refuse to save if held-out reprojection residual > 5 mm** — this residual is reported as a headline calibration metric.
- Robot pose per capture (`T_gripper2base`): manual `Here` entry initially; optional `WHERE` SPEL+ command to stream pose over TCP (deferred).

### 6.4 Perception — detection + depth fusion (parallax fix)
- `compute_top_surface_centroid(depth_crop, top_band_mm)` → 3D centroid in camera frame via the top-band filter.
- `correct_pick_target(...)` → `(x_base, y_base, z_top_base, yaw, completeness_ratio)`; transforms via the extrinsic **directly** (not through the legacy homography).
- **Completeness gate:** drop a detection if < `completeness_thresh` (default 70%) of OBB pixels return valid depth (partial occlusion guard — ties to Q4).
- **TCP offset:** commanded robot Z = `z_top_base − ee.tcp_z_mm`, so the tool *tip* lands on the top surface.

### 6.5 Obstacle detection & cross-check
- `segment_plane(cloud)` — RANSAC table-plane fit.
- `cluster_above_plane(points, eps_mm, min_pts)` — DBSCAN.
- `cross_check_clusters_vs_detections(clusters, detections, iou_thresh)` → `(matched, unknown_obstacles)`. A depth cluster with **no** YOLO match → `unknown_obstacle`; a YOLO detection with no depth support → dropped.
- *Rationale (viva-ready):* classical RANSAC+DBSCAN is interpretable, deterministic, needs no training data — appropriate for a 3-class lab cell; a learned segmenter is overkill and adds a data burden.

### 6.6 Height-aware pick sequencing — **H1, the central mechanism**
Sort surviving targets by **descending `z_top_base`** before dispatch. *Rationale:* picking a short item beside a tall neighbour risks the gripper colliding with the neighbour during a top-down approach; removing the tallest first leaves a progressively flatter residual scene. The policy is a single `sort()` on the depth-derived heights — **zero extra modelling or sensing** — which is precisely the "cheap and sufficient" claim of §3.2. The sequencing policy is a **config switch** (`pick_order: tallest_first | unordered`) so it can be toggled for the 2×2 experiment.

### 6.7 Dynamic approach-cone clearance — **H2, supporting mechanism**
For each pick, model the end-effector as an axis-symmetric cylinder (`radius_mm + safety_margin_mm`) and sample **N=64 points** on the cylinder surface from `z_top` up to `z_top + clearance`, in the base frame; transform each to camera frame, project to pixel, sample depth, and compare against the **full obstacle cloud minus the target's own cluster** (so a part isn't treated as its own obstacle). On intrusion, raise clearance in 10 mm steps to `max_clearance_mm` (default 200). If still blocked → **skip with a logged `BLOCKED` warning**; batch continues. Cone sampling in 3D (not a 2D circle) handles the elliptical perspective footprint correctly. Clearance is a **config switch** (`dynamic | fixed`) for the 2×2.

### 6.8 Grip-strength reporting
Per-label JSON lookup, **logged only** (not yet wired to the gripper — that's Thesis 2):
```json
{"_meta": {"unit": "grams", "default": 30}, "labels": {"lcd": 40, "arduino": 25, "esp32": 20}}
```
Unknown labels fall back to `_meta.default`.

### 6.9 Closed-loop pick verification & recovery (+ Thesis 2 interface)
- **Vision verification (core, mine):** after a pick attempt, re-scan the target ROI. Object absent ⇒ success; object still present/displaced ⇒ failure. Pure perception.
- **Recovery re-plan (stretch, mine):** on failure, re-detect, recompute the (possibly shifted) pose, re-issue the pick, up to N retries.
- **Interface contract (mine, first-class deliverable):** message protocol between subsystems — topic/field names, coordinate frames, who signals what, timing. Combined with Thesis 2's proprioceptive boolean this realises the **multimodal failure detection** pattern (§3.3): I own the *vision* modality + fusion/recovery logic; buddy owns the *proprioceptive* modality. **Signed off in writing with buddy + JP in Week 6.**

### 6.10 Robot protocol (extended `PICK`)
Backward-compatible extension of the SPEL+ receiver:
- `epsonPick(x, y, z, u, clearance=None)` — emits the optional 5th token only when set.
- `Main.prg` PICK handler: `Double clearance` default 50; if `UBound(indata$()) >= 5` and value `> 0`, use it; pass to the `Jump3` approach. Falls back to 50 on missing/invalid value. **No other SPEL+ commands touched.**
- `epsonPickAll(locations)` accepts 4- or 5-field tuples/strings (token-count detected); a `test_sender.py` regression covers both shapes.

### 6.11 Software modules, config & reproducibility
**New:** `pickerbot_depth.py` (entry), `pickerbot_lib/{depth,parallax,obstacle,extrinsics,grip,boq}.py`, `tools/{extrinsic_calibrator,record_bag}.py`, `grip_strengths.json`, `data/calibration/extrinsics.json`, `tests/test_{parallax,obstacle,grip,sender}.py`.
**`DepthSource` abstraction** with three backends — `LiveDepthSource` (pyrealsense2), `BagDepthSource` (`.bag`, `set_real_time(False)` for determinism), `NpyDepthSource` (paired `color.npy`/`depth.npy` for unit tests) — so the *entire pipeline runs and is tested without hardware*. This is the reproducibility guarantee: every number in the results regenerates from committed data.

**Config additions (`config.json`):**
```json
"end_effector":  {"tcp_z_mm":120, "radius_mm":25, "length_mm":80, "safety_margin_mm":8},
"realsense":     {"source":"bag", "bag_path":"data/recordings/dev.bag", "width":1280,"height":720,"fps":30},
"depth_pipeline":{"top_band_mm":8, "ransac_distance_mm":5, "dbscan_eps_mm":15, "dbscan_min_pts":50,
                  "iou_match_thresh":0.3, "completeness_thresh":0.7,
                  "approach_clearance_default_mm":50, "max_clearance_mm":200,
                  "pick_order":"tallest_first"},
"extrinsics_file":"data/calibration/extrinsics.json",
"grip_strengths_file":"grip_strengths.json"
```

---

## 7. Experimental Design & Evaluation

### 7.1 The central experiment — 2×2 factorial
Two binary factors, **fully crossed**:

| | **Clearance: fixed** | **Clearance: dynamic (H2)** |
|---|---|---|
| **Order: unordered** | baseline (legacy-like) | clearance only |
| **Order: tallest-first (H1)** | sequencing only | sequencing + clearance |

This single design delivers three things at once (per JP's notes): the **sequencing main effect** (H1 — "run with sorting and without"), the **clearance main effect** (H2 — "justify the dynamic calculation"), and the **interaction** (the complementarity story: sequencing solves cases clearance physically cannot, e.g. an obstacle taller than `max_clearance` over the approach path).

### 7.2 Independent variables (test matrix)
- **Order policy** {unordered, tallest-first} — factor A
- **Clearance** {fixed, dynamic} — factor B
- **Lighting** {bright, normal, dim}
- **Height spread** {flat, mixed-height, stacked}
- **Occlusion** {none, low, med, high} — **characterise-only** (Q4); report degradation, do not solve
- **Scene density** {sparse, dense}

> **Design note (de-risking the headline):** because H1 is now the central claim, scenes must be engineered where ordering *can* matter — **dense, tall-next-to-short layouts** where a top-down approach to the short item is plausibly blocked. A null result on sparse scenes would understate the effect; the design deliberately includes adversarial-for-collision layouts.

### 7.3 Dependent variables — **robotics-first** (JP note: "has to be robotics")
Every metric ties to *arm* performance, not vision in isolation:

| Metric | Definition | Targets |
|---|---|---|
| **Pick success rate** (%) | grasped & deposited without error | H1, H2 |
| **Collision / near-miss events** (count/batch) | gripper–neighbour contact on approach | H1, H2 |
| **Clearance height used** (mm) + total vertical travel | motion economy | H2 |
| **Skipped/blocked picks** (count) | abandoned (`max_clearance` exceeded) | H1×H2 interaction |
| **Cycle time** (s) | detection trigger → pick complete, per item & batch | all |
| **3D pose MAE** (mm, X/Y/Z) | vs ground truth; vs 2D homography baseline | Q3 |
| **Verification accuracy** (%) | correct success/failure calls | Q5 |

### 7.4 2D vs 3D baseline comparison (Q3)
Same 20 components at known positions/heights, estimated by (a) the 2D homography (at calibrated height only) and (b) the depth pipeline. Report MAE in X/Y/Z **as a function of object height** — the parallax error in the 2D baseline should grow with height while the 3D pipeline stays flat. This is the cleanest single figure in the report.

### 7.5 Ground-truth methodology
Pick-point truth obtained by (in order of preference): a **machined fixture / jig** with known coordinates; or a printed grid + vernier calipers; or **robot-touch** (jog the arm to contact each part, read `Here` — the robot's own repeatability becomes the ruler). Method fixed in Week 6; stated explicitly in the report (an examiner *will* ask "how do you know ground truth?").

### 7.6 Error budget (headline rigor)
Decompose pick-point error into sources and quantify each; the sum should predict observed MAE:

| Source | Estimate method |
|---|---|
| Camera intrinsics | calibration reprojection RMS |
| Hand-eye extrinsic | held-out residual (§6.3) |
| Depth noise @ working distance | repeated static captures, std of centroid |
| Segmentation/centroid | synthetic top-band test (`test_parallax.py`) |
| Robot positioning repeatability | datasheet + repeated `Here` returns |

### 7.7 Ablations
Each tunable choice becomes a small experiment: top-band filter on/off (Z-error), completeness threshold sweep, temporal-filter on/off, sequencing on/off (subsumed by 7.1), clearance step size. Ablations *prove* design choices instead of asserting them.

### 7.8 Statistical analysis plan
- **Proportions** (success, collision rate): chi-square / Fisher's exact, with 95% CIs.
- **Continuous** (cycle time, clearance used): two-way ANOVA on the 2×2 (main effects + interaction), or Mann-Whitney if non-normal.
- Report **effect sizes** (Cohen's d / odds ratios), not just p-values; pre-state N per cell for adequate power (JP: "big statistical difference is good" → power the design to detect it).

### 7.9 Failure-envelope characterisation
Deliberately locate and *report* breakdowns: glossy LCD depth dropouts, two parts touching, parts at the depth-FOV edge, high occlusion where YOLO fails. Documenting one's own limits is disarming in a viva and directly answers Q4.

### 7.10 Unit & integration verification (no hardware)
- `pytest tests/test_parallax.py` — synthetic crop centroid within 1 mm; camera→base→camera round-trip is identity.
- `pytest tests/test_obstacle.py` — plane + 2 clusters → exactly one obstacle.
- `pytest tests/test_grip.py` / `test_sender.py` — lookup + default; 4- vs 5-field protocol.
- Offline integration on a `.bag` (flat surface + Arduino + LCD + a tall "obstacle" mug): expect two picks (depth-corrected, tallest-first), mug flagged `unknown_obstacle`, clearances ≥ 50 mm; with the mug in the approach cone, expect clearance to grow or a logged `BLOCKED` skip.

---

## 8. Risks & Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Hardware (D435 / arm) unavailable or delayed | High | Offline `.bag`/`.npy` harness (Week 2) decouples perception dev + all unit tests from hardware; experiment weeks are back-loaded |
| **H1 effect small** (sparse scenes) → weak headline | Medium | Engineer dense tall-next-to-short scenes (§7.2); report collision/clearance DVs where the effect shows even if success-rate moves little; power the design |
| D435 Z-noise on small parts | Medium | RGB-for-XY/depth-for-Z split + many-point averaging + filter chain (§6.2); D405 named as future work |
| Glossy/specular dropouts | Medium | Scoped out; characterised, not solved (§7.9) |
| Thesis 2 interface mismatch | Medium | Written, signed-off interface contract in Week 6 |
| Calibration drift | Low | Held-out residual gate (<5 mm); re-calibrate if exceeded |
| 8-page limit vs. content volume | Medium | Blog (40%) carries the detailed narrative; report stays dense and selective |

---

## 9. Deliverables

1. **Project blog (40%)** — weekly updates + final video (due 26 Jul).
2. **Research article ~8 pages (40%)** — lit review (§3), design (§6), experiment (§7), results, conclusions (due 25 Sep).
3. **Final presentation (20%)** — supervisor + second marker (Oct).
4. **Code** — `pickerbot_depth.py` + `pickerbot_lib/*` + tests, on the existing Picker-Bot repo.
5. **Datasets** — calibration, pick-trial logs, pose-MAE measurements, cycle-time recordings (2D baseline + 3D pipeline).
6. **Interface specification** — perception ↔ end-effector contract for Thesis 2.

---

## 10. Open Questions / Deferred

- Robot pose ingestion for hand-eye (`WHERE` SPEL+ command vs manual entry).
- Calibration drift auto-detection at startup (fiducial check).
- Grip-strength → gripper wiring (Thesis 2; extend PICK with `grip_g` later).
- Full 6-DoF (tilted-part) pose via plane-normal estimation.
- ROS2/MoveIt2 + EPSON SPEL+ bridge (§3.5).

---

## 11. Indicative References (by cluster — to be expanded in Week 1–2)

**Sequencing / clutter (§3.2 — H1 lineage):**
- Nam et al. (2020), *Fast and resilient manipulation planning for target retrieval in clutter*. arXiv:2003.11420.
- *Real-Time Online Re-Planning for Grasping Under Clutter and Uncertainty* (2018). arXiv:1807.09049.
- *Taming Combinatorial Challenges in Optimal Clutter Removal* (2019). arXiv:1905.13530.
- *ClutterNav: Gradient-Guided Search for Efficient 3D Clutter Removal* (2025). arXiv:2511.12479.

**RGBD pose / grasp (§3.1):**
- Du et al. (2022), *Robotic Grasping from Classical to Modern: A Survey*. arXiv:2202.03631.
- *Deep Learning-Based Object Pose Estimation: A Comprehensive Survey*. IJCV (2025).
- Buchholz, Winkelbach & Wahl (2013), *RANSAC-based 3D point cloud segmentation for bin-picking*. RAS 61(12).
- Collet, Martinez & Srinivasa (2011), *The MOPED framework*. IJRR 30(10).

**Verification / failure recovery (§3.3):**
- *Failure Handling of Robotic Pick-and-Place with Multimodal Cues under Partial Occlusion*. Frontiers in Neurorobotics (2021).
- *Closed-Loop Open-Vocabulary Mobile Manipulation with GPT-4V* (2024). arXiv:2404.10220.

**Specular/transparent depth (§6.2 boundary):**
- *ClearGrasp: 3D Shape Estimation of Transparent Objects* (2020).
- *ASGrasp* (2024) arXiv:2405.05648; *GraspNeRF* (2022) arXiv:2210.06575.

**Sim-to-real / robustness (§3.4):**
- *Robust Visual Sim-to-Real Transfer for Robotic Manipulation* (2023). arXiv:2307.15320.
- *ClutterDexGrasp* (2025) arXiv:2506.14317.

**Middleware / integration (§3.5):**
- Macenski et al. (2022), *Robot Operating System 2*. Science Robotics 7(66).
- Coleman et al. (2014), *Reducing the barrier to entry... a MoveIt! case study*. JOSER 5(1).

**Sensor:**
- Intel RealSense D435 / D405 product datasheets, Intel (2023).
