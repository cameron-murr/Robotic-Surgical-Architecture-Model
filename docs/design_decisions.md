# Robotic Bronchoscopy Navigation Subsystem (RBNS)
## Functional Architecture Model — Design Decisions Document

**Author:** Portfolio Project — Systems Engineering Competency Demonstration  
**Notation:** SysML, implemented in Modelio  
**Version:** 1.4  
**Date:** 2026

---

## 1. Scope and Boundary Rationale

The RBNS is defined as the software subsystem responsible for intraoperative localization, path planning, and guidance command generation during a robotic bronchoscopy procedure. It accepts a preoperative airway model and target nodule coordinates as inputs, maintains a real-time estimate of bronchoscope tip position, and outputs motion guidance commands to a downstream robot motion controller.

**What is explicitly excluded:**

- **Robot kinematics and actuation stack.** The RBNS does not model joint torque, motor control, or mechanical drive. This boundary reflects a real architectural seam: guidance intent and motion execution are separated by a well-defined command interface (IF-EX-04), allowing either subsystem to be developed and validated independently.
- **Cone Beam Computed Tomography (CBCT) imaging hardware.** The CBCT imaging system is treated as an external system. The RBNS consumes volumetric data and issues acquisition triggers via a defined open interface (IF-EX-02). This boundary supports imaging hardware substitution without changes to navigation software.
- **Surgeon console rendering pipeline.** The RBNS outputs navigation overlay data to the console (IF-EX-05b) but does not manage display rendering. Display logic is a separate concern with different real-time constraints.

The resulting boundary produces a subsystem that is functionally cohesive — all internal blocks share the goal of knowing where the scope is and where it should go — and that has explicit, testable interfaces at every external boundary.

User needs and system requirements are defined in `requirements.md` (Tiers 1-2). System requirements (SYS-REQ) are scoped to the full robotic bronchoscopy system; subsystem requirements (SUB-REQ) are scoped to the RBNS and provide the upstream trace for the component requirements referenced throughout this document.

---

## 2. Sensor Fusion Architecture (Block 1.0 Design Decision)

The Localization and Tracking subsystem (1.0) uses a four-block architecture with an Extended Kalman Filter (EKF) as the integration point, rather than treating any single sensor source as ground truth.

**Three input modalities and their characteristics:**

| Source | Rate | Accuracy | Failure mode |
|---|---|---|---|
| Proprioceptive (Forward Kinematics, FK) — Block 1.1 | High (~100 Hz) | Low (accumulates error under flexion) | Scope deformation, cable stretch |
| Vision-based — Block 1.2 | Medium (~30 Hz) | Medium | Mucus occlusion, low-contrast airways |
| CBCT registration — Block 1.3 | Low (triggered, ~periodic) | High | Radiation dose limits frequency; latency |

**Why not use CBCT registration as ground truth?**

CBCT acquisition is periodic and triggered — it cannot provide continuous localization. Between acquisitions, the system must maintain a position estimate using FK and vision. If CBCT were treated as ground truth and discarded between acquisitions, the system would have no principled way to propagate uncertainty during the intervals. The EKF architecture treats CBCT registration as a high-accuracy periodic correction to a continuous low-accuracy estimate, which is the appropriate model for this sensor topology.

**FK initializes registration (dashed interface, Block 1.1 → 1.3).** The current FK estimate is passed to the Iterative Closest Point (ICP) registration algorithm as an initial pose estimate, improving convergence speed and reducing the risk of landing in a local minimum. This is a soft dependency — the registration can run without it — but FK initialization is expected to be used in nominal operation.

---

## 3. Interface IF-01 Design: Uncertainty as an Explicit Output

The primary output of Block 1.0 to Block 2.0 (Path Planning) carries not just a tip position and orientation estimate, but a 3D uncertainty ellipsoid.

**Why uncertainty is an explicit, typed output rather than implicit:**

A guidance system that ignores localization confidence is a safety problem. If the scope position is poorly known — due to rapid FK drift, vision failure, or a long interval since the last CBCT registration — a path updater that treats the position estimate as reliable may issue a replanning command based on a fictitious deviation, causing unnecessary or harmful motion.

By making uncertainty explicit on IF-01, the downstream consumer (Block 2.2, Real-Time Path Updater) can implement a principled gating rule: if the uncertainty ellipsoid exceeds a configurable threshold volume, replanning is inhibited and the system holds its current path. This rule is documented as a design decision in Block 2.2 rather than buried in an implementation detail.

This also produces a testable interface requirement: IF-01 must carry an uncertainty representation with defined units and coordinate frame, and Block 2.2 must have a documented threshold parameter with rationale.

---

## 4. Replanning Inhibition Under Uncertainty (Block 2.2 Design Decision)

Block 2.2 (Real-Time Path Updater) implements a replanning threshold that is parameterized, not hardcoded. The threshold defines the maximum tip position uncertainty (expressed as ellipsoid volume or semi-axis length) below which replanning is permitted.

**The design decision:**

When position uncertainty exceeds the threshold, Block 2.2 does not request a re-plan from Block 2.1 and does not issue a replanning event to the Supervisor (IF-02). Block 2.3 (Guidance Command Generator) outputs a HOLD command — zero advance, zero retract — until uncertainty drops back below threshold (typically following the next CBCT registration event).

**Why parameterized rather than hardcoded:**

The appropriate uncertainty threshold depends on airway diameter at the current position (a 3mm uncertainty is tolerable in a mainstem bronchus, unacceptable in a third-generation branch), procedure phase, and the specific clinical risk tolerance of the deployment. Making it a configurable parameter supports per-procedure-type tuning and enables clinical validation studies to characterize threshold effects on diagnostic yield and safety.

**Status of the threshold value itself:** The numeric default for this threshold is formally a derived component requirement (COMP-REQ-003 in `requirements.md`, Tier 4), produced by parametric analysis relating airway diameter at branch points to tolerable position uncertainty. It is currently marked TBD pending that analysis — consistent with the principle that derived values are outputs of analysis, not asserted defaults.

---

## 5. Procedure Supervisor as the Primary External Interface Point, with a Defined Exception for High-Rate Sensor Feeds

Most external interfaces connect to or through Block 4.0 (Procedure Supervisor). Command, control, status, and logging interfaces — IF-EX-01, IF-EX-02b, IF-EX-04, IF-EX-05a, IF-EX-05b, IF-EX-06a, IF-EX-06b, and IF-EX-07 — all route through the Supervisor. No internal block other than the Supervisor issues or receives these interfaces directly.

**Exception: continuous sensor and imaging feeds connect directly to their consuming subsystem, bypassing the Supervisor.** Specifically:

- IF-EX-03 (encoder telemetry, 100 Hz) and IF-EX-08 (endoscopic video, 30 Hz) connect directly to Block 1.0 (Localization & Tracking)
- IF-EX-02a (CBCT volume) connects directly to Block 3.0 (Image Processing & Registration), and also to Block 1.0 where the registered volume is consumed for fusion

**Rationale for the exception:**

Routing high-rate, continuous sensor data through a central relay adds latency and turns the Supervisor into a throughput bottleneck for data it has no need to inspect or act on. The Supervisor's role is procedure-level command and fault management — it does not need to see every encoder sample or video frame to do that job. Forcing 100 Hz telemetry through an intermediary that exists to manage a six-state procedure machine would add transport latency without any corresponding architectural or safety benefit, and would make the Supervisor a single point of failure for sensor data that has nothing to do with procedure state.

Command, control, and status interfaces remain centralized through the Supervisor because those interfaces are low-rate, event-driven, and directly relevant to the Supervisor's fault-handling and state-management responsibilities — centralizing them keeps fault response logic in one place (see the three practical consequences below). High-rate sensor data does not share those characteristics, so the same centralization argument does not apply.

**Practical consequences of centralizing the command/control/status interfaces:**

1. **Testability.** Blocks 2.0–4.0's command and control paths can be tested in isolation by injecting Supervisor-equivalent commands, without requiring a live console or safety interlock. (Blocks 1.0 and 3.0 additionally require their direct sensor/imaging feeds to be tested end-to-end, which is a reasonable trade for the latency benefit.)
2. **Fault handling centralization.** When Block 1.0 reports a lost-tracking alert (IF-03), the Supervisor decides the system response — hold, alert, or abort. The decision logic is in one place, not distributed across functional blocks.
3. **Interface stability.** Changes to external command/control systems (e.g., a new operator console) require changes only to the Supervisor's external interface handling, not to Path Planning or Localization. Changes to sensor hardware require changes to the directly-connected subsystem (Localization or Image Processing) and are appropriately scoped there rather than touching the Supervisor.

---

## 6. Notation Note

All diagrams in this portfolio artifact are built in Modelio with the SysML module, following SysML 1.6 Block Definition Diagram (BDD) and Internal Block Diagram (IBD) conventions, including block stereotypes, proxy ports, and item flows.

Because Modelio's SysML module enforces SysML semantics, block relationships, ports, and item flows are defined as elements in the underlying model and are consistent across diagrams, rather than existing only as independent graphical shapes.

---

## 7. Assumptions and Open Questions

**Assumptions made to bound this model:**

- Bronchoscope degrees of freedom (DOF) count is not specified; kinematics are treated as a black box consumed via IF-EX-03.
- The airway model from the preoperative planning system is assumed to be a centerline graph with segment IDs, not a raw mesh (influences Block 2.1 algorithm selection).
- Respiratory motion compensation is not modeled. Tissue deformation due to breathing is an open problem; the current architecture assumes it is handled by the CBCT registration frequency and EKF uncertainty bounds, which is a simplification.
- CBCT acquisition frequency is assumed to be clinically determined (radiation dose constraint), not RBNS-driven.

**Open systems questions not resolved in this model:**

- What is the correct uncertainty threshold for replanning inhibition, and how is it validated clinically? (Tracked as COMP-REQ-003, TBD.)
- How does Block 1.2 (vision-based localization) degrade gracefully when endoscopic visibility is lost entirely — does the system fall back to FK-only, and what does that do to the uncertainty ellipsoid?
- What is the maximum tolerable latency on IF-EX-04 (guidance commands to motion controller) for safe scope advancement, and does the current architecture meet it?
- How should the Supervisor handle simultaneous faults — e.g., lost tracking (IF-03 alert) coinciding with a safety interlock (IF-EX-06)?
- What is the required bronchoscope tip articulation range to satisfy SYS-REQ-001 (navigable reach into the segmented airway tree)? (Tracked as COMP-REQ-001, TBD pending kinematic reach analysis.)

These questions are intentionally surfaced rather than resolved. In a production program, they would be inputs to the system Failure Mode and Effects Analysis (FMEA) and hazard analysis.

---

*This document is a portfolio artifact demonstrating systems engineering functional decomposition, interface definition, and design decision documentation practices. It is not derived from proprietary information of any commercial product.*
