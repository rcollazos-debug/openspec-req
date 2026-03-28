# Delta Spec: risks — educational-project-management-platform

## ADDED Requirements

### REQ-RISKS-001: Risk Event Configuration Management

The system SHALL allow Administrators to configure risk events with probability parameters, impact levels, and execution conditions. Each risk event MUST have a unique identifier, descriptive name, probability percentage (1-100%), impact severity (Low/Medium/High/Critical), and optional zone targeting. The system SHALL validate that total probability across all active risk events does not exceed 100% per execution cycle.

**Escenarios:**

**Scenario 1: Administrator creates new risk event successfully**
- **Given** an Administrator is logged into the system
- **When** they create a risk event with name "Material Shortage", probability 15%, impact "High", and target zone "Zone 1"
- **Then** the system SHALL save the risk event with a unique identifier
- **And** the risk event SHALL appear in the active risks list

**Scenario 2: Risk event creation fails due to invalid probability**
- **Given** an Administrator attempts to create a risk event
- **When** they set probability to 150%
- **Then** the system SHALL reject the creation with error "Probability must be between 1-100%"
- **And** no risk event SHALL be created

**Scenario 3: Total probability validation prevents exceeding 100%**
- **Given** existing risk events total 95% probability
- **When** Administrator attempts to add a new risk with 10% probability
- **Then** the system SHALL reject the creation with error "Total probability would exceed 100%"
- **And** the system SHALL suggest reducing existing probabilities

---

### REQ-RISKS-002: Automated Risk Execution Engine

The system SHALL execute risk events automatically based on configured time intervals and probability calculations. The execution engine MUST run every 10 minutes (representing 1 month in simulation time) and SHALL use cryptographically secure random number generation for probability calculations. When a risk event is triggered, the system SHALL immediately notify affected zones and update the projection screen display.

**Escenarios:**

**Scenario 1: Risk event triggers successfully during execution cycle**
- **Given** a risk event "Equipment Failure" with 20% probability is configured for Zone 2
- **When** the execution engine runs and random calculation falls within the 20% threshold
- **Then** the system SHALL trigger the risk event for Zone 2
- **And** the projection screen SHALL display the risk event notification immediately
- **And** affected Coordinators SHALL receive email notifications within 30 seconds

**Scenario 2: No risk events trigger during low probability cycle**
- **Given** all configured risk events have probabilities totaling 30%
- **When** the execution engine runs and random calculation falls outside all thresholds
- **Then** no risk events SHALL be triggered
- **And** the system SHALL log "No risks triggered" in the audit trail

**Scenario 3: Multiple risk events trigger simultaneously**
- **Given** two risk events with 25% probability each are configured
- **When** both events trigger in the same execution cycle
- **Then** the system SHALL execute both risk events
- **And** the projection screen SHALL display both notifications sequentially
- **And** the system SHALL handle concurrent database updates without conflicts

---

### REQ-RISKS-003: Risk Impact Application and Zone Effects

The system SHALL automatically apply risk impacts to affected zones by modifying task completion percentages, extending deadlines, or blocking specific deliverables. Each risk impact MUST be reversible and SHALL maintain an audit trail of all modifications. The system SHALL calculate new zone completion percentages immediately after risk application and update all relevant dashboards and Gantt charts.

**Escenarios:**

**Scenario 1: Risk reduces zone completion percentage**
- **Given** Zone 3 has 80% completion and a "Resource Shortage" risk with -15% impact triggers
- **When** the system applies the risk impact
- **Then** Zone 3 completion SHALL be reduced to 65%
- **And** the consolidated project completion SHALL be recalculated automatically
- **And** the Gantt chart SHALL reflect the new completion status

**Scenario 2: Risk blocks deliverable completion**
- **Given** a deliverable "Foundation Design" is 90% complete in Zone 1
- **When** a "Technical Issue" risk triggers with "Block Deliverable" impact
- **Then** the deliverable status SHALL change to "Blocked"
- **And** assigned team members SHALL receive immediate notifications
- **And** the deliverable SHALL not contribute to zone completion until unblocked

**Scenario 3: Risk impact is reversed by Administrator**
- **Given** a risk impact has been applied reducing Zone 2 completion by 20%
- **When** Administrator selects "Reverse Risk Impact" for that specific event
- **Then** the system SHALL restore Zone 2 to its pre-risk completion percentage
- **And** all affected calculations SHALL be recalculated
- **And** the reversal SHALL be logged in the audit trail with timestamp and user

---

### REQ-RISKS-004: Real-time Risk Visualization on Projection Screen

The system SHALL display active risk events on the projection screen with visual alerts, countdown timers, and impact descriptions. Risk notifications MUST appear within 5 seconds of triggering and SHALL remain visible for a minimum of 60 seconds. The display SHALL include risk severity color coding (Green=Low, Yellow=Medium, Orange=High, Red=Critical) and affected zone indicators.

**Escenarios:**

**Scenario 1: Critical risk displays with maximum visibility**
- **Given** a Critical severity risk "System Failure" triggers for all zones
- **When** the risk notification appears on projection screen
- **Then** the display SHALL show red background with flashing border
- **And** the notification SHALL remain visible for 120 seconds minimum
- **And** an audio alert SHALL play if speakers are configured

**Scenario 2: Multiple risks display in priority queue**
- **Given** three risks of different severities trigger within 30 seconds
- **When** the projection screen processes the notifications
- **Then** Critical risks SHALL display first, followed by High, Medium, then Low
- **And** each notification SHALL display for its minimum required duration
- **And** a queue indicator SHALL show remaining notifications

**Scenario 3: Risk notification expires and clears from screen**
- **Given** a Medium severity risk has been displayed for 60 seconds
- **When** the minimum display time expires and no higher priority risks are queued
- **Then** the risk notification SHALL fade out over 3 seconds
- **And** the screen SHALL return to normal project status display
- **And** the risk SHALL be marked as "Displayed" in the system log

---

### REQ-RISKS-005: Risk History and Analytics Dashboard

The system SHALL maintain a comprehensive history of all triggered risk events with timestamps, affected zones, impacts applied, and resolution status. Administrators MUST be able to generate risk analytics reports showing frequency patterns, zone vulnerability analysis, and cumulative impact on project completion. The dashboard SHALL provide filtering capabilities by date range, risk type, severity, and affected zones.

**Escenarios:**

**Scenario 1: Administrator views comprehensive risk history**
- **Given** the system has executed risks over multiple simulation months
- **When** Administrator accesses the Risk Analytics Dashboard
- **Then** the system SHALL display a chronological list of all triggered risks
- **And** each entry SHALL show timestamp, risk name, affected zones, and current status
- **And** the dashboard SHALL calculate total risks triggered and average impact per zone

**Scenario 2: Risk analytics report generation with filtering**
- **Given** Administrator wants to analyze Zone 2 risk patterns
- **When** they apply filters for "Zone 2" and "Last 5 simulation months"
- **Then** the system SHALL generate a filtered report showing only Zone 2 risks
- **And** the report SHALL include frequency charts and impact trend analysis
- **And** the report SHALL be downloadable in PDF format within 10 seconds

**Scenario 3: Zone vulnerability analysis identifies high-risk areas**
- **Given** risk history data spans the complete simulation period
- **When** Administrator requests vulnerability analysis
- **Then** the system SHALL calculate risk frequency per zone
- **And** zones SHALL be ranked from most to least vulnerable
- **And** the analysis SHALL recommend risk mitigation strategies for high-vulnerability zones

---

### REQ-RISKS-006: Risk Notification and Alert System

The system SHALL send immediate notifications to affected Coordinators and team members when risks impact their zones. Notifications MUST be delivered via both email (Gmail API) and in-application alerts within 30 seconds of risk triggering. The system SHALL provide escalation notifications to Directors if risks remain unaddressed for more than 20 minutes (2 simulation months).

**Escenarios:**

**Scenario 1: Coordinator receives immediate risk notification**
- **Given** a "Supply Delay" risk triggers affecting Zone 4
- **When** the risk impact is applied to Zone 4
- **Then** the Zone 4 Coordinator SHALL receive an email notification within 30 seconds
- **And** an in-app notification SHALL appear in their dashboard immediately
- **And** the notification SHALL include risk details and recommended actions

**Scenario 2: Escalation notification sent to Director**
- **Given** a High severity risk has been active in Zone 1 for 20 minutes
- **When** no mitigation actions have been taken by the Coordinator
- **Then** the system SHALL send an escalation email to the Director
- **And** the Director's dashboard SHALL show an urgent alert for Zone 1
- **And** the escalation SHALL include risk history and current impact status

**Scenario 3: Risk resolution notification confirms mitigation**
- **Given** a Coordinator has successfully mitigated an active risk
- **When** they mark the risk as "Resolved" in the system
- **Then** all affected team members SHALL receive resolution confirmation emails
- **And** the risk SHALL be removed from active alerts
- **And** the resolution SHALL be logged with timestamp and mitigation actions taken

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]