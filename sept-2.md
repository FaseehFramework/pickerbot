---
layout: default
title: "September 05: Clutter clearing, Arduinos only"
parent: September 2026
nav_order: 2
---

# Clutter clearing, Arduinos only

*[Previously,]({% link sept-1.md %}) I did the first human-in-the-loop gripper pick — one part, stepped down by hand, with only the Arduino class provably grippable. This entry is the first time the system worked through a **cluttered, mixed-class scene** on its own initiative: filtering out everything that wasn't an Arduino, picking what was, and logging every attempt clean ones and failed ones alike.*

## The setup

The scene mixed Arduinos with other modules a real clutter. The clearing script was told `--only arduino`, and every detection was run through two gates before anything moved: a confidence floor (0.5) and the class filter. Neither gate is a soft suggestion both show up as `skipped` in the log, with a reason, before any hardware moves:

- **10 LCD detections** across the day, every one correctly skipped as `class not in --only ['arduino']` — zero false positives on class filtering.
- **3 Arduino detections** skipped on confidence alone (0.36, 0.41, 0.47 — all under the 0.5 floor), including one attached to a wildly out-of-range 3D point that the confidence gate happened to catch before the class filter even got a say.

Only what cleared both gates was ever physically attempted.

## The scoreboard

Thirteen real pick attempts went to hardware over the course of the afternoon. Per the meeting the day before this all started ([sept-1.md]({% link sept-1.md %})): every one of them is here, not just the ones that worked.

| Time | Label | Conf. | Outcome | Cycle (s) | Note |
|---|---|---|---|---|---|
| 14:00 | arduino | 0.973 | **placed** | 37.15 | |
| 14:02 | arduino | 0.975 | **placed** | 49.39 | |
| 14:12 | arduino | 0.963 | **placed** | 36.50 | |
| 14:14 | arduino | 0.967 | **placed** | 44.22 | |
| 14:35 | arduino | 0.959 | **release_failed** | 42.87 | gripped and lifted clean, then jammed open at 2017 |
| 16:09 | arduino | 0.959 | **arm_fault** | 34.02 | gripped clean, then arm returned a non-`OK` reply mid-cycle |
| 16:17 | arduino | 0.981 | **no_grasp** | 32.75 | |
| 16:17 | arduino | 0.962 | **no_grasp** | 33.84 | |
| 16:17 | arduino | 0.977 | **no_grasp** | 32.88 | three in a row, same run |
| 16:29 | arduino | 0.516 | **arm_fault** | 28.64 | commanded point was nowhere near the workspace — never even reached the grasp |
| 16:30 | arduino | 0.979 | **placed** | 38.70 | |
| 16:30 | arduino | 0.957 | **placed** | 45.71 | |
| 16:30 | arduino | 0.971 | **placed** | 53.16 | |

| Outcome | Count |
|---|---|
| placed | 7 |
| no_grasp | 3 |
| arm_fault | 2 |
| release_failed | 1 |
| **Total attempted** | **13** |

**7 of 13 — 53.8%.** Not the number I'd lead with if I only kept the best run, which is exactly why it's the number I'm reporting.

## Three ways a real pick fails

The failures were three genuinely different things going wrong, and the log tells them apart cleanly:

- **`no_grasp`** (3×, all in the same run, back to back): the servo closed and stopped, but never far enough to count as holding something. A real failure to grip, not a false read the gripper's own position feedback said so.
- **`arm_fault`** (2×): the arm itself refused the command — one came back with a non `OK` reply mid-cycle *after* a clean grasp; the other was a detection whose 3D point was nowhere near the physical workspace, and the arm faulted rather than obeying it. That second one is the confidence gate's near-miss: 0.516 cleared the 0.5 floor on a bad read, and it was the **arm's own refusal**, not the vision pipeline, that kept a nonsense coordinate from becoming a real problem the exact `OK`-reply guard built [back in August]({% link august-7.md %}) doing its job.
- **`release_failed`** (1×): gripped fine, lifted fine, carried to the place pose fine then the fingers wouldn't open. I'd already pulled the gripper aside earlier that day with a standalone script (`release_test.py`) to time three separate release strategies against each other, because the servo's current ceiling meant opening could never pull harder than gripping had. Whatever came out of that diagnosis held for the rest of the afternoon: nothing released badly again after 14:35.

## What actually worked

The day closed on a clean run: three Arduinos, three placements.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;max-width:100%;margin:1rem 0;">
  <iframe src="https://www.youtube.com/embed/538mh9guqAk" title="Clutter clearing, Arduinos only — Picker-Bot, 5 September 2026" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>
