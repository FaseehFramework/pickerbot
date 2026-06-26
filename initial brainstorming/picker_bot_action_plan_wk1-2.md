# Picker-Bot — Action Plan (Wk 1–2) · 26 Jun → 10 Jul 2026

Anchored to `picker_bot_thesis_v2.md`. Two windows:
- **Window A — now → Fri 3 Jul** (before JP meeting) = thesis **Week 1**
- **Window B — Fri 3 Jul → Fri 10 Jul** = thesis **Week 2**

Priority tags: **[P0]** = must be done for JP / non-negotiable · **[P1]** = important · **[P2]** = nice-to-have.

---

## The lens: what JP will expect on 3 Jul

You changed the headline since he last saw it (height-sequencing → **pile-clearing + failure-aware verification**). The meeting's job is to (1) bring him along on *why* the reframe is stronger and "feels right," (2) show the lit review is underway, and (3) get the PO signed. Everything below serves that.

**If you only do four things before 3 Jul:** blog live + reframe post · the reframed hypothesis & gap written up (1–2 pages) · 2–3 papers each on the *three pillar-critical* clusters · PO finalised and submitted.

---

## Window A — now → Fri 3 Jul (Week 1)

### 1. Blog — set up + first posts **[P0]** (40% of the grade; graded on *continuous* documentation, due 26 Jul)
- [ ] Stand up the blog platform (whatever JP/handbook specifies) and confirm it's visible to markers.
- [ ] **Post 1 — Project & problem:** the pile-clearing problem, the EPSON VT6 + depth setup, the three module classes. (~30 min, reuse v2 §1.)
- [ ] **Post 2 — The reframe (do this one well):** why "tallest-first" became "topmost-first pile clearing + verification," the difficulty ladder (free / stacked / lightly-tangled / strongly-tangled), and why it "feels right." This doubles as your JP talking points.
- [ ] Add a "this week" log entry stub you'll append to as you go.

### 2. Literature review — start, prioritised **[P0]**
JP asked for "2–3+ papers per area: existing solutions, why insufficient, how mine differs." Do the **three pillar-critical clusters first**, then the rest if time:
- [ ] **§3.2 Sequencing / clutter removal** (home of Pillar 1) — Nam 2020, the 2018 re-planning paper, ClutterNav 2025. Land the "cheap proxy, sufficient-not-optimal" framing.
- [ ] **§3.6 Entanglement** (Pillar 2's hard boundary) — **confirm exact titles/years** of the indicative refs (Matsumura/Domae IROS 2019; Fraunhofer IPA disentangling). Find 1–2 more.
- [ ] **§3.3 Verification / failure recovery** (Pillar 2) — Frontiers Neurorobotics 2021; one VLM closed-loop paper.
- [ ] **[P1]** §3.1 RGBD pose, §3.4 sim-to-real, §3.5 ROS2 — lighter pass.
- [ ] Keep a running BibTeX/reference file as you go (don't leave citations to the end).

### 3. The reframed justification & gap — write it up **[P0]**
- [ ] 1–2 pages: central hypotheses (H1 sequencing, H2 verification), the difficulty ladder, and the **research-gap statement** (v2 §3.7). This is the single most important artifact for the meeting — it's exactly what JP asked you to nail.

### 4. Hardware PO — finalise & submit **[P0]**
- [ ] Confirm **D405** price at the official store; finalise the BOM (rigid mount/gantry, rigid ChArUco board, dimmable diffuse LED + **cross-polariser**, **active USB-3 cable 2–3 m**, ground-truth fixture, matte surface).
- [ ] Note the **OAK-D Pro** role in the PO: active-stereo scene workhorse (can retire the D435), *not* a D405 substitute — and a 3rd sensor-comparison point.
- [ ] **[P0] Verify the D405 ≤50 cm mounting-height** against the VT6's `Jump3` approach arc + gripper length *before* the PO is locked — this is the one thing that could veto the D405.
- [ ] Submit PO / get JP sign-off lined up for the meeting.

### 5. Objectives + Gantt **[P1]**
- [ ] Finalise project objectives (lift from v2 §2 contributions).
- [ ] Update the Gantt (`picker_bot_dissertation_gantt.html`) to the v2 two-pillar plan + sensor comparison.

### 6. Track during Window A
Keep a one-line daily note (feeds the blog): papers read, decisions made, blockers. Bring a short bullet list of progress + open questions to the 3 Jul meeting.

---

## Window B — Fri 3 Jul → Fri 10 Jul (Week 2)

### 1. Blog **[P0]**
- [ ] Weekly update post: Week 1 outcomes, JP feedback, Week 2 plan. (Keep the cadence — markers reward consistency.)

### 2. Finish the literature review **[P0]**
- [ ] Close out all 6 clusters to draft quality (this is the §3 the report leans on).

### 3. Experiment design — §7 written **[P0]** (JP: "what you're going to test and how")
- [ ] The **2×2 factorial** (sequencing × clearance), IVs/DVs, **seeded replayed layouts** for common-mode tangle control.
- [ ] The **scene-composition study** (dump ≥30 parts, classify free/stacked/light/strong — your C3 histogram) — define the protocol now.
- [ ] Lock the **isolating metrics** (approach collisions; clean picks before first unrecoverable tangle) and the stats plan.
- [ ] Define the **2D-vs-3D + D405-vs-OAK-D Pro** pose-MAE comparison and ground-truth approach (rig built later in Wk 6, but decide the method now).

### 4. Code scaffold + offline harness **[P0]** (de-risks all later hardware weeks)
- [ ] Scaffold `pickerbot_depth.py` + `pickerbot_lib/{depth,parallax,obstacle,extrinsics,grip,verify,boq}.py`.
- [ ] Build the **`DepthSource` abstraction** (`Live` / `Bag` / `Npy`) so the pipeline runs and unit-tests **without hardware**.
- [ ] Stub the test suite (`tests/test_{parallax,obstacle,verify,grip,sender}.py`) — even failing stubs define the contract.

### 5. If hardware arrives early **[P1]**
- [ ] Begin sensor bring-up: intrinsics capture for D405 / OAK-D Pro; record a first `.bag` for the offline harness.

### 6. Track during Window B
- [ ] Daily one-liner → blog. Flag any slip on the experiment-design or harness milestones early; both are on the critical path to the Wk 8–9 experiments.

---

## Recurring (every week, don't let it slip)
- **Blog post + daily log** — it's 40% and the first hard deadline (26 Jul, end of Wk 4).
- **Reference file** — append as you read.
- **Decision log** — every design choice you make is a future ablation / viva answer; write down *why*.
