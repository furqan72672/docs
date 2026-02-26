# Section 1: Existing Content

(Existing content here)

---

## 2. System Architecture Overview

### 2.1 Platform Foundation
**AWS Microservices Architecture** (shared with Argus SaaS product)

```
┌─────────────────────────────────────────────────┐
│           Orchestrator Engine                    │
│   (Circuit management, handoffs, failures)      │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│              Agent Registry                      │
│   (Agent contracts: inputs, outputs, behavior)  │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│            Containerized Runtime                 │
│        (6 specialized agents execute)           │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│           State Management Layer                 │
│    (Artifacts, circuit state, project status)   │
└─────────────────────────────────────────────────┘
```

### 2.2 Six-Agent Circuit

| Agent | Type | Input | Output | Human Checkpoint |
|-------|------|-------|--------|------------------|
| **1. Intake & Prep** | Active | Raw uploads (OneDrive) | Organized folders, validation report | Coordinator reviews flags |
| **2. Spec Navigator** | Assistive | Spec PDF, project context | Structured section map (Div 22/23/40) | Estimator navigates/reviews |
| **3. Contention Detection** | Agentic | Drawings, specs, SOPs | Conflict alerts with precedence | Estimator resolves |
| **4. Assignment Recommendation** | Assistive | Estimator profiles, workload | Ranked recommendations | Coordinator assigns |
| **5. Project State** | Observational | Estimator actions, stage changes | Real-time status visibility | No action required |
| **6. QC & Audit** | Hybrid | System Chart, QuoteSoft export | Discrepancy report | QC Engineer reviews |

**Critical Design Principle:** Agents assist; humans decide. No automated decision-making at key checkpoints (assignment, contention resolution, QC approval).