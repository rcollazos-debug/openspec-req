# Tasks: educational-project-management-platform

## Progress
- Total: 32 tasks
- Completed: 0 / 32

---

## 1. Data Model & Database Setup

- [ ] **1.1** Create PostgreSQL database schema with domain separation
  - **Description:** Set up PostgreSQL database with separate schemas for each of the 11 domains (auth, users, roles, notifications, projects, tasks, risks, files, reports, audit, api). Create connection pooling configuration and establish database indexes for performance optimization.
  - **Files:** `database/migrations/001_create_schemas.sql`, `database/migrations/002_create_indexes.sql`, `config/database.js`
  - **Acceptance criteria:**
    - [ ] All 11 domain schemas are created with proper naming conventions
    - [ ] Connection pooling is configured for minimum 30 concurrent users
    - [ ] Primary indexes are created on all foreign key relationships

- [ ] **1.2** Implement Users and Authentication tables
  - **Description:** Create users table with email, password_hash, name, role, zone_id, status, and timestamps. Implement authentication-related tables for sessions, welcome_links, and password_resets with proper constraints and foreign key relationships.
  - **Files:** `database/migrations/003_users_auth_tables.sql`, `database/models/User.js`, `database/models/Session.js`
  - **Acceptance criteria:**
    - [ ] Users table supports all 4 roles (Administrador, Director, Coordinador, Integrante)
    - [ ] Email field has unique constraint and RFC 5322 validation
    - [ ] Password hashing uses bcrypt with minimum 12 salt rounds

- [ ] **1.3** Create Projects and Zones data model
  - **Description:** Implement projects table with duration_weeks, zone_count, current_month, status, and completion tracking. Create zones table with weight distribution (20% each), coordinator assignment, and completion percentage calculation fields.
  - **Files:** `database/migrations/004_projects_zones.sql`, `database/models/Project.js`, `database/models/Zone.js`
  - **Acceptance criteria:**
    - [ ] Projects support 1-10 zones with automatic 20% weight distribution for 5 zones
    - [ ] Zone completion percentage is calculated as DECIMAL(5,2) with CHECK constraint 0-100
    - [ ] Project lifecycle states are enforced: Planning → Active → Paused → Completed → Archived

- [ ] **1.4** Design Tasks workflow and state management tables
  - **Description:** Create tasks table with title, description, assignee_id, zone_id, status enum, due_date, and audit fields. Implement task_state_transitions table to enforce valid workflow: Creada → Asignada → En curso → En revisión → Finalizada.
  - **Files:** `database/migrations/005_tasks_workflow.sql`, `database/models/Task.js`, `database/models/TaskTransition.js`
  - **Acceptance criteria:**
    - [ ] Task status enum enforces exactly 5 valid states in correct order
    - [ ] State transition validation prevents invalid jumps (e.g., Creada → Finalizada)
    - [ ] Task reassignment history is preserved with timestamps and reasons

- [ ] **1.5** Implement Risks and Events data model
  - **Description:** Create risks table with probability percentages, impact levels, zone targeting, and execution parameters. Design risk_events table to log all triggered risks with timestamps, affected zones, and resolution status.
  - **Files:** `database/migrations/006_risks_events.sql`, `database/models/Risk.js`, `database/models/RiskEvent.js`
  - **Acceptance criteria:**
    - [ ] Risk probability is constrained between 1-100% with validation
    - [ ] Impact severity enum supports Low/Medium/High/Critical levels
    - [ ] Risk events maintain immutable execution history with cryptographic timestamps

- [ ] **1.6** Create Audit and Notifications tables
  - **Description:** Implement comprehensive audit_logs table with user_id, action_type, resource_type, old_values, new_values as JSONB, IP address, and session tracking. Create notifications table supporting both email and in-app delivery with status tracking.
  - **Files:** `database/migrations/007_audit_notifications.sql`, `database/models/AuditLog.js`, `database/models/Notification.js`
  - **Acceptance criteria:**
    - [ ] Audit logs are immutable with BIGSERIAL primary key and no UPDATE/DELETE permissions
    - [ ] JSONB fields support complex data structures for before/after state tracking
    - [ ] Notification status tracking supports queued, sent, delivered, failed states

---

## 2. Authentication & User Management Services

- [ ] **2.1** Implement JWT-based authentication service
  - **Description:** Create authentication service with email/password validation, JWT token generation with 24-hour expiration, refresh token mechanism, and account lockout after 5 failed attempts. Integrate bcrypt password hashing with minimum 12 salt rounds.
  - **Files:** `services/AuthService.js`, `middleware/authMiddleware.js`, `utils/tokenUtils.js`
  - **Acceptance criteria:**
    - [ ] JWT tokens expire after 24 hours with automatic refresh capability
    - [ ] Account lockout triggers after 5 consecutive failed login attempts for 15 minutes
    - [ ] Password complexity validation enforces 8+ characters with mixed case, numbers, special chars

- [ ] **2.2** Build welcome link generation and email delivery
  - **Description:** Implement welcome link generation with unique tokens valid for 48 hours, Gmail API integration for automated email delivery, and first-time password setup workflow. Include email template management and delivery status tracking.
  - **Files:** `services/WelcomeLinkService.js`, `services/EmailService.js`, `templates/welcomeEmail.html`
  - **Acceptance criteria:**
    - [ ] Welcome links expire after 48 hours or first use, whichever comes first
    - [ ] Gmail API integration sends emails within 30 seconds of user creation
    - [ ] Email delivery failures are logged and retry mechanism implemented

- [ ] **2.3** Create role-based permission system
  - **Description:** Implement role validation service supporting 4 roles with hierarchical permissions, zone-based access control for Coordinadores, and dynamic permission checking on all endpoints. Include role assignment audit logging.
  - **Files:** `services/RoleService.js`, `middleware/permissionMiddleware.js`, `config/permissions.js`
  - **Acceptance criteria:**
    - [ ] Coordinadores can only access their assigned zone data and functions
    - [ ] Directors have read access to all zones but cannot modify user roles
    - [ ] All permission checks are validated server-side with audit logging

- [ ] **2.4** Develop user profile and session management
  - **Description:** Build user profile management with email change verification, session tracking with automatic timeout, concurrent session limits (2 per user), and secure HTTP-only cookie implementation.
  - **Files:** `services/UserService.js`, `services/SessionService.js`, `middleware/sessionMiddleware.js`
  - **Acceptance criteria:**
    - [ ] Email changes require verification to new address before activation
    - [ ] Sessions automatically timeout after 8 hours of inactivity
    - [ ] Maximum 2 concurrent sessions per user with oldest session termination

---

## 3. Project Management & Business Logic

- [ ] **3.1** Build project creation and zone management service
  - **Description:** Implement project creation with configurable parameters (duration, zone count), automatic zone weight distribution (20% each for 5 zones), zone assignment to Coordinadores, and project lifecycle state management.
  - **Files:** `services/ProjectService.js`, `services/ZoneService.js`, `controllers/ProjectController.js`
  - **Acceptance criteria:**
    - [ ] Projects automatically create 5 zones with exactly 20% weight each
    - [ ] Zone assignment validates Coordinador role and prevents duplicate assignments
    - [ ] Project state transitions follow strict lifecycle: Planning → Active → Paused → Completed

- [ ] **3.2** Implement milestone and deliverable planning system
  - **Description:** Create milestone and deliverable management allowing Coordinadores to plan months (10-minute duration), create milestones with deliverables, and validate timeline constraints within month boundaries.
  - **Files:** `services/MilestoneService.js`, `services/DeliverableService.js`, `controllers/PlanningController.js`
  - **Acceptance criteria:**
    - [ ] Each month has exactly 10-minute duration representing 1 simulation month
    - [ ] Milestone dates must fall within their parent month boundaries
    - [ ] Deliverable completion drives milestone and zone completion calculations

- [ ] **3.3** Create real-time completion percentage calculation engine
  - **Description:** Implement automatic completion percentage calculation based on deliverable status, zone weight distribution, and real-time updates. Include validation that total project completion equals sum of zone contributions.
  - **Files:** `services/CompletionService.js`, `utils/calculationEngine.js`, `middleware/completionMiddleware.js`
  - **Acceptance criteria:**
    - [ ] Zone completion calculated as weighted average of deliverable completions
    - [ ] Overall project completion updates within 5 seconds of any deliverable change
    - [ ] Completion percentages are validated to ensure mathematical consistency

- [ ] **3.4** Build task management workflow engine
  - **Description:** Implement complete task workflow with state validation, assignment rules within zones, reassignment capabilities, and automatic notification triggers for state changes and deadline reminders.
  - **Files:** `services/TaskService.js`, `services/WorkflowService.js`, `controllers/TaskController.js`
  - **Acceptance criteria:**
    - [ ] Task state transitions strictly follow: Creada → Asignada → En curso → En revisión → Finalizada
    - [ ] Tasks can only be assigned to users within the same zone as creator
    - [ ] Deadline reminders sent automatically at 24 hours and 2 hours before due date

---

## 4. Risk Engine & Real-time Systems

- [ ] **4.1** Develop probabilistic risk execution engine
  - **Description:** Create risk engine with configurable probability parameters, cryptographically secure random number generation, automated execution every 10 minutes, and immediate impact application to affected zones.
  - **Files:** `services/RiskEngine.js`, `services/RandomService.js`, `schedulers/riskScheduler.js`
  - **Acceptance criteria:**
    - [ ] Risk execution runs automatically every 10 minutes using cron scheduling
    - [ ] Probability calculations use cryptographically secure random number generation
    - [ ] Risk impacts are applied immediately to zone completion percentages

- [ ] **4.2** Implement real-time projection screen service
  - **Description:** Build WebSocket-based projection screen service with countdown timer, zone completion display, risk event notifications, and automatic printing triggers every 5 minutes at month completion.
  - **Files:** `services/ProjectionService.js`, `websockets/projectionSocket.js`, `services/PrintService.js`
  - **Acceptance criteria:**
    - [ ] Projection screen updates every 10 seconds with current project status
    - [ ] Automatic printing triggers every 5 minutes when simulation month completes
    - [ ] WebSocket connections support automatic reconnection with state recovery

- [ ] **4.3** Create notification delivery system with Gmail integration
  - **Description:** Implement comprehensive notification system with Gmail API integration, email template management, in-app notifications, delivery status tracking, and retry mechanisms for failed deliveries.
  - **Files:** `services/NotificationService.js`, `services/GmailService.js`, `templates/notificationTemplates.js`
  - **Acceptance criteria:**
    - [ ] Email notifications delivered within 30 seconds via Gmail API
    - [ ] Failed email deliveries retry with exponential backoff up to 3 attempts
    - [ ] In-app notifications update in real-time via WebSocket connections

---

## 5. Google Drive & File Management

- [ ] **5.1** Build Google Drive API integration service
  - **Description:** Implement OAuth 2.0 authentication with Google Drive API, automatic token refresh, zone-based folder creation, and file permission management. Include error handling for API rate limits and connectivity issues.
  - **Files:** `services/GoogleDriveService.js`, `services/OAuth2Service.js`, `config/googleApi.js`
  - **Acceptance criteria:**
    - [ ] OAuth 2.0 tokens refresh automatically without user intervention
    - [ ] Zone folders created with appropriate read/write permissions per role
    - [ ] API rate limiting handled with exponential backoff and queuing

- [ ] **5.2** Implement file upload and versioning system
  - **Description:** Create file upload system with zone-based organization, automatic versioning, file type validation, and integration with task deliverables. Include file search and retrieval capabilities.
  - **Files:** `services/FileService.js`, `controllers/FileController.js`, `middleware/fileValidation.js`
  - **Acceptance criteria:**
    - [ ] Files automatically organized in zone-specific Google Drive folders
    - [ ] File versioning maintains history with timestamp suffixes
    - [ ] File uploads validate type and size limits with clear error messages

- [ ] **5.3** Create meeting minutes document generation
  - **Description:** Implement automated meeting minutes generation using Google Docs API, template population with participant data, and consolidation workflow for Director approval.
  - **Files:** `services/MeetingMinutesService.js`, `templates/meetingMinutesTemplate.js`, `services/DocumentService.js`
  - **Acceptance criteria:**
    - [ ] Meeting minutes auto-generated in Google Docs with participant information
    - [ ] Documents saved in zone "Actas" folders with standardized naming convention
    - [ ] Consolidation workflow allows Director approval of all zone minutes

---

## 6. API Endpoints & Controllers

- [ ] **6.1** Implement authentication and user management endpoints
  - **Description:** Create RESTful API endpoints for login, logout, welcome link generation, user profile management, and role assignment. Include proper HTTP status codes, error handling, and request validation.
  - **Files:** `routes/auth.js`, `routes/users.js`, `controllers/AuthController.js`, `controllers/UserController.js`
  - **Acceptance criteria:**
    - [ ] Authentication endpoints return proper JWT tokens and HTTP status codes
    - [ ] User management endpoints enforce role-based access control
    - [ ] All endpoints include comprehensive error handling and validation

- [ ] **6.2** Build project and zone management API endpoints
  - **Description:** Implement API endpoints for project creation, zone assignment, milestone planning, deliverable management, and completion percentage retrieval with real-time updates.
  - **Files:** `routes/projects.js`, `routes/zones.js`, `controllers/ProjectController.js`, `controllers/ZoneController.js`
  - **Acceptance criteria:**
    - [ ] Project endpoints support full CRUD operations with proper validation
    - [ ] Zone endpoints restrict access based on Coordinador assignments
    - [ ] Completion percentage endpoints return real-time calculated values

- [ ] **6.3** Create task management and workflow API endpoints
  - **Description:** Build comprehensive task API with creation, assignment, state transitions, filtering, and search capabilities. Include task comment system and file attachment support.
  - **Files:** `routes/tasks.js`, `controllers/TaskController.js`, `middleware/taskValidation.js`
  - **Acceptance criteria:**
    - [ ] Task endpoints enforce workflow state transition rules
    - [ ] Task filtering supports multiple criteria (status, assignee, zone, due date)
    - [ ] Task comments support file attachments via Google Drive integration

- [ ] **6.4** Implement reporting and analytics API endpoints
  - **Description:** Create reporting endpoints for dashboard metrics, PDF generation, Gantt chart data, audit trail retrieval, and historical data export with role-based filtering.
  - **Files:** `routes/reports.js`, `controllers/ReportController.js`, `services/PdfService.js`
  - **Acceptance criteria:**
    - [ ] Dashboard endpoints return role-specific metrics and visualizations
    - [ ] PDF reports generate within 10 seconds with comprehensive project data
    - [ ] Audit trail endpoints support filtering and pagination for large datasets

---

## 7. Frontend Web Application

- [ ] **7.1** Build responsive authentication and dashboard interface
  - **Description:** Create responsive web interface with login/logout functionality, role-based dashboards, and navigation menus. Implement real-time updates via WebSocket connections and mobile-responsive design.
  - **Files:** `frontend/src/components/Auth/`, `frontend/src/components/Dashboard/`, `frontend/src/styles/responsive.css`
  - **Acceptance criteria:**
    - [ ] Interface is fully responsive and works on tablets and mobile devices
    - [ ] Role-based dashboards show appropriate functionality per user type
    - [ ] Real-time updates display without page refresh using WebSocket connections

- [ ] **7.2** Implement project planning and task management interface
  - **Description:** Build user interfaces for project planning, milestone creation, task management, and Gantt chart visualization. Include drag-and-drop functionality and real-time collaboration features.
  - **Files