# RBNS Requirements Hierarchy

**Status:** Baseline v2.3 — supersedes v2.2 (rewrote SYS-REQ tier to be properly system-level, scoped to the full robotic bronchoscopy system rather than the RBNS; SUB-REQ tier unchanged).
**Structure:** User Needs → System Requirements → Subsystem Requirements → Component Requirements → Verification Method.
**Verification methods:** I = Inspection, A = Analysis, D = Demonstration, T = Test

---

## Tier 1: User Needs

| ID | User Need |
|---|---|
| UN-001 | Physicians need to navigate a bronchoscope to a target nodule identified in preoperative imaging in order to obtain a tissue biopsy. |
| UN-002 | Physicians need to reach target nodules located in peripheral (small-diameter, high-generation) airway branches that are difficult or impossible to access with manual bronchoscopy. |
| UN-003 | Patients need assurance that navigation will not cause unintended injury to the airway. |
| UN-004 | Physicians need to retain supervisory authority over the procedure, including the ability to pause or abort navigation at any time. |
| UN-005 | Clinical staff need a record of the navigation procedure to support post-procedure review. |

---

## Tier 2: System Requirements

Scoped to the full robotic bronchoscopy system. These requirements describe what the overall system must do from a clinical perspective, without specifying internal subsystem architecture or implementation. The RBNS exists as the subsystem that satisfies these requirements through its functional decomposition.

| ID | System Requirement | Derived From | Verif. |
|---|---|---|---|
| SYS-REQ-001 | The system shall enable a physician to navigate a flexible bronchoscope to target pulmonary nodules identified in preoperative imaging. | UN-001, UN-002 | A, D |
| SYS-REQ-002 | The system shall provide the physician with sufficient positional confidence to perform a tissue biopsy at the target location. | UN-001 | T, A |
| SYS-REQ-003 | The system shall prevent unintended scope advancement or steering that could result in airway wall injury. | UN-003 | T, A |
| SYS-REQ-004 | The system shall respond to physician pause and abort commands within a clinically acceptable time. | UN-004 | T |
| SYS-REQ-005 | The system shall generate a complete procedure record suitable for post-procedure clinical review. | UN-005 | T, I |
| SYS-REQ-006 | The system shall detect conditions under which navigation accuracy cannot be assured and shall prevent scope motion under those conditions. | UN-003 | T, A |

---

## Tier 3: Subsystem Requirements

Scoped to the Robotic Bronchoscopy Navigation Subsystem (RBNS). These requirements define what the RBNS must do to satisfy the system requirements above. Originally numbered RBNS-REQ-001 through 017; renamed to SUB-REQ-### for consistency with the document numbering convention.

| ID | Requirement | Allocated To | Traces To | Verif. |
|---|---|---|---|---|
| SUB-REQ-001 | The RBNS shall produce a fused bronchoscope tip pose estimate at a rate of no less than 30 Hz during the NAVIGATE phase. | 1.0 / 1.4 | SYS-REQ-001, SYS-REQ-002 | T |
| SUB-REQ-002 | Each fused tip pose estimate shall include a 3D position uncertainty representation expressed as an ellipsoid with semi-axis lengths in millimeters. | 1.4 | SYS-REQ-002, SYS-REQ-006 | I, T |
| SUB-REQ-003 | The RBNS shall complete image-to-patient registration within 4 seconds of receiving a Cone Beam Computed Tomography (CBCT) volume. | 1.3, 3.0 | SYS-REQ-002 | T |
| SUB-REQ-004 | The RBNS shall inhibit path replanning when the tip position uncertainty exceeds the configured replanning threshold. | 2.2 | SYS-REQ-003, SYS-REQ-006 | T |
| SUB-REQ-005 | The replanning uncertainty threshold shall be a configurable parameter modifiable without software rebuild. | 2.2 | SYS-REQ-003 | I, D |
| SUB-REQ-006 | The RBNS shall command zero scope advancement (HOLD) within one guidance cycle of the tip position uncertainty exceeding the configured threshold. | 2.3 | SYS-REQ-003, SYS-REQ-006 | T |
| SUB-REQ-007 | All guidance commands shall be magnitude-limited to configurable maximum advance rate and maximum steering angle values prior to transmission on IF-07. | 2.3 | SYS-REQ-003 | T |
| SUB-REQ-008 | The RBNS shall implement the procedure state machine with states IDLE, NAVIGATE, CONFIRM, BIOPSY, FAULT, and ABORT, with transitions as defined in the Supervisor state machine model. | 4.0 | SYS-REQ-004 | T, D |
| SUB-REQ-009 | All command, control, status, and logging communication between the RBNS and external systems shall be routed through the Procedure Supervisor, with the exception of continuous sensor and imaging feeds (IF-EX-02a, IF-EX-03, IF-EX-08), which connect directly to their consuming subsystem. | 4.0 (architecture constraint) | SYS-REQ-004, SYS-REQ-005 | I, A |
| SUB-REQ-010 | The RBNS shall report a lost-tracking alert to the Procedure Supervisor within 100 ms of tracking loss detection. | 1.4 | SYS-REQ-006 | T |
| SUB-REQ-011 | Upon deassertion of the motion enable signal (IF-EX-06a), the RBNS shall cease issuing guidance commands within 50 ms. | 4.0 | SYS-REQ-003, SYS-REQ-004 | T |
| SUB-REQ-012 | Upon receiving a lost-tracking alert, the Procedure Supervisor shall transition to the FAULT state and command HOLD on IF-EX-04 within 100 ms. | 4.0 | SYS-REQ-003, SYS-REQ-006 | T |
| SUB-REQ-013 | The RBNS shall maintain a degraded tip pose estimate upon failure of any single localization source, and shall flag the estimate as DEGRADED on IF-03. | 1.4 | SYS-REQ-002, SYS-REQ-006 | T, A |
| SUB-REQ-014 | The RBNS shall compute an airway path from current position to a newly commanded target within 1 second. | 2.1 | SYS-REQ-001 | T |
| SUB-REQ-015 | The end-to-end latency from a tip pose estimate (IF-01) to the corresponding guidance command transmission (IF-EX-04) shall not exceed 200 ms. | 2.0, 4.0 | SYS-REQ-001, SYS-REQ-003 | T, A |
| SUB-REQ-016 | The RBNS shall issue CBCT acquisition triggers (IF-EX-02b) only during the NAVIGATE and CONFIRM procedure phases. | 4.0 | SYS-REQ-002 | T |
| SUB-REQ-017 | The RBNS shall record all state transitions, replanning events, fault events, and the tip pose trace to the Procedure Data Recorder (IF-EX-07) with timestamps of 10 ms resolution or better. | 4.0 | SYS-REQ-005 | T, I |

---

## Tier 4: Component Requirements

Derived from subsystem requirements through analysis (kinematic, statistical, or parametric). These requirements translate a capability into a concrete engineering specification. TBD values are pending the indicated analysis.

| ID | Requirement | Allocated To | Derived From | Method of Derivation | Verif. |
|---|---|---|---|---|---|
| COMP-REQ-001 | The Guidance Command Generator shall support distal tip articulation commands of up to [TBD, kinematic analysis output] degrees per axis, sufficient to follow airway centerline curvature at branch points up to the generation depth required by SYS-REQ-001, given the bronchoscope outer diameter, bend radius, and insertion depth at the target. | GuidanceCommandGenerator | SYS-REQ-001, SUB-REQ-014 | Kinematic reach analysis: given the airway centerline geometry from the segmented model, compute the minimum tip articulation envelope required to traverse the tightest branch curvature on any path to a reachable target. | A, T |
| COMP-REQ-002 | The Sensor Fusion & Uncertainty Estimator shall bound the fused position uncertainty ellipsoid such that, given a registration confidence score above the minimum threshold, the 95% confidence volume does not exceed [TBD, statistical analysis output]. | SensorFusionUncertaintyEstimator | SYS-REQ-002, SUB-REQ-002 | Statistical analysis of Extended Kalman Filter (EKF) covariance propagation between CBCT corrections, parameterized by forward kinematics (FK) drift rate and vision update rate. | A, T |
| COMP-REQ-003 | The Real-Time Path Updater shall default the replanning inhibition threshold to an uncertainty ellipsoid semi-axis of [TBD, parametric analysis output], expressed in the same units as IF-01, configurable per SUB-REQ-005. | RealTimePathUpdater | SYS-REQ-003, SUB-REQ-004 | Parametric analysis relating airway diameter at branch points to the maximum tolerable position uncertainty before a guidance command risks wall contact. | A, T |

---

## Allocation Summary

| Block | Allocated Requirements |
|---|---|
| LocalizationTracking (incl. ImageToPatientRegistration, SensorFusionUncertaintyEstimator) | SUB-REQ-001, 002, 003, 010, 013 |
| PathPlanningGuidance (incl. AirwayGraphNavigator, RealTimePathUpdater, GuidanceCommandGenerator) | SUB-REQ-004, 005, 006, 007, 014, 015 (shared) |
| ImageProcessingRegistration | SUB-REQ-003 (shared) |
| ProcedureSupervisor | SUB-REQ-008, 009, 011, 012, 015 (shared), 016, 017 |
| GuidanceCommandGenerator | COMP-REQ-001 |
| SensorFusionUncertaintyEstimator | COMP-REQ-002 |
| RealTimePathUpdater | COMP-REQ-003 |

---

## Worked Example: Articulation Range — Correct Decomposition

This example demonstrates the correct system-requirement framing for "the bronchoscope must bend to reach targets."

**Incorrect framing (system-level):**
> "The system shall provide bronchoscope tip articulation of plus or minus 180 degrees."

This is incorrect at the system level because it bakes in a specific kinematic solution as if it were a clinical requirement. The actual need is about *reach*, and there may be multiple kinematic solutions that satisfy the same reach need.

**Correct framing, full chain:**

1. **User Need (UN-002):** Physicians need to reach target nodules located in peripheral airway branches that are difficult or impossible to access with manual bronchoscopy.
2. **System Requirement (SYS-REQ-001):** The system shall enable a physician to navigate a flexible bronchoscope to target pulmonary nodules identified in preoperative imaging. — *Stated in terms of clinical capability; no reference to articulation, bend angle, or mechanism.*
3. **Subsystem Requirement (SUB-REQ-014):** The RBNS shall compute an airway path from current position to a newly commanded target within 1 second. — *Addresses the planning side of reach.*
4. **Component Requirement (COMP-REQ-001):** The Guidance Command Generator shall support distal tip articulation of a defined number of degrees per axis, where that number is the output of a kinematic reach analysis against the airway geometry. — *This is where articulation range belongs — a number produced by analysis, not asserted from clinical need directly.*
