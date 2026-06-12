# Robotic Bronchoscopy Navigation Subsystem — Functional Architecture Model

A systems engineering portfolio project demonstrating functional decomposition, interface management, requirements allocation, and SysML modeling for a bounded robotic surgical subsystem.

## Overview

This project develops a functional architecture model for a **Robotic Bronchoscopy Navigation Subsystem (RBNS)** — the software subsystem responsible for intraoperative localization, path planning, and guidance command generation in a robotic bronchoscopy platform.

The system boundary is deliberately narrow: the RBNS receives a preoperative airway model and target nodule coordinates, maintains a real-time estimate of bronchoscope tip position using multi-source sensor fusion, and outputs motion guidance commands to a downstream robot motion controller. It excludes the robot kinematics stack, the CBCT imaging hardware, and the surgeon console rendering pipeline.

This scoping reflects a real architectural seam present in platforms like the J&J Monarch QUEST system, where a verified open interface separates intraoperative imaging hardware from navigation software.

## Repository Structure

```
├── docs/
│   ├── interface_registry.md    ← CANONICAL interface definitions (3 tiers, 26 interfaces)
│   ├── requirements.md          ← 17 subsystem requirements with allocation + verification methods
│   ├── requirements.csv         ← Tooling-readable export (medtrace-compatible)
│   ├── design_decisions.md      ← 5 key architectural decisions with rationale
│   └── modelio_build_plan.md    ← SysML diagram set specification and build sequence
├── model/                       ← Modelio SysML project (in progress)
└── sketches/                    ← Preliminary draw.io concept sketches (superseded)
```

## SysML Model

The formal model is built in **Modelio Open Source 5.x with the official SysML module**, comprising 8 diagrams:

| Diagram | Type | Content |
|---|---|---|
| D1 | Package | Model organization |
| D2 | BDD | System context — RBNS + 7 external actors |
| D3 | BDD | Structural hierarchy — full composition tree |
| D4 | IBD | RBNS internal structure — 4 subsystems, 8 internal interfaces |
| D5 | IBD | Localization & Tracking decomposition — EKF sensor fusion architecture |
| D6 | IBD | Path Planning & Guidance decomposition — replanning feedback loop |
| D7 | STM | Procedure Supervisor state machine |
| D8 | REQ | Requirements traceability — satisfy links to allocated blocks |

*Exported diagram SVGs will be embedded here as the Modelio build completes.*

## Architecture Summary

```
RBNS
├── 1.0 Localization & Tracking Subsystem
│   ├── 1.1 Proprioceptive State Estimator (FK, high-rate/low-accuracy)
│   ├── 1.2 Vision-Based Localization (feature matching, medium-rate)
│   ├── 1.3 Image-to-Patient Registration (ICP on CBCT, low-rate/high-accuracy)
│   └── 1.4 Sensor Fusion & Uncertainty Estimator (EKF → uncertainty ellipsoid on IF-01)
│
├── 2.0 Path Planning & Guidance Subsystem
│   ├── 2.1 Airway Graph Navigator (A* on preop airway graph)
│   ├── 2.2 Real-Time Path Updater (deviation detection; replanning inhibition under uncertainty)
│   └── 2.3 Guidance Command Generator (safety-bounded advance/steer commands)
│
├── 3.0 Image Processing & Registration Subsystem
│   (CBCT volume ingestion; registration + confidence score output)
│
└── 4.0 Procedure Supervisor
    (Sole external interface point; state machine; centralized fault handling)
```

## Key Design Decisions

1. **Sensor fusion over single-source ground truth** — CBCT registration is periodic (dose-limited), so an EKF fuses it with continuous proprioceptive and vision estimates, treating CBCT as a periodic high-accuracy correction.

2. **Uncertainty as an explicit interface signal** — IF-01 carries a 3D uncertainty ellipsoid alongside the pose estimate. Path planning gates replanning on this value (RBNS-REQ-004): the system will not replan from a position it cannot trust.

3. **Paired fault detection and response requirements** — Lost-tracking detection (REQ-010, ≤100 ms) and Supervisor response (REQ-012, ≤100 ms) are separately specified and allocated, making the fault chain testable end to end.

4. **Procedure Supervisor as sole external interface point** — All external systems interface only with the Supervisor (REQ-009). Functional blocks are testable in isolation.

5. **Interface registry as change-controlled single source of truth** — A three-tier registry governs all artifacts. The registry audit caught two real defects in the preliminary sketches: an undocumented boundary crossing (bronchoscope camera feed) and a routing contradiction (guidance commands bypassing the Supervisor).

## Process Note

The `/sketches` directory contains the initial draw.io concept diagrams. They were superseded when the project moved to a formal Modelio SysML model driven by the interface registry — and they contain the two defects the registry audit caught, retained deliberately as evidence of the review process.

## Related Work

Requirements in this project are structured for traceability analysis with [medtrace](https://github.com/cameron-murr/medtrace), a Python CLI requirements traceability tool built as a companion portfolio project.

---

*Portfolio artifact. Architecture is original work informed by publicly available information about robotic bronchoscopy platforms; not derived from proprietary product information.*
