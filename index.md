---
layout: default
title: Home
nav_order: 0
---

# Picker-Bot — Progress Blog

### Depth-Driven Pick Sequencing with Failure-Aware Verification for Clearing Unstructured Piles of Interlocking Microelectronic Modules on an EPSON VT6-A901S

**Faseeh Mohammed** (M01088120) · MSc Robotics, Middlesex University Dubai · Module **PDE4445**
Supervisor: **Dr. Sameer** · Companion thesis (end-effector & proprioceptive): **Aman Mishra**
Platform: **EPSON VT6-A901S** 6-axis arm · **Intel RealSense D405** (+ D435 / OAK-D Pro) · **YOLOv8-OBB**

---

## The project in one glance

Tip a bucket of small hobby modules (Arduino, ESP32, LCD) onto a table and you don't get a tidy flat scatter — you get a **pile**: some parts free, some stacked, and some **hooked together through their GPIO pins**. The existing Picker-Bot assumes every part lies flat at one calibrated height, so it mis-locates anything raised, has no idea what order to clear the pile in, and never checks whether a pick worked. This project rebuilds the pipeline around a depth camera and **two cheap, honest ideas**:

- **Pillar 1 — Topmost-first sequencing.** Clear the pile from the top down. Because the parts are near-identical in height, what matters isn't how *tall* a part is but how high its top sits *in the pile* — a single `sort()` on the depth map recovers the "what's accessible next" signal that heavier planners compute expensively.
- **Pillar 2 — Failure-aware verification.** After each pick, re-scan and ask *did exactly one object leave?* If a pick fails or drags a tangled neighbour, **detect it and skip** — bounding the one failure mode (pin entanglement) the system can't solve.

![The difficulty ladder — free singles, stacked/overlapping, lightly tangled, strongly tangled — shown on the actual modules.](img/june3/flatplane.png)

*The scene, as a difficulty ladder. Sequencing (Pillar 1) peels off the left; verification (Pillar 2) detects and bounds the right.*

> **Research question.** On a physical industrial arm clearing an unstructured pile of small, near-identical modules, does a cheap depth-only **topmost-first** sequence — with a lightweight vision re-scan that **detects and skips** tangled picks — reduce gripper collisions and improve clean-pick yield versus an unordered, open-loop baseline, by a statistically significant margin? *"Better" = fewer collisions and higher clean-pick yield at negligible compute — every metric tied to the arm.*

---

## Start here (reading path)

1. [Initial brainstorming]({% link june-week1.md %}) — the four features I started with.
2. [First supervisor meeting]({% link june-2.md %}) — the hard questions that reframed everything.
3. [Subject Area Review — Pt.1: Pick Sequencing]({% link june-3.md %}) — where "topmost-first" comes from.
4. [Subject Area Review — Pt.2: Entanglement]({% link june-4.md %}) — the hard boundary, and my object-class gap.
5. [Subject Area Review — Pt.3: Verification & Recovery]({% link july-1.md %}) — how "detect the tangle" actually works.
6. [Finalised — Problem, Question & Title]({% link july-2.md %}) — the locked framing.

---

## Timeline — 12 weeks

<div class="pb-gantt-scroll" markdown="0">
<style>
  .pb-gantt-scroll { overflow-x: auto; -webkit-overflow-scrolling: touch; }
  .pb-gantt { padding: 0.5rem 0 0.25rem; font-family: inherit; min-width: 720px; }
  .pb-gantt .g-title { font-size: 15px; font-weight: 600; color:#1b1b32; }
  .pb-gantt .g-sub { font-size: 12px; color:#5c5f66; margin: 2px 0 16px; }
  .pb-gantt .g-header { display:flex; margin-left:220px; }
  .pb-gantt .g-week { flex:1; text-align:center; font-size:11px; color:#5c5f66; font-weight:600; padding-bottom:2px; }
  .pb-gantt .g-week.now { color:#b0466b; }
  .pb-gantt .g-now { flex:1; text-align:center; font-size:10px; color:#b0466b; font-weight:700; margin-left:220px; display:flex; }
  .pb-gantt .g-now span { flex:1; text-align:center; }
  .pb-gantt .g-row { display:flex; align-items:center; margin-bottom:7px; }
  .pb-gantt .g-label { width:220px; min-width:220px; padding-right:12px; }
  .pb-gantt .g-phase { font-size:12px; font-weight:700; }
  .pb-gantt .g-task { font-size:11px; color:#4b4b57; line-height:1.35; }
  .pb-gantt .g-cells { flex:1; display:grid; grid-template-columns:repeat(12,1fr); gap:2px; align-items:center; }
  .pb-gantt .bar { height:24px; border-radius:4px; }
  .pb-gantt .tbar { height:17px; border-radius:3px; }
  .pb-gantt .nowcol { grid-column:2/3; height:100%; background:rgba(176,70,107,.08); border-left:2px solid rgba(176,70,107,.35); border-right:2px solid rgba(176,70,107,.35);}
  .pb-gantt .g-div { border-top:1px solid #e6e6ef; margin:8px 0 8px 220px; }
  .pb-gantt .done::after { content:" ✓"; color:#1f8a54; font-weight:700; }
  .pb-gantt .wip::after { content:" ◐"; color:#c9821f; font-weight:700; }
  .pb-gantt .g-legend { display:flex; gap:18px; flex-wrap:wrap; margin:14px 0 0 220px; font-size:11px; color:#5c5f66; }
  .pb-gantt .g-legend b { color:#1b1b32; }
</style>

<div class="pb-gantt">
  <div class="g-title">Dissertation timeline — 12 build weeks (w/c 30 Jun → 15 Sep 2026)</div>
  <div class="g-sub">✓ complete · ◐ in progress · ▼ current week</div>

  <div class="g-now"><span></span><span>▼ now (Wk&nbsp;2)</span><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span><span></span></div>
  <div class="g-header">
    <div class="g-week">W1</div><div class="g-week now">W2</div><div class="g-week">W3</div><div class="g-week">W4</div>
    <div class="g-week">W5</div><div class="g-week">W6</div><div class="g-week">W7</div><div class="g-week">W8</div>
    <div class="g-week">W9</div><div class="g-week">W10</div><div class="g-week">W11</div><div class="g-week">W12</div>
  </div>

  <!-- Phase 1 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#2f6fb0;">Phase 1 — Foundations &amp; framing</div><div class="g-task">Weeks 1–2 · done</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:1/3; background:#2f6fb0;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Literature review — 6 clusters</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Problem statement, research question &amp; title</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Blog live + subject-area-review posts</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Hardware PO (D405 + BOM) submitted</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/2; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task wip">Experiment design (2×2 factorial, metrics)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:2/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task wip">Offline .bag/.npy depth harness + code scaffold</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:2/3; background:#9cc2e8;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 2 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#159a72;">Phase 2 — Perception &amp; calibration</div><div class="g-task">Weeks 3–4 · needs camera delivery</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:3/5; background:#159a72;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task">Sensor bring-up + intrinsics (D405 / D435 / OAK)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:3/4; background:#7fd3b6;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Eye-to-hand calibration (residual &lt; 5 mm)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:3/4; background:#7fd3b6;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Parallax / top-surface-centroid pose pipeline</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:4/5; background:#7fd3b6;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Fuse depth with YOLOv8-OBB · <b>▲ Blog due 26 Jul</b></div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:4/5; background:#7fd3b6;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 3 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#5348b0;">Phase 3 — Mechanisms</div><div class="g-task">Weeks 5–6</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:5/7; background:#5348b0;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task">Obstacle detection (RANSAC + DBSCAN)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:5/6; background:#a9a2e0;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Topmost-first sequencing (H1) + dynamic clearance (H3)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:5/6; background:#a9a2e0;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Vision verification + tangle detection (H2)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:6/7; background:#a9a2e0;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Thesis-2 interface contract (with Aman) + GT rig</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:6/7; background:#a9a2e0;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 4 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#c9821f;">Phase 4 — Experiments &amp; analysis</div><div class="g-task">Weeks 7–10</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:7/11; background:#c9821f;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task">Pilot trials + scene-composition study (C3)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:7/8; background:#ecc38a;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Main experiment — block 1 (2×2 factorial)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:8/9; background:#ecc38a;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Block 2 (lighting/regime/occlusion) + 2D-vs-3D + sensor MAE</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:9/10; background:#ecc38a;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Analysis: significance, effect sizes, error budget, ablations</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:10/11; background:#ecc38a;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 5 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#b0466b;">Phase 5 — Writing &amp; delivery</div><div class="g-task">Weeks 11–12 → Oct</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:11/13; background:#b0466b;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task">Draft ~8-page research article + figures</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:11/12; background:#e0a3b8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task">Finalise report + polish blog &amp; final video</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:12/13; background:#e0a3b8;"></div></div></div>

  <div class="g-legend">
    <span><b>Milestones:</b></span>
    <span>▲ Blog (40%) — 26 Jul</span>
    <span>▲ Report (40%) — 25 Sep</span>
    <span>▲ Presentation (20%) — 26 Oct</span>
  </div>
</div>
</div>

*Weeks 1–2 (literature review, problem framing, blog, hardware PO) are complete; I'm currently in Week 2, on experiment design and the offline depth harness. **Phase 2 (sensor bring-up) is gated on the D405/OAK-D Pro delivery** — the PO is submitted; the offline harness is deliberately built first so perception work isn't blocked while the camera ships.*

---

## Deliverables & assessment

| Deliverable | Weight | Due |
|---|---|---|
| Project blog (this site) — weekly updates + final video | **40%** | 26 Jul 2026 |
| Project report — ~8-page research article | **40%** | 25 Sep 2026 |
| Final presentation — supervisor + second marker | **20%** | 26 Oct 2026 |
