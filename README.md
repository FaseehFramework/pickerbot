# Robust Depth-Driven Clearing of Unstructured Piles of Interlocking Microelectronic Modules: Height-Ordered Pick Sequencing with Failure-Aware Verification on an EPSON VT6 Manipulator

> *Working title (v2) — open to refinement. The two nouns that must survive any rewording are **pile clearing** (not "mixed-height workspace") and **failure-aware verification** (now a co-headline, not a footnote).*

| | |
|---|---|
| **Student** | Faseeh Mohammed (M01088120) |
| **Module** | PDE4445 — Robotics Dissertation Project |
| **Programme** | MSc Robotics — Middlesex University Dubai |
| **Supervisor** | JP |
| **Platform** | EPSON VT6-A901S 6-axis manipulator · Intel RealSense **D435** (held) · **D405** (close-range, under evaluation) · YOLOv8-OBB |
| **Companion thesis** | Thesis 2 (buddy) — custom end-effector design & pick verification |
| **Document** | Thesis design document — **v2 (consolidated)** |
| **Date** | June 2026 |

> **Note on scope vs. earlier proposal.** Earlier drafts (v3/v4) framed this as a ROS2 + MoveIt2 sim-to-real project quoting a ~15,000-word dissertation. That is superseded. The spine is a **direct (non-ROS) depth pipeline** — lower integration risk for a 12-week window — and the deliverable, per the PDE4445 handbook, is a **research article of ~8 pages** plus a **blog (40%)** and **presentation (20%)**, *not* a 15k-word dissertation. ROS2/MoveIt2 is retained only as documented future work.

---

## What changed in v2 (read this first)

v1's spine was "height-aware pick sequencing." On reflection — and per JP's challenge, *"where does this hypothesis come from… the answer should feel right"* — that framing was weak for the actual object set: flat microelectronic modules lying on a table are near-identical in height, so "tallest-first" was motivated mostly by a rare (~5%) tip-over case. The same critique quietly undermined the parallax-correction claim too, because parallax error only grows as an object departs from the calibration plane. Taken to its conclusion, the question becomes: *if the parts are flat and identical, why use depth at all?* v2 answers that by committing to a scene where vertical structure is the norm, and by rebalancing the contributions accordingly.

The substantive changes:

1. **Scene model reframed — from "mixed-height workspace" to "dumped pile of interlocking modules."** When a bucket of hobby modules is tipped onto a table, the result is not a flat scatter. It is a **difficulty ladder**: free singles, stacked/overlapping parts, *lightly* pin-tangled parts (a small nudge frees them), and *strongly* pin-tangled clusters (mechanically interlocked, inseparable with this hardware). Estimated composition on the bench: roughly ~20% strongly tangled, a further substantial fraction lightly tangled, the remainder free-or-stacked — **to be measured, not assumed (now an explicit early result).**

2. **H1 reconceived — "tallest-first" → "topmost-first."** Height is no longer claimed to matter because an LCD is 2 mm taller than an ESP32. Height is the **observable signal of which part is on top** in a pile. Topmost-first is not a statistical nicety; for stacked/overlapping parts it is *physically necessary* (you cannot grasp a pinned part). Height-ordering is positioned as a **cheap, training-free, depth-only proxy** for the occlusion/reachability scoring that heavier clutter-removal planners compute. The stance is unchanged: **"sufficient and cheap," not "optimal."**

3. **Verification promoted from supporting (old C4) to a co-pillar.** In a tangle-prone pile, **detecting what cannot be picked is as valuable as picking.** Vision re-scan verification now does double duty: confirm a successful extraction, *and* detect a non-separation (failed pick / co-moving tangle), then **skip gracefully and log it.** This converts entanglement from a silent catastrophic failure into a detected, bounded, reportable event.

4. **Entanglement is modelled explicitly — detected and bounded, not solved.** Mechanical disentanglement is a recognised open problem (new §3.6). Strongly tangled clusters are declared out of scope and *detected*; the single **stretch goal** is a cheap open-loop disturbance (lateral nudge / lift-jerk) that recovers a measurable fraction of the *lightly* tangled population.

5. **Research gap repositioned honestly.** v2 does **not** claim to advance bin-picking SOTA (mature). The defensible gaps are: (a) **empirical characterisation of an unstudied object class** — small, glossy, low-relief, *pin-interlocking* hobby modules dumped into a pile — including scene-composition statistics and a failure taxonomy; (b) a **rigorous, ablation-grade isolation** of how much benefit a minimal depth-only ordering heuristic recovers on *real hardware*; (c) a **lightweight exteroceptive entanglement detector** enabling graceful degradation. The depth pipeline, parallax fix, and RANSAC+DBSCAN are reframed as *engineering rigor*, not novelty.

6. **Hardware/sensor decision added (new §6.2).** The D435 is defensible but sits near its depth-noise floor for small, near parts — exactly where the headline measurement lives. Recommendation: **keep the D435 and add a D405**, and make the **sensor itself a controlled comparison** (D435-vs-D405 pose-MAE vs height). Full bill of materials and the D405 ≤50 cm mounting constraint are specified.

7. **Experiment updated for the new scene.** Tangling is a **common-mode** failure (it sinks picks under both orderings), so the design now (a) uses **seeded, replayed pile layouts** to hold tangle fraction constant across conditions, and (b) reports the sequencing effect on **isolating metrics** — collisions on approach, and clean picks achieved *before* the first unrecoverable tangle — not just final success rate.

The section structure below is unchanged from v1; the content within each section has been revised to reflect the above.

---

## Abstract

Vision-guided pick-and-place on industrial manipulators is mature for flat, uniform parts but remains brittle for the scene produced when a bucket of small components is tipped onto a table: an **unstructured pile** in which parts lie free, rest on one another, and — for modules carrying GPIO header pins — mechanically **interlock**. The existing *Picker-Bot* system (YOLOv8-OBB detection + a single-plane homography on an EPSON VT6-A901S) assumes every pickable object's top surface lies at a fixed height; this assumption produces parallax-induced XY error and collision risk for any part not lying flat at the calibration height, carries no model of obstacles or pick order, and has no way to recognise a pick that cannot succeed.

This work replaces the planar pipeline with a depth-driven one built on an Intel RealSense close-range sensor in a fixed eye-to-hand configuration, and is organised around **two central pillars**. **Pillar 1 (sequencing):** in an unstructured pile, **topmost-first** ordering — computed as a single sort on depth-derived top-surface height — peels separable parts off in dependency order, measurably improving clean-pick yield and reducing approach collisions relative to an unordered baseline, at negligible compute and sensing cost. Height is treated as a *cheap proxy* for the occlusion/reachability scoring used by heavier clutter-removal planners; the claim is sufficiency, not optimality. **Pillar 2 (verification):** a vision re-scan after each pick detects **non-separation** events — failed grasps and co-moving entangled clusters — enabling the system to **skip gracefully and bound its own failure mode** rather than thrashing. A supporting mechanism (dynamic approach-cone clearance) and a single stretch goal (an open-loop disturbance to recover *lightly* tangled parts) round out the system. The pillars are evaluated together in a 2×2 factorial (sequencing × clearance) on the physical arm, with seeded pile layouts controlling for common-mode entanglement, across controlled variation in lighting, scene regime, and occlusion, against the prior 2D baseline. The contributions are: an empirical characterisation of pick-and-place for an unstudied object class (small, glossy, pin-interlocking modules), a rigorous isolation of a minimal depth-only sequencing heuristic on real hardware, and a lightweight exteroceptive entanglement detector — together with a clean perception-side interface to a companion end-effector/verification subsystem.

---

## 1. Problem Definition

*Picker-Bot* is a computer-vision-guided pick-and-place system built around an EPSON VT6-A901S 6-axis manipulator. Its perception front-end — a YOLOv8 Oriented Bounding Box (OBB) detector — has been demonstrated on real camera frames against three microelectronic component classes (Arduino, ESP32, LCD module). Everything downstream of detection is fragile or untested on hardware:

1. **The planar-height assumption breaks under real conditions.** Coordinate resolution uses a 77-point homography grid collected once at a fixed camera height, and the robot descends to a fixed `robot_z = 360 mm`. Any component not lying flat at the calibration height yields (a) a wrong descent depth and (b) **parallax error**: an overhead camera projects the *top* of an elevated object to an image position offset from its true XY footprint, so the homography returns a laterally shifted pick point. The error grows with object height above the plane — i.e. precisely in the stacked and tangled regimes.

2. **There is no awareness of obstacles or of pick order — and the scene has real vertical structure.** When parts are tipped from a bucket they do not form a flat scatter. They form a **pile**: free singles, parts stacked or overlapping, and — because most modules carry protruding GPIO header pins — parts mechanically **tangled** in one another's pins. In this scene a top-down approach to a low or pinned part can collide with a taller neighbour, and attempting a part that is underneath or interlocked fails outright. The system has no principled order in which to clear the surface, and no model of which parts are even separable.

3. **Picks are open-loop with no failure detection.** detect → move → close → assume success. Nothing checks whether a pick succeeded, and nothing recognises a part that *cannot* be picked (pinned or tangled), so the arm will silently drag or thrash.

This dissertation addresses (1) and (2) directly, and (3) on the perception side — the gripper hardware and its proprioceptive grasp signal belong to the **companion Thesis 2**. The two scientific pillars are *pick ordering* (which separable parts to take, and in what order) and *failure-aware verification* (recognising the parts that cannot be taken); the depth pipeline is the instrument that makes ordering, parallax correction, obstacle clearance, and exteroceptive verification possible.

### Scope boundaries (stated up front)

- **In scope:** opaque microelectronic modules tipped onto a flat table forming an unstructured pile (free / stacked / tangled); 4-DoF picking `(x, y, z, yaw)` under a planar-rest assumption for the *graspable* parts; fixed overhead camera; **detection and graceful skipping of inseparable (strongly tangled) parts.**
- **The difficulty ladder (the organising idea):** *free single* → pick directly · *stacked/overlapping* → topmost-first peels it (Pillar 1) · *lightly tangled* → verification detects non-separation, optional disturbance-nudge recovery (stretch) · *strongly tangled* → detected and skipped, logged (declared out of scope).
- **Out of scope (declared limitations, not failures):** full 6-DoF pose of arbitrarily tilted parts; **mechanical disentanglement** of strongly interlocked clusters (an open research problem — §3.6); **specular/transparent** depth recovery (§3.1, §6.2); the end-effector mechanism and its force sensing (Thesis 2).

---

## 2. Research Questions & Hypotheses

**Central hypothesis 1 — sequencing (H1, Pillar 1).**
> In an unstructured pile, sequencing picks **topmost-first** (descending depth-derived top-surface height) improves end-to-end clean-pick yield and reduces gripper–neighbour collisions on approach, relative to an unordered (random/fixed) order — at negligible compute and sensing cost, using only the depth map.

**Central hypothesis 2 — verification (H2, Pillar 2).**
> A vision re-scan after each pick detects **non-separation** events (failed grasp; co-moving entangled cluster) with high accuracy, enabling the system to skip inseparable parts gracefully and **bound its failure rate**, rather than thrashing or dragging tangled clusters.

**Supporting hypothesis (H3).**
> **Dynamic** per-pick approach-cone clearance (lifting the approach to clear intruding obstacles) reduces collisions and skipped picks relative to a fixed clearance — and is *complementary* to H1: sequencing removes tall obstacles that clearance alone cannot lift over within the arm's limits, while clearance handles residual obstacles that cannot be reordered away.

**Perception question (Q4).**
> Does depth-based 3D pose estimation (parallax-corrected top-surface centroid) reduce pick-point error versus the 2D homography baseline as object height departs from the calibration plane, and by how much (MAE in X/Y/Z)? **And how does this differ between the D435 (active stereo, near its noise floor) and the close-range D405?**

**Robustness question (Q5).**
> How do detection and pick performance degrade as a function of lighting, scene regime (flat/stacked/tangled), occlusion (low/med/high), and **tangle density**? Occlusion and entanglement are **characterised, not solved**.

**Integration question (Q6).**
> Can a stable perception-side interface (object poses, pick/verify/skip signals) cleanly decouple this subsystem from the companion end-effector/verification subsystem?

**Stretch hypothesis (H7).**
> A cheap, open-loop disturbance primitive (lateral nudge or lift-jerk) applied after a detected non-separation recovers a measurable fraction of *lightly* tangled picks, without specialised tooling.

---

## 3. Existing Solutions & The Gap

Six clusters frame the related work. For each: the state of the art, why it is insufficient *for this problem*, and what this work does instead. (This is the spine of the Week 1–2 literature review JP asked for — "2–3+ papers, existing solutions, why not sufficient, how is mine different.")

### 3.1 RGBD 6-DoF pose estimation & grasp planning for bin-picking
**SOTA:** Deep 6-DoF grasp/pose estimation from RGBD is mature — comprehensive surveys (Du et al., *Robotic Grasping from Classical to Modern*, 2022; *Deep Learning-Based Object Pose Estimation: A Survey*, IJCV 2025) and industrial systems demonstrate reliable cluttered-scene picking.
**Why insufficient here:** these methods target generic objects and assume a capable sensor; almost none address **small, thin, low-texture, glossy microelectronic modules** on a budget close-range sensor, and they rarely isolate **pick *order*** as a controlled variable.
**What I do:** a deliberately lightweight depth-only pipeline tuned for small modules, with ordering as the object of study rather than a by-product.

### 3.2 Pick sequencing / clutter removal (the home of H1)
**SOTA:** Sequencing in clutter is studied — target retrieval in clutter (Nam et al., 2020), online re-planning (2018), optimal clutter removal (2019), cost-map sequencing (*ClutterNav*, 2025). Production bin-pickers rank targets by occlusion count, fail-counters, and confidence.
**Why insufficient here — and the trap to avoid:** these methods explicitly do **not** "just pick the topmost first" — they score reachability, occlusion, and stability, and often require full object pose, learned policies, or expensive combinatorial planning, evaluated largely in simulation. **I therefore do *not* claim topmost-first is the optimal policy** (that argument is lost to this literature).
**What I do — the defensible claim:** **topmost-first is a cheap, training-free, depth-only *proxy* for the occlusion/reachability ordering these planners compute.** Computed as one `sort()` on depth-derived heights with zero extra modelling, it recovers a *statistically significant share* of the collision-avoidance benefit on real hardware at negligible cost. The contribution is "sufficient and cheap," not "optimal" — and, critically, *measured* rather than asserted.

### 3.3 Closed-loop grasp verification & failure recovery (Pillar 2, now central)
**SOTA:** Most deployed industrial PnP is **open-loop**. Closed-loop verification is recent and not standard — multimodal failure handling fusing vision + gripper proprioception (Frontiers in Neurorobotics, 2021), VLM-driven closed-loop manipulation (2024).
**Why insufficient here:** these assume control of the gripper's sensing (Thesis 2's domain), and they mostly answer *"did I grasp something?"* — not *"did I extract a single **separable** object from a coupled cluster?"*
**What I do:** own the **exteroceptive (vision) modality** — re-scan the ROI after a pick to confirm the object left the scene, *and to detect a non-separation* (object still present, or two clusters that moved together = a tangle). Skip and log inseparable parts; fuse with Thesis 2's proprioceptive boolean across a defined interface. This is now a **co-pillar**, not a stretch.

### 3.4 Sim-to-real & robustness of vision-guided grasping (Q5)
**SOTA:** Robust visual sim-to-real and domain randomisation (2023–2025; *ClutterDexGrasp*, 2025; ICRA 2025 Sim2Real PnP).
**Terminology caution:** "sim-to-real" in the literature means transferring *learned policies* from simulation via randomisation. This project's "sim-to-real" means **moving from the RC8 virtual controller to the physical arm** — deployment/commissioning, not policy transfer. Used carefully to avoid overclaiming.
**What I do:** empirically characterise robustness to lighting/scene-regime/occlusion/tangle-density on real hardware rather than via randomised training.

### 3.5 ROS2/MoveIt2 integration with proprietary arms (deferred context)
**SOTA:** `ros2_control` hardware-interface plugins and vendor gateways exist, but **no community ROS2 driver exists for EPSON SPEL+ arms.**
**Why deferred:** the EPSON RC8 controller does its own trajectory planning in SPEL+; feeding it MoveIt2 trajectories means fighting the controller or down-sampling to waypoints it re-plans anyway — high risk for 12 weeks.
**What I do:** retain the working SPEL+ TCP link and treat ROS2/MoveIt2 integration as **documented future work.**

### 3.6 Entanglement / separating interlocking parts (new — the source of Pillar 2's hard boundary)
**SOTA:** Separating mechanically entangled parts in random bin-picking is a recognised, *unsolved* sub-problem — studied for hooks, S-shaped parts, and wire harnesses (e.g. Matsumura, Domae et al. on bin-picking of potentially tangled objects, IROS 2019; Fraunhofer IPA work on disentangling in random bin-picking, ~2020–2022). Solutions rely on **specialised tooling, learned entanglement classifiers, or active shake/regrasp/disturbance strategies.**
**Why insufficient here:** all require either hardware (shaker bins, custom grippers) or training data this project does not have; none are aimed at GPIO-pin interlocking of small PCBs.
**What I do:** I **do not attempt to solve disentanglement.** I *detect* it exteroceptively (non-separation re-scan, Pillar 2), **bound** it (declared out of scope, reported in the failure taxonomy), and — as a **stretch only** — test whether a single cheap open-loop disturbance recovers the *lightly* tangled subset (H7). This is the honest, defensible boundary.

### 3.7 The research gap (synthesis)
> Existing clutter-removal and bin-picking methods achieve strong performance using full 6-DoF pose estimation, learned policies, or combinatorial planning, evaluated largely in simulation or on generic rigid objects. They neither establish how much collision-avoidance benefit is recoverable by a **minimal, depth-only, training-free ordering heuristic**, nor characterise pick-and-place behaviour for **small, glossy, pin-interlocking microelectronic modules** on a budget close-range sensor and a proprietary industrial arm. This work addresses both empirically: it isolates the effect of topmost-first sequencing under controlled conditions on real hardware, adds a lightweight exteroceptive **entanglement detector**, and characterises the resulting **failure envelope** — including pin-entanglement, which it detects and bounds rather than solves.

---

## 4. Contributions

1. **C1 — Pillar 1 (sequencing):** An empirical demonstration, on a physical industrial manipulator, that a lightweight depth-only **topmost-first pick sequence** significantly improves clean-pick yield / reduces approach collisions in unstructured piles — with effect sizes and significance, against an unordered baseline. Positioned explicitly as a *cheap, sufficient proxy* for heavier occlusion/reachability planners (§3.2).
2. **C2 — Pillar 2 (verification):** A **vision-based non-separation detector** that confirms successful extractions *and* detects failed/entangled picks, enabling graceful skip-and-log and **bounding the system's failure rate** — promoted to co-headline because the scene is tangle-prone.
3. **C3 — object-class & scene characterisation (contribution to knowledge):** The first empirical **scene-composition statistics** (free/stacked/lightly-tangled/strongly-tangled) and **failure taxonomy** for dumped piles of small pin-bearing hobby modules. Descriptive, measured, and previously unpublished.
4. **C4 — depth pipeline & sensor study (engineering rigor):** A depth-driven **parallax/descent-correction** pipeline quantitatively compared to the prior 2D homography baseline (pose MAE vs height), **and a D435-vs-D405 sensor comparison** turning the hardware choice into measured evidence.
5. **C5 — dynamic approach-cone clearance:** A supporting mechanism, justified independently and shown complementary to C1 via a 2×2 factorial.
6. **C6 — interface contract:** A **published interface contract** decoupling this subsystem from the companion end-effector/verification thesis.
7. **(stretch) C7 — disturbance recovery:** A cheap open-loop nudge/jerk evaluated for recovery of the *lightly* tangled subset (H7).

---

## 5. Twelve-Week Timeline & Milestones

**Official PDE4445 deliverables (from the module handbook, Table 1):**

| # | Deliverable | Weight | Due |
|---|---|---|---|
| 1 | **Project blog** — weekly updates, multimedia, final video; evidence of planning/time-management | **40%** | **26 Jul 2026** |
| 2 | **Project report** — *research article of ~8 pages*; clarity, organisation, referencing, figures/tables, design-choice documentation, technical quality | **40%** | **25 Sep 2026** |
| 3 | **Final presentation** — oral, to supervisor + second marker | **20%** | **26 Oct 2026** |

> ⚠️ The report is **~8 pages (research-article format)**, not 15,000 words. Plan for density and selectivity. The **blog is worth as much as the report** — treat continuous documentation as a recurring deliverable.

**12-week plan (anchored to the handbook's own week schedule).** Blog upkeep is implicit every week.

| Week | Dates (w/c) | Focus | Output / milestone |
|---|---|---|---|
| **1** | 30 Jun | **Literature review** across the 6 clusters (§3), incl. the new entanglement cluster; finalise objectives + Gantt; set up blog; **raise hardware PO (D405 + BOM, §6.2)** | Blog live; objectives; §3 draft; PO submitted |
| **2** | 07 Jul | Lit review cont'd; **experiment design** (§7); scaffold depth pipeline + offline `.npy`/`.bag` harness | Experiment plan; reproducible test harness |
| **3** | 14 Jul | **Sensor bring-up** (D435 + D405): intrinsics + **eye-to-hand calibration**; report reprojection residual; report-writing template | Calibration residual logged (<5 mm target) |
| **4** | 21 Jul | Parallax / top-surface-centroid pose pipeline; fuse with YOLOv8-OBB | **📌 BLOG DUE 26 Jul (40%)** — problem, lit review, plan, early results |
| **5** | 28 Jul | Obstacle detection (RANSAC + DBSCAN) + **dynamic clearance (H3)**; **topmost-first ordering (H1)** | Both mechanisms implemented |
| **6** | 04 Aug | **Vision verification + non-separation/tangle detection (H2)** + Thesis 2 **interface contract** (signed off with buddy + JP); build **ground-truth rig** | Interface spec v1; GT method fixed |
| **7** | 11 Aug | Pilot trials; **measure scene composition (C3 histogram)**; finalise **scene designs** + seeded layouts; metric logging | Pilot data; scene stats; scenes frozen |
| **8** | 18 Aug | **Main experiment — block 1:** 2×2 factorial (sequencing × clearance), seeded layouts, baseline lighting | Core dataset (H1/H3) |
| **9** | 25 Aug | **Main experiment — block 2:** lighting × scene-regime × occlusion sweeps; **2D-vs-3D** and **D435-vs-D405** pose-MAE | Robustness + baseline dataset |
| **10** | 01 Sep | **Analysis:** significance tests, effect sizes, **error budget**, **failure/entanglement taxonomy**, ablations; (stretch) disturbance recovery trials | Results finalised |
| **11** | 08 Sep | **Write the ~8-page research article** (draft); figures/tables; supervisor review | Report draft (handbook: 14 Sep review) |
| **12** | 15 Sep | Finalise report; polish blog + final video; buffer | **📌 REPORT DUE 25 Sep (40%)** |
| **post** | Oct | Presentation prep & rehearsal | **📌 PRESENTATION 26 Oct (20%)** |

**Critical-path risk:** every experiment week depends on the sensors and arm being available; the offline `.bag`/`.npy` harness (Week 2) lets perception development and unit tests proceed *without* hardware. The D405 PO is raised in Week 1 to protect the Week 8–9 sensor comparison.

---

## 6. System Architecture & Technical Design

### 6.1 Platform & data flow
Fixed **eye-to-hand** close-range sensor mounted above the work surface. A single aligned capture per cycle yields `(color_bgr, depth_mm, intrinsics)`. The pipeline runs alongside the existing webcam workflow as a **parallel entry point** (`pickerbot_depth.py`) so the legacy path keeps working untouched.

```
sensor → aligned (RGB, depth) ─┬─ YOLOv8-OBB ───────────► detections (cx,cy,yaw,label,conf,obb)
                               └─ point cloud ─ RANSAC plane ─ DBSCAN ─► clusters
                                            │
              cross-check (IoU, completeness) ─► matched targets + unknown obstacles
                                            │
     per target: top-surface centroid (parallax fix) → (x,y,z_top) in base frame
                                            │
       HEIGHT-SORT → TOPMOST-FIRST (H1) → per pick: dynamic approach-cone clearance (H3)
                                            │
           grip-strength lookup (log) → PICK x y z u clearance → EPSON RC8 (SPEL+)
                                            │
   after pick: VISION RE-SCAN → success? / NON-SEPARATION? (H2) ─► skip+log tangle
                                            │                         └─(stretch) disturbance nudge → retry (H7)
                                            │
                       ↕ interface contract ↔ Thesis 2 (end-effector + proprioceptive verify)
```

### 6.2 Hardware & sensor selection (new — the PO decision)
**Decision: keep the held D435, add a D405, and treat the sensor as a measured comparison (C4), not an assumption.**

- **Why the D435 alone is the weak link.** Its minimum range is 0.1 m and it is tuned for 0.3–3 m, so small, thin, near modules sit near its **depth-noise floor** — which is exactly where the headline measurement (Z/pose MAE) lives. Its one genuine advantage is the **IR projector**: active stereo paints texture onto the scene, so it stays robust in **dim light** and on low-texture surfaces (this directly serves the lighting sweep, Q5).
- **Why add the D405.** Purpose-built for this regime: 7–50 cm range, sub-mm object detection at 7 cm, ≲0.7% error at 25 cm (≈1.7 mm), 87°×58° FOV. Explicitly marketed for close-range robotic pick-and-place. The cost: **no IR projector** (passive stereo → degrades in dim light and on truly textureless surfaces) and a **≤50 cm range cap**. Two nuances: PCBs carry abundant fine natural texture (traces, silkscreen, pads) that passive stereo resolves well at close range; and the genuinely hard surfaces (glossy LCD glass, shiny IC tops) are **specular**, defeating *both* sensors regardless of projector. So the D405's weakness bites mainly in the *dim-lighting* condition — which the D435 covers.
- **Mounting constraint (verify before commissioning).** The D405's ~50 cm useful range caps the overhead mount height. At 50 cm the FOV covers ≈95×55 cm — ample for hobby modules — but the camera must clear the VT6's `Jump3` approach arc and gripper length, and not occlude the descent path; **mount offset/angled, not directly over the pick column.** The D435 has no such limit and serves as the wide-coverage workhorse.
- **Bill of materials (PO), priority order:** (1) **RealSense D405** (~US$250–300; confirm at official store) — keep D435 as second sensor; (2) **rigid camera mount/gantry** holding a *fixed* eye-to-hand pose (any drift invalidates the extrinsic — do not economise); (3) **ChArUco board** on a *rigid flat* substrate (dibond/acrylic, not paper); (4) **dimmable diffuse LED panel + diffuser**, plus optional **cross-polarising filter + polarised illumination** to suppress specular dropouts on glossy parts (cheap, high-leverage, mitigates the glossy failure mode and supports the lighting sweep); (5) **active USB-3 cable (2–3 m)** — RealSense is fussy about cables/hubs; the stock cable won't reach an overhead mount; (6) **ground-truth fixture** — CNC'd/3D-printed jig locating parts at known XY *and* known heights, incl. an elevated/stacked tier (the ruler for the 2D-vs-3D figure); (7) **matte work surface**; (8) **compute** — any CUDA GPU already held suffices for one-frame-per-cycle YOLOv8 inference; no Jetson needed.
- **Sensor noise mitigations (carried from v1, still in force on either sensor):** RGB-for-XY / depth-for-Z split (reduces the small-object problem to a Z-accuracy problem); many-point top-surface averaging (noise ≈ 1/√N); RealSense filter chain (decimation → disparity → spatial → temporal → disparity-back); top-band percentile filter; closest clearance-safe working distance.
- **Documented limitation:** specular/transparent depth recovery (ClearGrasp / ASGrasp / GraspNeRF territory) is scoped out; glossy breakdown is *characterised* in the failure envelope (§7.9).

### 6.3 Coordinate frames & hand-eye calibration
- **Z convention:** EPSON base frame is **+Z up** (confirmed from `Main.prg`). Camera-frame Z is positive *into* the scene; the extrinsic flips it.
- **Extrinsic:** full `T_cam2base` (4×4) via a ChArUco board + `cv2.calibrateHandEye` (eye-to-hand, `CALIB_HAND_EYE_DANIILIDIS`). ≥15 robot poses with rotational diversity. **Refuse to save if held-out reprojection residual > 5 mm** — reported as a headline calibration metric. *Calibrated separately per sensor (D435, D405) for the comparison.*
- Robot pose per capture (`T_gripper2base`): manual `Here` entry initially; optional `WHERE` SPEL+ command to stream pose over TCP (deferred).

### 6.4 Perception — detection + depth fusion (parallax fix)
- `compute_top_surface_centroid(depth_crop, top_band_mm)` → 3D centroid in camera frame via the top-band filter.
- `correct_pick_target(...)` → `(x_base, y_base, z_top_base, yaw, completeness_ratio)`; transforms via the extrinsic **directly** (not the legacy homography).
- **Completeness gate:** drop a detection if < `completeness_thresh` (default 70%) of OBB pixels return valid depth (partial-occlusion guard — ties to Q5).
- **TCP offset:** commanded robot Z = `z_top_base − ee.tcp_z_mm`, so the tool *tip* lands on the top surface.

### 6.5 Obstacle detection & cross-check
- `segment_plane(cloud)` — RANSAC table-plane fit.
- `cluster_above_plane(points, eps_mm, min_pts)` — DBSCAN.
- `cross_check_clusters_vs_detections(clusters, detections, iou_thresh)` → `(matched, unknown_obstacles)`. A depth cluster with **no** YOLO match → `unknown_obstacle`; a YOLO detection with no depth support → dropped.
- *Rationale (viva-ready):* classical RANSAC+DBSCAN is interpretable, deterministic, needs no training data — appropriate for a 3-class lab cell; a learned segmenter adds a data burden for no clear gain here.

### 6.6 Topmost-first pick sequencing — **H1, Pillar 1**
Sort surviving targets by **descending `z_top_base`** before dispatch. *Rationale (the reframed, "feels-right" version):* in a pile, the highest top surface is the part most likely to be **on top and free**; removing it first leaves a progressively flatter, more separable residual scene, and avoids descending onto a low/pinned part beside a taller neighbour. Height is a **cheap depth-only proxy** for the occlusion/reachability ordering that heavier planners compute (§3.2) — **zero extra modelling or sensing**, a single `sort()`. *Honest boundary (state in the report):* the globally tallest part is not guaranteed to be the most accessible (a tall part can lean under another); the claim is that height captures *most* of the accessibility signal in shallow piles of small flat modules, not that it is exact. The policy is a **config switch** (`pick_order: topmost_first | unordered`) for the 2×2.

### 6.7 Dynamic approach-cone clearance — **H3, supporting mechanism**
For each pick, model the end-effector as an axis-symmetric cylinder (`radius_mm + safety_margin_mm`) and sample **N=64 points** on the cylinder surface from `z_top` up to `z_top + clearance`, in the base frame; transform to camera frame, project to pixel, sample depth, and compare against the **full obstacle cloud minus the target's own cluster**. On intrusion, raise clearance in 10 mm steps to `max_clearance_mm` (default 200). If still blocked → **skip with a logged `BLOCKED` warning**; batch continues. 3D cone sampling (not a 2D circle) handles the elliptical perspective footprint. Config switch (`dynamic | fixed`) for the 2×2.

### 6.8 Grip-strength reporting
Per-label JSON lookup, **logged only** (not wired to the gripper — Thesis 2):
```json
{"_meta": {"unit": "grams", "default": 30}, "labels": {"lcd": 40, "arduino": 25, "esp32": 20}}
```
Unknown labels fall back to `_meta.default`.

### 6.9 Closed-loop verification, tangle detection & recovery — **H2, Pillar 2 (+ Thesis 2 interface)**
- **Vision verification + non-separation detection (core, mine):** after a pick attempt, re-scan the target ROI and compare to the pre-pick scene. **Object absent ⇒ success.** **Object still present / displaced but not removed ⇒ failed pick.** **Two or more clusters that moved *together* ⇒ entanglement** (a tangle dragged a neighbour). On a detected tangle of a *strongly* interlocked cluster → **skip and log** (out of scope, counted in the failure taxonomy, C3). Pure perception.
- **Disturbance recovery (stretch, H7, mine):** on a *lightly* tangled detection, apply one cheap open-loop disturbance — a short lateral nudge with the closed tool tip, or a lift-and-jerk — then re-detect and retry, up to N attempts. **Arm-motion only** (no special gripper action), so it stays in my lane, not Thesis 2's. Reported as "% of lightly-tangled picks recovered."
- **Interface contract (mine, first-class deliverable):** message protocol between subsystems — topic/field names, coordinate frames, who signals what (`success | fail | skip-tangle`), timing. Combined with Thesis 2's proprioceptive boolean this realises the **multimodal failure-detection** pattern (§3.3): I own the *vision* modality + fusion/skip/recovery logic; buddy owns the *proprioceptive* modality. **Signed off in writing with buddy + JP in Week 6.**

### 6.10 Robot protocol (extended `PICK` + optional disturbance)
- `epsonPick(x, y, z, u, clearance=None)` — emits the optional 5th token only when set.
- `Main.prg` PICK handler: `Double clearance` default 50; if `UBound(indata$()) >= 5` and value `> 0`, use it; pass to the `Jump3` approach. Falls back to 50 on missing/invalid value. **No other SPEL+ commands touched.**
- `epsonPickAll(locations)` accepts 4- or 5-field tuples/strings; `test_sender.py` covers both shapes.
- *(stretch)* a minimal `epsonNudge(x, y, dx, dy)` lateral-disturbance move for H7, gated behind a config flag and added only if the two pillars are secure.

### 6.11 Software modules, config & reproducibility
**New:** `pickerbot_depth.py` (entry), `pickerbot_lib/{depth,parallax,obstacle,extrinsics,grip,verify,boq}.py`, `tools/{extrinsic_calibrator,record_bag}.py`, `grip_strengths.json`, `data/calibration/extrinsics_{d435,d405}.json`, `tests/test_{parallax,obstacle,grip,verify,sender}.py`.
**`DepthSource` abstraction** with backends — `LiveDepthSource` (pyrealsense2), `BagDepthSource` (`.bag`, `set_real_time(False)`), `NpyDepthSource` (paired `color.npy`/`depth.npy`) — so the *entire pipeline runs and is tested without hardware*.

**Config additions (`config.json`):**
```json
"end_effector":  {"tcp_z_mm":120, "radius_mm":25, "length_mm":80, "safety_margin_mm":8},
"realsense":     {"source":"bag", "model":"d405", "bag_path":"data/recordings/dev.bag", "width":1280,"height":720,"fps":30},
"depth_pipeline":{"top_band_mm":8, "ransac_distance_mm":5, "dbscan_eps_mm":15, "dbscan_min_pts":50,
                  "iou_match_thresh":0.3, "completeness_thresh":0.7,
                  "approach_clearance_default_mm":50, "max_clearance_mm":200,
                  "pick_order":"topmost_first", "clearance_mode":"dynamic"},
"verification":  {"enabled":true, "roi_change_thresh":0.5, "tangle_comotion_iou":0.3, "max_retries":2,
                  "recovery_nudge":false},
"extrinsics_file":"data/calibration/extrinsics_d405.json",
"grip_strengths_file":"grip_strengths.json"
```

---

## 7. Experimental Design & Evaluation

### 7.1 The central experiment — 2×2 factorial
Two binary factors, **fully crossed**:

| | **Clearance: fixed** | **Clearance: dynamic (H3)** |
|---|---|---|
| **Order: unordered** | baseline (legacy-like) | clearance only |
| **Order: topmost-first (H1)** | sequencing only | sequencing + clearance |

This delivers the **sequencing main effect** (H1), the **clearance main effect** (H3), and the **interaction** (sequencing solves cases clearance physically cannot, e.g. an obstacle taller than `max_clearance`). **Pillar 2 (H2)** is evaluated across all four cells as verification/tangle-detection accuracy.

> **Design note (de-risking the headline + controlling for tangling):** scenes must be engineered where ordering *can* matter — **dense, stacked, tall-next-to-short layouts**. Because entanglement is a **common-mode** failure (it sinks picks under both orderings and would dilute the effect), each pile is built from a **seeded, replayed layout** so the *same* arrangement is presented under every condition, holding tangle fraction constant across cells.

### 7.2 Independent variables (test matrix)
- **Order policy** {unordered, topmost-first} — factor A
- **Clearance** {fixed, dynamic} — factor B
- **Lighting** {bright, normal, dim} — *(D435 carries the dim condition; see §6.2)*
- **Scene regime** {flat, stacked, tangled}
- **Tangle density** {none, low, high} — common-mode; controlled via seeded layouts
- **Occlusion** {none, low, med, high} — **characterise-only** (Q5)
- **Scene density** {sparse, dense}

### 7.3 Dependent variables — **robotics-first** (JP: "has to be robotics")
| Metric | Definition | Targets |
|---|---|---|
| **Clean-pick yield** (%) | grasped & deposited, single object, without error | H1, H3 |
| **Collision / near-miss on approach** (count/batch) | gripper–neighbour contact during descent | **H1 isolating metric** |
| **Clean picks before first unrecoverable tangle** (count) | how far the batch progresses before blocking | **H1 isolating metric** |
| **Verification accuracy** (%) | correct success / fail / tangle calls | **H2** |
| **Tangle-detection rate & false-alarm rate** (%) | non-separation events correctly flagged | **H2** |
| **Clearance used** (mm) + total vertical travel | motion economy | H3 |
| **Skipped/blocked picks** (count) | abandoned (`max_clearance` or tangle) | H1×H3 / H2 |
| **Cycle time** (s) | detection → pick complete, per item & batch | all |
| **3D pose MAE** (mm, X/Y/Z) | vs ground truth; vs 2D baseline; **vs sensor** | Q4 |
| **(stretch) lightly-tangled recovery rate** (%) | disturbance-nudge successes | H7 |

### 7.4 2D-vs-3D and D435-vs-D405 baseline comparison (Q4)
Same 20 components at known positions/heights, estimated by (a) the 2D homography (calibrated height only), (b) the depth pipeline on the **D435**, and (c) the depth pipeline on the **D405**. Report MAE in X/Y/Z **as a function of object height**: parallax error in the 2D baseline should grow with height while the 3D pipelines stay flatter, and the D405 should hold tighter Z than the D435 on small near parts. This is the cleanest single figure in the report and converts the sensor choice into measured evidence.

### 7.5 Scene-composition characterisation (C3 — new)
Repeatedly dump a fixed inventory of modules (N≥30) and classify the resting state of each part: **free / stacked / lightly-tangled / strongly-tangled.** Report the distribution and its variance across drops. This **histogram is a contribution in its own right** — the first such characterisation for pin-bearing hobby modules — and it calibrates how often each rung of the difficulty ladder actually occurs, which frames the whole results section.

### 7.6 Ground-truth methodology
Pick-point truth by (in order of preference): a **machined fixture / jig** with known coordinates incl. an **elevated/stacked tier**; or printed grid + vernier calipers; or **robot-touch** (jog to contact, read `Here`). Method fixed in Week 6; stated explicitly (an examiner *will* ask "how do you know ground truth?").

### 7.7 Error budget (headline rigor)
Decompose pick-point error into sources and quantify each; the sum should predict observed MAE:

| Source | Estimate method |
|---|---|
| Camera intrinsics | calibration reprojection RMS (per sensor) |
| Hand-eye extrinsic | held-out residual (§6.3, per sensor) |
| Depth noise @ working distance | repeated static captures, std of centroid (per sensor) |
| Segmentation/centroid | synthetic top-band test (`test_parallax.py`) |
| Robot positioning repeatability | datasheet + repeated `Here` returns |

### 7.8 Ablations
top-band filter on/off (Z-error), completeness-threshold sweep, temporal-filter on/off, sequencing on/off (subsumed by 7.1), clearance step size, verification ROI-change threshold, **sensor (D435 vs D405)**. Ablations *prove* design choices instead of asserting them.

### 7.9 Statistical analysis plan
- **Proportions** (yield, collision rate, verification accuracy): chi-square / Fisher's exact, 95% CIs.
- **Continuous** (cycle time, clearance used): two-way ANOVA on the 2×2 (main effects + interaction), or Mann-Whitney if non-normal.
- **Common-mode control:** seeded layouts replayed across conditions (paired analysis where applicable) so entanglement does not confound the sequencing effect.
- Report **effect sizes** (Cohen's d / odds ratios), not just p-values; pre-state N per cell for power (JP: "big statistical difference is good").

### 7.10 Failure-envelope & entanglement taxonomy
Deliberately locate and *report* breakdowns: glossy LCD depth dropouts; two parts touching; parts at the depth-FOV edge; high occlusion where YOLO fails; and a **named entanglement taxonomy** (light vs strong pin-interlock) with detection rates. Documenting one's own limits is disarming in a viva and directly answers Q5/§3.6.

### 7.11 Unit & integration verification (no hardware)
- `pytest tests/test_parallax.py` — synthetic crop centroid within 1 mm; camera→base→camera round-trip is identity.
- `pytest tests/test_obstacle.py` — plane + 2 clusters → exactly one obstacle.
- `pytest tests/test_verify.py` — pre/post ROI deltas → correct success / fail / tangle (co-motion) calls.
- `pytest tests/test_grip.py` / `test_sender.py` — lookup + default; 4- vs 5-field protocol.
- Offline integration on a `.bag` (flat surface + Arduino + LCD + a tall "obstacle" mug + a deliberately stacked pair): expect topmost-first depth-corrected picks, mug flagged `unknown_obstacle`, clearances ≥ 50 mm; with the mug in the cone, expect clearance to grow or a logged `BLOCKED` skip; on a simulated non-separation, expect a `skip-tangle` log.

---

## 8. Risks & Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Hardware (sensors / arm) unavailable or delayed | High | Offline `.bag`/`.npy` harness (Week 2) decouples dev + unit tests; experiment weeks back-loaded; D405 PO raised Week 1 |
| **Tangling is common-mode** → dilutes the sequencing effect | Medium | Seeded replayed layouts hold tangle fraction constant; report isolating metrics (approach collisions; clean picks before first unrecoverable tangle); ~20% strong-tangle estimate is low enough to leave headroom |
| **H1 effect small** on sparse scenes → weak headline | Medium | Engineer dense, stacked, tall-next-to-short scenes (§7.1); power the design |
| D405 no-projector → poor depth in **dim light** | Medium | D435 (active projector) carries the dim-lighting condition; D405 used where it is strongest (bright/normal, close-range Z) |
| D405 ≤50 cm range vs arm clearance / occlusion | Medium | Verify mount height vs `Jump3` arc + gripper length before commissioning; mount offset/angled; D435 as wide-coverage fallback |
| Depth Z-noise on small parts | Medium | RGB-for-XY/depth-for-Z + many-point averaging + filter chain (§6.2); D405 added specifically to attack this |
| Glossy/specular dropouts | Medium | Scoped out; characterised (§7.10); cross-polarised lighting mitigation (§6.2) |
| Thesis 2 interface mismatch | Medium | Written, signed-off interface contract in Week 6 |
| Calibration drift | Low | Held-out residual gate (<5 mm); re-calibrate if exceeded |
| 8-page limit vs content volume | Medium | Blog (40%) carries the detailed narrative; report stays dense and selective |
| Stretch (recovery) eats time | Medium | H7 gated behind both pillars being secure; detect-and-skip is the guaranteed deliverable |

---

## 9. Deliverables

1. **Project blog (40%)** — weekly updates + final video (due 26 Jul).
2. **Research article ~8 pages (40%)** — lit review (§3), design (§6), experiment (§7), results, conclusions (due 25 Sep).
3. **Final presentation (20%)** — supervisor + second marker (26 Oct).
4. **Code** — `pickerbot_depth.py` + `pickerbot_lib/*` (incl. `verify.py`) + tests, on the existing Picker-Bot repo.
5. **Datasets** — calibration (per sensor), pick-trial logs, **scene-composition statistics (C3)**, pose-MAE measurements (2D / D435 / D405), verification logs, cycle-time recordings.
6. **Interface specification** — perception ↔ end-effector contract for Thesis 2 (incl. `skip-tangle` signal).

---

## 10. Open Questions / Deferred

- Robot pose ingestion for hand-eye (`WHERE` SPEL+ command vs manual entry).
- Calibration drift auto-detection at startup (fiducial check).
- Grip-strength → gripper wiring (Thesis 2; extend PICK with `grip_g` later).
- Full 6-DoF (tilted-part) pose via plane-normal estimation.
- **Active disentanglement** of strongly tangled clusters (shaker/regrasp/learned classifier — §3.6).
- ROS2/MoveIt2 + EPSON SPEL+ bridge (§3.5).

---

## 11. Indicative References (by cluster — to be expanded in Week 1–2)

**Sequencing / clutter (§3.2 — H1 lineage):**
- Nam et al. (2020), *Fast and resilient manipulation planning for target retrieval in clutter*. arXiv:2003.11420.
- *Real-Time Online Re-Planning for Grasping Under Clutter and Uncertainty* (2018). arXiv:1807.09049.
- *Taming Combinatorial Challenges in Optimal Clutter Removal* (2019). arXiv:1905.13530.
- *ClutterNav: Gradient-Guided Search for Efficient 3D Clutter Removal* (2025). arXiv:2511.12479.

**Entanglement / interlocking parts (§3.6 — Pillar 2 boundary; *verify exact titles/years in Week 1*):**
- Matsumura, Domae, Wan & Harada — bin-picking of potentially tangled objects (IROS, 2019).
- Fraunhofer IPA — separating entangled workpieces in random bin-picking (~2020–2022).
- *(seek 1–2 more on wire-harness / hook disentangling and learned entanglement classifiers.)*

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
- Coleman et al. (2014), *Reducing the barrier to entry… a MoveIt! case study*. JOSER 5(1).

**Sensor (§6.2):**
- Intel RealSense **D435** / **D405** product datasheets, Intel / RealSense (2023–2025).
