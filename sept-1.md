---
layout: default
title: "September 02: First human-in-the-loop gripper pick"
parent: September 2026
nav_order: 1
---

# First human-in-the-loop gripper pick

*[Previously,]({% link august-8.md %}) the vision-to-motion chain drove the real VT6 for the first time, with two bounded residuals left to close: a constant Z offset and an unknown yaw offset. This entry is where both get measured, the gripper finally comes online, and for the first time the arm picks a part it saw, and grasps it, with me stepping the descent by hand.*

## Setting the evaluation bar, the day before

yesterday, in a meeting with Dr. Sameer, the brief for evaluating Picker Bot got a lot sharper. don't evaluate on the best run, evaluate on all of them. Individual picks *and* closed loop runs, across scene densities from scattered to regular to bunched-up, with a deliberately adversarial case thrown in "put a spoon from my kitchen" was the actual suggestion, to see what a truly out-of-distribution object does to the pipeline. Track it with real metrics: accuracy, time, false positives. And the reassurance that matters most for how I now run every session: **a bad number in a hard condition is not a problem for the grade only hiding it is.**

That framing is exactly why tonight is the first entry I can put real tables in.

## The scene

A fresh capture through `pose_seg.py`, sorted **topmost-first by robot Z** — five detections:

| Order | Label | Conf. | X (mm) | Y (mm) | Z (mm) | Yaw (°) | Aspect | Grippable today? |
|---|---|---|---|---|---|---|---|---|
| 1 | esp | 0.974 | 32.3 | 718.0 | 249.5 | 62.5 | 1.82 | No — too thin |
| 2 | ultrasonic | 0.954 | −74.7 | 795.1 | 249.3 | 90.8 | 1.68 | No — too thin |
| 3 | lcd | 0.984 | 48.9 | 824.9 | 248.9 | 37.1 | 2.27 | No — too thin |
| 4 | arduino | 0.976 | −126.4 | 686.1 | 247.9 | −2.6 | 1.35 | Yes |
| 5 | arduino | 0.985 | 35.8 | 598.4 | 245.8 | −31.6 | 1.42 | Yes |

Confidence is strong across the board (0.95–0.99), and the ordering is correctly topmost-first by Z.

## One laptop, one driver

The earlier two-laptop plan for the gripper (Aman's servo bridged over the LAN, from the [August 15 session]({% link august-8.md %})) is gone. `picker-bot` and Aman's `GripSense` now sit side by side on the same machine, and a small locator (`gripsense_path.py`) finds his repo and puts its driver on my path so no more sockets, no server process, just an `import`. 

## Measuring the grasp, not guessing it

Before touching a real part, I measured whether "closed" actually means "holding something." `grip_verify.py` closes the gripper empty, then closed on an Arduino, three times each, and reads the servo's own position:

| Condition | Reading 1 | Reading 2 | Reading 3 | Mean |
|---|---|---|---|---|
| Empty (nothing between the fingers) | 1829 | 1829 | 1829 | 1829 |
| Loaded (on an Arduino) | 2126 | 2126 | 2126 | 2126 |

A **297-tick gap, with zero variance on either side** the two cases don't overlap at all. `HOLD_THRESHOLD` is set at **1977**, almost exactly the midpoint. Above it, something is between the fingers; at or below it, they closed on air.

That measurement immediately explained a result from earlier testing: **80 raw current never reached the object.** It stalled at ~1829 for every part regardless of thickness — indistinguishable from gripping nothing, because it *was* gripping nothing. That's the fingers' own friction-limited closed position, not a grasp. It took **110 raw** to actually arrest on the Arduino with zero slip on the lift.

| Constant | Value | Why |
|---|---|---|
| `GRIP_CURRENT` | 110 raw | Held the Arduino with zero slip on the lift; 80 raw never reached the object. |
| `OPEN_CURRENT` | 160 raw | Must exceed the grip current — opening at 100 after a 110 grip moved the fingers 0 ticks. |
| `OPEN_RETRY_CURRENT` | 200 raw | One firmer retry if the first open stalls. |
| `RELAX_CURRENT` | 50 raw | Drops the ceiling first so compressed padding decompresses before opening. |
| `HOLD_THRESHOLD` | 1977 ticks | Midpoint of the empty/loaded gap above. |
| `SLIP_DROP_TICKS` | 150 ticks | Fingers running on this much further during the lift = the part escaped. |

And the same threshold test drew the honest boundary on the whole system: **the esp, LCD, and ultrasonic modules don't arrest the fingers at all they stall at 1829, exactly like air.** `holding()` correctly reports "no grasp" for them, because there genuinely isn't one. Only the Arduino is thick enough to grip today; the other three classes in the scene above are detected and ranked correctly, but not yet pickable. That's a quantified limitation.

## Calibrating the wrist, not eyeballing it

The other open residual from the last session was the yaw offset between what `pose_seg` reports and what the wrist should actually command. Rather than nudge it by eye, `yaw_calib.py` took five known vision/commanded pairs and solved for it properly:

| Vision yaw (°) | Commanded U (°) |
|---|---|
| 62.5 | −28.0 |
| 90.8 | 0.0 |
| 37.1 | 130.0 |
| −2.6 | 90.0 |
| −31.6 | 60.0 |

The fit: **sign = +1, offset = −88.8°, RMS = 1.5°** over the five pairs.

> `U = sign × yaw + offset`, wrapped to (−180°, 180°]. That single line replaces a hand-eyeballed guess of **+54°** a correction of well over 140° from a value that had *looked* plausible on the bench.

With that solved, both residuals flagged last time are closed: yaw comes from `yaw_calib.json`, and the fingertip height comes from a single measured constant, `TOOL_Z_OFFSET_MM = 70 mm`.

## The pick

Before wiring in the gripper at all, I re-ran August's no-gripper rehearsal fresh, in the confirmed gripper-down orientation, hovering and descending over every part in the new scene with nothing attached. Clean. Only then did the gripper go live.

The actual pick was deliberately **not autonomous**. `pick_one.py` hovers above the target, waits for me to confirm the jaws are aligned across the part's short edge, then descends in 5 mm steps under my own key presses `Enter` down, `u` up, `g` to grip, `a` to abort with a hand on the e-stop the entire time. I targeted one of the two Arduinos in the table above, since the threshold test had already ruled out the other three classes for tonight.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;max-width:100%;margin:1rem 0;">
  <iframe src="https://www.youtube.com/embed/iAxgKESYfMg" title="First human-in-the-loop gripper pick — Picker-Bot, 2 September 2026" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

Stepped down, gripped, `holding()` read true, lifted with no drop beyond `SLIP_DROP_TICKS`, moved to the fixed place pose `(0, 550, 330)` deliberately *not* the camera-down capture pose, which is unreachable gripper down and faults the controller without a reply and released. 

## What this actually proves

> This is the first time both pillars of the thesis ran together on real hardware: **topmost-first sequencing** picked the ordering (the table above), and a **failure-aware check** the gripper's own position feedback against a measured threshold confirmed the grasp rather than assuming it. It's human-supervised, not autonomous, and it's one part, not a pile. But the ordering signal, the grasp-verification signal, and the motion chain are no longer three separate demos they're one run, on video.

## Where this leaves me

Both residuals from the last session are closed with measured constants, not guesses. The grasp decision is a calibrated threshold with a 297-tick separation and zero overlap, not a hope. And exactly one of four classes is provably grippable today ,a limitation I now have numbers for, not just a hunch.

**Next:** run this properly, the way the meeting yesterday described it — every attempt through `runlog.py`, individual and closed-loop, across scattered / regular / bunched-up scenes, and start closing the gap on the three ungrippable classes.
