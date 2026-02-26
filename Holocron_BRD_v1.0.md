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

---

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
- **Multi-estimator support:** Display multiple avatars when 2+ estimators assigned (per Q3 clarification)
- Role tags replace location flags (role-based design, not region-based)

**FR-PD-005:** Effort tracking
- Display estimated effort in hours (font: Azeret Mono)
- Source: Assignment Recommendation Agent calculation + actual time logged

#### Data Sources
- **Agent:** Project State Agent (real-time stage/status)
- **Input:** Estimator actions, coordinator assignments, QC outcomes
- **Update Frequency:** WebSocket/SSE for live updates (V1: polling acceptable with 30s interval)

---

### 3.3 Panel 2 — Spec Navigator

**Sidebar Icon:** 📄  
**Default Landing For:** Jr Estimator, Sr Estimator  
**Primary Purpose:** Eliminate 2-hour spec hunt → 30-minute structured review

#### Functional Requirements

**FR-SN-001:** Two-column layout
- Left sidebar: 220px (project/document controls)
- Right main area: Section list (full width minus sidebar)

**FR-SN-002:** Left sidebar components
1. **Project selector dropdown** — switch between assigned projects
2. **Document tree** — Spec Manual / Drawings / System Chart (click to switch active document)
3. **Division quick-nav buttons** — Div 22 Plumbing, Div 23 Mechanical/HVAC, Div 40 Process Piping (jump to section without scrolling)
4. **Client SOP callout** — Inline display of active client rules (sourced from SOP Management panel)

**FR-SN-003:** Section list row structure

Each row must display:
- Page range (e.g., "pp. 234-238")
- Section title (e.g., "Ductwork Construction")
- Brief description (extracted from spec)
- Relevance flag (one of four states):

| Flag | Meaning | Visual Treatment |
|------|---------|------------------|
| **Relevant** | HVAC content confirmed | Cyan accent |
| **Client Specific** | Content differs from standard | Orange badge |
| **⚠ Contention** | Conflict detected | Red badge + link to Contention Review |
| **Skip** | Not applicable | NOT DISPLAYED (per Q7 clarification) |

**FR-SN-004:** Irrelevant section handling (Q7 clarification)
- Sections marked "Skip" by Spec Navigator Agent **must not appear** in the UI
- No toggle to show hidden sections — estimators open full PDF separately if needed
- **Critical validation requirement:** Agent must correctly identify relevance (false negatives worse than false positives)

**FR-SN-005:** Spec processing time (Q8 clarification)
- Maximum acceptable wait for 900-page spec: **5 minutes**
- Display progress indicator during processing
- Faster is better, but 5min is acceptable for large projects

**FR-SN-006:** Section-by-section navigation (not keyword search)
- Mirror experienced estimator workflow: read heading, determine relevance, skip irrelevant
- Structured document view, not search results page
- Direct links to page ranges in source PDF

**FR-SN-007:** System Chart upload (Q6 clarification)
- **Upload location:** Within Spec Navigator panel (dedicated upload widget)
- **Validation:** If estimator attempts to mark project "Ready for QC" without uploading System Chart, display blocking prompt
- **File format:** Excel (.xlsx, .xls)
- **Storage:** Link to project in state management layer for QC Agent access

#### Data Sources
- **Agent:** Spec Navigator Agent
- **Input:** Spec manual PDF, project context, client SOPs
- **Processing:** Extract Div 22/23/40 content, flag client-specific deviations, identify contentions
- **Output:** Structured section map with relevance flags

#### Non-Functional Requirements
- **NFR-SN-001:** Section list must render <500ms for 100 sections
- **NFR-SN-002:** PDF page links must open native PDF viewer at correct page
- **NFR-SN-003:** Client SOP callout must update in real-time when SOPs change

---

### 3.4 Panel 3 — Contention Review

**Sidebar Icon:** ⚠️ (with live pending count badge)  
**Default Landing For:** None (reached via sidebar or conflict row click)  
**Primary Purpose:** Human-in-the-loop conflict resolution

#### Functional Requirements

**FR-CR-001:** Contention card structure

Each card must display:
- Contention ID (format: CON-XXXX)
- Source project ID
- Conflict title (concise description)
- Priority badge (Critical / High)
- Three-column comparison (SOP / Drawing / Spec)
- Agent reasoning box (explanation + confidence score)
- Three action buttons (Accept / Override / Escalate)

**FR-CR-002:** Source column precedence hierarchy

**Fixed order (left to right):** SOP → Drawing → Spec
- Recommended source highlighted in cyan with "RECOMMENDED" label
- Each column shows: source label, confidence badge (High/Medium), value text, reference citation
- Agent always recommends highest-precedence source that resolves conflict

**FR-CR-003:** Action button behaviors

| Button | Action | System Response |
|--------|--------|-----------------|
| **Accept** | Estimator confirms agent recommendation | • Record resolution with source reference<br>• Decrement sidebar badge count<br>• Remove card from view<br>• Update project status |
| **Override** | Estimator selects different source | • Prompt for reason (text input required)<br>• Record override with justification<br>• Decrement badge, remove card |
| **Escalate** | Send to senior reviewer or client | • Assign to escalation queue<br>• Log escalation reason<br>• Badge count persists until external resolution |

**FR-CR-004:** Customer clarification tracking (Q9 clarification)
- Escalated contentions requiring contractor clarification happen **outside Holocron**
- **Email attachment capability:** Must support attaching email threads that document spec/scope clarifications
- Attachment linked to contention ID for audit trail

**FR-CR-005:** Additive/deductive options (Q10 clarification)
- "Additive/deductive" refers to **quantity alternates only** (add/remove footage)
- **Cost alternates are OUT OF SCOPE** for V1
- If contention resolution involves alternate quantities, display as separate rows in resolution

**FR-CR-006:** Conflict scope validation
- Only surface contentions within **Div 22/23/40** (HVAC scope)
- No structural, civil, or architectural conflicts
- Sample contentions in mockup (all HVAC-scoped):
  - VAV terminal unit classification (SOP vs Drawing terminology)
  - Duct liner length after equipment (3-way conflict: SOP/Drawing/Spec)
  - Supply main pressure class (Drawing 1" w.g. vs Spec 2" w.g., no SOP)

**FR-CR-007:** Live badge count
- Sidebar icon displays pending contention count
- Updates in real-time as contentions resolved/added
- Badge color: orange (#ffa940)

#### Data Sources
- **Agent:** Contention Detection Agent
- **Input:** Drawings, specs, SOPs, historical patterns
- **Processing:** Detect conflicts, apply precedence rules (SOP > Drawing > Spec), propose resolution
- **Output:** Contention cards with source comparison and recommendation

#### Non-Functional Requirements
- **NFR-CR-001:** Badge count must update <1s after resolution action
- **NFR-CR-002:** Three-column comparison must render side-by-side on screens ≥1200px wide
- **NFR-CR-003:** Agent reasoning must include confidence score (High: ≥85%, Medium: 70-84%, Low: <70%)

---

### 3.5 Panel 4 — QC / Audit Review

**Sidebar Icon:** ✓  
**Default Landing For:** QC Engineer  
**Primary Users:** QC Engineers (India + Pakistan), Sr Estimators with override capability

#### Functional Requirements

**FR-QC-001:** Progress overview section
- **Overall Completeness:** Progress bar + percentage (0-100%)
- Calculation: (completed checklist items / total items) × 100
- Visual: Gradient progress bar (cyan to blue)

**FR-QC-002:** Critical flags section

Display agent-detected discrepancies:
- Flag title (concise issue description)
- Flag description (details with quantities/references)
- Flag location (Division + Drawing/Spec reference)
- Flag severity (Error: red border | Warning: orange border)

**Sample flags (HVAC scope only):**
- VAV box count discrepancy: System Chart 42 units vs QuoteSoft export 39 — 3 units missing
- Duct insulation thickness: System Chart 1" vs MRSM SOP 1.5" requirement

**FR-QC-003:** System checklist (HVAC-scoped items only)

Each checklist item shows:
- Check icon (Complete: green ✓ | Flagged: red ! | Missing: empty border)
- Item label
- Item detail (status or issue description)

**Mandatory checklist items:**
1. Project metadata verified (name, location, client)
2. Spec sections extracted (data point count)
3. Contentions resolved (count cleared)
4. System Chart populated (before QuoteSoft)
5. VAV count match (System Chart vs QuoteSoft export)
6. Client SOP applied (specific rule validation)

**Items explicitly EXCLUDED (per IPR):**
- ❌ Unit rates verified (pricing out of scope)
- ❌ Cost data validated (quantity takeoffs only)
- ❌ Bid accuracy (not priced bids)

**FR-QC-004:** Automated clash detection (Q4 clarification — CRITICAL)

**Minimum QC requirement for ALL projects:**
- **Automated check:** Compare System Chart against QuoteSoft export (.zip containing CSV + JSON)
- **Clash detection scope:**
  - Component count mismatches (e.g., 42 VAV boxes in Chart vs 39 in export)
  - Specification mismatches (e.g., insulation thickness, pressure classes, material types)
  - Missing entries (items in Chart not found in export)
- **Output:** Discrepancy report flagging all conflicts
- **Role independence:** This check runs regardless of estimator seniority

**FR-QC-005:** Tiered QC depth by role (Q4 clarification)

**Jr Takeoff Engineers:**
- Full manual QC review required (in addition to automated checks)
- QC Engineer reviews complex systems in detail

**Sr Takeoff Engineers:**
- Automated clash detection (mandatory minimum)
- Spot-checks for complex systems only (QC Engineer discretion)
- May bypass full manual review (per Q1 clarification — Senior role grants override capability)

**QC Role:**
- Can submit job without additional QC (self-approval capability)

**FR-QC-006:** QC submission workflow
1. Estimator uploads System Chart in Spec Navigator (FR-SN-007)
2. Estimator marks project "Ready for QC" (triggers automated checks)
3. QC Agent runs clash detection against QuoteSoft export
4. QC Engineer reviews:
   - Automated discrepancy report
   - Manual spot-checks (if required by estimator role or complexity)
5. QC Engineer actions:
   - **Approve:** Project moves to Complete stage
   - **Return for Rework:** Project returns to estimator with logged reason (FR-QC-007)

**FR-QC-007:** Rework tracking (Q5 clarification)
- **Action:** "Return for Rework" button in QC panel
- **Required input:** Rework reason (text field, mandatory)
- **System behavior:**
  - Log rework event with reason, timestamp, QC Engineer name
  - Reassign project to original estimator(s)
  - Update project status to "Rework Required"
  - Increment rework counter for Admin/Ops metrics
- **Audit trail:** All rework events stored for metric calculation (15-20% baseline → <10% target)

**FR-QC-008:** QuoteSoft export validation
- **File format:** .zip containing CSV (data) + JSON (annotation alignment)
- **Upload location:** QC panel (dedicated upload widget)
- **Validation scope:**
  - Component counts
  - Pressure classes
  - Material types
  - Insulation specifications
  - Any other System Chart fields with export equivalents

#### Data Sources
- **Agent:** QC & Audit Agent
- **Input:** System Chart (Excel), QuoteSoft export (.zip)
- **Processing:** Compare Chart vs Export, flag discrepancies, assess completeness
- **Output:** Discrepancy report, completeness percentage, checklist status

#### Non-Functional Requirements
- **NFR-QC-001:** Automated clash detection must complete in <2 minutes for projects with 1000+ items
- **NFR-QC-002:** Progress bar must animate on load (smooth fill transition)
- **NFR-QC-003:** Critical flags must sort by severity (Error before Warning)

---

### 3.6 Panel 5 — SOP Management

**Sidebar Icon:** 📚  
**Default Landing For:** Account Manager  
**V1 Scope:** ⚠️ **DEFERRED** (per Q2 clarification — MRSM excluded from V1)

#### Functional Requirements (V2 Scope)

**FR-SM-001:** Two-column layout
- Left nav (200px): Client selector + category list
- Right main area: Rule management interface

**FR-SM-002:** Client SOP compilation
- **V1 reality:** Manual multi-week effort per client (one-time setup)
- **V2 goal:** Automate capture from emails, QC feedback, production notes

**FR-SM-003:** Rule structure

Each SOP rule must include:
- Rule text (full specification)
- Source attribution (email / QC feedback / verbal instruction + date) — **MANDATORY**
- Category tag (Liner Requirements / Pressure Classes / Insulation / Material Preferences / etc.)
- Active/Inactive status

**FR-SM-004:** Pending approvals workflow
- Estimators/QC flag new SOP observations during production
- Rules enter pending queue
- Account Manager reviews → Approve or Reject
- Approved rules activate and surface in Spec Navigator (FR-SN-002, item 4)

**FR-SM-005:** Source attribution validation
- All rules must trace to documented origin
- Prevents undocumented tribal knowledge re-entering system
- If source is "verbal instruction," must include date + person

#### V1 Workaround
- SOPs entered directly into database (manual admin process)
- No UI for rule management in V1
- Estimators consume SOPs via Spec Navigator callout (FR-SN-002)

---

### 3.7 Panel 6 — Admin / Ops

**Sidebar Icon:** 📈  
**Default Landing For:** Manager  
**Primary Purpose:** Production workflow metrics (not platform infrastructure costs)

#### Functional Requirements

**FR-AO-001:** Metric cards (6 total, 2×3 grid)

| Metric | Description | Baseline | V1 Target |
|--------|-------------|----------|-----------|
| **Projects Delivered (MTD)** | Completed projects this month | — | Track actual |
| **Rework Rate** | % of projects requiring correction | 15-20% (validated) | <10% |
| **Avg Spec Review Time** | Time from Deep Dive start to System Chart upload | ~2 hours (validated) | <30 min |
| **Avg Effort Per Project** | Total estimator hours per project | — | Track actual |
| **Contentions This Month** | Total conflicts detected + % auto-resolved | — | Track actual |
| **Active Estimators** | Current team count by role | — | Track actual |

**FR-AO-002:** Metric card structure
- Metric label (uppercase, small font)
- Metric value (large font, Azeret Mono)
- Metric change indicator (green ↑ positive / red ↓ negative / gray neutral)

**FR-AO-003:** Rework rate calculation (Q5 clarification)
- **Numerator:** Count of projects with "Return for Rework" action (FR-QC-007)
- **Denominator:** Total projects delivered in period
- **Formula:** (Rework count / Total delivered) × 100
- **Validation source:** Rehman interview (15-20% baseline)

**FR-AO-004:** Spec review time calculation
- **Start time:** Deep Dive stage begins (estimator first opens Spec Navigator)
- **End time:** System Chart uploaded (FR-SN-007 timestamp)
- **Validation source:** Josh Waldecker interview (~2 hours for large projects)

**FR-AO-005:** Projects by stage (horizontal bar chart)
- Visual distribution of active projects across workflow stages
- Stages: Intake / Deep Dive / Takeoff / QC Review / Complete
- Provides pipeline view without requiring tracker updates

**FR-AO-006:** Recent deliveries list

Each delivery item shows:
- Project name
- Project ID + delivery date
- Tier badge (Small / Medium / Large — based on scope, not cost)
- Delivery status (Billed / Pending)

**FR-AO-007:** Metrics explicitly OUT OF SCOPE (per IPR)
- ❌ Token usage (platform-operator concern, not production manager)
- ❌ Infrastructure cost (AWS costs not relevant to Bimteq workflow)
- ❌ API metrics (SaaS product concern, not internal tool)
- ⚠️ If displayed in mockup, remove from V1 implementation

#### Data Sources
- **Agent:** Project State Agent (observational — tracks all workflow events)
- **Input:** Stage transitions, rework logs, QC outcomes, timestamps
- **Processing:** Aggregate metrics, calculate rates, generate charts
- **Output:** Dashboard cards + visualizations

#### Non-Functional Requirements
- **NFR-AO-001:** Metrics must update every 5 minutes (acceptable lag for management view)
- **NFR-AO-002:** Bar chart must be responsive (scales to panel width)
- **NFR-AO-003:** Recent deliveries limited to last 30 days (performance optimization)

---

---

## 4. Role-Based Access Control

### 4.1 Role Selector (Top Bar)

**FR-RBAC-001:** Role selector dropdown
- Displays current role + dropdown arrow
- Available roles: Jr Estimator / Sr Estimator / QC Engineer / Coordinator / Manager / Account Manager (V2)
- Selecting role changes **default landing panel** (does not restrict access)

**FR-RBAC-002:** Default landing panels

| Role | Default Panel | Rationale |
|------|--------------|-----------|
| Jr Estimator | Spec Navigator | Primary work surface |
| Sr Estimator | Spec Navigator | Primary work surface |
| QC Engineer | QC / Audit Review | Post-takeoff validation |
| Coordinator | Project Dashboard | Assignment + tracking |
| Manager | Admin / Ops | Production metrics |
| Account Manager | SOP Management (V2) | Client rule maintenance |

**FR-RBAC-003:** Sidebar access independence
- **All panels accessible via sidebar regardless of role**
- Role selector sets default, not restrictions
- Design principle: Role-based presentation, not role-based permissions

### 4.2 Role-Specific Capabilities (Q1 clarification)

**FR-RBAC-004:** QC bypass capability
- **QC Engineer role:** Can submit project without additional QC review (self-approval)
- **Sr Estimator role:** Can bypass full QC review (spot-checks only)
- **Complexity override:** System may require full QC for high-complexity projects regardless of role (future enhancement)

**FR-RBAC-005:** Multi-estimator assignment (Q3 clarification)
- **Coordinator action:** Must support assigning 2+ estimators to single project
- **Display:** Project Dashboard shows multiple avatars for team assignments
- **Workflow:** System Chart upload + QC submission allowed by any assigned estimator

### 4.3 Geographic vs. Role-Based Design

**Design Principle (per IPR):**
- **Role-based, not region-based** — No geographic constraints applied
- **Current operating reality:**
  - Pakistan: ~90% Jr Estimators (execution engine)
  - India: ~15 QC Engineers + Senior Management (QC authority for organization)
  - Mexico (MRSM): Standalone team with own QC (India backup)
- **System design:** Assumes any role performable from any region (scaling flexibility)

**FR-RBAC-006:** Location flag removal
- Mockup shows location flags (🇵🇰 🇲🇽 🇮🇳) — **Remove from V1**
- Replace with role tags only (Jr Estimator / Sr Estimator / QC / Coordinator)
- Location may be stored in user profile but not displayed in production UI

---

---

## 5. Workflow Requirements

### 5.1 Stage-Based Workflow (per Bimteq VSM)

**FR-WF-001:** Core workflow stages

**Stage 1: Intake & Prep**
- Coordinator uploads documents to OneDrive
- Intake Agent validates files, organizes folders
- Missing file alerts surface to Coordinator

↓

**Stage 2: Assignment**
- Assignment Agent recommends estimators (ranked list)
- Coordinator confirms assignment (1 or more estimators)
- Project State updates: status → "Assigned"

↓

**Stage 3: Deep Dive (Spec Review)**
- Estimator opens Spec Navigator
- Reviews structured HVAC sections (Div 22/23/40)
- Accesses client SOPs inline
- Resolves contentions (Contention Review panel)
- Populates System Chart (Excel)
- Timer: Tracks spec review duration (for metrics)

↓

**Stage 4: Takeoff (QuoteSoft — Manual)**
- Estimator works in QuoteSoft (UNCHANGED WORKFLOW)
- Manually places components on drawings
- Generates quantity counts
- Exports .zip (CSV + JSON) when complete
- Uploads System Chart + QuoteSoft export to Holocron
- Marks project "Ready for QC"

↓

**Stage 5: QC Review**
- QC Agent runs automated clash detection
- Compares System Chart vs QuoteSoft export
- Flags discrepancies (counts, specs, missing items)
- QC Engineer reviews:
  - Automated discrepancy report (mandatory)
  - Manual spot-checks (role-dependent, FR-QC-005)
- QC Engineer actions:
  - Approve → Stage 6
  - Return for Rework → Back to estimator (FR-QC-007)

↓

**Stage 6: Delivery (OUT OF SCOPE for V1)**
- Manual package creation (highlighted PDFs, etc.)
- Delivered outside Holocron

### 5.2 Workflow Variations (per IPR + Q1 clarification)

**FR-WF-002:** Configurable stage skipping
- **Some teams skip Initial Review entirely** (Scott's estimators)
- **Some teams skip Stage 6** (internal QC packaging)
- **V1 approach:** Manual override by Coordinator (no per-client configuration UI)
- **V2 enhancement:** Per-client workflow configuration

**FR-WF-003:** Stage transition validation
- **System Chart upload required before "Ready for QC"** (FR-SN-007)
- **QuoteSoft export upload required before QC validation** (FR-QC-006)
- **All contentions must be resolved** (no "Pending" conflicts when marking Ready for QC)

### 5.3 Human Checkpoint Enforcement

**FR-WF-004:** Mandatory human approvals (orchestrator enforces)

| Checkpoint | Stage | Human Action | System Behavior |
|------------|-------|--------------|-----------------|
| **Assignment** | 2 | Coordinator confirms estimator(s) | No auto-progression to Deep Dive |
| **Contention Resolution** | 3 | Estimator accepts/overrides/escalates | No auto-resolution; blocks QC if unresolved |
| **System Chart Upload** | 4 | Estimator uploads Excel file | Blocks "Ready for QC" if missing |
| **QC Approval** | 5 | QC Engineer approves or returns | No auto-approval (even for automated checks) |

**FR-WF-005:** No automatic decisions at checkpoints
- Agents recommend; humans decide
- Orchestrator pauses circuit at checkpoints until human action recorded
- Exception: Observational agents (Project State) run continuously without requiring input

---

---

## 6. Agent Specifications

### 6.1 Agent 1: Intake & Prep Agent

**Type:** Active  
**Trigger:** Coordinator uploads files to OneDrive project folder  
**Human Checkpoint:** Coordinator reviews validation report

#### Inputs
- Raw uploads: PDFs (drawings, specs), Excel (schedules), OneDrive folder path
- Project metadata: Name, client, expected document types

#### Processing Logic
1. **File validation:**
   - Check for required document types (spec manual, mechanical drawings, system schedules)
   - Validate file formats (PDF, Excel)
   - Flag missing files (e.g., "Mechanical drawings missing")
2. **Folder organization:**
   - Create standardized folder structure: `/Drawings`, `/Specifications`, `/Schedules`, `/SystemChart`
   - Move uploaded files to appropriate folders
   - Rename files per convention (e.g., `PROJ-0089_SpecManual.pdf`)
3. **Version control:**
   - Detect duplicate filenames
   - Flag potential version conflicts (e.g., "Spec Manual uploaded twice — confirm latest version")

#### Outputs
- **Organized folder structure** (OneDrive)
- **Validation report:**
  - ✓ All required files present
  - ⚠️ Missing files list
  - ⚠️ Version conflicts detected
- **Project entry created** in state management layer

#### Success Criteria
- All uploaded files moved to correct folders
- Missing file alerts visible to Coordinator in Project Dashboard
- Project becomes visible in dashboard with "Intake" stage badge (if not skipped)

---

### 6.2 Agent 2: Spec Navigator Agent

**Type:** Assistive  
**Trigger:** Estimator opens Spec Navigator panel  
**Human Checkpoint:** Estimator reviews sections and populates System Chart

#### Inputs
- Spec manual PDF (raw document)
- Project context: Client name, project type, region
- Client SOPs (from SOP Management database)
- Historical patterns (optional V2 enhancement)

#### Processing Logic
1. **Content extraction:**
   - Parse PDF, extract text by page
   - Identify section headings (CSI MasterFormat structure)
   - Isolate Divisions 22 (Plumbing), 23 (Mechanical/HVAC), 40 (Process Piping)
2. **Relevance classification:**
   - For each section, determine: Relevant / Client Specific / Skip
   - **Relevant:** Contains HVAC-applicable content (ductwork, piping, insulation, equipment specs)
   - **Client Specific:** Content differs from standard practices or mentions client-specific requirements
   - **Skip:** Outside HVAC scope (structural, civil, architectural)
3. **Contention detection integration:**
   - Cross-reference spec sections with drawings and SOPs
   - Flag sections with conflicts → mark as "⚠ Contention" and link to Contention Review
4. **SOP matching:**
   - Compare extracted content against client SOP rules
   - Surface applicable SOPs in left sidebar callout (FR-SN-002, item 4)

#### Outputs
- **Structured section map:**
  - Page ranges (e.g., "pp. 234-238")
  - Section titles (e.g., "Ductwork Accessories")
  - Descriptions (brief excerpt from spec)
  - Relevance flags (Relevant / Client Specific / Contention / Skip)
- **Division quick-nav data:** Section IDs for Div 22/23/40 buttons
- **Client SOP callout content:** Active rules for this project

#### Performance Requirements (Q8 clarification)
- **900-page spec:** ≤5 minutes processing time (acceptable)
- **Progress indicator:** Display % complete during processing
- **Incremental display:** Show sections as they're processed (don't wait for full completion)

#### Critical Validation (Q7 clarification)
- **False negative cost >> False positive cost**
- If uncertain whether section is relevant, mark as "Relevant" (let estimator decide)
- "Skip" classification must be high-confidence (≥90%) — missing HVAC content is worse than showing extra sections

---

### 6.3 Agent 3: Contention Detection Agent

**Type:** Agentic  
**Trigger:** Continuous during Deep Dive stage; surfaces conflicts as detected  
**Human Checkpoint:** Estimator resolves each contention (Accept / Override / Escalate)

#### Inputs
- Drawings (PDF — annotations, notes, schedules)
- Specifications (parsed sections from Spec Navigator Agent)
- Client SOPs (from SOP Management database)
- Precedence hierarchy: **SOP > Drawing > Spec** (Bimteq rule)

#### Processing Logic
1. **Conflict detection:**
   - Compare same specification across three sources (e.g., duct liner length)
   - Identify discrepancies:
     - **Direct contradiction:** Drawing says 10 ft, Spec says 15 ft
     - **Missing specification:** Drawing silent, Spec defines, SOP defines differently
     - **Terminology mismatch:** Same item called different names (e.g., "VAV box" vs "Supply Air Unit")
2. **Precedence application:**
   - Rank sources by Bimteq hierarchy: SOP (highest) → Drawing → Spec (lowest)
   - Recommend highest-precedence source that resolves conflict
   - **If no SOP:** Drawing overrides Spec
   - **If conflict within same source:** Escalate (cannot auto-resolve)
3. **Confidence scoring:**
   - High confidence (≥85%): Clear precedence rule applies, sources unambiguous
   - Medium confidence (70-84%): Terminology interpretation required, or partial information
   - Low confidence (<70%): Complex conflict, missing context, or three-way disagreement
4. **Conservative bias (per IPR):**
   - When uncertain, default to **higher-cost assumption** (safer for client)
   - Only recommend lower-cost option if clearly justified
   - Flag high-risk contentions for escalation (e.g., structural impact, client relationship risk)

#### Outputs
- **Contention card** (displayed in Contention Review panel):
  - Contention ID (format: CON-XXXX, sequential)
  - Source project ID
  - Conflict title (e.g., "Duct Liner Length After Equipment")
  - Priority badge (Critical / High)
  - Three-column comparison:
    - SOP column: Rule text + source reference (or "No SOP on file")
    - Drawing column: Value + drawing reference (e.g., "Dwg M-401, Note 3")
    - Spec column: Value + spec section (e.g., "Spec 23 31 00, Para 2.3.A")
  - Recommended source: Highlighted in cyan with confidence score
  - Agent reasoning: Text explanation (2-3 sentences)

#### Sample Contentions (HVAC-scoped)
1. **CON-0027:** VAV terminal unit classification — SOP vs Drawing terminology mismatch
2. **CON-0028:** Duct liner length after equipment — SOP 15 ft (VAV only), Drawing 15 ft (all), Spec 10 ft (all)
3. **CON-0029:** Supply main pressure class — Drawing 1" w.g. vs Spec minimum 2" w.g., no SOP

#### Scope Boundaries
- **IN SCOPE:** Divisions 22, 23, 40 only (Plumbing, Mechanical/HVAC, Process Piping)
- **OUT OF SCOPE:** Structural, civil, architectural, electrical, pricing/cost conflicts

#### Integration Points
- **Spec Navigator:** Contentions link back to source spec sections (⚠ flag)
- **Project Dashboard:** Projects with unresolved contentions show "N Conflicts" status chip
- **QC Agent:** Unresolved contentions block "Ready for QC" transition

---

### 6.4 Agent 4: Assignment Recommendation Agent

**Type:** Assistive  
**Trigger:** Coordinator initiates assignment workflow (Project Dashboard action)  
**Human Checkpoint:** Coordinator confirms final assignment

#### Inputs
- **Estimator profiles:**
  - Role (Jr Estimator / Sr Estimator / QC)
  - Skills (e.g., healthcare projects, complex piping, duct fabrication)
  - Familiarity (clients previously worked, project types)
  - Availability (current workload, hours allocated this week)
- **Project requirements:**
  - Client name
  - Project type (office, healthcare, industrial, mixed-use)
  - Estimated effort (hours)
  - Complexity flags (large scope, unfamiliar client, tight deadline)
- **Current workload:**
  - Active projects per estimator
  - Total hours allocated
  - Upcoming deadlines

#### Processing Logic
1. **Skill matching:**
   - Match project type to estimator experience
   - Prioritize estimators with prior client familiarity (faster Deep Dive)
   - Consider role: Jr Estimators for standard projects, Sr Estimators for complex/high-risk
2. **Workload balancing:**
   - Calculate available capacity per estimator (40 hrs/week - current allocation)
   - Avoid overloading individuals during peak workloads
   - Distribute work to prevent bottlenecks
3. **Team assignment logic (Q3 clarification):**
   - For large projects (>1000 items) or complex scope, recommend **2+ estimators**
   - Split work by division (e.g., Estimator A: Div 22/23, Estimator B: Div 40)
   - Assign Sr Estimator as lead + Jr Estimator as support
4. **Ranking algorithm:**
   - Score each estimator: (Skill match × 0.4) + (Client familiarity × 0.3) + (Availability × 0.3)
   - Sort by score descending
   - Return top 3-5 recommendations

#### Outputs
- **Ranked recommendation list:**
  - Estimator name + role
  - Match score (0-100)
  - Reasoning (e.g., "Previously worked 3 MRSM projects, 12 hrs available this week")
  - Current workload indicator (e.g., "3 active projects, 28 hrs allocated")
- **Team assignment flag:** If large/complex, suggest multi-estimator approach

#### Display (Project Dashboard — Future Enhancement)
- Coordinator clicks "Assign" button on project row
- Modal displays ranked recommendations
- Coordinator selects 1+ estimators, confirms assignment
- **V1 Simplification:** Manual assignment without agent recommendations (agent runs but UI not built)

---

### 6.5 Agent 5: Project State Agent

**Type:** Observational  
**Trigger:** Continuous (listens to all workflow events)  
**Human Checkpoint:** None (no human action required)

#### Inputs
- **Workflow events:**
  - Project created (Intake Agent completes)
  - Estimator assigned (Coordinator action)
  - Stage transitions (Deep Dive started, Takeoff started, QC submitted, etc.)
  - Contention resolved (Estimator action in Contention Review)
  - QC outcome (Approved / Returned for Rework)
  - Timestamps for all actions

#### Processing Logic
1. **State tracking:**
   - Maintain current stage per project (Intake / Deep Dive / Takeoff / QC Review / Complete)
   - Track status per project (In Progress / N Conflicts / Rework Required / Complete)
   - Log all state changes with timestamps
2. **Real-time updates:**
   - Push stage/status changes to Project Dashboard (WebSocket/SSE in production, polling in V1)
   - Update badge counts (Contention Review pending, active projects)
3. **Metric aggregation:**
   - Calculate metrics for Admin/Ops panel:
     - Projects delivered (MTD)
     - Avg effort per project (sum of logged hours / project count)
     - Contentions this month (total detected + % auto-resolved)
     - Active estimators by role
   - Track rework events for rework rate calculation (FR-AO-003)
4. **Timeline construction:**
   - Build audit trail per project: all actions, who performed them, when
   - Useful for post-project analysis and dispute resolution

#### Outputs
- **Project status updates:**
  - Current stage badge (Project Dashboard)
  - Current status chip (In Progress / N Conflicts / etc.)
  - Assigned estimator(s) display
  - Effort logged (hours)
- **Dashboard metrics:**
  - Real-time counts (active projects, pending contentions)
  - Aggregated metrics (Admin/Ops panel, FR-AO-001)
- **Audit trail:**
  - Complete project history (for QC review, management analysis)

#### Non-Functional Requirements
- **Real-time latency:** <3 seconds from event to dashboard update (V1: 30s polling acceptable)
- **Scalability:** Handle 50+ concurrent projects without performance degradation
- **Data retention:** Store audit trail indefinitely (or per compliance requirements)

---

### 6.6 Agent 6: QC & Audit Agent

**Type:** Hybrid (automated + human-supervised)  
**Trigger:** Estimator marks project "Ready for QC" (uploads System Chart + QuoteSoft export)  
**Human Checkpoint:** QC Engineer reviews discrepancy report and approves/rejects

#### Inputs
- **System Chart** (Excel file uploaded in Spec Navigator, FR-SN-007)
- **QuoteSoft export** (.zip containing CSV data + JSON annotation alignment)
- **Client SOPs** (for spec compliance validation)
- **Project contentions** (resolved conflicts from Contention Review)

#### Processing Logic

**Phase 1: Automated Clash Detection (Q4 clarification — MANDATORY FOR ALL PROJECTS)**

1. **Component count validation:**
   - Parse System Chart: Extract counts by component type (e.g., VAV boxes, duct runs, diffusers)
   - Parse QuoteSoft CSV: Extract counts by same component types
   - **Clash check:** For each component type:
     - If counts match → ✓ Pass
     - If counts differ → ⚠ Flag discrepancy (e.g., "VAV boxes: Chart 42, Export 39 — 3 missing")

2. **Specification validation:**
   - Compare System Chart specifications vs QuoteSoft item properties:
     - **Pressure class:** Chart says 2" w.g., export items tagged 1" w.g. → Flag
     - **Material type:** Chart says galvanized steel, export says aluminum → Flag
     - **Insulation thickness:** Chart says 1.5", export says 1" → Flag
     - **Duct size ranges:** Chart specifies 8"-12", export includes 14" duct → Flag

3. **Missing entry detection:**
   - For each System Chart row, search for corresponding export item
   - If not found → Flag as "Missing from takeoff" (e.g., "AHU-03 listed in Chart but not in export")

4. **SOP compliance check:**
   - Cross-reference System Chart specs against client SOPs
   - Flag violations (e.g., "MRSM SOP requires 1.5" insulation, Chart shows 1"")

**Phase 2: Completeness Assessment**

5. **Checklist validation (FR-QC-003):**
   - Project metadata verified (name, location, client match across documents)
   - Spec sections extracted (Spec Navigator ran successfully)
   - Contentions resolved (zero pending conflicts)
   - System Chart populated (file uploaded, non-empty)
   - VAV count match (specific check from #1 above)
   - Client SOP applied (from #4 above)

6. **Progress calculation:**
   - Overall completeness = (passed checklist items / total items) × 100

#### Outputs

**Automated Outputs (shown in QC / Audit Review panel):**

1. **Overall Completeness:**
   - Percentage (0-100%)
   - Progress bar visual (FR-QC-001)

2. **Critical Flags:**
   - List of discrepancies with severity (Error / Warning)
   - Each flag shows:
     - Title (e.g., "VAV Box Count Mismatch")
     - Description (e.g., "System Chart: 42 units, QuoteSoft export: 39 units — 3 missing")
     - Location (e.g., "Division 23 | Drawing M-401")

3. **System Checklist:**
   - 6 items with status (Complete ✓ / Flagged ! / Missing ○)
   - Detail text for each item

**Human QC Review (FR-QC-005 — Tiered by Role):**

- **Jr Takeoff Engineers:**
  - QC Engineer performs full manual review (in addition to automated checks)
  - Reviews complex systems in detail
  - Validates assumptions, checks drawing interpretation

- **Sr Takeoff Engineers:**
  - Automated checks (mandatory minimum)
  - Spot-checks only for complex systems (QC Engineer discretion)
  - May bypass full manual review (Q1 clarification)

- **QC Role:**
  - Self-approval capability (can submit without additional review)

#### QC Actions (FR-QC-006)

**Action: Approve**
- Project moves to Complete stage
- Deliverable ready for packaging (Stage 6, outside Holocron)

**Action: Return for Rework (FR-QC-007)**
- QC Engineer clicks "Return for Rework" button
- **Required input:** Rework reason (text field, mandatory)
- **System behavior:**
  - Log rework event: reason, timestamp, QC Engineer name
  - Reassign to original estimator(s)
  - Update project status to "Rework Required"
  - Increment rework counter (for Admin/Ops metrics)
  - Send notification to estimator(s)

#### Scope Boundaries
- **IN SCOPE:** Quantity validation, spec compliance, completeness checks
- **OUT OF SCOPE:** Pricing validation (no unit rates, no cost data — per IPR)
- **HVAC-only:** Validation limited to Div 22/23/40 items

#### Non-Functional Requirements
- **Processing time:** <2 minutes for projects with 1000+ items (NFR-QC-001)
- **Accuracy:** ≥95% detection rate for count discrepancies (false negatives are critical failures)
- **Reporting:** Discrepancy report must be exportable (PDF or Excel for audit trail)

---

## 7. Integration Requirements

### 7.1 OneDrive Integration

**INT-001:** File upload monitoring
- Monitor designated project folders for new uploads
- Trigger Intake Agent on file addition
- Support concurrent uploads (multiple projects)

**INT-002:** Folder structure management
- Create standardized folder structure per project
- Move files to appropriate subfolders
- Maintain folder permissions (team access)

**INT-003:** Version control
- Detect duplicate filenames
- Store file upload timestamps
- Flag version conflicts for Coordinator review

### 7.2 QuoteSoft Integration

**INT-004:** Export file format
- **Input format:** .zip containing:
  - CSV file (quantity data: component type, count, dimensions, location)
  - JSON file (annotation alignment: coordinates, zone references)
- **Parsing requirements:**
  - Extract component counts by type
  - Map CSV rows to System Chart entries (by component name/type)
  - Handle non-standard naming conventions (fuzzy matching)

**INT-005:** Zone structure understanding
- QuoteSoft zone = one drawing page at one scale
- Each zone has background image (PDF or .tif)
- Zones may be split for pricing breakouts (same drawing, different cost centers)
- **V1 scope:** Validate quantities only (not zone structure)

**INT-006:** Deep integration (V2 scope — OUT OF SCOPE for V1)
- Pre-populate QuoteSoft data from System Chart
- Understand annotation format (JSON structure)
- Potential challenges: Proprietary format, limited API access

### 7.3 Microsoft Teams Integration (Future Enhancement)

**INT-007:** Notifications (V2 scope)
- Send Teams message on project assignment
- Send Teams message on QC completion / rework
- Send Teams message on contention escalation

**INT-008:** Status updates (V2 scope)
- Post daily summary to team channel (projects completed, pending QC)
- Replace manual morning call check-ins

### 7.4 Email Integration (V2 scope)

**INT-009:** SOP capture from emails
- Parse email threads for client instructions
- Extract SOPs automatically (NLP-based)
- Route to SOP Management pending approvals

**INT-010:** Contention clarification attachment (Q9 clarification — V1)
- Allow attaching email threads to escalated contentions
- Store email as PDF or .eml format
- Link to contention ID for audit trail

---
---

## 8. Data Model (Simplified)

### 8.1 Core Entities

**Project**
{
  "project_id": "PRJ-2025-0089",
  "name": "Toronto Office Tower",
  "client": "ABC Mechanical",
  "stage": "Takeoff",
  "status": "3 Conflicts",
  "estimated_effort_hours": 18.5,
  "assigned_estimators": ["user_ahmed_k", "user_maria_h"],
  "created_at": "2025-02-01T08:00:00Z",
  "deep_dive_start": "2025-02-01T09:15:00Z",
  "system_chart_uploaded_at": null,
  "qc_submitted_at": null,
  "completed_at": null
}

**User**
{
  "user_id": "user_ahmed_k",
  "name": "Ahmed K.",
  "role": "Jr Estimator",
  "region": "Pakistan",
  "skills": ["healthcare", "office", "duct_fabrication"],
  "clients_worked": ["ABC Mechanical", "XYZ Contractors"],
  "current_workload_hours": 28,
  "active_projects": ["PRJ-2025-0089", "PRJ-2025-0085"]
}

**Contention**
{
  "contention_id": "CON-0027",
  "project_id": "PRJ-2025-0089",
  "title": "VAV Terminal Unit Classification",
  "priority": "Critical",
  "sources": {
    "sop": {"value": "Supply Air Units", "reference": "MRSM SOP 2024", "confidence": "High"},
    "drawing": {"value": "VAV Boxes", "reference": "Dwg M-401", "confidence": "High"},
    "spec": {"value": "Variable Volume Terminal Units", "reference": "Spec 23 36 00", "confidence": "High"}
  },
  "recommended_source": "sop",
  "reasoning": "SOP terminology takes precedence per Bimteq hierarchy. Client uses 'Supply Air Units' consistently across MRSM projects.",
  "status": "Pending",
  "resolved_by": null,
  "resolved_at": null,
  "resolution_action": null
}

**System Chart Entry** (simplified)
{
  "project_id": "PRJ-2025-0089",
  "chart_uploaded_at": "2025-02-05T14:30:00Z",
  "entries": [
    {
      "component_type": "VAV Box",
      "count": 42,
      "pressure_class": "2 in w.g.",
      "material": "Galvanized Steel",
      "insulation_thickness": "1.5 in",
      "notes": "Per MRSM SOP, 15 ft liner after VAV"
    }
  ]
}

**QC Result**
{
  "project_id": "PRJ-2025-0089",
  "qc_submitted_at": "2025-02-06T10:00:00Z",
  "qc_engineer": "user_qc_india_1",
  "automated_checks": {
    "completeness_pct": 87,
    "flags": [
      {"severity": "Error", "title": "VAV Box Count Mismatch", "description": "Chart: 42, Export: 39"},
      {"severity": "Warning", "title": "Insulation Thickness", "description": "Chart: 1 in, SOP: 1.5 in"}
    ],
    "checklist": [
      {"item": "Project metadata verified", "status": "Complete"},
      {"item": "Spec sections extracted", "status": "Complete"},
      {"item": "Quantities complete", "status": "Flagged"}
    ]
  },
  "qc_outcome": "Returned for Rework",
  "rework_reason": "Missing 3 VAV boxes in takeoff — verify Drawing M-401 zones 3-4",
  "qc_approved_at": null
}

---

## 9. Non-Functional Requirements

### 9.1 Performance

**NFR-PERF-001:** Page load time
- Dashboard panels must render in <2 seconds on standard broadband (10 Mbps)
- Spec Navigator section list must render in <500ms for 100 sections

**NFR-PERF-002:** Real-time updates
- Project status changes visible in <3 seconds (production target; 30s polling acceptable for V1)
- Contention badge count updates in <1 second after resolution

**NFR-PERF-003:** Spec processing
- 900-page spec: ≤5 minutes (Q8 clarification)
- Display progress indicator during processing
- Incremental display (show sections as processed, don't block UI)

**NFR-PERF-004:** QC clash detection
- 1000+ item project: ≤2 minutes (NFR-QC-001)
- Must scale to projects with 5000+ items (future-proofing)

### 9.2 Usability

**NFR-USE-001:** Estimator workflow efficiency
- Spec review time: Target <30 minutes (from 2-hour baseline)
- Zero training required for Spec Navigator (intuitive section navigation)
- Contention cards readable without scrolling (fit in viewport on 1200px+ screens)

**NFR-USE-002:** Error handling
- Graceful degradation when agents fail (show last known state + error message)
- If Spec Navigator fails, provide link to raw PDF
- If QC Agent fails, allow manual QC submission

**NFR-USE-003:** Accessibility
- WCAG 2.1 AA compliance (color contrast, keyboard navigation)
- Screen reader support for panel navigation
- Focus indicators on interactive elements

### 9.3 Reliability

**NFR-REL-001:** Availability
- 99.5% uptime during business hours (7 AM - 7 PM Pakistan time, Mon-Fri)
- Planned maintenance windows announced 48 hours in advance

**NFR-REL-002:** Data integrity
- Zero data loss on project state transitions
- Audit trail for all human actions (who, what, when)
- Automatic backups every 6 hours

**NFR-REL-003:** Agent failure handling
- Orchestrator retries failed agents (max 3 attempts with exponential backoff)
- If agent fails after retries, pause circuit and alert operations team
- Display failure reason to user (not generic "something went wrong")

### 9.4 Security

**NFR-SEC-001:** Authentication
- Single Sign-On (SSO) via Microsoft 365 (Bimteq uses Teams/OneDrive)
- Role assignment stored in user profile
- No passwords stored in Holocron (delegate to Microsoft identity provider)

**NFR-SEC-002:** Authorization
- Role-based access control (RBAC) per FR-RBAC-* requirements
- Audit log for sensitive actions (QC approval, rework assignment, SOP changes)

**NFR-SEC-003:** Data protection
- Encrypt data at rest (AWS S3 encryption)
- Encrypt data in transit (TLS 1.3)
- Client project data isolated (no cross-client data leakage)

### 9.5 Scalability

**NFR-SCALE-001:** Concurrent users
- Support 50+ concurrent users (Pakistan + India + Mexico teams)
- Handle 20+ simultaneous Spec Navigator sessions (different projects)

**NFR-SCALE-002:** Project volume
- Support 100+ active projects simultaneously
- Handle 500+ projects per month (future growth)

**NFR-SCALE-003:** Document size
- Spec PDFs: Up to 1000 pages (current max observed: 900)
- Drawing PDFs: Up to 200 pages per project
- QuoteSoft exports: Up to 10,000 line items per project

---

---

## 10. Validation & Testing Requirements

### 10.1 Agent Validation

**VAL-001:** Spec Navigator accuracy
- **Test corpus:** 20 real spec manuals (mix of small/large, simple/complex)
- **Success criteria:**
  - ≥95% of HVAC sections flagged as "Relevant" (no false negatives)
  - ≤10% of non-HVAC sections flagged as "Relevant" (minimize false positives)
  - Zero Division 22/23/40 sections marked "Skip" (false negatives are critical failures)
- **Validation method:** Manual review by Sr Estimators (Josh Waldecker, Scott)

**VAL-002:** Contention Detection accuracy
- **Test corpus:** 10 projects with known conflicts (documented in QC feedback)
- **Success criteria:**
  - ≥90% of known conflicts detected (recall)
  - ≤15% false positive rate (precision acceptable — estimators can dismiss)
  - All Critical priority contentions detected (zero false negatives for high-impact conflicts)
- **Validation method:** Blind test (agent runs on historical projects, results compared to actual QC findings)

**VAL-003:** QC clash detection accuracy
- **Test corpus:** 15 completed projects (System Chart + QuoteSoft export pairs)
- **Success criteria:**
  - ≥95% of count discrepancies detected (e.g., 42 vs 39 VAV boxes)
  - ≥90% of spec mismatches detected (e.g., pressure class, insulation thickness)
  - Zero false negatives for count mismatches >5% (e.g., missing 3 of 42 items = 7% error must be caught)
- **Validation method:** Compare agent output to manual QC review (India QC team ground truth)

### 10.2 Workflow Validation

**VAL-004:** End-to-end workflow test
- **Scenario:** Upload → Assign → Spec Review → Resolve Contention → Upload System Chart → QC Review → Approve
- **Success criteria:**
  - All stage transitions occur correctly
  - Project Dashboard updates in real-time (or <30s for V1 polling)
  - No data loss during transitions
  - Audit trail complete (all actions logged)

**VAL-005:** Rework workflow test
- **Scenario:** QC Engineer returns project for correction
- **Success criteria:**
  - Project reassigned to original estimator(s)
  - Rework reason displayed in Spec Navigator or Project Dashboard
  - Rework event logged for Admin/Ops metrics
  - Estimator can resubmit after corrections

**VAL-006:** Multi-estimator workflow test (Q3 clarification)
- **Scenario:** Assign 2 estimators to large project
- **Success criteria:**
  - Both estimators see project in their assigned list
  - Either estimator can upload System Chart
  - Project Dashboard shows both avatars
  - QC submission allowed by either estimator

### 10.3 Performance Validation

**VAL-007:** Spec processing time
- **Test:** 900-page spec manual (real MRSM project)
- **Success criteria:** Completes in ≤5 minutes (Q8 clarification)
- **Measurement:** Time from upload to section list display

**VAL-008:** QC clash detection time
- **Test:** Project with 1000+ items (typical large project)
- **Success criteria:** Completes in ≤2 minutes (NFR-QC-001)
- **Measurement:** Time from "Ready for QC" button click to discrepancy report display

**VAL-009:** Dashboard responsiveness
- **Test:** 50 concurrent users, 100 active projects
- **Success criteria:** Panel load time <2 seconds (NFR-PERF-001)
- **Measurement:** Time to first contentful paint (FCP)

### 10.4 Usability Validation (Stakeholder Testing)

**VAL-010:** Jr Estimator usability test
- **Participants:** 3 Jr Estimators (Pakistan team)
- **Task:** Complete Deep Dive on unfamiliar client project using Spec Navigator
- **Success criteria:**
  - Task completion time <30 minutes (vs 2-hour baseline)
  - Zero questions asked about how to navigate interface
  - Post-task satisfaction score ≥4/5 ("would use this instead of manual spec reading")

**VAL-011:** QC Engineer usability test
- **Participants:** 2 QC Engineers (India team)
- **Task:** Review automated discrepancy report, approve or return for rework
- **Success criteria:**
  - QC review time <10 minutes (vs 20-minute manual review baseline)
  - Zero critical discrepancies missed (validated by second QC engineer)
  - Post-task satisfaction score ≥4/5

**VAL-012:** Coordinator usability test
- **Participants:** 1 Coordinator (Pakistan or India)
- **Task:** Review Project Dashboard, check project statuses, identify bottlenecks
- **Success criteria:**
  - Task completion time <5 minutes (vs 15-minute manual tracker check)
  - Correctly identifies which projects have conflicts (no missed flags)
  - Post-task satisfaction score ≥4/5

---

## 11. Success Metrics (V1 Goals)

### 11.1 Primary Metrics (Validated Baselines from IPR)

| Metric | Baseline | V1 Target | Measurement Method |
|--------|----------|-----------|-------------------|
| **Rework Rate** | 15-20% (validated) | <10% | (Rework count / Total delivered) × 100 |
| **Avg Spec Review Time** | ~2 hours (validated) | <30 min | Deep Dive start → System Chart upload |
| **Projects Delivered per Month** | Baseline TBD | +20% | Count of projects reaching Complete stage |
| **Contention Detection Recall** | N/A (manual today) | ≥90% | Known conflicts detected / Total conflicts |
| **QC Clash Detection Accuracy** | N/A (manual today) | ≥95% | Count discrepancies detected / Total discrepancies |

### 11.2 Adoption Metrics

| Metric | V1 Target | Measurement Method |
|--------|-----------|-------------------|
| **Estimator Adoption Rate** | ≥80% of team uses Spec Navigator weekly | Active users / Total estimators |
| **System Chart Upload Compliance** | ≥90% of projects have uploaded Chart before QC | Projects with Chart / Total projects in QC stage |
| **Contention Resolution Rate** | ≥85% resolved without escalation | Accepted+Overridden / Total contentions |

### 11.3 User Satisfaction Metrics

| Metric | V1 Target | Measurement Method |
|--------|-----------|-------------------|
| **Estimator Satisfaction** | ≥4/5 (post-task survey) | Likert scale: "Holocron makes spec review faster/easier" |
| **QC Satisfaction** | ≥4/5 (post-task survey) | Likert scale: "Automated checks reduce manual QC time" |
| **Coordinator Satisfaction** | ≥4/5 (post-task survey) | Likert scale: "Project Dashboard improves status visibility" |

### 11.4 Failure Metrics (Monitor for Issues)

| Metric | Acceptable Threshold | Action if Exceeded |
|--------|---------------------|-------------------|
| **Spec Navigator False Negatives** | ≤5% of HVAC sections missed | Retrain Spec Navigator Agent |
| **QC Clash Detection False Negatives** | ≤5% of count discrepancies missed | Retrain QC Agent, add validation rules |
| **Agent Failure Rate** | ≤2% of agent runs fail | Investigate orchestrator logs, improve error handling |
| **User Error Rate** | ≤10% of users report confusion/errors | Conduct usability testing, improve UI clarity |

---

---

## 12. Implementation Phases

### Phase 1: Foundation (Weeks 1-4)
**Deliverables:**
- Orchestrator engine (circuit management, handoffs)
- Agent registry (define contracts for 6 agents)
- State management layer (project status, artifacts)
- Basic UI framework (top bar, sidebar, panel container)
- Authentication (Microsoft SSO integration)

### Phase 2: Core Agents (Weeks 5-10)
**Deliverables:**
- Agent 1: Intake & Prep (OneDrive monitoring, validation)
- Agent 2: Spec Navigator (PDF parsing, section extraction, relevance classification)
- Agent 5: Project State (event tracking, dashboard updates)
- Panel 1: Project Dashboard (basic table, no assignment UI yet)
- Panel 2: Spec Navigator (section list, division nav, SOP callout — no upload yet)

### Phase 3: Conflict & QC (Weeks 11-16)
**Deliverables:**
- Agent 3: Contention Detection (conflict identification, precedence rules)
- Agent 6: QC & Audit (clash detection, discrepancy reporting)
- Panel 3: Contention Review (three-column cards, action buttons)
- Panel 4: QC / Audit Review (flags, checklist, rework workflow)
- System Chart upload (Spec Navigator integration, FR-SN-007)
- QuoteSoft export upload (QC panel integration, FR-QC-006)

### Phase 4: Metrics & Polish (Weeks 17-20)
**Deliverables:**
- Agent 4: Assignment Recommendation (skill matching, workload balancing — backend only, no UI)
- Panel 6: Admin / Ops (metric cards, recent deliveries, projects by stage chart)
- Real-time updates (WebSocket/SSE for Project Dashboard badge counts)
- Email attachment capability (contention escalation, FR-CR-004)
- Multi-estimator assignment support (Project Dashboard + workflow, FR-RBAC-005)

### Phase 5: Validation & Rollout (Weeks 21-24)
**Deliverables:**
- Agent accuracy validation (VAL-001, VAL-002, VAL-003)
- End-to-end workflow testing (VAL-004, VAL-005, VAL-006)
- Performance testing (VAL-007, VAL-008, VAL-009)
- Usability testing with stakeholders (VAL-010, VAL-011, VAL-012)
- Bug fixes + UI polish
- **Pilot rollout:** 5 projects with Pakistan Jr Estimators + India QC
- **Full rollout:** All estimators (Pakistan + India), excluding MRSM (Q2 clarification)

---

## 13. Out of Scope (V2 Enhancements)

### Deferred to V2
- **MRSM integration:** Basis Board workflow (Q2 clarification)
- **SOP Management UI:** Panel 5 (Account Manager workflow)
- **Automated SOP capture:** Extract rules from emails and QC feedback
- **Assignment Recommendation UI:** Modal in Project Dashboard (agent runs, but no UI)
- **Highlighted PDF tooling:** Automated creation (V1: manual upload only, Q11 clarification)
- **QuoteSoft deep integration:** Pre-populate takeoff data, understand annotation format
- **Teams notifications:** Automated messages on assignment, QC completion, escalation
- **Mobile interface:** Responsive design for tablets/phones (desktop-only for V1)
- **Cost alternates:** Pricing functionality explicitly out of scope (Q10 clarification)
- **Workflow configuration UI:** Per-client stage skipping (V1: manual override only)
- **Historical pattern analysis:** Agent learning from past projects (V1: rule-based only)

### Explicitly NOT Planned (Ever)
- **Automated takeoff:** QuoteSoft workflow remains manual (Holocron assists before/after, not during)
- **Pricing system:** Bimteq produces quantity takeoffs, not priced bids (IPR scope boundary)
- **Project management:** Post-delivery tracking (Holocron is estimation-focused)
- **Client-facing features:** Holocron is internal tool only (Argus is the external SaaS product)

---

---

## 14. Risks & Mitigations

### Risk 1: Spec Navigator False Negatives (HIGH IMPACT)
**Description:** Agent incorrectly marks HVAC section as "Skip," estimator misses critical content  
**Impact:** Missing specifications lead to takeoff errors, client rejects deliverable  
**Mitigation:**
- Conservative classification: Mark uncertain sections as "Relevant" (Q7 clarification)
- Validation testing with 20 real spec manuals (VAL-001)
- Confidence threshold: Only mark "Skip" if ≥90% confidence
- Fallback: Estimators can always open full PDF manually

### Risk 2: QuoteSoft Export Format Changes (MEDIUM IMPACT)
**Description:** QuoteSoft updates export format, breaks QC clash detection  
**Impact:** Automated checks fail, QC Engineer forced to manual validation  
**Mitigation:**
- Version detection: Check export file headers for format version
- Flexible parsing: Handle minor format variations (fuzzy matching)
- Manual fallback: QC panel still functional without automated checks
- Monitoring: Alert if parsing fails for >10% of exports

### Risk 3: Estimator Workflow Resistance (MEDIUM IMPACT)
**Description:** Estimators continue using manual spec reading, bypass Holocron  
**Impact:** Low adoption, no productivity gains, metrics don't improve  
**Mitigation:**
- Stakeholder testing during Phase 5 (VAL-010, VAL-011, VAL-012)
- Pilot rollout with 5 projects (gather feedback, iterate)
- Visible time savings: Track spec review time, show estimators their own improvement
- Management buy-in: Coordinators enforce System Chart upload (mandatory for QC)

### Risk 4: Agent Processing Time Exceeds Tolerance (MEDIUM IMPACT)
**Description:** 900-page spec takes >5 minutes, estimators go back to manual  
**Impact:** Low adoption, spec review time doesn't improve  
**Mitigation:**
- Performance testing in Phase 5 (VAL-007)
- Incremental display: Show sections as processed (don't block UI)
- Optimization: Parallel processing, caching, pre-processing on upload
- Fallback: If processing stalls, display link to raw PDF

### Risk 5: Multi-Region Coordination Complexity (LOW IMPACT)
**Description:** Pakistan/India/Mexico teams have different workflows, Holocron doesn't fit all  
**Impact:** One region adopts, others resist  
**Mitigation:**
- Flexible workflow: Support stage skipping, manual overrides (FR-WF-002)
- V1 excludes MRSM: Avoid Mexico-specific complexity (Q2 clarification)
- Role-based design: System adapts to role, not region (FR-RBAC-003)

### Risk 6: OneDrive Integration Reliability (LOW IMPACT)
**Description:** OneDrive API rate limits or outages break Intake Agent  
**Impact:** Projects stuck in Intake, estimators blocked  
**Mitigation:**
- Polling interval: 60 seconds (avoid rate limits)
- Retry logic: Exponential backoff on failures (NFR-REL-003)
- Manual bypass: Coordinator can manually mark project as "Ready" if upload detected but Intake fails

---

## 15. Assumptions & Dependencies

### Assumptions
1. **Estimators have stable internet access** (10 Mbps minimum for spec PDF downloads)
2. **Microsoft 365 SSO available** for authentication
3. **OneDrive folder structure** can be standardized across clients
4. **QuoteSoft export format** remains consistent (CSV + JSON in .zip)
5. **System Chart format** is Excel (.xlsx/.xls) with predictable structure
6. **Client SOPs** will be manually compiled during V1 (multi-week effort per client)
7. **Estimators work on desktop** (1920×1080 or larger screens)

### Dependencies
1. **OneDrive API access** (Microsoft Graph API)
2. **AWS infrastructure** (S3, Lambda, ECS for agent runtime)
3. **PDF parsing library** (e.g., PyMuPDF, pdfplumber)
4. **QuoteSoft export documentation** (format spec for CSV/JSON parsing)
5. **Stakeholder availability** for validation testing (Josh Waldecker, Scott, Rehman, estimators)
6. **Pilot project selection** (5 real projects for Phase 5 rollout)

---

## 16. Clarifications Needed (Before Implementation)

### Open Questions
1. **System Chart structure:** Is there a standardized template, or does it vary by estimator/client? (Need sample files)
2. **QuoteSoft CSV schema:** What are the exact column names and data types? (Need export documentation)
3. **SOP database schema:** How should SOPs be stored in V1 (manual entry)? Flat file, database table?
4. **Project metadata source:** Where does project name, client, region come from? (OneDrive folder name parsing, or manual entry?)
5. **Error notification:** When agent fails, who gets alerted? (Coordinator, Manager, or operations team?)
6. **Historical data:** Do we have past projects with known rework reasons for training/validation? (Need access to QC feedback logs)

### Clarifications from Stakeholders
- **Josh Waldecker:** Confirm typical spec review time for small projects (<500 items) vs large (1000+ items)
- **Scott:** Provide 2-3 example spec manuals (small, medium, large) for agent training
- **Rehman:** Confirm QC checklist items (are there standard items beyond the 6 in FR-QC-003?)
- **Coordinators:** Confirm OneDrive folder naming convention (how is client/project encoded in folder path?)
- **Jr Estimators:** Which SOPs are most frequently needed? (prioritize for manual entry in V1)

---

## 17. Appendices

### Appendix A: Glossary (from IPR)
- **Quantity Takeoff:** Detailed list of materials/components for HVAC installation (quantities only, no pricing)
- **System Chart:** Excel document capturing specifications and assumptions (filled before QuoteSoft, not related to QuoteSoft software)
- **SOP:** Standard Operating Procedure (client-specific rules overriding drawings/specs)
- **Division 22/23/40:** CSI MasterFormat sections (Plumbing, Mechanical/HVAC, Process Piping)
- **QuoteSoft:** Industry software for HVAC takeoff execution (manual component placement)
- **Zone:** In QuoteSoft, one drawing page at one scale (background PDF or .tif)
- **Contention:** Conflict between specification sources (e.g., drawing contradicts spec)
- **Precedence:** Bimteq rule for conflict resolution (SOP > Drawing > Spec)
- **Basis Board:** MRSM's project request system (V2 scope, not V1)
- **Command Center:** Holocron's unified interface (6 panels, sidebar navigation)

### Appendix B: Stakeholder Interview Sources (from IPR)
- **Josh Waldecker:** Role structure, team organization, MRSM workflow, QuoteSoft integration
- **Scott:** Terminology corrections, precedence hierarchy, spec reading method, workflow variations
- **Rehman:** Production hierarchy, QC process, rework metrics, operating reality across regions

### Appendix C: Q&A Clarifications (Incorporated into BRD)
- **Q1:** QC bypass by role (Senior Estimators + QC Engineers)
- **Q2:** MRSM deferred to V2
- **Q3:** Multi-estimator assignment required
- **Q4:** Automated clash detection mandatory for all projects (minimum QC level)
- **Q5:** Rework tracking logged inside Holocron
- **Q6:** System Chart upload within Spec Navigator panel
- **Q7:** Irrelevant sections NOT displayed (no toggle to show)
- **Q8:** 5-minute spec processing acceptable for 900 pages
- **Q9:** Customer clarification outside system, but email attachment supported
- **Q10:** Additive/deductive = quantity alternates only (cost out of scope)
- **Q11:** Highlighted PDF upload required, no tooling support in V1
- **Q12:** India QC team in scope for V1 (QC authority for organization)

### Appendix D: Visual Design References
- **Mockup file:** bimteq-command-center_022026_v1.html
- **Design doc:** bimteq_command_center_explanation_022026_v1.md
- **Theme:** Mission control aesthetic (Azeret Mono + Sora fonts, dark void #050810, cyan accent #00e5cc)
- **Layout:** Single-panel with 56px sidebar, top bar 56px height, panels fill remaining space

---

## 18. Approval & Sign-Off

**Document Prepared By:** [Engineering Team]  
**Date:** February 25, 2026  
**Version:** 1.0

**Stakeholder Approval Required:**
- [ ] **Josh Waldecker** (Role structure, workflow validation)
- [ ] **Scott** (Spec reading method, terminology accuracy)
- [ ] **Rehman** (QC process, production metrics)
- [ ] **Engineering Lead** (Technical feasibility)
- [ ] **Product Owner** (Scope alignment with IPR)

**Next Steps:**
1. Stakeholder review (target: 1 week)
2. Address open questions (Section 16)
3. Finalize agent contracts (detailed input/output schemas)
4. Begin Phase 1 implementation (Foundation)

---

**END OF BRD**

---

## Summary for Engineering Team

This BRD translates the Holocron vision into actionable requirements. Key takeaways:

1. **Six specialized agents** (Intake, Spec Navigator, Contention Detection, Assignment Recommendation, Project State, QC & Audit) work in a circuit orchestrated by the platform.

2. **Six UI panels** (Project Dashboard, Spec Navigator, Contention Review, QC/Audit, SOP Management [V2], Admin/Ops) with sidebar navigation — role-based defaults, but all accessible.

3. **Human-in-the-loop at critical checkpoints:** Assignment, contention resolution, System Chart upload, QC approval — no automated decisions.

4. **HVAC scope only:** Divisions 22/23/40 (Plumbing, Mechanical, Process Piping) — no structural, civil, or pricing.

5. **Automated QC clash detection mandatory for all projects** (Q4 clarification) — compares System Chart vs QuoteSoft export, flags discrepancies.

6. **Role-based design, not region-based:** Jr/Sr Estimators, QC Engineers, Coordinators, Managers — geography not a constraint.

7. **V1 excludes MRSM** (Q2 clarification) — focus on Pakistan/India teams, Mexico MRSM deferred to V2.

8. **Success metric targets:** Rework rate <10% (from 15-20%), spec review time <30 min (from 2 hours), ≥90% contention detection recall, ≥95% QC clash detection accuracy.

If you have questions, refer to the IPR (product vision), design doc (UI details), or this BRD (functional requirements). Clarifications needed before starting? See Section 16.

Let's build a tool our estimators actually want to use. 🚀

---
