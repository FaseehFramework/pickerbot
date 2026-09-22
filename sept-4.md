---
layout: default
title: "September 15 - September 19: BOQ retrieval, and clearing a known blocker"
parent: September 2026
nav_order: 4
---

# BOQ retrieval, and clearing a known blocker

*[Previously,]({% link sept-3.md %}) every class could finally be gripped. This week's two highlights: the system takes an actual **order** rather than "clear everything," and it can deliberately move a known part aside to reach the one underneath it.*

## The bill of quantities

The idea: vision reports what it sees on the bench, then I say what I actually want from it. `boq_fetch.py` scans, prints the inventory, asks for the order, shows the plan — including what it will deliberately leave behind — and waits for confirmation before anything moves.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;max-width:100%;margin:1rem 0;">
  <iframe src="https://www.youtube.com/embed/yW7HbDkuMaY" title="BOQ retrieval — Picker-Bot, September 2026" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

*[Watch on YouTube ↗](https://youtu.be/yW7HbDkuMaY?si=ZrjwDnjehUMLpH6p) — a cluttered scene with all five known classes on the bench; the bill of quantities was 1 esp, 1 arduino, 1 ultrasonic, and only those three came off the table.*

Asking for **two of the same class** (two ultrasonics on the bench, one requested) exercises the same logic from the other side — the quota is met after the first, and the second is correctly refused rather than picked anyway.

## Clearing a known blocker

The harder case: an LCD sitting squarely on top of the Arduino I actually wanted. The system recognised it as a **known class blocking a known target**, moved it aside on purpose, re-scanned, then fetched the Arduino underneath — logged as `blocker_cleared`, a distinct outcome from an ordinary pick.

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;max-width:100%;margin:1rem 0;">
  <iframe src="https://www.youtube.com/embed/OeUeevRCT-4" title="Clearing a known blocker — Picker-Bot, September 2026" style="position:absolute;top:0;left:0;width:100%;height:100%;border:0;" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

*[Watch on YouTube ↗](https://youtu.be/OeUeevRCT-4?si=VSAlF2gAjGjFio2a) — the LCD moved aside, a re-scan, then the Arduino it was sitting on delivered.*

> Three separate times this week the gripper's own feedback said it was holding something, and the camera's re-scan said otherwise. One signal alone isn't enough — that's the case for verifying with both, made with real repeats rather than one lucky anecdote.

## The numbers that finally separate

| Condition | Attempts | OK | Rate | Units | Delivered | Fulfilment |
|---|---|---|---|---|---|---|
| easy | 34 | 28 | 82% | 30 | 28 | 93% |
| hard | 33 | 17 | 52% | 30 | 17 | 57% |

Matched at 30 units each, and unlike an earlier version of this same test — the two conditions now actually differ in crowding (0.000 vs 0.123), so "hard" means something real this time.