# RBNS Modelio Build Plan — Full SysML Diagram Set

**Tool:** Modelio Open Source 5.x + official SysML module
**Source of truth:** `interface_registry.md` v1.1 and `requirements.md` v2.1 — build the model from these, not from the draw.io sketches.

---

## Diagram Set (8 diagrams)

| # | Diagram | SysML Type | Owner Element | Purpose |
|---|---|---|---|---|
| D1 | Model Organization | Package Diagram | Root | Shows package structure: Structure / Behavior / Requirements / Interfaces |
| D2 | System Context | BDD | Context package | RBNS block + 7 external actor blocks («external» stereotype) + boundary interface associations |
| D3 | Structural Hierarchy | BDD | RBNS block | Composition tree: RBNS ◆→ 4 subsystems ◆→ 7 L2 blocks. The "one diagram that explains everything" |
| D4 | RBNS Internal Structure | IBD | RBNS block | 4 part properties, proxy ports, connectors with item flows per Tier 2 registry (IF-01..IF-08) |
| D5 | Localization Internal Structure | IBD | Block 1.0 | 4 part properties, item flows per Tier 3 registry (IF-1.1..IF-1.4) |
| D6 | Path Planning Internal Structure | IBD | Block 2.0 | 3 part properties, item flows per Tier 3 registry (IF-2.1..IF-2.3), replanning feedback connector |
| D7 | Procedure Supervisor Behavior | State Machine | Block 4.0 | States: IDLE, NAVIGATE, CONFIRM, BIOPSY, FAULT, ABORT. Transitions triggered by IF-EX-05a commands, IF-03 alerts, IF-EX-06a deassert. Satisfies SUB-REQ-008 |
| D8 | Requirements Traceability | Requirements Diagram | Requirements package | All requirements tiers (user needs, system, subsystem, component) as «requirement» elements; «satisfy» links to blocks; «verify» placeholders |

**Optional stretch (D9):** Parametric diagram for the uncertainty propagation constraint (ellipsoid volume vs. replanning threshold). High signal, but only if time permits — do not block v1 publication on it.

---

## Build Order (model tree first, diagrams second)

### Phase 1 — Package and block structure (~2 hrs)
1. Create project `RBNS-Architecture`
2. Packages: `01_Context`, `02_Structure`, `03_Interfaces`, `04_Behavior`, `05_Requirements`
3. In `02_Structure`, create blocks: `RBNS`, `LocalizationTracking`, `PathPlanningGuidance`, `ImageProcessingRegistration`, `ProcedureSupervisor`, then L2 blocks: `ProprioceptiveStateEstimator`, `VisionBasedLocalization`, `ImageToPatientRegistration`, `SensorFusionUncertaintyEstimator`, `AirwayGraphNavigator`, `RealTimePathUpdater`, `GuidanceCommandGenerator`
4. Set composition relationships (RBNS owns the four subsystems; subsystems own their L2 blocks)
5. In `01_Context`, create external blocks with «external» stereotype: `PreopPlanningSystem`, `CBCTImagingSystem`, `RobotMotionController`, `OperatorConsole`, `SafetyInterlockSystem`, `ProcedureDataRecorder`, `BronchoscopeCamera`

### Phase 2 — Interface definitions (~2 hrs)
6. In `03_Interfaces`, create an InterfaceBlock per registry entry (Tier 1 and Tier 2 minimum). Name them exactly per registry IDs: `IF_01_FusedPose`, `IF_EX_04_GuidanceCommand`, etc.
7. Define value properties on each interface block for the data items (e.g., `IF_01_FusedPose` gets `position: mm[3]`, `orientation: quaternion`, `uncertaintyEllipsoid: mm[3]`)
8. Create proxy ports on blocks, typed by the interface blocks

### Phase 3 — Diagrams D1–D6 (~4-5 hrs)
9. D3 first (structural hierarchy BDD) — it's automatic once composition exists and validates your Phase 1 work
10. D2 (context), D4 (RBNS IBD), then D5, D6
11. Add item flows on every connector, referencing the interface blocks

### Phase 4 — Behavior and requirements (~3 hrs)
12. D7 state machine on `ProcedureSupervisor`. Transitions:
    - IDLE → NAVIGATE [START + plan loaded]
    - NAVIGATE → CONFIRM [distance-to-target < threshold]
    - CONFIRM → BIOPSY [operator CONFIRM_TARGET]
    - BIOPSY → NAVIGATE [operator REPLAN] / → IDLE [procedure complete]
    - any → FAULT [lost-tracking alert (IF-03) OR motion enable deassert (IF-EX-06a)]
    - FAULT → NAVIGATE [fault cleared + operator confirm] / → ABORT [operator ABORT]
    - any → ABORT [operator ABORT]
    - Entry action on FAULT: command HOLD on IF-EX-04 (satisfies SUB-REQ-012)
13. D8: import the full requirements hierarchy from `requirements.md` (user needs, system requirements, subsystem requirements, component requirements) as «requirement» elements, draw «satisfy» dependencies to allocated blocks per the allocation table. Component requirements marked TBD (COMP-REQ-001 through 003) should appear in the diagram as open items, not omitted.

### Phase 5 — Export and publish (~2 hrs)
14. Export every diagram: right-click diagram → Export image → SVG, white background
15. Commit the Modelio project folder to the repo under `/model` (the project directory is the model — no separate save-as needed; zip it as a release asset too)
16. Embed SVGs in README in this order: D3 (hero candidate), D2, D4, D7, D8, with D5/D6 linked below the fold

---

## Export Quality Settings

- White background, not transparent (GitHub dark mode renders transparent SVGs poorly)
- Hide the Modelio grid before export
- Set diagram zoom to 100% before export to keep font rendering consistent
- Keep diagram names file-safe: `D3_structural_hierarchy.svg` etc.

## Known Modelio Pitfalls

- IBD part properties: drag the *property* from the model tree, not the block — dragging the block creates a new untyped element
- The SysML module's item flow dialog occasionally fails to list interface blocks; workaround is to create the item flow first, then set the conveyed element in the Properties panel
- Save after every diagram. Modelio autosave is unreliable with the SysML module loaded
- Undo across model-tree operations is inconsistent — prefer deleting and recreating over undo for structural mistakes

## Definition of Done (v1)

- [ ] All 8 diagrams exist and conform to interface registry v1.1
- [ ] Every connector in D4–D6 carries an item flow named per registry
- [ ] D7 state machine satisfies SUB-REQ-008 transition set
- [ ] All requirements tiers (user needs, system, subsystem, component) appear in D8 with at least one «satisfy» link each where applicable
- [ ] SVGs exported and embedded in README
- [ ] Modelio project committed to `/model`, zipped release asset created
- [ ] draw.io XMLs moved to `/sketches` with README note: "Preliminary concept sketches, superseded by the Modelio model"
