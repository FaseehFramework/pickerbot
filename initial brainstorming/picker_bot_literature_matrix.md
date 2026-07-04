# Picker-Bot — Literature Matrix / Starter Bibliography

Working artifact for the **subject area review**. Aligned to `picker_bot_thesis_v2.md` §3 clusters. This is your rigor layer → it becomes report §3. Keep it here (excluded from the published blog); post *narrative summaries* to the blog (see bottom).

**All papers below were confirmed to exist via web search (June 2026).** Verify exact author lists / years / DOIs in your reference manager before citing.

---

## How many papers?
- **JP's floor:** 2–3+ per area, with "existing solution → why insufficient → how mine differs."
- **Practical target:** ~**3–4 deeply-engaged** papers for each *pillar-critical* cluster (§3.2 sequencing, §3.3 verification, §3.6 entanglement) + **2–3** for the lighter clusters. → **~15–20 references total** in the report, ~8–12 read deeply, the rest cited lightly.
- For an 8-page research article, **relevance and density beat count.** Do not pad to 40.

## Reading workflow (per cluster)
1. Start from the **anchor paper(s)** seeded below.
2. **Snowball:** backward (their reference list) + forward (Google Scholar "Cited by"). Fastest way to saturate a cluster.
3. Read efficiently: **abstract → figures/tables → conclusion →** method only if relevant.
4. Capture in the fields below *as you read* (don't write prose first — prose is generated from these notes).
5. Use **Zotero** (free) from day one → auto-citations + BibTeX for the report.

## Databases & accelerators
- Google Scholar · IEEE Xplore · arXiv · Scopus (use **Middlesex library login** for paywalled PDFs).
- Curated paper lists to mine: `github.com/GeorgeDu/vision-based-robotic-grasping` · `github.com/rhett-chen/Robotic-grasping-papers`.
- Background primer: **MIT "Robotic Manipulation," Ch. 5 Bin Picking** — `manipulation.csail.mit.edu/clutter.html`.

---

## §3.2 Pick sequencing / clutter removal — **Pillar 1 (H1)** *(priority)*
> Your job: show these compute *expensive* ordering (reachability/occlusion/learned) — and position topmost-first as a cheap depth-only **proxy**, measured not asserted.

**Nam et al. (2020), "Fast and resilient manipulation planning for target retrieval in clutter," ICRA 2020** — arXiv:2003.11420
- Problem: __ · Method (TAMP, minimise pick-place actions): __ · Key result (≥28% fewer actions vs baseline): __
- Why insufficient for me: __ · How mine differs: __

**ClutterNav (2025), "Gradient-Guided Search for Efficient 3D Clutter Removal with Learned Costmaps"** — arXiv:2511.12479
- …why insufficient / how mine differs: learned cost-maps vs my zero-training sort → __

**"Towards Reliable Sequential Object Picking in Clutter" (Runner-up, RGMC 2025)** — arXiv:2606.12954
- Very close to your task (sequential picking in clutter) → good for the "how mine differs" contrast. __

**Zeng et al. (2018), "Robotic Pick-and-Place of Novel Objects in Clutter with Multi-Affordance Grasping…" (Amazon Robotics Challenge), IJRR** — foundational clutter picking. __

---

## §3.6 Entanglement / interlocking parts — **Pillar 2 hard boundary** *(priority — your strongest novelty axis)*
> Your job: establish that separating tangled parts is an *open, hardware/learning-heavy problem* → you **detect and bound**, not solve.

**Matsumura, Domae, Wan & Harada (2019), "Learning Based Robotic Bin-picking for Potentially Tangled Objects," IROS 2019** — the anchor. __

**Moosmann et al. (2021), "Separating Entangled Workpieces in Random Bin Picking using Deep Reinforcement Learning," Procedia CIRP** — __

**Moosmann et al. (2022), "Transfer Learning for ML-based Detection and Separation of Entanglements in Bin-Picking," IROS 2022** — (detection angle, closest to your Pillar 2). __

**"Learning to Dexterously Pick or Separate Tangled-Prone Objects for Industrial Bin Picking" (2023)** — arXiv:2302.08152 — __

**"Industrial Bin Picking of Potential Entangled Objects… Skeletonized Shape Restoration" (Springer, 2024)** — __

---

## §3.3 Closed-loop verification & failure recovery — **Pillar 2 (H2)** *(priority)*
> Your job: most verification asks "did I grasp anything?"; yours asks "did I extract a *single separable* object?" + you own the *vision* modality, buddy owns proprioception.

**"Failure Handling of Robotic Pick and Place Tasks With Multimodal Cues Under Partial Object Occlusion," Frontiers in Neurorobotics (2021)** — the anchor; fuses vision + gripper proprioception = exactly your two-thesis split. __

**"A Unified Framework for Real-Time Failure Handling… VLMs, Reactive Planner & Behavior Trees" (2025)** — arXiv:2503.15202 — modern execution-monitoring contrast. __

**"Simultaneous Pick and Place Detection by Combining SE(3) Diffusion Models with Differential Kinematics" (2025)** — arXiv:2504.19502 — __

---

## §3.1 RGBD 6-DoF pose / bin-picking (context / SOTA framing)
**Cordeiro et al. (2022), "Bin Picking Approaches Based on Deep Learning Techniques: A State-of-the-Art Survey" (INESC TEC)** — anchor survey to frame the field. __

**Du et al. (2021), "Vision-based robotic grasping… a review," Artificial Intelligence Review** — __

**"Review of Learning-Based Robotic Manipulation in Cluttered Environments," Sensors (2022)** — __

---

## §6.2 boundary — small / low-texture / specular & reachability
**"Learning Suction Graspability Considering Grasp Quality and Robot Reachability for Bin-Picking," Frontiers in Neurorobotics (2022)** — supports the "ordering needs reachability" point you approximate cheaply. __

**GlassLoc (2019), "Plenoptic Grasp Pose Detection in Transparent Clutter"** — arXiv:1909.04269 — specular/transparent boundary. __

**ClearGrasp (2020), "3D Shape Estimation of Transparent Objects"** — field boundary (from v2). __

---

## §3.4 Sim-to-real / robustness & §3.5 ROS2 (lighter — carry-over from v2, verify)
- Robust Visual Sim-to-Real Transfer (2023) arXiv:2307.15320 — *terminology caution* (policy transfer ≠ your RC8→physical commissioning).
- Macenski et al. (2022), "Robot Operating System 2," Science Robotics 7(66).
- Coleman et al. (2014), "Reducing the barrier to entry… a MoveIt! case study."

---

## Blog vs. report — where the lit review lives
- **Report §3 (rigor):** the full matrix above, prose-ified. Exhaustive.
- **Blog (progress + planning evidence):** a *narrative synthesis*, not a dump. e.g. **"This week: the clutter-sequencing literature — why nobody just picks the tallest first, and why a cheap proxy is still defensible."** 2–3 key papers, one sentence each, your takeaway, how it shaped the design. This is what JP gives formative feedback on and what the rubric rewards. Link the takeaways; keep the matrix out of the blog.
