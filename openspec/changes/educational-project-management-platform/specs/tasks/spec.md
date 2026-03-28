# Delta Spec: tasks — educational-project-management-platform

## ADDED Requirements

### REQ-TASKS-001: Task Creation and Assignment System

The system SHALL allow Coordinators to create tasks with mandatory fields including title, description, assignee, due date, and zone association. The system MUST validate that only users with Coordinator or higher privileges can create tasks. Each task SHALL be automatically assigned a unique identifier and creation timestamp. The system SHALL enforce that tasks can only be assigned to users within the same zone as the creating Coordinator.

**Escenarios:**

**Scenario 1: Successful task creation by Coordinator**
- **Given** a user with Coordinator role is authenticated and accessing their zone dashboard
- **When** they submit a new task form with valid title, description, assignee from their zone, and future due date
- **Then** the system SHALL create the task with status "Creada" and generate a unique task ID
- **And** the system SHALL send an automatic notification to the assigned user via email and in-app notification

**Scenario 2: Task creation with invalid assignee**
- **Given** a Coordinator is creating a new task
- **When** they attempt to assign the task to a user from a different zone
- **Then** the system SHALL reject the creation with error message "Cannot assign tasks to users outside your zone"
- **And** the task SHALL NOT be saved to the database

**Scenario 3: Task creation by unauthorized user**
- **Given** a user with Integrante role is authenticated
- **When** they attempt to access the task creation interface
- **Then** the system SHALL deny access with HTTP 403 Forbidden status
- **And** display message "Insufficient privileges to create tasks"

---

### REQ-TASKS-002: Task State Management Workflow

The system SHALL implement a strict task state workflow: Creada → Asignada → En curso → En revisión → Finalizada. The system MUST prevent invalid state transitions and SHALL log all state changes with timestamp and user information. Only assigned users MAY transition tasks from "Asignada" to "En curso", and only Coordinators or Directors SHALL approve transitions from "En revisión" to "Finalizada".

**Escenarios:**

**Scenario 1: Valid state progression by assigned user**
- **Given** a task exists with status "Asignada" and user is the assigned person
- **When** the user clicks "Start Task" button
- **Then** the system SHALL update task status to "En curso" and record timestamp
- **And** the system SHALL log the state change with user ID and datetime

**Scenario 2: Invalid state transition attempt**
- **Given** a task with status "Creada"
- **When** a user attempts to change status directly to "En revisión"
- **Then** the system SHALL reject the change with error "Invalid state transition"
- **And** the task status SHALL remain unchanged

**Scenario 3: Task completion approval by Coordinator**
- **Given** a task with status "En revisión" and user has Coordinator role for that zone
- **When** the Coordinator clicks "Approve Completion"
- **Then** the system SHALL update status to "Finalizada" and set completion timestamp
- **And** the system SHALL calculate zone completion percentage automatically

---

### REQ-TASKS-003: Task Reassignment Functionality

The system SHALL allow Coordinators and Directors to reassign tasks to different users within the same zone. The system MUST preserve the complete reassignment history including previous assignee, new assignee, reason, and timestamp. When a task is reassigned, the system SHALL automatically notify both the previous and new assignees via email and in-app notifications.

**Escenarios:**

**Scenario 1: Successful task reassignment within zone**
- **Given** a Coordinator has a task assigned to User A in their zone
- **When** they reassign the task to User B with reason "Workload balancing"
- **Then** the system SHALL update the assignee to User B and preserve reassignment history
- **And** the system SHALL send notifications to both User A and User B about the reassignment

**Scenario 2: Attempted reassignment to different zone**
- **Given** a Coordinator is reassigning a task from their zone
- **When** they attempt to assign it to a user from a different zone
- **Then** the system SHALL reject the reassignment with error "Cannot reassign to users outside your zone"
- **And** the original assignment SHALL remain unchanged

**Scenario 3: Reassignment history tracking**
- **Given** a task has been reassigned multiple times
- **When** a user views the task details
- **Then** the system SHALL display complete reassignment history with dates, users, and reasons
- **And** the system SHALL show current assignee prominently

---

### REQ-TASKS-004: Task Progress Tracking and Zone Completion Calculation

The system SHALL automatically calculate zone completion percentage based on completed tasks weighted by their estimated effort or priority. The system MUST update completion percentages in real-time when task statuses change to "Finalizada". Each zone SHALL contribute exactly 20% to the overall project completion, and the system SHALL display this information on the projection screen updated every 10 seconds.

**Escenarios:**

**Scenario 1: Zone completion calculation after task completion**
- **Given** Zone A has 10 tasks with 3 completed and 7 pending
- **When** another task is marked as "Finalizada"
- **Then** the system SHALL recalculate zone completion to 40% (4/10 tasks)
- **And** the system SHALL update the overall project completion percentage on the projection screen

**Scenario 2: Real-time projection screen update**
- **Given** the projection screen is displaying current zone progress
- **When** any task status changes to "Finalizada" in any zone
- **Then** the system SHALL update the projection screen within 10 seconds
- **And** the system SHALL highlight the zone that had progress updated

**Scenario 3: Weighted task completion calculation**
- **Given** Zone B has tasks with different priority levels (High=3 points, Medium=2 points, Low=1 point)
- **When** a High priority task is completed
- **Then** the system SHALL calculate completion percentage based on weighted points
- **And** the system SHALL reflect the weighted progress in zone completion metrics

---

### REQ-TASKS-005: Task Deadline Management and Notifications

The system SHALL monitor task due dates and automatically send reminder notifications 24 hours and 2 hours before deadline expiration. The system MUST identify overdue tasks and highlight them in red on all dashboards. Coordinators SHALL receive daily digest emails with overdue and approaching deadline tasks for their zones. The system SHALL prevent task completion if the deadline has passed without explicit Coordinator approval.

**Escenarios:**

**Scenario 1: Automatic reminder notifications**
- **Given** a task has due date set to tomorrow at 14:00
- **When** the system time reaches today at 14:00 (24 hours before)
- **Then** the system SHALL send reminder notification to assigned user via email and in-app
- **And** the system SHALL send another reminder 2 hours before deadline

**Scenario 2: Overdue task identification**
- **Given** a task with due date of yesterday and status "En curso"
- **When** the system performs its daily overdue check
- **Then** the system SHALL mark the task as overdue and highlight it in red
- **And** the system SHALL include it in the Coordinator's daily digest email

**Scenario 3: Overdue task completion approval**
- **Given** an overdue task that user attempts to mark as "Finalizada"
- **When** the user submits the completion without Coordinator approval
- **Then** the system SHALL require explicit Coordinator approval before allowing completion
- **And** the system SHALL notify the Coordinator of the overdue completion request

---

### REQ-TASKS-006: Task Filtering and Search Functionality

The system SHALL provide comprehensive filtering options for tasks including status, assignee, due date range, zone, and priority level. The system MUST support full-text search across task titles and descriptions with results highlighted. Users SHALL be able to save custom filter combinations as "views" for quick access. The system SHALL display task counts for each filter category to provide immediate feedback on data volume.

**Escenarios:**

**Scenario 1: Multi-criteria task filtering**
- **Given** a user is viewing the tasks dashboard with 100+ tasks
- **When** they apply filters for "En curso" status, "Zone A", and due date "This week"
- **Then** the system SHALL display only tasks matching all criteria
- **And** the system SHALL show task count "Showing 15 of 100 tasks" at the top

**Scenario 2: Full-text search with highlighting**
- **Given** a user enters "database migration" in the search box
- **When** they execute the search
- **Then** the system SHALL return tasks containing those terms in title or description
- **And** the system SHALL highlight the matching text in yellow in the results

**Scenario 3: Custom view creation and retrieval**
- **Given** a user has applied specific filters (overdue tasks in their zone)
- **When** they click "Save as View" and name it "My Overdue Tasks"
- **Then** the system SHALL save the filter combination under that name
- **And** the system SHALL make it available in a "Saved Views" dropdown for quick access

---

### REQ-TASKS-007: Task Comments and Collaboration System

The system SHALL allow users to add timestamped comments to tasks for collaboration and progress updates. The system MUST notify task assignees and Coordinators when new comments are added. Comments SHALL support basic formatting (bold, italic, links) and file attachments via Google Drive integration. The system SHALL maintain a chronological comment thread with user avatars and timestamps for each task.

**Escenarios:**

**Scenario 1: Adding comment with notification**
- **Given** a task is assigned to User A and User B adds a comment
- **When** User B submits comment "Please review the database schema before proceeding"
- **Then** the system SHALL save the comment with timestamp and User B's information
- **And** the system SHALL notify User A and the zone Coordinator via email and in-app notification

**Scenario 2: Comment with file attachment**
- **Given** a user is adding a comment to a task
- **When** they attach a file from Google Drive and submit the comment
- **Then** the system SHALL embed the Google Drive link in the comment
- **And** the system SHALL display file preview if supported (images, PDFs)

**Scenario 3: Comment thread chronological display**
- **Given** a task has multiple comments from different users over time
- **When** a user views the task details
- **Then** the system SHALL display comments in chronological order with user avatars
- **And** the system SHALL show relative timestamps ("2 hours ago", "yesterday")

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]