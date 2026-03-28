# Delta Spec: notifications — educational-project-management-platform

## ADDED Requirements

### REQ-NOTIFICATIONS-001: Email notification delivery system

The system SHALL send automated email notifications through Gmail API integration for all critical project events including task assignments, deadline reminders, risk events, and milestone completions. The system MUST authenticate with Gmail API using OAuth 2.0 and SHALL maintain token refresh capabilities to ensure uninterrupted service during the 3-week workshop duration.

**Escenarios:**

**Scenario 1: Successful email notification delivery**
- **Given** a user has a valid email address in the system and Gmail API is authenticated
- **When** a task is assigned to the user by a Coordinator
- **Then** the system SHALL send an email notification within 30 seconds
- **And** the notification SHALL include task details, deadline, and direct link to the task

**Scenario 2: Gmail API authentication failure**
- **Given** the Gmail API token has expired or is invalid
- **When** the system attempts to send an email notification
- **Then** the system SHALL attempt token refresh automatically
- **And** if refresh fails, the system SHALL log the error and queue the notification for retry

**Scenario 3: Email delivery failure handling**
- **Given** an email notification fails to deliver due to invalid recipient address
- **When** the Gmail API returns a delivery failure response
- **Then** the system SHALL mark the notification as failed in the audit log
- **And** the system SHALL display an in-app notification as fallback

---

### REQ-NOTIFICATIONS-002: In-app notification system

The system SHALL provide real-time in-app notifications displayed in the user interface for immediate awareness of project events. Notifications MUST be categorized by priority (High, Medium, Low) and SHALL support read/unread status tracking. The system MUST update notification counts in real-time using WebSocket connections.

**Escenarios:**

**Scenario 1: Real-time notification display**
- **Given** a user is logged into the system with an active WebSocket connection
- **When** a high-priority event occurs (risk execution, deadline approaching)
- **Then** the system SHALL display the notification immediately in the user interface
- **And** the notification counter SHALL increment without page refresh

**Scenario 2: Notification marking as read**
- **Given** a user has unread notifications in their notification panel
- **When** the user clicks on a specific notification
- **Then** the system SHALL mark the notification as read
- **And** the notification counter SHALL decrease by one

**Scenario 3: WebSocket connection failure**
- **Given** a user's WebSocket connection is interrupted
- **When** the user refreshes the page or reconnects
- **Then** the system SHALL load all unread notifications from the database
- **And** the notification counter SHALL display the correct count

---

### REQ-NOTIFICATIONS-003: Task deadline reminder system

The system SHALL automatically generate reminder notifications for tasks approaching their deadlines. Reminders MUST be sent at configurable intervals (24 hours, 4 hours, 1 hour before deadline) and SHALL be delivered through both email and in-app channels. The system MUST consider the accelerated time scale where 10 minutes equals 1 month.

**Escenarios:**

**Scenario 1: Automated deadline reminder generation**
- **Given** a task has a deadline set for tomorrow in project time (1 minute in real time)
- **When** the system runs the scheduled reminder check
- **Then** the system SHALL generate a 24-hour reminder notification
- **And** the notification SHALL be sent via both email and in-app channels

**Scenario 2: Multiple reminder intervals**
- **Given** a task deadline is approaching with multiple reminder intervals configured
- **When** each reminder threshold is reached
- **Then** the system SHALL send only one notification per threshold
- **And** subsequent reminders SHALL reference previous notifications to avoid spam

**Scenario 3: Task completion before deadline**
- **Given** a task has pending deadline reminders scheduled
- **When** the task status changes to "Finalizada"
- **Then** the system SHALL cancel all pending reminder notifications for that task
- **And** no further deadline reminders SHALL be sent

---

### REQ-NOTIFICATIONS-004: Risk event notification broadcasting

The system SHALL broadcast immediate notifications to all relevant users when random risk events are executed by the risk engine. Risk notifications MUST be displayed on the projection screen and sent to affected zone Coordinators and the Director. The system SHALL include risk impact details and recommended actions in the notification content.

**Escenarios:**

**Scenario 1: Zone-specific risk event notification**
- **Given** a risk event is executed affecting Zone 1 construction progress
- **When** the risk engine triggers the event
- **Then** the system SHALL send notifications to Zone 1 Coordinator and the Director
- **And** the notification SHALL appear on the projection screen with visual emphasis

**Scenario 2: Project-wide risk event notification**
- **Given** a risk event affects all zones (e.g., material shortage)
- **When** the risk engine executes the global risk
- **Then** the system SHALL notify all Coordinators, the Director, and Administrator
- **And** the projection screen SHALL display the risk with countdown timer for resolution

**Scenario 3: Risk resolution notification**
- **Given** a Coordinator marks a risk event as resolved
- **When** the resolution is submitted through the system
- **Then** the system SHALL notify the Director of the resolution
- **And** the projection screen SHALL update to show risk status as resolved

---

### REQ-NOTIFICATIONS-005: Milestone and deliverable completion alerts

The system SHALL generate automatic notifications when milestones or deliverables are marked as completed, updating zone completion percentages and notifying relevant stakeholders. Completion notifications MUST trigger recalculation of overall project progress and SHALL be sent to the Director and affected zone teams.

**Escenarios:**

**Scenario 1: Deliverable completion notification**
- **Given** a Coordinator marks a deliverable as completed in their zone
- **When** the completion is saved in the system
- **Then** the system SHALL notify the Director and all zone team members
- **And** the notification SHALL include updated zone completion percentage

**Scenario 2: Milestone achievement notification**
- **Given** all deliverables for a milestone are completed
- **When** the system calculates milestone completion automatically
- **Then** the system SHALL send milestone achievement notifications to all project participants
- **And** the projection screen SHALL display milestone celebration animation

**Scenario 3: Completion percentage threshold alerts**
- **Given** a zone reaches 50%, 75%, or 100% completion
- **When** the system recalculates zone progress after deliverable completion
- **Then** the system SHALL send threshold achievement notifications
- **And** the Administrator SHALL receive progress summary notifications

---

### REQ-NOTIFICATIONS-006: Meeting minutes approval workflow notifications

The system SHALL manage notification workflows for meeting minutes creation, consolidation, and approval processes. Notifications MUST be sent to Coordinators when creating initial minutes, to the Director when consolidated minutes are ready for review, and to all participants when minutes are approved.

**Escenarios:**

**Scenario 1: Meeting minutes creation reminder**
- **Given** a new project month begins (every 10 real minutes)
- **When** the system detects the month transition
- **Then** the system SHALL notify all Coordinators to create their zone meeting minutes
- **And** the notification SHALL include a direct link to the minutes creation form

**Scenario 2: Consolidated minutes approval request**
- **Given** all Coordinators have submitted their meeting minutes
- **When** the system automatically consolidates the minutes
- **Then** the system SHALL notify the Director that consolidated minutes are ready for approval
- **And** the notification SHALL include a summary of all zone submissions

**Scenario 3: Minutes approval confirmation**
- **Given** the Director approves the consolidated meeting minutes
- **When** the approval is submitted through the system
- **Then** the system SHALL notify all project participants of the approved minutes
- **And** the approved minutes SHALL be automatically uploaded to Google Drive

---

### REQ-NOTIFICATIONS-007: System maintenance and error notifications

The system SHALL provide administrative notifications for system maintenance events, integration failures, and critical errors that may impact workshop operations. Administrator notifications MUST include detailed error information and suggested resolution steps. The system SHALL maintain notification delivery even during partial system failures.

**Escenarios:**

**Scenario 1: Google API integration failure alert**
- **Given** the Google Drive or Gmail API integration encounters an authentication error
- **When** the system detects the integration failure
- **Then** the system SHALL immediately notify the Administrator via email and in-app
- **And** the notification SHALL include specific error codes and troubleshooting steps

**Scenario 2: Database connection issue notification**
- **Given** the PostgreSQL database connection becomes unstable
- **When** the system detects connection timeouts or failures
- **Then** the system SHALL send critical alerts to the Administrator
- **And** the system SHALL attempt automatic reconnection while logging all attempts

**Scenario 3: Projection screen connectivity alert**
- **Given** the projection screen loses connection to the main system
- **When** the WebSocket connection fails for more than 30 seconds
- **Then** the system SHALL alert the Administrator and provide backup projection options
- **And** the system SHALL attempt to restore connection automatically

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]