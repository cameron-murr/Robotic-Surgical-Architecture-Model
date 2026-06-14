# RBNS Requirements Hierarchy

**Status:** Baseline v2.0 — supersedes v1.0, which contained only the Subsystem Requirements tier.
**Structure:** User Needs → System Requirements → Subsystem Requirements → Component Requirements → Verification Method.
**Verification methods:** I = Inspection, A = Analysis, D = Demonstration, T = Test

This hierarchy exists for two reasons: (1) it gives the D8 requirements traceability diagram a real multi-level trace rather than a flat allocation table, and (2) it demonstrates the correct decomposition pattern for solution-space details (like articulation range) that are easy to mis-level as system requirements. See the worked example at the end of this document.

---

## Tier 1: User Needs

| ID | User Need |
|---|---|
| UN-001 | The physician shall be able to navigate a bronchoscope to a target nodule identified in preoperative imaging, for the purpose of biopsy. |
| UN-002 | The physician shall be able to reach target nodules located in peripheral (small-diameter, high-generation) airway branches not reliably accessible with manual bronchoscopy. |
| UN-003 | The system shall not cause unintended injury to the airway during navigation. |
| UN-004 | The physician shall retain supervisory authority over the procedure and shall be able to pause or abort navigation at any time. |
| UN-005 | The procedure shall be recorded in sufficient detail to support post-procedure review. |

---

## Tier 2: System Requirements

Derived from user needs. Stated at the level of the RBNS as a whole, without specifying internal decomposition or kinematic implementation.

| ID | System Requirement | Derived From | Verif. |
|---|---|---|---|
| SYS-REQ-001 | The RBNS shall provide navigation guidance sufficient to reach target nodules located anywhere within the segmented airway tree provided by the preoperative planning system, including branches up to the generation depth represented in that model. | UN-002 | A, D |
| SYS-REQ-002 | The RBNS shall provide a tip position estimate at the target location accurate to within a defined tolerance sufficient to support biopsy confirmation. | UN-001 | T, A |
| SYS-REQ-003 | The RBNS shall not issue guidance commands that would result in airway wall contact forces exceeding safe tissue limits. | UN-003 | T, A |
| SYS-REQ-004 | The RBNS shall cease issuing guidance commands within a bounded time of an operator pause/abort command or a safety interlock deassertion. | UN-004 | T |
| SYS-REQ-005 | The RBNS shall record navigation events and system state with sufficient fidelity to support post-procedure audit. | UN-005 | T, I |
| SYS-REQ-006 | The RBNS shall detect loss of reliable position information and shall respond without issuing guidance commands based on that unreliable information. | UN-003 | T, A |

**Note on SYS-REQ-001:** This requirement is intentionally stated as a *reach/workspace* capability, not as a bend angle, degrees-of-freedom count, or articulation specification. At the system level, the requirement is about what clinical targets must be accessible — it says nothing about how the bronchoscope achieves that access. The "how" (articulation range, insertion mechanics, steering authority) is a derived design solution and belongs at the component level. See the worked example below.

---

## Tier 3: Subsystem Requirements

Carried over from v1.0 (RBNS-REQ-001 through 017), with a "Traces To" column added linking each to its parent System Requirement(s).

| ID | Requirement | Allocated To | Traces To | Verif. |
|---|---|---|---|---|
| RBNS-REQ-001 | The RBNS shall produce a fused bronchoscope tip pose estimate at a rate of no less than 30 Hz during the NAVIGATE phase. | 1.0 / 1.4 | SYS-REQ-001, SYS-REQ-002 | T |
| RBNS-REQ-002 | Each fused tip pose estimate shall include a 3D position uncertainty representation expressed as an ellipsoid with semi-axis lengths in millimeters. | 1.4 | SYS-REQ-002, SYS-REQ-006 | I, T |
| RBNS-REQ-003 | The RBNS shall complete image-to-patient registration within 4 seconds of receiving a CBCT volume. | 1.3, 3.0 | SYS-REQ-002 | T |
| RBNS-REQ-004 | The RBNS shall inhibit path replanning when the tip position uncertainty exceeds the configured replanning threshold. | 2.2 | SYS-REQ-003, SYS-REQ-006 | T |
| RBNS-REQ-005 | The replanning uncertainty threshold shall be a configurable parameter modifiable without software rebuild. | 2.2 | SYS-REQ-003 | I, D |
| RBNS-REQ-006 | The RBNS shall command zero scope advancement (HOLD) within one guidance cycle of the tip position uncertainty exceeding the configured threshold. | 2.3 | SYS-REQ-003, SYS-REQ-006 | T |
| RBNS-REQ-007 | All guidance commands shall be magnitude-limited to configurable maximum advance rate and maximum steering angle values prior to transmission on IF-07. | 2.3 | SYS-REQ-003 | T |
| RBNS-REQ-008 | The RBNS shall implement the procedure state machine with states IDLE, NAVIGATE, CONFIRM, BIOPSY, FAULT, and ABORT, with transitions as defined in the Supervisor state machine model. | 4.0 | SYS-REQ-004 | T, D |
| RBNS-REQ-009 | All communication between the RBNS and external systems shall be routed through the Procedure Supervisor. | 4.0 (architecture constraint) | SYS-REQ-004, SYS-REQ-005 | I, A |
| RBNS-REQ-010 | The RBNS shall report a lost-tracking alert to the Procedure Supervisor within 100 ms of tracking loss detection. | 1.4 | SYS-REQ-006 | T |
| RBNS-REQ-011 | Upon deassertion of the motion enable signal (IF-EX-06a), the RBNS shall cease issuing guidance commands within 50 ms. | 4.0 | SYS-REQ-003, SYS-REQ-004 | T |
| RBNS-REQ-012 | Upon receiving a lost-tracking alert, the Procedure Supervisor shall transition to the FAULT state and command HOLD on IF-EX-04 within 100 ms. | 4.0 | SYS-REQ-003, SYS-REQ-006 | T |
| RBNS-REQ-013 | The RBNS shall maintain a degraded tip pose estimate upon failure of any single localization source, and shall flag the estimate as DEGRADED on IF-03. | 1.4 | SYS-REQ-002, SYS-REQ-006 | T, A |
| RBNS-REQ-014 | The RBNS shall compute an airway path from current position to a newly commanded target within 1 second. | 2.1 | SYS-REQ-001 | T |
| RBNS-REQ-015 | The end-to-end latency from a tip pose estimate (IF-01) to the corresponding guidance command transmission (IF-EX-04) shall not exceed 200 ms. | 2.0, 4.0 | SYS-REQ-001, SYS-REQ-003 | T, A |
| RBNS-REQ-016 | The RBNS shall issue CBCT acquisition triggers (IF-EX-02b) only during the NAVIGATE and CONFIRM procedure phases. | 4.0 | SYS-REQ-002 | T |
| RBNS-REQ-017 | The RBNS shall record all state transitions, replanning events, fault events, and the tip pose trace to the Procedure Data Recorder (IF-EX-07) with timestamps of 10 ms resolution or better. | 4.0 | SYS-REQ-005 | T, I |

---

## Tier 4: Component Requirements

Derived from subsystem requirements through analysis (kinematic, statistical, or parametric). These are the requirements that translate a capability into a concrete engineering specification.

| ID | Requirement | Allocated To | Derived From | Method of Derivation | Verif. |
|---|---|---|---|---|---|
| COMP-REQ-001 | The Guidance Command Generator (2.3) shall support distal tip articulation commands of up to [TBD, kinematic analysis output] degrees per axis, sufficient to follow airway centerline curvature at branch points up to the generation depth required by SYS-REQ-001, given the bronchoscope's outer diameter, bend radius, and insertion depth at the target. | 2.3 | SYS-REQ-001, RBNS-REQ-014 | Kinematic reach analysis: given the airway centerline geometry from the segmented model, compute the minimum tip articulation envelope (angle and rate) required to traverse the tightest branch curvature on any path to a reachable target. | A, T |
| COMP-REQ-002 | The Sensor Fusion & Uncertainty Estimator (1.4) shall bound the fused position uncertainty ellipsoid such that, given a registration confidence score above the minimum threshold, the 95% confidence volume does not exceed [TBD, statistical analysis output]. | 1.4 | SYS-REQ-002, RBNS-REQ-002 | Statistical analysis of EKF covariance propagation between CBCT corrections, parameterized by FK drift rate and vision update rate. | A, T |
| COMP-REQ-003 | The Real-Time Path Updater (2.2) shall default the replanning inhibition threshold to an uncertainty ellipsoid semi-axis of [TBD, parametric analysis output], expressed in the same units as IF-01, configurable per RBNS-REQ-005. | 2.2 | SYS-REQ-003, RBNS-REQ-004 | Parametric analysis relating airway diameter at branch points (from the segmented model) to the maximum tolerable position uncertainty before a guidance command risks wall contact. | A, T |

---

## Worked Example: Articulation Range — Correct Decomposition

This example exists to demonstrate the correct system-requirement framing for "the bronchoscope must bend to reach targets," contrasting with the framing error of stating articulation range as a system requirement directly.

**Incorrect framing (system-level):**
> "The system shall provide bronchoscope tip articulation of plus or minus 180 degrees."

This is incorrect at the system level for two reasons. First, it bakes in a specific kinematic solution (a particular articulation mechanism and range) as if it were a clinical or mission requirement — but the actual need is about *reach*, and there may be multiple kinematic solutions (different bend radii, multi-segment articulation, insertion-depth strategies) that satisfy the same reach need. Second, it is not independently verifiable against a clinical need — "is plus or minus 180 degrees correct?" cannot be answered without first knowing what reach is required, which means the requirement as stated is actually a *derived* value masquerading as a top-level one.

**Correct framing, full chain:**

1. **User Need (UN-002):** The physician shall be able to reach target nodules located in peripheral airway branches not reliably accessible with manual bronchoscopy.
2. **System Requirement (SYS-REQ-001):** The RBNS shall provide navigation guidance sufficient to reach target nodules located anywhere within the segmented airway tree, including branches up to the generation depth represented in that model. — *Note: this is stated purely in terms of clinical workspace (airway generations), with no reference to articulation, bend angle, or mechanism.*
3. **Subsystem Requirement (RBNS-REQ-014):** The RBNS shall compute an airway path from current position to a newly commanded target within 1 second. — *This addresses the planning side of "reach": the system must be able to find a route through that workspace.*
4. **Component Requirement (COMP-REQ-001):** The Guidance Command Generator shall support distal tip articulation of a defined number of degrees per axis, where that number is the output of a kinematic reach analysis against the airway geometry. — *This is where articulation range actually belongs. It is a number produced by analysis, not asserted from clinical need directly.*

**Why this matters in an interview:** When asked for "a system requirement for needing the bronchoscope to bend to reach targets," the correct answer is a workspace/reach statement (SYS-REQ-001-style), not a degrees-of-articulation statement. If pressed for the articulation number itself, the correct response is to identify it as a *derived component requirement* and to name the analysis that produces it (kinematic reach analysis against the segmented airway model) — demonstrating that you understand the requirement is an output of a design trade, not an input to it.
