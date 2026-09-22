---
layout: default
title: "September 02: First human-in-the-loop gripper pick"
parent: September 2026
nav_order: 1
---

# First human-in-the-loop gripper pick

*[Previously,]({% link august-8.md %}) the vision-to-motion chain drove the real VT6, with two residuals left: a Z offset and an unknown yaw offset. This entry is where both get measured, the gripper comes online, and the arm grasps a part it saw — with me stepping the descent down by hand.*

Yesterday, in a meeting with Dr. Sameer, the evaluation bar got sharper: report every run, not the best one — **a bad number in a hard condition is not a problem for the grade, only hiding it is.** That's why tonight is the first entry with real tables in it.

## The scene

A fresh capture through `pose_seg.py`, sorted **topmost-first by robot Z** — five detections:

| Order | Label | Conf. | Z (mm) | Yaw (°) | Grippable today? |
|---|---|---|---|---|---|
| 1 | esp | 0.974 | 249.5 | 62.5 | No — too thin |
| 2 | ultrasonic | 0.954 | 249.3 | 90.8 | No — too thin |
| 3 | lcd | 0.984 | 248.9 | 37.1 | No — too thin |
| 4 | arduino | 0.976 | 247.9 | −2.6 | Yes |
| 5 | arduino | 0.985 | 245.8 | −31.6 | Yes |

## Measuring the grasp, not guessing it

`grip_verify.py` closed the gripper empty, then on an Arduino, three times each:

| Condition | Readings | Mean |
|---|---|---|
| Empty | 1829, 1829, 1829 | 1829 |
| Loaded (Arduino) | 2126, 2126, 2126 | 2126 |

A **297-tick gap, zero variance either side.** `HOLD_THRESHOLD` = **1977**, the midpoint. This also explained why **80 raw current never reached the object** — it stalled at ~1829 regardless of thickness, indistinguishable from air. It took **110 raw** to arrest on the Arduino with zero slip on the lift — but esp, lcd, and ultrasonic never arrest the fingers at all; they stall at 1829 like nothing's there. Only the Arduino is grippable tonight.

## Calibrating the wrist, not eyeballing it

`yaw_calib.py` took five known vision/commanded pairs and solved for the offset properly:

| Vision yaw (°) | Commanded U (°) |
|---|---|
| 62.5 | −28.0 |
| 90.8 | 0.0 |
| 37.1 | 130.0 |
| −2.6 | 90.0 |
| −31.6 | 60.0 |

> `U = sign × yaw + offset` → **sign = +1, offset = −88.8°, RMS = 1.5°.** That replaces a hand-eyeballed guess of **+54°** — over 140° off a value that had *looked* plausible on the bench.

## The pick

`pick_one.py` hovers above the target, waits for me to confirm the jaws are aligned, then descends in 5 mm steps under my own key presses — hand on the e-stop the whole time. I targeted one of the two Arduinos, since the threshold test had already ruled out the rest.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;max-width:100%;margin:1rem 0;">
  <iframe src="https://www.youtube.com/embed/iAxgKESYfMg" title="First human-in-the-loop gripper pick — Picker-Bot, 2 September 2026" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

*Stepped down, gripped, `holding()` read true, lifted with no drop, placed, released.*

> Both pillars ran together on real hardware for the first time: **topmost-first sequencing** picked the ordering, and a **failure-aware check** — the gripper's own position feedback against a measured threshold — confirmed the grasp rather than assuming it. Human-supervised, one part, not a pile — but no longer three separate demos. One run, on video.

## Where this leaves me

Both residuals are closed with measured constants, not guesses. The grasp decision is a calibrated threshold with zero overlap. One of four classes is provably grippable today — a limitation I now have numbers for.

**Next:** run this properly through `runlog.py`, individual and closed-loop, across scattered / regular / bunched-up scenes — and close the gap on the three ungrippable classes.
