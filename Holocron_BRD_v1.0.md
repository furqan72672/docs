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
- Multi-estimator support: Display multiple avatars when 2+ estimators assigned
- Role tags replace location flags (role-based design, not region-based)

**FR-PD-005:** Effort tracking
- Display estimated effort in hours (font: Azeret Mono)
- Source: Assignment Recommendation Agent calculation + actual time logged

#### Data Sources
- **Agent:** Project State Agent (real-time stage/status)
- **Input:** Estimator actions, coordinator assignments, QC outcomes
- **Update Frequency:** WebSocket/SSE for live updates (V1: polling acceptable with 30s interval)
