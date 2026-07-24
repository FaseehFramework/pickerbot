---
layout: default
title: "24/06/2026 : Initial meeting with Dr. Sameer"
parent: June 2026
nav_order: 2
---

# First Supervisor Meeting

*Second entry. My first meeting with my supervisor, Dr. Sameer, with four features and a plan. I walked out with something more useful: a set of hard questions I couldn't yet answer.*

## The meeting

This was our first proper sit down about the thesis. I laid out the [four features from my brainstorming]({% link june-week1.md %}).

I'd expected the conversation to be about *how* to build it. Instead, Dr. Sameer pushed on *why* and that turned out to be the right question.

## What he actually challenged

His feedback was blunt and, in hindsight, exactly what I needed. I've grouped it into the themes that came up.

### 1. Justify the hypothesis — don't just assert it

The core question:

> *"Where does this hypothesis come from? The answer should feel right."*

Right now, "pick the tallest item first" is an assertion. This is an **unconstrained problem**  there's no single correct pick order handed to me so I need to show *where* the tallest-first idea comes from, **how others have approached the same problem**, and build a justification that genuinely holds up rather than one I've reverse-engineered to fit what I already built.

### 2. Prove it experimentally, against a baseline

The way to move from assertion to evidence: **run the system both ways sorting by height, and without any ordering and compare.** If height-ordering is worth claiming, the difference should show up as a **large, statistically significant** gap between the two. That comparison becomes the spine of my evaluation.

The same standard applies to the **dynamic clearance calculation**. it needs its own justification, not just "it seemed sensible."

### 3. Ground it in the literature — first, before building

Do the **literature review in the initial weeks.** For each part of the problem, find **2–3+ papers** and, for each:

- what the existing solution is,
- *why it isn't sufficient* for my case,
- and **how mine differs and what "better" even means** here (faster? fewer collisions? cheaper? more robust?).

"Better" isn't allowed to be a vibe; I have to define the axis I'm claiming to win on.

### 4. Define what I'm going to test, and how

Testing and validation are on me — he wants to see *my* decisions defended. Concretely, the evaluation should sweep the conditions that actually stress the system:

- **Lighting** conditions,
- **toggling the z-height** (the whole point of going depth-based),
- **occlusion** at low / medium / high — including the honest failure case where **YOLO can't even read a part** because it's occluded.

### 5. Keep it *robotics*

The framing lens I'll carry through the whole project:

> Even if I find a better, faster, more efficient way to train a YOLO model — that alone is AI/ML, not robotics. But if I **correlate it to the hardware** — the robot arm — and the *arm* measurably performs better because of it, **then it counts as robotics.**

Every metric I report has to tie back to the arm's behaviour, not to vision quality in isolation.

## What I'm taking away

The honest takeaway: I'd been thinking like a builder ("here are four features") when I needed to be thinking like a researcher ("here is a problem, here is why my approach is justified, here is how I'll prove it"). The tallest-first idea might well survive but it has to *earn* its place against the literature and against a baseline, not just sound reasonable.

The other thing that stuck: **the answer should feel right.** That's a high bar, and my current framing doesn't quite clear it yet.

## Next step

Straight into the literature. I need to find where pick-ordering and clutter-clearing have been studied, work out where my problem genuinely differs, and come back with a **sharper problem statement, a defensible hypothesis, and a title** before I write a single line of the new pipeline.

→ *Continued in the [next entry]({% link june-3.md %}): finalising the project.*
