---
layout: default
title: "August 14: Gripper API and a dry run rehearsal"
parent: August 2026
nav_order: 7
---

# Building own gripper API, and rehearsing the whole pick before touching hardware

*[Previously,]({% link august-6.md %}) the whole perception → pose → pick-list chain ran end-to-end on recordings. Today was a build day rather than a bench day wrapping Aman's gripper driver into my own API, and rehearsing the entire pick in simulation before the arm ever moves.*

## Wrapping Aman's driver, not forking it

I have access to Aman's gripper driver repository ,his GripSense stack and rather than treat his code as a black box I called over the network, I decided to **build my own gripper API directly on top of his driver**. Everything runs on the same PC, his layers are cleanly separated for reuse, and same machine means I can simply `import` his module. His current/slip handshake gets reused as-is, my TCP stays unchanged, and his mechanism opens well past 60 mm, which clears every part in my scene.

The reason this matters for the thesis is that it keeps a clean separation of concerns. Aman owns the *actuation* layer talking to the Dynamixel servo, enforcing travel limits, exposing calibrated open/close ticks. I own the *task* layer deciding when to open, when to close, and how to interpret whether a grasp succeeded. Wrapping his driver instead of forking it means his calibration flows straight through to me without duplication.

## What I actually wrote

Two files came out of the session.

**`gripper_api.py`** is my wrapper over Aman's real driver. It exposes a small, intention-revealing surface: `open()`, `close(current)`, `holding()`, `present_current()`, and `release()`. The interesting design choice is how a grasp works. Instead of commanding a target position and hoping, I use his **current-based position mode**: the fingers drive closed but are capped by a force ceiling, so they *stall on the object* rather than crushing it or slamming shut on empty air. This is compliant grasping in the truest sense the object itself decides where the fingers stop.

That single idea also gives me contact sensing for free. My `holding()` check is simply: *did the fingers stop short of fully closed?* If they did, something is physically between them; if they closed all the way, the grasp missed. No dedicated force sensor, no extra hardware the servo's own position feedback under a current cap becomes an implicit grasp-success signal. It reads Aman's `gripper_settings` and `travel_limits()` so it inherits his calibrated open/close ticks, and I made all of his imports lazy so the module still runs offline against a `MockGripper` when the hardware isn't present.

**`orchestrate_pick.py`** is the conductor. It reads my `pick_list.json` and drives the arm (my existing EPSON sender: `epsonJump` / `epsonGo` / `epsonStandby`) and the gripper together. The per-part logic is: open → JUMP above the part → descend to straddle it → close → *holding?* → if yes, lift, move, drop, open; if no, skip and log.

## Proving it before the bench

I did not want to walk into the lab with untested code, so I ran the whole thing in `DRY_RUN` mode against my real four-part `pick_list.json`. It sequenced all four parts correctly.

The tunables that remain are the ones that fundamentally cannot be known without the hardware in front of me: `YAW_OFFSET` (the wrist-to-jaw alignment), `GRIP_DZ_MM` (how far below the top surface to descend so the open fingers straddle the part), `APPROACH_MM` and the drop pose, and `GRIP_CURRENT` / `HOLD_MARGIN_TICKS` (grip force and the "did I grab it" threshold). Each of these is a physical quantity, and pretending to guess them in code would have been false precision.

## A small but important safety guard

One thing I added after the dry run was an **`OK`-reply handshake guard** on the orchestrator. My EPSON receiver already replies after each motion, but the orchestrator wasn't checking it, it was firing the next command optimistically. Now any non-`OK` reply from the arm aborts the run, releases the gripper, and returns to standby. This is the kind of defensive interlock that feels unnecessary in simulation and becomes essential the moment a real arm is moving near a real table.

## Where this leaves me

Where the project stands tonight: perception → pose estimation → robot coordinates → full pick orchestration is all built and rehearsed offline.

**Next:** it meets the VT6 — the first live pick, a single isolated part, low speed, hand on the e-stop.
