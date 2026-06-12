# RBNS Subsystem Requirements

**Status:** Baseline v1.0
**Scope:** Subsystem-level requirements for the Robotic Bronchoscopy Navigation Subsystem (RBNS). Parent system requirements and clinical/user needs are out of scope for this portfolio artifact but are referenced notionally in the Rationale column.
**Verification methods:** I = Inspection, A = Analysis, D = Demonstration, T = Test

---

## Functional Requirements

| ID | Requirement | Allocated To | Verif. | Rationale / Trace Notes |
|---|---|---|---|---|
| RBNS-REQ-001 | The RBNS shall produce a fused bronchoscope tip pose estimate at a rate of no less than 30 Hz during the NAVIGATE phase. | 1.0 / 1.4 | T | Guidance loop stability; matches endoscopic video rate. Drives IF-01 rate. |
| RBNS-REQ-002 | Each fused tip pose estimate shall include a 3D position uncertainty representation expressed as an ellipsoid with semi-axis lengths in millimeters. | 1.4 | I, T | Design Decision 3: uncertainty is a typed mandatory output on IF-01, consumed by replanning gate. |
| RBNS-REQ-003 | The RBNS shall complete image-to-patient registration within 4 seconds of receiving a CBCT volume. | 1.3, 3.0 | T | One respiratory cycle; bounds the staleness of the high-accuracy correction channel (IF-06 latency). |
| RBNS-REQ-004 | The RBNS shall inhibit path replanning when the tip position uncertainty exceeds the configured replanning threshold. | 2.2 | T | Design Decision 4: system shall not replan from a position estimate it cannot trust. |
| RBNS-REQ-005 | The replanning uncertainty threshold shall be a configurable parameter modifiable without software rebuild. | 2.2 | I, D | Threshold depends on airway generation and procedure type; supports clinical validation tuning. |
| RBNS-REQ-006 | The RBNS shall command zero scope advancement (HOLD) within one guidance cycle of the tip position uncertainty exceeding the configured threshold. | 2.3 | T | Safety behavior paired with REQ-004; uncertain position must not produce motion. |
| RBNS-REQ-007 | All guidance commands shall be magnitude-limited to configurable maximum advance rate and maximum steering angle values prior to transmission on IF-07. | 2.3 | T | Soft tissue protection; limits are safety parameters subject to hazard analysis. |
| RBNS-REQ-008 | The RBNS shall implement the procedure state machine with states IDLE, NAVIGATE, CONFIRM, BIOPSY, FAULT, and ABORT, with transitions as defined in the Supervisor state machine model. | 4.0 | T, D | Single authoritative state machine; modeled as SysML STM diagram. |
| RBNS-REQ-009 | All communication between the RBNS and external systems shall be routed through the Procedure Supervisor. | 4.0 (architecture constraint) | I, A | Design Decision 5; verified by inspection of the model (no port on blocks 1.0–3.0 crosses the system boundary except sensor feeds IF-EX-03/08 and imaging feed IF-EX-02a, which are receive-only). |
| RBNS-REQ-010 | The RBNS shall report a lost-tracking alert to the Procedure Supervisor within 100 ms of tracking loss detection. | 1.4 | T | Bounds fault detection latency; feeds Supervisor fault response (REQ-012). |
| RBNS-REQ-011 | Upon deassertion of the motion enable signal (IF-EX-06a), the RBNS shall cease issuing guidance commands within 50 ms. | 4.0 | T | Safety interlock response; hardware interlock provides backstop, software response bounds nuisance motion. |
| RBNS-REQ-012 | Upon receiving a lost-tracking alert, the Procedure Supervisor shall transition to the FAULT state and command HOLD on IF-EX-04 within 100 ms. | 4.0 | T | Closes the fault response chain: detection (REQ-010) → response (REQ-012). |
| RBNS-REQ-013 | The RBNS shall maintain a degraded tip pose estimate upon failure of any single localization source (proprioceptive, vision, or registration), and shall flag the estimate as DEGRADED on IF-03. | 1.4 | T, A | Single-source fault tolerance; EKF architecture rationale (Design Decision 2). |
| RBNS-REQ-014 | The RBNS shall compute an airway path from current position to a newly commanded target within 1 second. | 2.1 | T | Bounds operator wait on target change; sizes the graph search implementation. |
| RBNS-REQ-015 | The end-to-end latency from a tip pose estimate (IF-01) to the corresponding guidance command transmission (IF-EX-04) shall not exceed 200 ms. | 2.0, 4.0 | T, A | Guidance loop responsiveness; allocated across IF-01 age, 2.2/2.3 processing, IF-07 transport, Supervisor forwarding. |
| RBNS-REQ-016 | The RBNS shall issue CBCT acquisition triggers (IF-EX-02b) only during the NAVIGATE and CONFIRM procedure phases. | 4.0 | T | Radiation dose discipline; acquisition is procedure-phase-gated. |
| RBNS-REQ-017 | The RBNS shall record all state transitions, replanning events, fault events, and the tip pose trace to the Procedure Data Recorder (IF-EX-07) with timestamps of 10 ms resolution or better. | 4.0 | T, I | Auditability and post-procedure review. |

---

## Allocation Summary

| Block | Allocated Requirements |
|---|---|
| 1.0 Localization & Tracking (incl. 1.3, 1.4) | REQ-001, 002, 003, 010, 013 |
| 2.0 Path Planning & Guidance (incl. 2.1, 2.2, 2.3) | REQ-004, 005, 006, 007, 014, 015 (shared) |
| 3.0 Image Processing & Registration | REQ-003 (shared) |
| 4.0 Procedure Supervisor | REQ-008, 009, 011, 012, 015 (shared), 016, 017 |

## Verification Method Distribution

Test: 14 | Analysis: 3 | Inspection: 4 | Demonstration: 2
(Several requirements carry multiple methods.)

---

## Notes for medtrace Integration

This requirements set is structured for ingestion by `medtrace` (github.com/cameron-murr/medtrace). Suggested trace chain:

```
User Need (notional) → RBNS-REQ-XXX → Allocated Block → Verification Method → [future: Test Case ID]
```

A CSV export (`requirements.csv`) accompanies this document for tooling use. Adding notional test case IDs (RBNS-TC-XXX) in a future revision would complete a four-level trace demonstration.
