---
layout: default
title: Home
nav_order: 0
---

# Picker-Bot — Progress Blog

### Depth-Driven Pick Sequencing and Failure-Aware Verification for Bill-of-Quantities Retrieval of Microelectronic Modules from a Cluttered Workspace

**Faseeh Mohammed** (M01088120) · MSc Robotics, Middlesex University Dubai · Module **PDE4445**
Supervisors: **Dr. Sameer Kishore** · Companion thesis (end-effector & proprioceptive): **Aman Mishra**
Platform: **EPSON VT6-A901S** 6-axis arm · **Intel RealSense D435i** · **YOLOv8n-seg instance segmentation**
Code & work: **[GitHub repository](https://github.com/MrRox1337/picker-bot/tree/pde4445-pickerbot-faseeh/pde4445-dev)**

*This project has concluded*

---

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;max-width:100%;margin:1rem 0;">
  <iframe src="https://www.youtube.com/embed/RvMV7aG49nk" title="Picker-Bot — Full Demo" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

*[Watch on YouTube ↗](https://youtu.be/RvMV7aG49nk) — the full Picker-Bot demo: BOQ in, fulfilment report out.*

---

## The project in one glance

An electronics bench doesn't hold one part at a time it holds many kinds together: Arduinos, ESP32s, LCD panels, ultrasonic rangefinders, left resting on and against each other after use. Clearing the bench isn't actually the job; **retrieving the specific parts a schematic calls for** is. The project ended up reframed around that: a **bill of quantities (BOQ)** in, a **fulfilment report** out .

- **Depth-ordered, topmost-first sequencing.** These parts are near-uniform in thickness, so top-surface height is a free, depth-only proxy for what's accessible next one sort recovers it, instead of the occlusion reasoning heavier planners compute. Reported with the **null result** it returns: under continuous re-scanning, a depth-agnostic order clears just as well.
- **Two-signal, failure-aware verification.** A grasp is confirmed by the gripper's own proprioceptive check *and* a position-matched visual re-scan, because on real hardware they fail differently.

The operating envelope is declared: every part must be separable by a vertical lift alone — evenly spaced, neighbouring, overlapping, or stacked flat all qualify. Parts mechanically **interlocked** through their GPIO pins are excluded.

![The scene that motivated the project, tipped from a bucket, before the operating envelope narrowed to a single separability criterion.](img/june3/flatplane.png)

*The original difficulty spectrum this project started from. It's since been replaced by a precise boundary: separable by a vertical lift, or excluded.*

> **Aim.** Convert a planar, open-loop, order-agnostic predecessor into a depth-aware, class-conditional retrieval system that fulfils a BOQ inside that envelope, verifies every attempt, and reports its performance and failure modes as measured.

**Four research questions:**

- **RQ1 — Perception.** How reliably does segmentation + depth recover class, pose and height, and how does that degrade across scene density, lighting, and out-of-distribution objects? → **89.8% recall** over 45 recordings; 74% under clutter, 80% under low light.
- **RQ2 — Sequencing.** Does topmost-first improve retrieval over an order-agnostic baseline, and does that depend on re-planning? → Only once re-planning is switched off.
- **RQ3 — Verification.** Can two signals separate grasp failure from perception failure? → Yes — nothing evaded both signals in these trials.
- **RQ4 — The deficit.** What governs the gap between easy and hard scenes? → Perception, not the gripper: **93% fulfilment isolated vs. 57% crowded**, traced to mask contamination where parts touch.

---

## Start here (reading path)

1. [Initial brainstorming]({% link june-week1.md %}) — the four features I started with.
2. [First supervisor meeting]({% link june-2.md %}) — the hard questions that reframed everything.
3. [Subject Area Review — Pt.1: Pick Sequencing]({% link june-3.md %}) — where "topmost-first" comes from.
4. [Subject Area Review — Pt.2: Entanglement]({% link june-4.md %}) — the hard boundary, and my object-class gap.
5. [Subject Area Review — Pt.3: Verification & Recovery]({% link july-1.md %}) — how "detect the tangle" actually works.
6. [Finalised — Problem, Question & Title]({% link july-2.md %}) — the locked framing, later reframed again around BOQ retrieval.


---

## Timeline

<div class="pb-gantt-scroll" markdown="0">
<style>
  .pb-gantt-scroll { overflow-x: auto; -webkit-overflow-scrolling: touch; }
  .pb-gantt { padding: 0.5rem 0 0.25rem; font-family: inherit; min-width: 720px; }
  .pb-gantt .g-title { font-size: 15px; font-weight: 600; color:#1b1b32; }
  .pb-gantt .g-sub { font-size: 12px; color:#5c5f66; margin: 2px 0 16px; }
  .pb-gantt .g-header { display:flex; margin-left:220px; }
  .pb-gantt .g-week { flex:1; text-align:center; font-size:11px; color:#5c5f66; font-weight:600; padding-bottom:2px; }
  .pb-gantt .g-row { display:flex; align-items:center; margin-bottom:7px; }
  .pb-gantt .g-label { width:220px; min-width:220px; padding-right:12px; }
  .pb-gantt .g-phase { font-size:12px; font-weight:700; }
  .pb-gantt .g-task { font-size:11px; color:#4b4b57; line-height:1.35; }
  .pb-gantt .g-cells { flex:1; display:grid; grid-template-columns:repeat(12,1fr); gap:2px; align-items:center; }
  .pb-gantt .bar { height:24px; border-radius:4px; }
  .pb-gantt .tbar { height:17px; border-radius:3px; }
  .pb-gantt .g-div { border-top:1px solid #e6e6ef; margin:8px 0 8px 220px; }
  .pb-gantt .done::after { content:" ✓"; color:#1f8a54; font-weight:700; }
  .pb-gantt .g-legend { display:flex; gap:18px; flex-wrap:wrap; margin:14px 0 0 220px; font-size:11px; color:#5c5f66; }
  .pb-gantt .g-legend b { color:#1b1b32; }
</style>

<div class="pb-gantt">
  <div class="g-header">
    <div class="g-week">W1</div><div class="g-week">W2</div><div class="g-week">W3</div><div class="g-week">W4</div>
    <div class="g-week">W5</div><div class="g-week">W6</div><div class="g-week">W7</div><div class="g-week">W8</div>
    <div class="g-week">W9</div><div class="g-week">W10</div><div class="g-week">W11</div><div class="g-week">W12</div>
  </div>

  <!-- Phase 1 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#2f6fb0;">Phase 1 — Foundations &amp; framing</div><div class="g-task">Weeks 1–2 · done</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:1/3; background:#2f6fb0;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Literature review — three clusters</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Problem statement, research question &amp; title</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Blog live + subject-area-review posts</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Hardware PO submitted (D435i arrived, not the D405)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:1/2; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Evaluation design (later superseded by isolated/crowded batteries)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:2/3; background:#9cc2e8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Offline .db3 depth harness + code scaffold</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:2/3; background:#9cc2e8;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 2 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#159a72;">Phase 2 — Perception &amp; calibration</div><div class="g-task">Weeks 3–6 · done — twice as long as planned</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:3/7; background:#159a72;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Sensor bring-up — D435i arrives, first frames (17–19 Jul)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:3/4; background:#7fd3b6;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Depth segmentation discovery — the pile-in-one-box finding</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:3/4; background:#7fd3b6;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Camera mount + capture pose, tilt to 0.2°</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:4/5; background:#7fd3b6;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Unplanned pivot: oriented boxes → instance segmentation</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:5/6; background:#7fd3b6;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Hand–eye calibration — RMS 2.67 mm (target was &lt;5 mm)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:6/7; background:#7fd3b6;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 3 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#5348b0;">Phase 3 — Mechanisms</div><div class="g-task">Weeks 7–10 · done — shifted two weeks later</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:7/11; background:#5348b0;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Full pipeline dry run + gripper API (13–14 Aug)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:7/8; background:#a9a2e0;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">First live vision-driven motion on the VT6 (15 Aug)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:7/8; background:#a9a2e0;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">First human-in-the-loop gripper pick (2 Sep)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:10/11; background:#a9a2e0;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Clutter clearing, Arduino-only (5 Sep)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:10/11; background:#a9a2e0;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 4 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#c9821f;">Phase 4 — Experiments &amp; analysis</div><div class="g-task">Weeks 11–12 · done — compressed from four weeks to two</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:11/13; background:#c9821f;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Current-range fix — every class finally grips (8–12 Sep)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:11/12; background:#ecc38a;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Re-scan ablation — the null result (12 Sep)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:11/12; background:#ecc38a;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">BOQ retrieval + blocker_cleared (15–19 Sep)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:12/13; background:#ecc38a;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Gated PnP battery — 93% isolated vs. 57% crowded (19 Sep)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:12/13; background:#ecc38a;"></div></div></div>

  <div class="g-div"></div>

  <!-- Phase 5 -->
  <div class="g-row">
    <div class="g-label"><div class="g-phase" style="color:#b0466b;">Phase 5 — Writing &amp; delivery</div><div class="g-task">Week 12 → 25 Sep · done</div></div>
    <div class="g-cells"><div class="bar" style="grid-column:12/13; background:#b0466b;"></div></div>
  </div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Draft ~8-page IEEE-format research article</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:12/13; background:#e0a3b8;"></div></div></div>
  <div class="g-row"><div class="g-label"><div class="g-task done">Report submitted (25 Sep)</div></div>
    <div class="g-cells"><div class="tbar" style="grid-column:12/13; background:#e0a3b8;"></div></div></div>

</div>
</div>

---

## Deliverables & assessment

| Deliverable | Weight | Due | Status |
|---|---|---|---|
| Project blog (this site) | **40%** | 26 Jul 2026 | ✓ Done |
| Project report — ~8-page research article | **40%** | 25 Sep 2026 | ✓ Done |
| Final presentation — supervisor + second marker | **20%** | 30 Sep 2026 | ✓ Done |
