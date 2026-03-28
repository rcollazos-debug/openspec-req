# Delta Spec: projects — educational-project-management-platform

## ADDED Requirements

### REQ-PROJECTS-001: Project Creation and Configuration

The system SHALL allow Administrators to create new educational project instances with configurable parameters including duration (default 3 weeks), number of zones (default 5), and time scale mapping (10 minutes = 1 month). Each project MUST be assigned a unique identifier and creation timestamp. The system SHALL validate that zone count is between 1 and 10, and duration is between 1 and 52 weeks.

**Escenarios:**

**Scenario 1: Successful project creation**
- **Given** an authenticated Administrator user
- **When** they create a new project with valid parameters (name="Ciudad Lego 2024", zones=5, duration=3 weeks)
- **Then** the system SHALL generate a unique project ID
- **And** the project SHALL be created with status "Planning"
- **And** the system SHALL create 5 empty zones with 20% weight each

**Scenario 2: Invalid zone count**
- **Given** an authenticated Administrator user
- **When** they attempt to create a project with 15 zones
- **Then** the system SHALL reject the request with error "Zone count must be between 1 and 10"
- **And** no project SHALL be created

**Scenario 3: Duplicate project name**
- **Given** a project named "Ciudad Lego 2024" already exists
- **When** an Administrator attempts to create another project with the same name
- **Then** the system SHALL reject the request with error "Project name must be unique"
- **And** no duplicate project SHALL be created

---

### REQ-PROJECTS-002: Zone Management and Weight Distribution

The system SHALL automatically distribute project completion weight equally across all zones (20% per zone for 5 zones). Each zone MUST be assigned to exactly one Coordinator role user. The system SHALL prevent zone deletion if it contains active tasks or deliverables. Zone reassignment MUST trigger recalculation of all dependent completion percentages.

**Escenarios:**

**Scenario 1: Automatic zone weight calculation**
- **Given** a project with 5 zones is created
- **When** the system initializes the zones
- **Then** each zone SHALL be assigned exactly 20% weight
- **And** the total weight SHALL equal 100%

**Scenario 2: Zone assignment to Coordinator**
- **Given** a project with available zones and a Coordinator user
- **When** an Administrator assigns the Coordinator to Zone 1
- **Then** the zone SHALL be marked as "Assigned"
- **And** the Coordinator SHALL gain access to Zone 1 planning functions
- **And** the system SHALL send notification to the assigned Coordinator

**Scenario 3: Prevent deletion of active zone**
- **Given** Zone 1 contains 3 active tasks
- **When** an Administrator attempts to delete Zone 1
- **Then** the system SHALL reject the deletion with error "Cannot delete zone with active tasks"
- **And** the zone SHALL remain unchanged

---

### REQ-PROJECTS-003: Month and Milestone Planning

Coordinators SHALL create months, milestones, and deliverables within their assigned zones. Each month MUST have a duration of 10 real minutes and contain at least one milestone. Milestones MAY contain multiple deliverables with individual completion criteria. The system SHALL validate that milestone dates fall within the month boundaries and prevent overlapping critical path dependencies.

**Escenarios:**

**Scenario 1: Month creation by Coordinator**
- **Given** a Coordinator assigned to Zone 1
- **When** they create "Month 1" with start date and 2 milestones
- **Then** the system SHALL create the month with 10-minute duration
- **And** the milestones SHALL be linked to the month
- **And** the month status SHALL be "Planned"

**Scenario 2: Milestone with deliverables**
- **Given** a month exists in Zone 1
- **When** a Coordinator creates milestone "Foundation Complete" with 3 deliverables
- **Then** each deliverable SHALL be created with status "Not Started"
- **And** the milestone completion SHALL be calculated as average of deliverable completions
- **And** the system SHALL validate deliverable due dates are within milestone timeframe

**Scenario 3: Invalid milestone date**
- **Given** a month spanning January 1-31
- **When** a Coordinator attempts to create a milestone with due date February 5
- **Then** the system SHALL reject with error "Milestone date must fall within month boundaries"
- **And** no milestone SHALL be created

---

### REQ-PROJECTS-004: Real-time Progress Calculation

The system SHALL calculate project completion percentage in real-time based on zone weights and deliverable completion status. Each zone's completion MUST be computed as the weighted average of its deliverables' completion percentages. The overall project completion SHALL be the sum of all zone completions multiplied by their respective weights. Updates MUST propagate to the projection screen within 5 seconds.

**Escenarios:**

**Scenario 1: Zone completion calculation**
- **Given** Zone 1 (20% weight) has 4 deliverables: 100%, 75%, 50%, 25% complete
- **When** the system calculates zone completion
- **Then** Zone 1 completion SHALL be 62.5% ((100+75+50+25)/4)
- **And** Zone 1 contributes 12.5% to overall project (62.5% × 20%)

**Scenario 2: Real-time update propagation**
- **Given** a deliverable completion is updated from 50% to 75%
- **When** the change is saved
- **Then** the zone completion SHALL recalculate within 2 seconds
- **And** the overall project percentage SHALL update within 5 seconds
- **And** the projection screen SHALL display the new percentage

**Scenario 3: Multiple zone completion aggregation**
- **Given** 5 zones with completions: 80%, 60%, 40%, 90%, 70%
- **When** the system calculates overall project completion
- **Then** the result SHALL be 68% ((80+60+40+90+70)/5)
- **And** the calculation SHALL be logged for audit purposes

---

### REQ-PROJECTS-005: Project Lifecycle Management

The system SHALL manage project states through a defined lifecycle: Planning → Active → Paused → Completed → Archived. State transitions MUST be restricted based on user roles and current project conditions. Only Administrators MAY transition projects to Archived state. Active projects SHALL automatically transition to Completed when all zones reach 100% completion or the duration expires.

**Escenarios:**

**Scenario 1: Project activation**
- **Given** a project in "Planning" state with all zones assigned to Coordinators
- **When** an Administrator activates the project
- **Then** the project state SHALL change to "Active"
- **And** the countdown timer SHALL start
- **And** all assigned users SHALL receive activation notification

**Scenario 2: Automatic completion**
- **Given** an active project with all 5 zones at 100% completion
- **When** the system performs its periodic check
- **Then** the project state SHALL automatically change to "Completed"
- **And** the completion timestamp SHALL be recorded
- **And** all participants SHALL receive completion notification

**Scenario 3: Unauthorized state transition**
- **Given** a Coordinator user and a project in "Active" state
- **When** the Coordinator attempts to change project state to "Archived"
- **Then** the system SHALL reject the request with error "Insufficient permissions"
- **And** the project state SHALL remain "Active"

---

### REQ-PROJECTS-006: Project Timeline and Duration Control

The system SHALL enforce the configured project timeline with automatic progression through months based on the 10-minute real-time mapping. Each month transition MUST trigger automatic notifications, progress snapshots, and risk evaluation cycles. The system SHALL provide timeline override capabilities for Administrators to pause, resume, or manually advance the timeline during the educational workshop.

**Escenarios:**

**Scenario 1: Automatic month progression**
- **Given** an active project in Month 1 with 10 minutes elapsed
- **When** the system timer reaches the month boundary
- **Then** the project SHALL automatically advance to Month 2
- **And** a progress snapshot SHALL be captured and stored
- **And** all participants SHALL receive month transition notification

**Scenario 2: Timeline pause by Administrator**
- **Given** an active project in Month 2
- **When** an Administrator pauses the timeline
- **Then** the countdown timer SHALL stop
- **And** the project state SHALL change to "Paused"
- **And** no automatic month transitions SHALL occur until resumed

**Scenario 3: Manual month advancement**
- **Given** a paused project in Month 1
- **When** an Administrator manually advances to Month 3
- **Then** the system SHALL skip Month 2 and set current month to 3
- **And** the timeline SHALL resume from Month 3
- **And** the skip action SHALL be logged in the audit trail

---

### REQ-PROJECTS-007: Project Dashboard and Monitoring

The system SHALL provide role-specific project dashboards with real-time metrics, progress visualization, and key performance indicators. Directors MUST see consolidated views across all zones with Gantt charts and critical path analysis. Coordinators SHALL access detailed zone-specific dashboards with task assignments and deliverable tracking. All dashboards MUST refresh automatically every 30 seconds during active project phases.

**Escenarios:**

**Scenario 1: Director consolidated dashboard**
- **Given** a Director user accessing the project dashboard
- **When** the dashboard loads
- **Then** it SHALL display overall project completion percentage
- **And** it SHALL show individual zone progress bars
- **And** it SHALL present a consolidated Gantt chart with all zones
- **And** it SHALL highlight critical path milestones

**Scenario 2: Coordinator zone dashboard**
- **Given** a Coordinator assigned to Zone 3
- **When** they access their dashboard
- **Then** it SHALL display only Zone 3 metrics and tasks
- **And** it SHALL show detailed deliverable completion status
- **And** it SHALL list assigned team members and their task loads
- **And** it SHALL provide zone-specific Gantt view

**Scenario 3: Automatic dashboard refresh**
- **Given** a user viewing any project dashboard
- **When** 30 seconds elapse since last update
- **Then** the dashboard SHALL automatically refresh all metrics
- **And** new data SHALL be fetched without page reload
- **And** the refresh timestamp SHALL be displayed to the user

---

### REQ-PROJECTS-008: Project Archive and Historical Data

The system SHALL maintain complete historical records of all project activities, decisions, and outcomes for educational analysis and future reference. Archived projects MUST preserve all associated data including tasks, deliverables, communications, and audit trails. The system SHALL provide search and filtering capabilities across archived projects for comparative analysis and lesson learned extraction.

**Escenarios:**

**Scenario 1: Project archival process**
- **Given** a completed project with all data intact
- **When** an Administrator archives the project
- **Then** all project data SHALL be moved to archive storage
- **And** the project SHALL be marked as "Archived" with timestamp
- **And** access permissions SHALL be restricted to read-only for all users

**Scenario 2: Historical data preservation**
- **Given** an archived project from 6 months ago
- **When** a user with appropriate permissions accesses it
- **Then** all original tasks, deliverables, and communications SHALL be viewable
- **And** the complete audit trail SHALL be preserved and searchable
- **And** all file attachments SHALL remain accessible through Google Drive links

**Scenario 3: Cross-project analysis**
- **Given** multiple archived projects from different workshop sessions
- **When** an Administrator performs comparative analysis
- **Then** the system SHALL provide filtering by date range, completion rate, and zone performance
- **And** aggregated metrics SHALL be available for educational assessment
- **And** trend analysis reports SHALL be exportable in PDF format

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]