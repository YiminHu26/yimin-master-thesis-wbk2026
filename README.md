# CoG-Based Geometric Constraints and Custom Fingertip Design for Reliable Industrial Deployment of vMF-Contact Grasping on a UR10e Collaborative Robot

**Author:** Yimin Hu  
**Institution:** Karlsruhe Institute of Technology (KIT) — SFB-1574 Circular Factory  
**Year:** 2026  
**Citation style:** IEEE

---

## Research Question

> **Primary RQ:** To what extent do CoG-based geometric constraints improve grasping success rate and placement orientation consistency of an angle grinder compared to unconstrained vMF-Contact?

> **Sub-RQ:** Under what orientations, positions, and failure modes does the CoG-constrained method produce unsuccessful grasps?

The deployment context is a CT-cell inside a circular factory, where an AGV delivers angle grinders (~2 kg, asymmetric) at variable positions. Hardcoded pick poses are fundamentally inadequate under AGV positioning tolerances and uncontrolled return streams — flexible, model-free grasping is a structural requirement, not an optimisation.

---

## Research Gap

Flexible grasping research has evolved through three acts:

| Act | Method | Limitation |
|-----|--------|-----------|
| **1 — Traditional** | ICP-based pose estimation (Besl & McKay, 1992) | Requires CAD model; brittle under novel objects and variable placement |
| **2 — Learning-based** | GPD, PointNet, GraspNet-1Billion, Contact-GraspNet | Deterministic quality scoring; cannot distinguish a poor grasp from an uncertain one |
| **3 — vMF-Contact** | Shi et al. (ICRA 2025) — von Mises-Fisher uncertainty modeling | Solves epistemic uncertainty but encodes **no object-level geometric priors** |

vMF-Contact achieves 72.7% on lab benchmarks but leaves two gaps unaddressed:

- **Gap A (algorithmic):** no constraint on the spatial relationship between grasp point and object centroid, and no alignment between gripper axis and object principal axis — producing grip-force failures and upside-down placements on asymmetric heavy objects.
- **Gap B (systems):** no prior work reports a systematic closed-loop evaluation in a real industrial deployment with downstream task orientation requirements.

---

## Contributions

| # | Type | Description | Effect |
|---|------|-------------|--------|
| **A** | Software | CoG-based geometric constraints added to vMF-Contact: annular grip zone (`grasp_cog_dist_th`, `grasp_cog_min_dist_th`) + gripper axis / principal axis alignment (`cog_axis`, `cog_axis_projection_th`) | Baseline 1 (<5%) → Baseline 2 (~50%) |
| **B** | Hardware | Custom fingertip with curved geometry (matches cylindrical profile) and silicone overlay (increases friction) for Robotiq 2F-140 | Baseline 2 (~50%) → Final (80%) |
| **C** | Systems | ROS2 + Docker closed-loop pipeline: vMF-Contact inference on Jetson Orin Nano → MoveIt on Ubuntu 22.04 → UR10e pick-transport-place | Enables end-to-end evaluation |

> **Framing note:** Contribution B is method-agnostic. It is presented as a necessary deployment cost — the thesis contribution is the combined system (software + hardware + integration) evaluated end-to-end.

---

## Failure Modes

| Code | Type | Layer | Mechanism |
|------|------|-------|-----------|
| **F1** | Grip force failure | Execution | Gripper closes; object drops mid-trajectory |
| **F2** | No valid pose | Inference | CoG constraints filter all candidates |
| **F3** | Kinematic infeasibility | Motion planning | Valid pose found; UR10e cannot reach it |
| **F4** | Orientation error | Placement | Grasp succeeds; object placed upside-down |

Baseline 1 is dominated by F4 and F1 — confirming Gap A as the proximate cause. Contribution A (CoG constraints) nearly eliminates F4; Contribution B (custom fingertip) substantially reduces F1. F3 is structural (UR10e workspace limits) and unaffected by either contribution.

---

## Experimental Setup and Results

**Protocol:** 5 trials × 8 orientations × 9 positions = 360 trials per condition, randomised order.  
**Success criterion:** complete pick-transport-place cycle with correct object orientation on CT-cell holder.  
**Hardware:** Orbbec Femto Mega (RGBD) → Docker on Nvidia Jetson Orin Nano → MoveIt on Ubuntu 22.04 workstation → UR10e + Robotiq 2F-140.

| Condition | Algorithm | Fingertip | Success Rate |
|-----------|-----------|-----------|:------------:|
| Baseline 1 | vMF-Contact (no CoG) | Robotiq original | **<5%** |
| Baseline 2 | vMF-Contact + CoG | Robotiq original | **~50%** |
| Final | vMF-Contact + CoG | Custom + silicone | **80%** |

The factorial decomposition isolates each contribution independently: Baseline 1 → Baseline 2 quantifies the software contribution (+~45 pp); Baseline 2 → Final quantifies the hardware contribution (+~30 pp).

The Final condition (80%) exceeds the Shi et al. (2025) lab benchmark (72.7%) despite three systematic disadvantages: a heavier object (~2 kg vs. ≤1 kg), a weaker gripper (2F-140 at 125 N vs. 2F-85 at 185 N), and a compound pick-transport-place metric vs. isolated grasp success.

---

## Conclusion

Domain-specific adaptation — both algorithmic (CoG constraints) and physical (custom fingertip) — is necessary and sufficient to bring vMF-Contact to industrial reliability in the circular factory context. Neither contribution alone closes the gap; together they exceed the lab benchmark in a strictly harder scenario.

The Baseline 2 result (~50% vs. 72.7% reported) provides structured empirical evidence that lab-reported success rates systematically underestimate real deployment difficulty. The meta-contribution of this thesis is a methodology for quantifying that gap through a controlled three-condition factorial evaluation.

---

## Key Repositories

| Repository | Purpose |
|-----------|---------|
| [YiminHu26/vMF-Contact](https://github.com/YiminHu26/vMF-Contact/tree/dev_2) (`dev_2`) | Algorithm fork with CoG constraint implementation |
| [SFB-Circular-Factory-WG-C/Yimin_Dochub](https://github.com/SFB-Circular-Factory-WG-C/Yimin_Dochub) | Main ROS2 pipeline |
| [SFB-Circular-Factory-WG-C/docker_vmf_ur10e](https://github.com/SFB-Circular-Factory-WG-C/docker_vmf_ur10e) | Docker inference pipeline for Jetson Orin Nano |

**Base paper:** Shi et al., *vMF-Contact: Uncertainty-aware Evidential Learning for Probabilistic Contact-grasp in Noisy Clutter*, ICRA 2025. [arXiv:2411.03591](https://arxiv.org/abs/2411.03591)
