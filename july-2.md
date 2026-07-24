---
layout: default
title: "05/07/2026 : Finalised — Problem, Question & Title"
parent: July 2026
nav_order: 2
---

# Finalising the Project: Problem, Question & Title

*The [subject-area review]({% link july-1.md %}) is done — three clusters, three roles. Before I write a line of the new pipeline, Dr. Sameer asked me to come back with three things the review should have earned me: a **sharpened problem statement**, a **defensible research question**, and a **title**. Here they are.*

## The sharpened problem statement

When a bucket of small microelectronic modules (Arduino, ESP32, LCD) is tipped onto a table, it does **not** form the flat, one-object-per-place scene the legacy Picker-Bot assumes. It forms a **pile** with a difficulty ladder: free singles, stacked/overlapping parts, and because the boards carry protruding GPIO headers, **pin-tangled** clusters that mechanically interlock. Against that scene the current system fails three ways: its fixed-height homography mis-locates anything not lying flat at the calibration plane (parallax + wrong descent); it has no principled **order** in which to clear the pile; and it is **open-loop**, so it can neither confirm a pick nor recognise a pick it *cannot* make.

The parts are also **near-identical in height**, which is the subtle bit: intrinsic height carries almost no signal, so the useful quantity isn't "how tall is this object" but "how high does its top sit *in the pile*" — i.e. which object is on top and therefore accessible.

## The research question

> **Primary RQ.** On a physical industrial manipulator clearing an unstructured pile of small, near-identical microelectronic modules, does a cheap, depth-only **topmost-first** pick sequence supported by a lightweight vision re-scan that **detects and skips** inseparable (tangled) picks reduce gripper–neighbour collisions and improve clean-pick yield relative to an **unordered, open-loop baseline**, by a statistically significant margin?

**What "better" means** Not "smarter" and not "optimal." Specifically: **fewer approach collisions** and **higher clean-pick yield**, at **negligible added compute and sensing** the whole ordering policy is a single `sort()` on the depth map, no training and no extra models. Every metric is tied to the *arm's* behaviour, not to vision quality in isolation, so the claim stays **robotics**, not computer vision.

### Sub-questions (each maps to a pillar and a measurable outcome)

- **SQ1 — Sequencing (Pillar 1).** How large, and how significant, is the topmost-first effect on collisions and clean-pick yield versus unordered and does it hold up in the dense, stacked scenes where ordering *should* matter?
- **SQ2 — Verification (Pillar 2).** How accurately can a post-pick re-scan classify *success / failed grasp / tangle*, and does "skip-and-bound" keep a batch robust rather than thrashing on an inseparable cluster?
- **SQ3 — Enabling perception.** Does depth-based parallax/descent correction reduce pick-point error (MAE in X/Y/Z) versus the 2D homography baseline as an object's top departs from the calibration plane?
- **SQ4 — Characterisation.** What is the actual **free / stacked / tangled composition** of a dumped pile of these parts, data nobody has reported for pin-headed hobby modules?

## Scope — what I am and am not claiming

- **In:** 4-DoF picking `(x, y, z, yaw)` of opaque modules in a pile; a fixed overhead depth camera; topmost-first ordering; vision verification that **detects and skips** tangles.
- **Out (declared, not hidden):** mechanically **separating** strongly tangled parts (an open, hardware/learning-heavy problem — Part 2); full 6-DoF pose of arbitrarily tilted parts; specular/transparent depth recovery; and the gripper mechanism with its proprioceptive force signal, which is **Aman's companion thesis** — I own the vision side and the interface between us.

## The title

The two ideas that had to survive were **pile-clearing** and **failure-aware verification**; the mechanism itself (topmost-first) lives in the body, so the title stays accurate to the whole system rather than one heuristic. My title:

> **Depth-Driven Pick Sequencing with Failure-Aware Verification for Clearing Unstructured Piles of Interlocking Microelectronic Modules on an EPSON VT6-A901S**

I went with "depth-driven" rather than "height-ordered": *both* pillars lean on the depth camera — the sequencing and the parallax-corrected verification — so it describes the system honestly instead of over-committing the title to a single heuristic. Naming the VT6-A901S keeps it grounded in real hardware.

## Where this leaves me

That's the research framing locked: a positioned problem, a falsifiable question with a baseline and a defined win-condition, and a title. The naive "tallest-first" hunch has become a **cheap proxy for accessibility, to be proven against an unordered baseline** , exactly the move from *builder* to *researcher* Dr. Sameer pushed for.

**Next:** out of the library and onto the bench.Sensor bring-up and the offline depth harness.
