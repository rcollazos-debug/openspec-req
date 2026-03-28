# Delta Spec: api — educational-project-management-platform

## ADDED Requirements

### REQ-API-001: Authentication Endpoint Implementation

The system SHALL implement RESTful authentication endpoints supporting email/password login and automated welcome link generation. The API SHALL return JWT tokens with 24-hour expiration and refresh token capabilities. Authentication endpoints MUST integrate with Gmail API for automated welcome email delivery.

**Escenarios:**

**Scenario 1: Successful user authentication**
- **Given** a registered user with valid email "coordinator@uao.edu.co" and password "SecurePass123"
- **When** the user sends POST request to "/api/auth/login" with valid credentials
- **Then** the API SHALL return HTTP 200 with JWT access token and refresh token
- **And** the token SHALL be valid for 24 hours

**Scenario 2: Invalid credentials authentication attempt**
- **Given** a user with email "coordinator@uao.edu.co" and incorrect password "WrongPass"
- **When** the user sends POST request to "/api/auth/login" with invalid credentials
- **Then** the API SHALL return HTTP 401 with error message "Invalid credentials"
- **And** no authentication tokens SHALL be generated

**Scenario 3: Welcome link generation and email delivery**
- **Given** an administrator creates a new user account for "student@uao.edu.co"
- **When** the system generates a welcome link via POST "/api/auth/welcome-link"
- **Then** the API SHALL create a unique token valid for 48 hours
- **And** the system SHALL send automated email via Gmail API with welcome link

---

### REQ-API-002: Role-Based Access Control Endpoints

The system SHALL implement role-based API endpoints with granular permission validation for four roles: Administrador, Director, Coordinadores, and Integrantes. Each API endpoint MUST validate user permissions before processing requests and return appropriate HTTP status codes for unauthorized access attempts.

**Escenarios:**

**Scenario 1: Coordinator accessing zone management endpoints**
- **Given** a user with "Coordinador" role and valid JWT token for zone "Zona-A"
- **When** the user sends GET request to "/api/zones/zona-a/milestones"
- **Then** the API SHALL return HTTP 200 with zone-specific milestone data
- **And** the response SHALL include only milestones assigned to "Zona-A"

**Scenario 2: Unauthorized role attempting restricted operation**
- **Given** a user with "Integrante" role attempting administrative operation
- **When** the user sends POST request to "/api/admin/users" to create new user
- **Then** the API SHALL return HTTP 403 with error message "Insufficient permissions"
- **And** no user creation operation SHALL be executed

**Scenario 3: Director accessing consolidated project data**
- **Given** a user with "Director" role and valid authentication token
- **When** the user sends GET request to "/api/projects/consolidated-view"
- **Then** the API SHALL return HTTP 200 with aggregated data from all 5 zones
- **And** the response SHALL include cumulative completion percentages

---

### REQ-API-003: Project Planning and Zone Management Endpoints

The system SHALL provide RESTful endpoints for creating and managing months, milestones, and deliverables by zone. The API MUST automatically calculate completion percentages where each zone represents 20% of total project completion. Zone-specific endpoints SHALL enforce coordinator permissions and validate zone assignments.

**Escenarios:**

**Scenario 1: Coordinator creating milestone for assigned zone**
- **Given** a coordinator user authenticated for "Zona-B" with 2 existing milestones
- **When** the user sends POST request to "/api/zones/zona-b/milestones" with milestone data
- **Then** the API SHALL create the milestone and return HTTP 201 with milestone ID
- **And** the system SHALL recalculate zone completion percentage automatically

**Scenario 2: Invalid zone assignment attempt**
- **Given** a coordinator assigned to "Zona-A" attempting to modify "Zona-C"
- **When** the user sends PUT request to "/api/zones/zona-c/milestones/123"
- **Then** the API SHALL return HTTP 403 with error message "Zone access denied"
- **And** no modifications SHALL be applied to "Zona-C" data

**Scenario 3: Completion percentage calculation validation**
- **Given** 5 zones with milestones: Zone-A (80% complete), Zone-B (60% complete), Zone-C (40% complete), Zone-D (100% complete), Zone-E (20% complete)
- **When** the system calculates overall project completion via GET "/api/projects/completion"
- **Then** the API SHALL return HTTP 200 with total completion of 60% (average of all zones)
- **And** each zone contribution SHALL be weighted at 20% of total project

---

### REQ-API-004: Task Management Workflow Endpoints

The system SHALL implement comprehensive task management endpoints supporting the complete workflow: Creada → Asignada → En curso → En revisión → Finalizada. The API MUST validate state transitions, enforce assignment rules, and provide task filtering capabilities by status, assignee, and zone.

**Escenarios:**

**Scenario 1: Task creation and assignment workflow**
- **Given** a coordinator creates a task in "Creada" status for "Zona-A"
- **When** the coordinator sends POST request to "/api/tasks" with task details and assignee
- **Then** the API SHALL create task with status "Asignada" and return HTTP 201
- **And** the system SHALL send notification to assigned user via notification endpoints

**Scenario 2: Invalid task state transition attempt**
- **Given** a task in "Creada" status with ID "task-456"
- **When** a user attempts to change status directly to "Finalizada" via PUT "/api/tasks/task-456/status"
- **Then** the API SHALL return HTTP 400 with error message "Invalid state transition"
- **And** the task status SHALL remain unchanged as "Creada"

**Scenario 3: Task reassignment with status validation**
- **Given** a task in "En curso" status assigned to user "student1@uao.edu.co"
- **When** a coordinator sends PUT request to "/api/tasks/task-789/reassign" with new assignee
- **Then** the API SHALL change assignee and reset status to "Asignada"
- **And** both old and new assignees SHALL receive notification via integrated notification system

---

### REQ-API-005: Real-Time Projection Screen Data Endpoints

The system SHALL provide real-time data endpoints for projection screen display including countdown timer, current month, zone completion percentages, and automatic printing triggers every 5 minutes at month completion. The API MUST support WebSocket connections for live updates and maintain projection state consistency.

**Escenarios:**

**Scenario 1: Real-time projection data retrieval**
- **Given** the projection screen is active and connected via WebSocket
- **When** the system updates zone completion percentages during active month
- **Then** the API SHALL broadcast updated data to all connected projection clients within 2 seconds
- **And** the projection display SHALL show current month, countdown timer, and all 5 zone percentages

**Scenario 2: Automatic month completion and printing trigger**
- **Given** the current simulated month timer reaches zero (10 minutes elapsed)
- **When** the month completion event occurs
- **Then** the API SHALL trigger automatic printing via POST "/api/projection/print"
- **And** the system SHALL advance to next month and reset countdown timer to 10 minutes

**Scenario 3: Projection screen connection failure recovery**
- **Given** the projection screen WebSocket connection is lost during active session
- **When** the connection is re-established within 30 seconds
- **Then** the API SHALL immediately send current state data to restore projection display
- **And** no data loss SHALL occur during the disconnection period

---

### REQ-API-006: Risk Management and Random Execution Endpoints

The system SHALL implement risk management endpoints with configurable random execution parameters and real-time visualization integration. The API MUST support risk event creation, probability configuration, and automated execution triggers with immediate projection screen updates.

**Escenarios:**

**Scenario 1: Risk event configuration and execution**
- **Given** an administrator configures a risk with 30% probability for "Zona-C"
- **When** the system evaluates risks via scheduled POST "/api/risks/evaluate"
- **Then** the API SHALL execute the risk based on probability calculation
- **And** if executed, the risk impact SHALL be applied to zone metrics and displayed on projection screen

**Scenario 2: Risk impact application to zone completion**
- **Given** a risk event "Material Shortage" with -15% completion impact is executed for "Zona-B"
- **When** the risk is applied via POST "/api/risks/apply/risk-123"
- **Then** the API SHALL reduce "Zona-B" completion percentage by 15%
- **And** the updated completion SHALL be reflected in real-time on projection screen

**Scenario 3: Risk visualization and notification integration**
- **Given** a high-impact risk event is executed affecting multiple zones
- **When** the risk execution completes successfully
- **Then** the API SHALL send risk notification to all affected zone coordinators
- **And** the projection screen SHALL display risk alert banner for 30 seconds

---

### REQ-API-007: Document Management and Google Drive Integration Endpoints

The system SHALL provide document management endpoints with complete Google Drive API integration for file storage by zone. The API MUST support file upload, download, sharing, and folder organization with automatic zone-based folder creation and permission management.

**Escenarios:**

**Scenario 1: Zone-specific file upload to Google Drive**
- **Given** a coordinator for "Zona-D" uploads a deliverable document
- **When** the user sends POST request to "/api/files/upload" with zone parameter
- **Then** the API SHALL upload file to Google Drive in "Zona-D" folder
- **And** the system SHALL return file metadata with Google Drive sharing link

**Scenario 2: Automated folder creation for new zones**
- **Given** a new project is created with 5 zones
- **When** the system initializes project via POST "/api/projects/initialize"
- **Then** the API SHALL create 5 zone folders in Google Drive with appropriate permissions
- **And** each zone coordinator SHALL have read/write access to their respective folder only

**Scenario 3: File sharing and permission management**
- **Given** a file exists in "Zona-A" Google Drive folder
- **When** the Director requests access via GET "/api/files/zona-a/file-456"
- **Then** the API SHALL grant temporary read access and return file content
- **And** the access SHALL be logged in audit trail with timestamp and user information

---

### REQ-API-008: Notification System and Gmail Integration Endpoints

The system SHALL implement comprehensive notification endpoints with Gmail API integration for automated email delivery and in-app notification management. The API MUST support notification templates, delivery scheduling, and notification status tracking with guaranteed delivery confirmation.

**Escenarios:**

**Scenario 1: Automated task deadline notification**
- **Given** a task is due within 24 hours for user "student@uao.edu.co"
- **When** the system runs scheduled notification check via POST "/api/notifications/check-deadlines"
- **Then** the API SHALL send email notification via Gmail API and create in-app notification
- **And** the notification status SHALL be tracked as "sent" with delivery timestamp

**Scenario 2: Bulk notification for zone milestone completion**
- **Given** "Zona-E" completes a major milestone affecting all team members
- **When** the milestone completion triggers notification via POST "/api/notifications/milestone-complete"
- **Then** the API SHALL send notifications to all zone members and stakeholders
- **And** each notification SHALL be personalized based on recipient role and permissions

**Scenario 3: Notification delivery failure and retry mechanism**
- **Given** Gmail API returns temporary failure for notification delivery
- **When** the initial notification attempt fails with HTTP 429 rate limit error
- **Then** the API SHALL implement exponential backoff retry mechanism
- **And** the system SHALL attempt redelivery up to 3 times before marking as failed

---

### REQ-API-009: Reporting and Analytics Endpoints

The system SHALL provide comprehensive reporting endpoints with downloadable PDF generation, completion metrics, and historical data analysis. The API MUST support real-time dashboard data, Gantt chart visualization, and audit trail reporting with role-based data filtering.

**Escenarios:**

**Scenario 1: Administrator dashboard metrics retrieval**
- **Given** an administrator requests comprehensive project metrics
- **When** the user sends GET request to "/api/reports/admin-dashboard"
- **Then** the API SHALL return HTTP 200 with completion percentages, task statistics, and zone performance data
- **And** the response SHALL include historical trends and comparative analysis

**Scenario 2: PDF report generation and download**
- **Given** a Director requests consolidated project report
- **When** the user sends POST request to "/api/reports/generate-pdf" with date range parameters
- **Then** the API SHALL generate PDF report within 10 seconds and return download link
- **And** the PDF SHALL include Gantt charts, completion metrics, and audit trail summary

**Scenario 3: Zone-specific Gantt chart data for Coordinator**
- **Given** a coordinator requests Gantt visualization for their assigned zone
- **When** the user sends GET request to "/api/reports/gantt/zona-c"
- **Then** the API SHALL return HTTP 200 with zone-specific milestones, deliverables, and timeline data
- **And** the response SHALL include dependency relationships and critical path information

---

### REQ-API-010: Audit Trail and Change Tracking Endpoints

The system SHALL implement comprehensive audit trail endpoints capturing all user actions with detailed change tracking (who, what, when) for complete system accountability. The API MUST provide audit log retrieval, filtering capabilities, and immutable record storage with tamper-proof timestamps.

**Escenarios:**

**Scenario 1: Complete audit trail retrieval with filtering**
- **Given** an administrator requests audit logs for specific date range and user
- **When** the user sends GET request to "/api/audit/logs" with filter parameters
- **Then** the API SHALL return HTTP 200 with filtered audit records including user, action, timestamp, and affected entities
- **And** the response SHALL support pagination for large result sets

**Scenario 2: Real-time audit logging for deliverable changes**
- **Given** a coordinator modifies a deliverable status from "En progreso" to "Completado"
- **When** the change is submitted via PUT "/api/deliverables/deliv-789/status"
- **Then** the API SHALL immediately create audit record with user ID, timestamp, old value, and new value
- **And** the audit record SHALL be immutable and include IP address and session information

**Scenario 3: Audit trail integrity verification**
- **Given** audit records exist for the past 30 days of system activity
- **When** the system performs integrity check via POST "/api/audit/verify-integrity"
- **Then** the API SHALL validate all audit records for tampering and return verification status
- **And** any integrity violations SHALL be immediately reported to system administrators

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]