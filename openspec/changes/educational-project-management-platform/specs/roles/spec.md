# Delta Spec: roles — educational-project-management-platform

## ADDED Requirements

### REQ-ROLES-001: Role Definition and Hierarchy

The system SHALL define four distinct roles with hierarchical permissions: Administrator (highest), Director, Coordinador, and Integrante (lowest). Each role SHALL have specific capabilities and access restrictions within the educational project management platform.

**Escenarios:**

**Scenario 1: Role hierarchy validation**
- **Given** the system has defined the four roles with their hierarchy levels
- **When** a user attempts to access a function restricted to a higher role
- **Then** the system SHALL deny access and display an appropriate error message
- **And** the system SHALL log the unauthorized access attempt

**Scenario 2: Role assignment validation**
- **Given** an Administrator is assigning roles to users
- **When** the Administrator selects a valid role for a user
- **Then** the system SHALL assign the role successfully
- **And** the user SHALL inherit all permissions associated with that role

---

### REQ-ROLES-002: Administrator Role Permissions

The Administrator role SHALL have complete system access including user management, role assignment, system configuration, global reporting, and audit trail access. The Administrator MUST be able to override any system restriction and access all data across all zones.

**Escenarios:**

**Scenario 1: Complete system access**
- **Given** a user with Administrator role is logged in
- **When** the Administrator navigates to any system module
- **Then** the system SHALL grant full access to all functionalities
- **And** the system SHALL display all available options without restrictions

**Scenario 2: User role management**
- **Given** an Administrator is managing user roles
- **When** the Administrator attempts to assign or modify any user's role
- **Then** the system SHALL allow the role change
- **And** the system SHALL update the user's permissions immediately
- **And** the system SHALL log the role change with timestamp and Administrator ID

**Scenario 3: System configuration access**
- **Given** an Administrator needs to configure system parameters
- **When** the Administrator accesses configuration settings
- **Then** the system SHALL display all configurable parameters
- **And** the system SHALL allow modifications to risk parameters, time settings, and zone configurations

---

### REQ-ROLES-003: Director Role Permissions

The Director role SHALL have access to consolidated project views, approval workflows for actas de inicio, global Gantt charts, cross-zone reporting, and team coordination functionalities. The Director MUST NOT have user management or system configuration capabilities.

**Escenarios:**

**Scenario 1: Consolidated project oversight**
- **Given** a user with Director role is logged in
- **When** the Director accesses the project dashboard
- **Then** the system SHALL display consolidated information from all five zones
- **And** the system SHALL show overall project completion percentage
- **And** the system SHALL display cross-zone dependencies and critical path

**Scenario 2: Actas de inicio approval workflow**
- **Given** Coordinadores have submitted actas de inicio for approval
- **When** the Director reviews pending actas
- **Then** the system SHALL display all submitted actas with consolidation status
- **And** the Director SHALL be able to approve or reject each acta with comments
- **And** the system SHALL notify relevant Coordinadores of approval decisions

**Scenario 3: Access restriction validation**
- **Given** a Director attempts to access user management functions
- **When** the Director clicks on user administration
- **Then** the system SHALL deny access with message "Insufficient permissions"
- **And** the system SHALL redirect to the Director's authorized dashboard

---

### REQ-ROLES-004: Coordinador Role Permissions

The Coordinador role SHALL have zone-specific management capabilities including creation of meses/hitos/entregables, task assignment within their zone, actas de inicio generation, zone-specific Gantt view, and team member coordination. Each Coordinador MUST be assigned to exactly one zone and SHALL NOT access other zones' data.

**Escenarios:**

**Scenario 1: Zone-specific project management**
- **Given** a Coordinador is assigned to Zone 3
- **When** the Coordinador accesses project planning functions
- **Then** the system SHALL display only Zone 3 data and options
- **And** the system SHALL allow creation of meses, hitos, and entregables for Zone 3
- **And** the system SHALL calculate Zone 3's contribution to overall project completion (20%)

**Scenario 2: Task assignment within zone**
- **Given** a Coordinador has Integrantes assigned to their zone
- **When** the Coordinador creates or assigns tasks
- **Then** the system SHALL display only Integrantes from the Coordinador's zone
- **And** the system SHALL allow task assignment, reassignment, and status updates
- **And** the system SHALL send notifications to assigned Integrantes

**Scenario 3: Cross-zone access restriction**
- **Given** a Coordinador from Zone 2 attempts to view Zone 4 data
- **When** the Coordinador tries to access Zone 4 information
- **Then** the system SHALL deny access with message "Access restricted to assigned zone only"
- **And** the system SHALL log the unauthorized access attempt

---

### REQ-ROLES-005: Integrante Role Permissions

The Integrante role SHALL have limited access focused on task execution, status updates, file uploads for assigned tasks, and viewing zone-specific information. Integrantes MUST NOT create tasks, assign work to others, or access administrative functions.

**Escenarios:**

**Scenario 1: Task execution and status updates**
- **Given** an Integrante has assigned tasks
- **When** the Integrante accesses their task list
- **Then** the system SHALL display only tasks assigned to that Integrante
- **And** the Integrante SHALL be able to update task status through the workflow (En curso → En revisión → Finalizada)
- **And** the system SHALL notify the assigning Coordinador of status changes

**Scenario 2: File upload for assigned tasks**
- **Given** an Integrante is working on a task requiring deliverables
- **When** the Integrante uploads files related to the task
- **Then** the system SHALL accept the file upload to the zone's Google Drive folder
- **And** the system SHALL associate the file with the specific task and Integrante
- **And** the system SHALL notify the Coordinador of the file submission

**Scenario 3: Administrative function restriction**
- **Given** an Integrante attempts to create new tasks
- **When** the Integrante looks for task creation options
- **Then** the system SHALL NOT display task creation buttons or menus
- **And** if accessed directly via URL, the system SHALL return "Insufficient permissions" error

---

### REQ-ROLES-006: Role-Based Authentication and Session Management

The system SHALL authenticate users based on email/password credentials and maintain role-based sessions with appropriate timeouts. Each user session MUST include role information and zone assignment (for Coordinadores and Integrantes).

**Escenarios:**

**Scenario 1: Successful role-based authentication**
- **Given** a user provides valid email and password credentials
- **When** the system validates the credentials
- **Then** the system SHALL create a session with the user's role and permissions
- **And** the system SHALL redirect to the appropriate role-based dashboard
- **And** the session SHALL include zone assignment if applicable

**Scenario 2: Session timeout handling**
- **Given** a user has been inactive for the configured timeout period
- **When** the user attempts to perform any action
- **Then** the system SHALL terminate the session
- **And** the system SHALL redirect to the login page with message "Session expired"
- **And** the system SHALL require re-authentication to continue

**Scenario 3: Invalid credentials handling**
- **Given** a user provides incorrect email or password
- **When** the system attempts authentication
- **Then** the system SHALL reject the login attempt
- **And** the system SHALL display "Invalid credentials" message
- **And** the system SHALL log the failed login attempt with timestamp and IP address

---

### REQ-ROLES-007: Dynamic Permission Enforcement

The system SHALL enforce role-based permissions dynamically across all modules and SHALL prevent privilege escalation attempts. Permission checks MUST occur on every request and SHALL be validated server-side.

**Escenarios:**

**Scenario 1: Real-time permission validation**
- **Given** a user is navigating through the application
- **When** the user accesses any functionality or data
- **Then** the system SHALL validate current role permissions before granting access
- **And** the system SHALL deny access if permissions are insufficient
- **And** the validation SHALL occur server-side regardless of client-side restrictions

**Scenario 2: Privilege escalation prevention**
- **Given** a user attempts to modify their session to gain higher privileges
- **When** the system detects session tampering or privilege escalation attempts
- **Then** the system SHALL immediately terminate the session
- **And** the system SHALL log the security violation with full details
- **And** the system SHALL require fresh authentication

**Scenario 3: Module-specific permission enforcement**
- **Given** different modules require different permission levels
- **When** a user accesses reporting, task management, or configuration modules
- **Then** the system SHALL apply module-specific permission rules based on user role
- **And** the system SHALL display only authorized functions within each module
- **And** unauthorized functions SHALL be completely hidden from the user interface

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]