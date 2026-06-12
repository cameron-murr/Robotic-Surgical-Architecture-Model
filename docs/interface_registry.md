# RBNS Interface Registry

**Status:** CANONICAL — this registry is the single source of truth for all interface definitions.
All diagrams, documents, and models shall conform to this registry. Discrepancies are defects in the downstream artifact, not the registry.

**Version:** 1.0
**Change control:** Any interface addition, removal, or redefinition requires a version increment and a changelog entry.

---

## Tier 1: External Interfaces (System Boundary)

| ID | Source | Sink | Type | Data Items | Rate / Trigger | Latency Req | Notes |
|---|---|---|---|---|---|---|---|
| IF-EX-01 | Preoperative Planning System | 4.0 Supervisor | Data | Segmented airway centerline graph (segment IDs, branch topology), surface mesh, target nodule coordinates (x,y,z, patient frame) | Aperiodic (procedure setup) | N/A | Loaded once per procedure; re-load permitted on target change |
| IF-EX-02a | CBCT Imaging System | 3.0 Image Processing | Data | Intraoperative volumetric image (DICOM volume), acquisition metadata (timestamp, geometry) | Triggered (see IF-EX-02b) | Volume available ≤ 10 s post-acquisition | GE OEC 3D via OEC Open-style interface |
| IF-EX-02b | 4.0 Supervisor | CBCT Imaging System | Control | Acquisition trigger command, acquisition mode | Event-driven (procedure phase) | N/A | Trigger permitted only in defined procedure phases (REQ-016) |
| IF-EX-03 | Robot Motion Controller | 1.0 Localization | Data | Joint encoder positions, articulation bend angles, insertion depth | 100 Hz continuous | ≤ 10 ms transport | Primary proprioceptive feed |
| IF-EX-04 | 4.0 Supervisor | Robot Motion Controller | Control | Guidance command packet: advance/retract magnitude, steering vector, HOLD flag | 30 Hz when in NAVIGATE | ≤ 50 ms transport | Supervisor forwards packets received on IF-07; sole motion output of RBNS |
| IF-EX-05a | Operator Console | 4.0 Supervisor | Control | Procedure state commands (START, PAUSE, ABORT, CONFIRM_TARGET, REPLAN) | Event-driven | ≤ 100 ms to state transition | Operator authority supersedes autonomous replanning |
| IF-EX-05b | 4.0 Supervisor | Operator Console | Data | Navigation overlay data (tip pose, path, uncertainty visualization), alert states | 30 Hz | ≤ 100 ms | Display rendering is console-side, out of RBNS scope |
| IF-EX-06a | Safety & Interlock System | 4.0 Supervisor | Control | Motion enable signal (discrete) | Continuous monitor | Deassert response ≤ 50 ms (REQ-011) | Hardware-backed interlock |
| IF-EX-06b | 4.0 Supervisor | Safety & Interlock System | Control | Fault acknowledgment, RBNS health status | Event-driven + 1 Hz heartbeat | N/A | |
| IF-EX-07 | 4.0 Supervisor | Procedure Data Recorder | Data | Navigation log: timestamped tip pose trace, uncertainty history, state transitions, replanning events, fault events | 10 Hz + events | N/A (non-real-time) | Supports post-procedure review and audit |
| IF-EX-08 | Bronchoscope Camera | 1.0 Localization | Data | Endoscopic video frames (RGB) | 30 Hz continuous | ≤ 33 ms transport | **Added in v1.0 registry audit** — previously undocumented boundary crossing consumed by Block 1.2 |

## Tier 2: Internal Interfaces (Level 1, between subsystem blocks)

| ID | Source | Sink | Type | Data Items | Rate / Trigger | Latency Req | Notes |
|---|---|---|---|---|---|---|---|
| IF-01 | 1.0 Localization | 2.0 Path Planning | Data | Fused tip position (x,y,z), orientation quaternion, velocity estimate, 3D uncertainty ellipsoid (semi-axes + orientation, mm) | 30 Hz min | ≤ 33 ms age at delivery | Uncertainty is a mandatory typed field, not optional metadata (Design Decision 3) |
| IF-02 | 2.0 Path Planning | 4.0 Supervisor | Data | Current waypoint ID, distance-to-target (mm), replanning event notifications | 10 Hz + events | ≤ 100 ms | Status only; commands are on IF-07 |
| IF-03 | 1.0 Localization | 4.0 Supervisor | Data | Tracking status enum (NOMINAL / DEGRADED / LOST), per-source health flags, lost-tracking alert | 10 Hz + alert events | Alert ≤ 100 ms from detection (REQ-010) | |
| IF-04 | 4.0 Supervisor | 2.0 Path Planning | Control | Procedure phase enum, target nodule coordinates, mode command (PLAN / NAVIGATE / HOLD / ABORT) | Event-driven | ≤ 50 ms | |
| IF-05 | 4.0 Supervisor | 3.0 Image Processing | Control | CBCT acquisition trigger relay, registration mode command (FULL / REFINE) | Event-driven | N/A | |
| IF-06 | 3.0 Image Processing | 1.0 Localization | Data | Registered CBCT volume reference, airway centerline update, registration confidence score [0,1] | Per acquisition (~aperiodic) | ≤ 4 s from volume receipt (REQ-003) | Confidence score is mandatory (Design Decision 3 corollary) |
| IF-07 | 2.0 Path Planning | 4.0 Supervisor | Control | Guidance command packet (identical schema to IF-EX-04 payload) | 30 Hz when in NAVIGATE | ≤ 20 ms | **Added in v1.0 registry audit** — resolves routing contradiction: Block 2.3 does NOT output directly to IF-EX-04; Supervisor forwards (Design Decision 5 preserved) |
| IF-08 | 4.0 Supervisor | 1.0 Localization | Control | Localization mode command (RUN / RESET / REACQUIRE), configuration parameters | Event-driven | ≤ 50 ms | **Added in v1.0 registry audit** — required for fault recovery: Supervisor must be able to command tracking re-acquisition after lost-tracking |

## Tier 3: Intra-Subsystem Flows (Level 2, within blocks)

### Within 1.0 Localization & Tracking

| ID | Source | Sink | Data Items | Rate | Notes |
|---|---|---|---|---|---|
| IF-1.1 | 1.1 Proprioceptive State Estimator | 1.4 Sensor Fusion | FK-derived tip pose estimate, velocity, FK covariance | 100 Hz | High-rate / low-accuracy channel |
| IF-1.2 | 1.2 Vision-Based Localization | 1.4 Sensor Fusion | Landmark matches, relative displacement estimate, vision confidence | 30 Hz | Fails gracefully to zero-confidence on occlusion |
| IF-1.3 | 1.3 Image-to-Patient Registration | 1.4 Sensor Fusion | CBCT-refined tip pose, registration covariance | Per acquisition | Low-rate / high-accuracy correction channel |
| IF-1.4 | 1.1 Proprioceptive State Estimator | 1.3 Image-to-Patient Registration | Current FK estimate (ICP initialization) | On registration start | Soft dependency; registration can run uninitialized |

### Within 2.0 Path Planning & Guidance

| ID | Source | Sink | Data Items | Rate | Notes |
|---|---|---|---|---|---|
| IF-2.1 | 2.1 Airway Graph Navigator | 2.2 Real-Time Path Updater | Planned path (ordered airway segment IDs, branch decisions) | On plan/replan | |
| IF-2.2 | 2.2 Real-Time Path Updater | 2.1 Airway Graph Navigator | Re-plan request (current position, reason code) | On deviation detect | Inhibited when uncertainty > threshold (REQ-004) |
| IF-2.3 | 2.2 Real-Time Path Updater | 2.3 Guidance Command Generator | Next waypoint, path segment geometry, HOLD directive | 30 Hz | |

---

## Changelog

| Version | Change | Rationale |
|---|---|---|
| 1.0 | Initial registry established as canonical. Added IF-EX-08 (bronchoscope camera), IF-07 (guidance forwarding), IF-08 (localization mode command). Renumber conflict with pre-registry working notes resolved in favor of artifact numbering. | Registry audit found one undocumented boundary crossing and one routing contradiction with Design Decision 5; both resolved. |

## Conformance Notes for Downstream Artifacts

- `L0_system_context` diagram: **requires update** — add Bronchoscope Camera actor and IF-EX-08
- `L2_path_planning_ibd` diagram: **requires update** — Block 2.3 output port relabeled IF-07 (to Supervisor), not IF-EX-04
- `design_decisions.md`: conformant (Design Decision 5 language now accurately reflects implemented routing)
- Modelio model: build directly from this registry
