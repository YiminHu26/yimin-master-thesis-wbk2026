# Thesis Outline — vMF-Contact + UR10e Pick-and-Place
**`structure_architect_agent` | outline-only mode**
**Date confirmed:** 2026-06-17
**Status:** CONFIRMED — ready for `/ars-full`

---

## Paper Configuration

| Parameter | Value |
|-----------|-------|
| **Topic** | CoG-based geometric constraints for vMF-Contact deployed on UR10e + Robotiq 2F-140 + Orbbec Femto Mega |
| **Paper Type** | Master's Thesis — IMRaD (Engineering Variant) |
| **Discipline** | Robotics / Mechanical Engineering / Computer Vision |
| **Citation Format** | IEEE |
| **Word Count Target** | ~10,000 words |
| **Structure Pattern** | IMRaD (6-chapter engineering thesis variant) |

---

## Confirmed Experimental Results

| Condition | Algorithm | Fingertip | Success Rate |
|-----------|-----------|-----------|--------------|
| Baseline 1 | vMF-Contact (no CoG) | Robotiq original | **<5%** ✅ |
| Baseline 2 | vMF-Contact + CoG | Robotiq original | **~50%** ✅ |
| Final | vMF-Contact + CoG | Custom + silicone | **80%** ✅ |

- Benchmark comparison: Shi et al. (2024) reports 72.7%; Final (80%) exceeds this in a harder scenario
- Baseline 1 vs. Baseline 2 → isolates software contribution (+~45 pp)
- Baseline 2 vs. Final → isolates hardware contribution (+~30 pp)

---

## Overview

The thesis opens by establishing that flexible, model-free grasping is a structural necessity in circular manufacturing and that current methods fail two object-level geometric priors. Chapter 2 traces the field from rigid CAD-based methods through ML approaches to vMF-Contact, exposing both gaps. Chapter 3 presents the combined system as a principled response: CoG-based geometric constraints (software), a custom fingertip (hardware), and a ROS2+Docker closed-loop pipeline (integration). Chapter 4 reports the three-condition evaluation across 5×8×9 trials with per-orientation and per-position breakdowns. Chapter 5 interprets the results in context of the benchmark, critiques evaluation methodology, acknowledges limitations, and provides deployment guidance. Chapter 6 closes the narrative loop and states the meta-implication beyond the specific system.

---

## Word Count Summary

| Chapter | Section | Target Words |
|---------|---------|-------------|
| 1 | Introduction | ~900 |
| — | 1.1 Context and Motivation | ~250 |
| — | 1.2 Problem Statement | ~200 |
| — | 1.3 Research Gap | ~200 |
| — | 1.4 RQs and Contributions | ~150 |
| — | 1.5 Thesis Structure | ~100 |
| 2 | Related Work | ~1,750 |
| — | 2.1 Traditional Methods | ~350 |
| — | 2.2 Learning-Based Methods | ~700 |
| — | 2.3 vMF-Contact | ~350 |
| — | 2.4 Industrial Deployment Gap | ~350 |
| 3 | System Design & Methodology | ~2,250 |
| — | 3.1 System Overview | ~400 |
| — | 3.2 CoG Constraint Design | ~650 |
| — | 3.3 Custom Fingertip | ~400 |
| — | 3.4 ROS2+Docker Pipeline | ~400 |
| — | 3.5 Experimental Design | ~400 |
| 4 | Experimental Setup & Results | ~2,250 |
| — | 4.1 Experimental Setup | ~400 |
| — | 4.2 Overall Results | ~350 |
| — | 4.3 Per-Orientation Analysis | ~550 |
| — | 4.4 Per-Position Analysis | ~350 |
| — | 4.5 Failure Mode Distribution | ~600 |
| 5 | Discussion | ~2,250 |
| — | 5.1 Interpretation of Results | ~500 |
| — | 5.2 Dialogue with Existing Literature | ~500 |
| — | 5.3 Limitations | ~500 |
| — | 5.4 Future Work | ~450 |
| — | 5.5 Deployment Recommendations | ~300 |
| 6 | Conclusion | ~900 |
| — | 6.1 Summary of Contributions | ~300 |
| — | 6.2 Key Findings | ~250 |
| — | 6.3 Broader Implication | ~200 |
| — | 6.4 Closing Statement | ~150 |
| **Total** | | **~10,300 words** |

---

## Detailed Outline

---

### Chapter 1 — Introduction (~900 words)

**Purpose:** Establish the operational and structural necessity of flexible, model-free grasping in circular manufacturing; expose the two missing geometric priors; state research questions and contributions.

---

#### 1.1 Operational Context and Motivation (~250 words)

**Purpose:** Make the problem concrete before abstracting it.

**Content:**
- Opening hook: angle grinder on AGV arrives at CT-cell → why can't a robot pick it up reliably?
- Operational argument: AGV positioning tolerances render hardcoded pick poses non-repeatable; flexible grasping is a functional requirement, not a convenience
- Structural argument: circular manufacturing involves uncontrolled return streams — products arrive in unknown states, orientations, and configurations; a single hardcoded pose is fundamentally inadequate
- Establish CT-cell deployment as the concrete instantiation of this broader context

**Sources:** System operational context (self-derived); Shi et al. (2024) arXiv:2411.03591

**Key claim:** Flexible grasping is a structural necessity of the circular factory paradigm, not an optimisation over hardcoding.

---

#### 1.2 Problem Statement (~200 words)

**Purpose:** Narrow from the broad context to the two specific technical gaps.

**Content:**
- vMF-Contact: current state-of-the-art for uncertainty-aware grasp generation — 72.7% on lab benchmarks
- **Gap A (algorithmic):** encodes local contact geometry but not the spatial relationship between grasp point and object centroid
- **Gap B (algorithmic):** does not encode alignment between gripper axis and object's principal axis
- Consequence: on a heavy asymmetric object (angle grinder, ~2 kg), these missing priors produce F1 grip failures (insufficient force) and F4 orientation errors (upside-down placement)

**Sources:** Shi et al. (2024); Robotiq 2F-140 + angle grinder geometry (system documentation)

---

#### 1.3 Research Gap (~200 words)

**Purpose:** State Gap A and Gap B precisely.

**Content:**
- **Gap A:** learning-based methods encode local contact geometry but fail to encode (a) centroid-relative position bound and (b) gripper axis/principal axis alignment
- **Gap B:** no prior work systematically evaluated such methods in a closed-loop ROS2 deployment feeding downstream task orientation requirements back into grasp selection
- Gap A → Contribution A; Gap B → Contributions B + C

**Sources:** Shi et al. (2024); Sundermeyer et al. (2021); [IEEE Xplore deployment papers]

---

#### 1.4 Research Questions and Contributions (~150 words)

**Content:**
- **Primary RQ:** To what extent do CoG-based geometric constraints improve grasping success rate and placement orientation consistency of an angle grinder compared to unconstrained vMF-Contact?
- **Sub-RQ 1:** Under what orientations, positions, and failure modes (F1–F4) does the CoG-constrained method produce unsuccessful grasps?
- **Contribution A:** CoG-based geometric constraints (software) — <5% → ~50%
- **Contribution B:** Custom fingertip design (hardware) — ~50% → 80%
- **Contribution C:** ROS2+Docker closed-loop pipeline (systems) — enables end-to-end evaluation

---

#### 1.5 Thesis Structure (~100 words)

**Content:** Chapter signpost (Ch. 2–6); note that Ch. 4 directly answers primary RQ and Sub-RQ 1; Ch. 5 interprets relative to Shi et al. (2024) benchmark.

**Transition to Chapter 2:** Chapter 1 established *why* the problem exists; Chapter 2 traces *how* the field arrived at vMF-Contact and identifies precisely where both gaps originate.

---

### Chapter 2 — Related Work (~1,750 words)

**Purpose:** Trace three-act evolution (traditional → ML → vMF-Contact); expose Gap A and Gap B at their origin point; critique evaluation methodology.

**Three-act spine:**

| Act | Transition | Mechanism |
|-----|-----------|-----------|
| Traditional → ML | CAD + hardcoding cannot handle novel objects or variable positions | Flexibility gap |
| ML → vMF-Contact | Deterministic methods cannot distinguish confident from uncertain predictions | Epistemic uncertainty gap |
| vMF-Contact → **This thesis** | vMF-Contact handles uncertainty but encodes no object-level geometric priors | Gap A + Gap B |

---

#### 2.1 Traditional Model-Based Grasping (~350 words)

**Purpose:** Establish Act 1 — why rigid/CAD approaches fail as the baseline.

**Content:**
- ICP-based pose estimation (Besl & McKay, 1992): requires CAD model, fails on novel/occluded objects, brittle under variable conditions
- Hardcoded trajectory approaches: deterministic under controlled conditions; fail when geometry or placement varies (AGV tolerance problem from §1.1)
- Establish the "flexibility gap" as the motivation to Act 2

**Sub-sections:**
- 2.1.1 CAD-Based Pose Estimation (ICP and variants)
- 2.1.2 Failure Mode under Circular Manufacturing Conditions

**Sources:** Besl & McKay (1992) [⚠️ seminal, >10 years — flag in citation]

**Key claim:** Traditional methods are reliable only when object geometry and placement are controlled — both conditions fail in circular manufacturing.

---

#### 2.2 Learning-Based Grasp Generation (~700 words)

**Purpose:** Trace Act 2 — ML solved the flexibility gap but introduced the uncertainty modeling problem.

**2.2.1 Point-Cloud-Based Methods (~350 words)**
- GPD (Ten Pas et al. 2017): first large-scale ML grasp generation; generalizes to novel objects without CAD
- PointNet/PointNet++ (Qi et al. 2017): backbone for direct learning from unordered 3D point clouds
- GraspNet-1Billion (Fang et al. 2020): scale solves the flexibility gap — near-universal object coverage
- Limitation: deterministic quality scoring; cannot distinguish poor grasp from uncertain prediction

**2.2.2 Transition to Uncertainty-Aware Methods (~100 words)**
- "Epistemic uncertainty gap": novel orientations → low-quality candidates indistinguishable from ungraspable poses
- Observable in deployment: certain orientations produce near-zero candidates → F2 failures in this thesis

**2.2.3 Contact-GraspNet as Act 2 Ceiling (~250 words)**
- Contact-GraspNet (Sundermeyer et al. 2021): contact-point + approach-direction prediction; strong benchmark performance
- Still deterministic: no uncertainty representation; degradation on OOD objects has no signal
- Sets the ceiling that vMF-Contact targets via uncertainty modeling

**Sources:** Ten Pas et al. (2017); Qi et al. (2017); Fang et al. (2020); Sundermeyer et al. (2021)

---

#### 2.3 vMF-Contact: The Base Method (~350 words)

**Purpose:** Act 3 — establish vMF-Contact's mechanism and precisely where Gap A and Gap B originate.

**Content:**
- Shi et al. (2024): models grasp candidate quality as von Mises-Fisher distribution; explicitly represents aleatoric uncertainty (sensor noise — irreducible) and epistemic uncertainty (OOD poses — reducible with more data)
- Reported success: 72.7% on lab benchmark
- **The gap:** vMF-Contact encodes *local contact geometry* but no *object-level geometric priors* — no centroid-relative position bound (Gap A), no principal-axis alignment (Gap A, axis component)
- Practical consequence for angle grinder: candidates generated at extremities → F1 failures; no axis constraint → F4 upside-down placements

**Sources:** Shi et al. (2024) arXiv:2411.03591

**Key claim:** vMF-Contact is the correct base method (it solves uncertainty modeling), but its missing geometric priors are the precise target of Contribution A.

---

#### 2.4 Industrial Deployment Gap and Evaluation Methodology Critique (~350 words)

**Purpose:** Establish Gap B — the absence of systematic closed-loop industrial evaluation.

**Content:**
- Existing evaluation: isolated grasp success under controlled lab conditions (no downstream task constraints, moderate object weights, no orientation requirements)
- Deployment success is system-level: hardware-object compatibility, sensor noise, downstream task orientation requirements all compound
- No prior work reports a structured orientation × position trial matrix in a real industrial deployment
- The 72.7% benchmark is not directly comparable to deployment: different gripper (2F-85, 185N), lighter objects (≤1 kg), isolated grasp metric only
- Motivates Contribution C and the 5×8×9 evaluation protocol

**Sources:** [IEEE Xplore: "ROS2 MoveIt UR10 pick and place" — ⚠️ search pending]; Shi et al. (2024)

**Key claim:** Lab evaluation systematically underestimates deployment complexity — this thesis provides empirical evidence.

**Transition to Chapter 3:** Chapter 2 identified the gaps; Chapter 3 presents the system designed to fill them.

---

### Chapter 3 — System Design and Methodology (~2,250 words)

**Purpose:** Present the full system as a principled response to Gap A (CoG constraints) and Gap B (closed-loop evaluation); justify each design decision by its target failure mode.

**Opening statement (§3.0 intro paragraph):** The custom fingertip improvement is method-agnostic. The thesis frames this as a necessary deployment cost; the contribution is the combined system (software + hardware + integration) evaluated end-to-end. State explicitly in §3.0 to pre-empt the reviewer objection that Contribution B is not an ML contribution.

---

#### 3.1 System Overview (~400 words)

**Purpose:** Complete architectural picture before detail sections.

**Content:**
- **Figure 3.1 — Full pipeline diagram:**
  ```
  Orbbec Femto Mega (RGBD)
    → point cloud (base_link frame, shifted to agv_table_center_link)
    → Docker container on Nvidia Jetson Orin Nano
      → vMF-Contact inference with CoG parameters
      → grasp pose published to ROS2 domain
    → MoveIt on Ubuntu 22.04 workstation
    → UR10e + Robotiq 2F-140 (Baseline 1+2) / Custom fingertip (Final)
    → Phase 1: Pick (CoG-constrained grasp)
    → Phase 2: Cobot turn manoeuvre (arm reorientation)
    → Phase 3: Place on CT-cell holder (hardcoded trajectory)
  ```
- Three-sentence summary of each major subsystem
- 2F-140 vs. 2F-85 selection rationale: angle grinder geometry requires wider jaw opening; lower grip force (125N vs. 185N) → motivates Contribution B

**Sub-sections:**
- 3.1.1 System Architecture Diagram
- 3.1.2 Hardware Bill of Materials and Selection Rationale

**Sources:** System documentation; Robotiq 2F-140 datasheet; UR10e datasheet; Orbbec Femto Mega datasheet

---

#### 3.2 CoG-Based Geometric Constraint Design (~650 words)

**Purpose:** Present Contribution A — geometric motivation and implementation.

**3.2.1 Motivation and Geometric Rationale (~250 words)**
- Angle grinder (~2 kg, asymmetric elongated): geometric centroid ≠ geometric center
- Problem 1 (grip force): vMF-Contact selects grasps at extremities where grip force insufficient → F1
- Problem 2 (orientation): no axis encoding → F4 upside-down placements
- **Annular grip zone:** `grasp_cog_dist_th` (0.05 m) outer bound, `grasp_cog_min_dist_th` (0.03 m) inner bound — too far → F1; too close → F2
- **Axis alignment:** `cog_axis` + `cog_axis_projection_th` (0.5) constrains gripper y-axis to object's longest axis (OBB rotation matrix)
- **Precision note:** "CoG" is a geometric approximation: `cog_mean = 0.2 × obb_center + 0.8 × cog_pcd` — use "geometric centroid-based constraints" in thesis, not "centre of mass"

**3.2.2 Implementation (~400 words)**
- Inference call with CoG parameters (Figure 3.2 — code snippet):
  ```python
  prediction = self.model.inference(
      pcd.to("cuda"),
      graspness_th=0.3,
      grasp_height_th=0.003,
      grasp_cog_dist_th=0.05,
      grasp_cog_min_dist_th=0.03,
      cog=cog_mean,
      cog_axis=obb_rot,
      cog_axis_projection_th=0.5,
  )
  ```
- Parameter justification: each parameter tied to a geometric measurement of the angle grinder; framing = geometry-driven design, not empirical tuning
- Object-specificity acknowledged as limitation (→ §5.3)
- CoG limitation: requires single-object scene — instance segmentation needed for multi-object capability (→ §5.4)

**Sources:** Shi et al. (2024); GitHub: YiminHu26/vMF-Contact (dev_2 branch)

---

#### 3.3 Hardware Design: Custom Fingertip (~400 words)

**Purpose:** Present Contribution B — custom fingertip targeting F1.

**Content:**
- F1 sub-mechanism decomposition: Robotiq 2F-140 flat fingertip → insufficient contact area and friction on cylindrical angle grinder body
- **Two design interventions:**
  - Curved geometry: matches cylindrical profile → increases contact area, distributes normal force
  - Silicone overlay: increases friction coefficient → resists axial slippage during transport
- Figure 3.3: CAD drawing (available)
- Framing: method-agnostic improvement; necessary deployment cost, not ML-exclusive contribution (restatement of §3.0)

**Sources:** CAD documentation; Robotiq 2F-140 gripper datasheet

---

#### 3.4 Software Integration: ROS2 and Docker Pipeline (~400 words)

**Purpose:** Present Contribution C — the integration enabling closed-loop evaluation.

**Content:**
- Architecture: Docker on Jetson Orin Nano → ROS2 topic → MoveIt on Ubuntu 22.04 → UR10e driver
- **Key engineering fix:** Docker networking fix enabling bidirectional ROS2 domain bridge between Jetson container and host workstation — this was the integration blocker
- Repositories:
  - Main pipeline: github.com/SFB-Circular-Factory-WG-C/Yimin_Dochub
  - Algorithm fork: github.com/YiminHu26/vMF-Contact (dev_2)
  - Docker pipeline: github.com/SFB-Circular-Factory-WG-C/docker_vmf_ur10e
- Closed-loop operation: complete pick-turn-place cycle without human intervention

**Sources:** GitHub repositories above; ROS2/MoveIt/Docker documentation

---

#### 3.5 Experimental Design (~400 words)

**Purpose:** Define the three-condition factorial design, trial protocol, and failure taxonomy.

**3.5.1 Three-Condition Factorial Design (~200 words)**
- Proper factorial decomposition: two independent contributions → two independent comparisons
- Baseline 1 vs. Baseline 2 → isolates software contribution
- Baseline 2 vs. Final → isolates hardware contribution

**3.5.2 Trial Protocol and Failure Taxonomy (~200 words)**
- Trial structure: 5 trials × 8 orientations × 9 positions = 360 trials per condition; randomised order
- Success criterion: complete pick-transport-place cycle with correct object orientation on CT-cell holder
- **F1–F4 failure taxonomy:**

| # | Type | Layer | Mechanism |
|---|------|-------|-----------|
| F1 | Grip force failure | Execution | Gripper closes; object drops mid-trajectory |
| F2 | No valid pose | Inference | CoG constraints filter all candidates |
| F3 | Kinematic infeasibility | Motion planning | Valid pose found; UR10e cannot reach it |
| F4 | Orientation error | Orientation | Grasp succeeds; object placed upside-down |

**Sources:** Original experimental design (self-derived)

**Transition to Chapter 4:** Chapter 3 presented the system design; Chapter 4 reports what happened when tested under the 5×8×9 protocol.

---

### Chapter 4 — Experimental Setup and Results (~2,250 words)

**Purpose:** Report three-condition evaluation results with per-orientation and per-position breakdowns; demonstrate both contributions are independently necessary and jointly sufficient.

---

#### 4.1 Experimental Setup (~400 words)

**Content:**
- Hardware table (three conditions):

| Condition | Algorithm | Fingertip | Result |
|-----------|-----------|-----------|--------|
| Baseline 1 | vMF-Contact (no CoG) | Robotiq original | <5% |
| Baseline 2 | vMF-Contact + CoG | Robotiq original | ~50% |
| Final | vMF-Contact + CoG | Custom + silicone | 80% |

- Camera: Orbbec Femto Mega, fixed, calibrated to agv_table_center_link frame
- Object: angle grinder (~2 kg), randomised placement within 9-position grid
- Environment: SFB Circular Factory CT-cell, controlled lighting, single-object scene
- Data logging: per-trial outcome (success/F1/F2/F3/F4) logged automatically via pipeline

**Sources:** System documentation; trial logs

---

#### 4.2 Overall Results (~350 words)

**Content:**
- Table 4.1: three-condition success rate table
- **Headline claim:** Baseline 1 (<5%) → Baseline 2 (~50%): +~45 pp software contribution; Baseline 2 (~50%) → Final (80%): +~30 pp hardware contribution
- Both transitions large and independent → both adaptations load-bearing
- Final (80%) exceeds Shi et al. (2024) benchmark (72.7%) in a stricter scenario (heavier object, weaker gripper, compound metric)
- Note: margin over benchmark is +7.3 pp; do not overclaim — present as "exceeds the benchmark despite three systematic disadvantages"

**Sources:** Trial data (original); Shi et al. (2024)

---

#### 4.3 Per-Orientation Analysis (~550 words)

**Content:**
- **Figure 4.1:** grouped bar chart, 8 orientations × 3 conditions (colorblind-safe palette, IEEE formatting)
- Baseline 2: success rate ranges 5–88% across orientations (high variance)
- Final: consistent lift across all orientations; variance structure preserved
- **F3 cluster:** one or two orientations produce near-zero success in both Baseline 2 and Final — kinematic infeasibility (UR10e workspace limit), NOT algorithm failure
  - Report full-matrix success rate AND kinematically-feasible-orientation success rate separately
  - Do NOT exclude F3 orientations from trial count
- **Low-candidate / F1 cluster:** at orientations where vMF-Contact produces few CoG-filtered candidates, system selects best available → F1 (not F2), because candidate still passes hard CoG distance threshold but grip quality is marginal
  - This is the epistemic uncertainty cascade: low model confidence → few candidates → marginal grasp selected → F1

**Sources:** Trial data (original)

---

#### 4.4 Per-Position Analysis (~350 words)

**Content:**
- **Figure 4.2:** heatmap or bar chart, 9 positions × success rate
- F2 failures are rare and position-sensitive: extreme corner positions occasionally fall outside annular grip zone
- Small repositioning (~10 cm) resolves F2 in virtually all cases → position retry heuristic would have near-zero operational cost (→ §5.4)
- F2 position sensitivity consistent across Baseline 2 and Final; absent in Baseline 1 (no CoG filtering)

**Sources:** Trial data (original)

---

#### 4.5 Failure Mode Distribution (~600 words)

**Content:**
- **Figure 4.3:** stacked bar chart, F1–F4 per orientation (3 conditions)
- **Baseline 1:** primarily F4 (upside-down placement) and F1 (grip force failure at extremity grasps) — confirms Gap A and Gap B are the proximate causes
- **Baseline 2:** F1 dominant at low-candidate orientations (epistemic uncertainty cascade); F2 rare; F3 cluster at workspace-limited orientations; F4 nearly eliminated by CoG axis alignment
- **Final:** F1 substantially reduced by custom fingertip; F3 unchanged (workspace constraint); F2 unchanged (CoG parameter issue); F4 eliminated
- **Table 4.2:** failure mode count by condition
- **Key distinction:** F3 cluster vs. low-candidate F1 cluster: distinguished by whether a valid pose was found (F3: yes, robot cannot reach; low-candidate F1: yes, robot reaches but grip fails)

**Sources:** Trial data (original)

**Transition to Chapter 5:** Chapter 4 reported what happened; Chapter 5 explains why, contextualises against the benchmark, and draws the broader methodological lesson.

---

### Chapter 5 — Discussion (~2,250 words)

**Purpose:** Interpret results relative to benchmark; critique evaluation methodology; acknowledge limitations; provide deployment guidance; state future work.

---

#### 5.1 Interpretation of Results (~500 words)

**Content:**
- Baseline 2 (~50%) vs. Shi et al. (2024) reported (72.7%) — **three-reason gap:**
  1. Gripper force: 2F-140 (125N) vs. paper's 2F-85 (185N)
  2. Object weight: ~2 kg angle grinder vs. ≤1 kg lab objects
  3. Metric scope: pick-transport-place vs. isolated grasp only
- Gap is methodologically explainable — not a method failure; evidence that lab conditions systematically underrepresent deployment difficulty
- Final (80%) exceeds paper (72.7%) despite all three disadvantages → domain-specific adaptation is necessary AND sufficient
- Framing: "exceeds the benchmark despite three systematic disadvantages" — do not overclaim the +7.3 pp margin

**Sources:** Shi et al. (2024); trial data

---

#### 5.2 Dialogue with Existing Literature (~500 words)

**Content:**
- Gap B argument validated: 72.7% benchmark does not predict ~50% Baseline 2 deployment result — evaluation methodology critique is empirically supported
- Epistemic uncertainty clusters (F2/F1 at certain orientations) provide empirical evidence of vMF-Contact's training distribution limits
- Uniform lift (Baseline 2 → Final) confirms hardware contribution is consistent, not orientation-specific — custom fingertip adds a constant floor lift
- Orientation sensitivity persists into Final condition — not addressable by fingertip design; requires model-side intervention (→ §5.4)

**Sources:** Shi et al. (2024); Sundermeyer et al. (2021); Fang et al. (2020); Ten Pas et al. (2017)

---

#### 5.3 Limitations (~500 words)

**Content:**
- **Single object class:** CoG parameters tuned to angle grinder geometry; generalizability requires re-parameterisation
- **Single deployment environment:** CT-cell, controlled lighting, SFB lab — environmental robustness untested
- **Multi-object scene capability removed:** CoG computed from full point cloud — meaningless in cluttered multi-object scenes; direct trade-off of the approach
- **2F-140 grip force disadvantage:** mitigated by custom fingertip but not identical to paper's 2F-85 setup

**Sources:** Self-derived

---

#### 5.4 Future Work (~450 words)

**Content:**
- **Instance segmentation (primary):** Mask3D or PointNet++ segmentation head to restore multi-object capability while preserving CoG constraint logic — enabling step for generalisation at scale
- **Confidence-weighted candidate selection:** weight vMF concentration parameter against CoG distance to avoid marginal grasps at high-uncertainty orientations (addresses low-candidate → F1 cascade)
- **Collision-aware motion planning:** eliminate F3 workspace cluster without sacrificing orientation coverage
- **F2 position retry heuristic:** practical deployment fix at negligible operational cost
- **Generalisation to other elongated objects:** drill, torque wrench — geometric similarity analysis to establish applicability scope

**Sources:** Self-derived / forward-looking

---

#### 5.5 Deployment Recommendations (~300 words)

**Content:**
- **Suitable:** single object class, controlled single-object placement, moderate takt time, 80% success rate acceptable, circular factory return streams
- **Not suitable:** high-takt automated assembly lines, safety-critical (≥99% reliability), multi-object cluttered scenes, novel geometries without re-tuning
- The circular factory context fits the system's operating envelope. High-speed assembly lines do not.

**Sources:** Self-derived

**Transition to Chapter 6:** Chapter 5 interpreted and qualified; Chapter 6 closes the loop and states what the results mean beyond this specific system.

---

### Chapter 6 — Conclusion (~900 words)

**Purpose:** Close the narrative loop from §1.1; state three contributions precisely; articulate the meta-implication.

---

#### 6.1 Summary of Contributions (~300 words)

**Content:**
- **Contribution A (software):** CoG-based constraints (`grasp_cog_dist_th`, `grasp_cog_min_dist_th`, `cog_axis`, `cog_axis_projection_th`) added to vMF-Contact; Baseline 1 (<5%) → Baseline 2 (~50%)
- **Contribution B (hardware):** Custom fingertip with curved geometry + silicone overlay targeting F1; Baseline 2 (~50%) → Final (80%)
- **Contribution C (systems):** ROS2+Docker closed-loop pipeline enabling real-time inference, motion planning, and pick-transport-place on UR10e; enables the end-to-end evaluation

---

#### 6.2 Key Findings (~250 words)

**Content:**
- Final (80%) exceeds Shi et al. (2024) benchmark (72.7%) in a harder scenario
- Baseline 2 gap vs. paper: three explainable reasons — not a method failure
- Orientation sensitivity is structural — persists across all conditions; addressable only by future model-side work
- Both adaptations independently necessary and jointly sufficient — confirmed by factorial decomposition

**Sources:** Trial data; Shi et al. (2024)

---

#### 6.3 Broader Implication (~200 words)

**Content:**
- Lab-to-real gap is systematic, not accidental — this thesis provides a structured counter-example at the level of evaluation methodology
- Domain-specific adaptation (hardware + software) is the path to industrial reliability — not waiting for a better algorithm
- Meta-contribution: empirical evidence that lab-reported success rates systematically overestimate real deployment performance

---

#### 6.4 Closing Statement (~150 words)

**Content:** Return to Chapter 1 hook — the angle grinder is on the AGV, the CT-cell is waiting. With CoG-based constraints and a purpose-designed fingertip, the robot picks it up — reliably, in the actual CT-cell. The loop is closed.

---

## Evidence Map

| Section | Assigned Sources | Evidence Type | Status |
|---------|-----------------|---------------|--------|
| §1.1–1.2 | System operational context | Problem framing | Self-derived |
| §1.3 | Shi et al. (2024), Sundermeyer et al. (2021) | Gap statement | ⚠️ Verify DOIs |
| §2.1 | Besl & McKay (1992) | Traditional method anchor | ⚠️ Seminal, >10 years — flag |
| §2.2.1 | Ten Pas et al. (2017), Qi et al. (2017), Fang et al. (2020) | ML grasp generation | ⚠️ Verify DOIs |
| §2.2.3 | Sundermeyer et al. (2021) | Act 2 ceiling | ⚠️ Verify DOI |
| §2.3 | Shi et al. (2024) arXiv:2411.03591 | Base method | ⚠️ Verify arXiv ID + check if journal-published |
| §2.4 | [IEEE Xplore deployment papers] | Deployment gap | ⚠️ Search pending: "ROS2 MoveIt UR10 pick and place" |
| §3.1 | Robotiq 2F-140 / UR10e / Orbbec datasheets | Hardware specs | ⚠️ Confirm IEEE datasheet citation format |
| §3.2 | Shi et al. (2024), GitHub fork | Algorithm extension | ⚠️ Cite code repo per IEEE software citation |
| §3.3 | Robotiq 2F-140 datasheet, CAD documentation | Hardware design | Self-produced |
| §3.4 | ROS2 / MoveIt / Docker documentation | Integration | ⚠️ Confirm citation format |
| §3.5–4.5 | Trial logs (original data) | Experimental results | Self-produced |
| §5.1–5.2 | Shi et al. (2024) + Act 1–3 papers | Comparative interpretation | ⚠️ Verify all DOIs |
| §5.3–6.4 | Self-derived | Limitations / future / conclusion | No external sources |

---

## Open Items Before Drafting

| # | Item | Status |
|---|------|--------|
| 1 | Baseline 1 trials | ✅ RESOLVED — <5% confirmed |
| 2 | Final condition trials | ✅ RESOLVED — 80% confirmed |
| 3 | IEEE Xplore deployment paper search (§2.4) | 🟡 Pending |
| 4 | DOI verification for 7 literature sources | 🟡 Pending (IRON RULE before drafting) |

---

## Key Repositories

- Main pipeline: https://github.com/SFB-Circular-Factory-WG-C/Yimin_Dochub
- Algorithm fork: https://github.com/YiminHu26/vMF-Contact/blob/dev_2/vmf_contact_main/test_11_save_logs.py
- Docker pipeline: https://github.com/SFB-Circular-Factory-WG-C/docker_vmf_ur10e
- Base algorithm paper: https://arxiv.org/abs/2411.03591
