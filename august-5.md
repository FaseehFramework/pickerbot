---
layout: default
title: "August 10: calibration solver"
parent: August 2026
nav_order: 5
---

# The calibration solver .the camera→robot bridge

*[Previously,]({% link august-4.md %}) I collected the touch-correspondence pairs into `calib_pairs.csv` each marker's position in both the camera frame and the robot frame.

## The solver

At its heart the solver is a classic **rigid-transform fit**: given the matched camera points `A` and robot points `B`, it finds the single rotation `R` and translation `t` that best line them up (via SVD of the cross-covariance, with a reflection guard so it stays a proper rotation). The diagnostic `scale` should come out ≈ 1.0 if the two sets really are the same rigid shape in different frames.

```python
def rigid_transform(A, B):
    """Best-fit R,t so that R@A_i + t ~= B_i.  A,B: (N,3), same units.
    Returns R (3,3), t (3,), and a diagnostic scale (should be ~1.0)."""
    A = np.asarray(A, float); B = np.asarray(B, float)
    cA, cB = A.mean(0), B.mean(0)
    AA, BB = A - cA, B - cB
    H = AA.T @ BB
    U, S, Vt = np.linalg.svd(H)
    d = np.sign(np.linalg.det(Vt.T @ U.T))          # reflection guard
    R = Vt.T @ np.diag([1, 1, d]) @ U.T
    t = cB - R @ cA
    scale = float((S * np.array([1, 1, d])).sum() / (AA ** 2).sum())
    return R, t, scale
```

The runnable script is `python pde4445-dev/calib_solve.py`.

## Verifying the math before trusting the data

Before pointing it at anything real, I checked the solver on **synthetic data** with a known transform plus 0.5 mm of noise. It recovered the rotation to ~1e-3, the translation to **0.19 mm**, and an RMS of **0.66 mm** essentially the noise floor. So the code itself is sound; time to point it at my real data.

## Running it on the real pairs

The script reads `calib_pairs.csv`, solves the camera→robot transform, and prints a report:

```text
Pairs used: 11   |   scale (want ~1.00): 0.994
  id    residual(mm)
    5      4.35
   10      2.97
    7      2.94
    0      2.85
    6      2.76
    9      2.74
    8      2.11
    3      2.01
    1      1.99
    4      1.93
    2      1.63
RMS residual: 2.67 mm   |   max: 4.35 mm
Target RMS < 5 mm
```

**RMS 2.67 mm, worst point 4.35 mm all under the 5 mm target**, and a scale of 0.994 confirms the units and setup are consistent. No gross outliers either: the residuals are tidily spread between 1.6 and 4.4 mm.

## The transform itself

The solver writes out `handeye.json`. Here's a snippet — the rotation, translation, and the headline diagnostics:

```json
{
  "note": "p_robot_mm = R @ (p_camera_metres * 1000) + t_mm",
  "R": [
    [-0.01376468094338832, 0.9995712138223241,  0.025844188056355136],
    [ 0.9997291865123626,  0.01424269128649426, -0.018403787117001198],
    [-0.01876398661950528, 0.025583866843836865, -0.9994965625571005]
  ],
  "t_mm": [-31.532555348129883, 758.2624546810478, 640.2694841265707],
  "rms_mm": 2.6710169971115914,
  "n_pairs": 11,
  "scale": 0.9937545472184207
}
```

That `note` line is the whole point of this: `p_robot_mm = R @ (p_camera_metres * 1000) + t_mm` feed in any camera detection and get back a coordinate the arm can move to.

## Why I trust it

The numbers are good, but I wanted to be sure the transform is *legit* and not just internally consistent:

- **It's a mathematically perfect rotation.** `det(R) = 1.0` and orthonormality error ~1e-15 — no reflection, no shear.
- **The geometry cross-checks.** The camera origin lands at `t = (-31.5, 758.3, 640.3)` mm — about **310 mm above** the touched markers (they sat at Z ≈ 330). That matches the ~334 mm working distance the camera measured back at the very start. An *independent* confirmation that the whole chain is correct, not just self-consistent.
- **2.67 mm RMS is comfortably fine** for grabbing cm-scale PCBs — the gripper's own tolerance absorbs it.

> This is a real milestone: the **camera→robot bridge exists.** The arm can now be told where to go from what the camera sees.

## Where this leaves me

Perception knows *what* and *where* (in the camera frame); the calibration now translates that into the robot's own coordinates, verified to under 3 mm RMS and cross-checked against the physical geometry. The two halves of the front-end finally speak the same language.