# Business Requirements Document (BRD)
## Holocron Command Center — Production Estimation Platform

**Document Version:** 1.0  
**Date:** February 25, 2026  
**Project:** Holocron V1  
**Prepared For:** Engineering Team  
**Authority Documents:**  
- Holocron_IPR_022026_v1.md (Product Vision)  
- bimteq_command_center_explanation_022026_v1.md (Design Reference)  
- bimteq-command-center_022026_v1.html (Interface Mockup)

---

## 1. Executive Summary

### 1.1 Purpose
This BRD translates the Holocron Internal Press Release and Command Center design documentation into actionable engineering requirements. It defines the functional and non-functional requirements for V1, incorporating stakeholder feedback from the Q&A clarifications.

### 1.2 Product Vision
Holocron eliminates the document hunt and tribal knowledge bottleneck in HVAC estimation by providing estimators with organized project documents, HVAC-relevant content navigation, client-specific rules access, and automated conflict detection — reducing the 15-20% rework rate and transforming 2-hour spec reviews into 30-minute sessions.

### 1.3 Core User Groups

| Role | Primary Function | V1 Scope |
|------|-----------------|----------|
| Jr Estimator | Spec review, takeoff execution (Pakistan) | ✓ Full access |
| Sr Estimator | Complex takeoffs, mentorship | ✓ Full access + QC override |
| QC Engineer | Post-takeoff validation (India + Pakistan) | ✓ QC panel access |
| Coordinator | Project intake, assignment, tracking | ✓ Dashboard + assignment |
| Manager | Production oversight, metrics | ✓ Admin/Ops panel |
| Account Manager | Client SOP management | ⚠ V2 scope (MRSM excluded) |

### 1.4 Out of Scope (V1)
- **MRSM integration:** Basis Board workflow deferred to V2
- **Pricing functionality:** Holocron is quantity-only; cost alternates are explicitly out of scope
- **Highlighted PDF tooling:** Manual creation required, upload only
- **QuoteSoft deep integration:** V1 uses export validation only
- **Mobile interface:** Desktop-only workflow
- **Automated takeoff:** QuoteSoft remains manual

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

## 3. User Interface Requirements

### 3.1 Layout Architecture

**Navigation Model:** Single-panel view with left sidebar navigation (56px rail)

**Top Bar (56px height):**
- Logo: "BIMTEQ•COMMAND"
- Role selector dropdown (determines default landing panel)
- System status indicator (live connection)
- User avatar + name

**Sidebar (56px width):**
- 6 icon-based navigation items with hover tooltips
- Active item highlighted in cyan (#00e5cc)
- Live badge on Contention Review showing pending count
- All panels accessible regardless of role

**Panel Container:**
- Fills remaining viewport space
- One panel active at time (display toggle)
- Subtle fadeIn animation on panel transitions

### 3.2 Panel 1 — Project Dashboard

**Sidebar Icon:** 📊  
**Default Landing For:** Coordinator  
**Primary Users:** All roles (shared operational view)

#### Functional Requirements

**FR-PD-001:** Display all active projects in sortable table format
- Columns: Project Name/ID, Stage, Status, Effort (hrs), Estimator(s)
- Real-time updates from Project State Agent
- Click row with conflicts → navigate to Contention Review for that project

**FR-PD-002:** Stage badges must reflect VSM workflow
- Intake (not shown in mockup — projects visible after assignment)
- Deep Dive (blue)
- Takeoff (cyan)
- QC Review (orange)
- Complete (green)

**FR-PD-003:** Status chips indicate project health
- In Progress (cyan border)
- N Conflicts (orange border, animated pulse)
- Complete (green border)

**FR-PD-004:** Estimator assignment display
- Show avatar initials, full name, role tag (Jr/Sr Estimator, QC)
- Multi-estimator support: Display multiple avatars when 2+ estimators assigned
- Role tags replace location flags (role-based design, not region-based)

**FR-PD-005:** Effort tracking
- Display estimated effort in hours (font: Azeret Mono)
- Source: Assignment Recommendation Agent calculation + actual time logged

#### Data Sources
- **Agent:** Project State Agent (real-time stage/status)
- **Input:** Estimator actions, coordinator assignments, QC outcomes
- **Update Frequency:** WebSocket/SSE for live updates (V1: polling acceptable with 30s interval)
