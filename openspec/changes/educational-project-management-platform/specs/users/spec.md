# Delta Spec: users — educational-project-management-platform

## ADDED Requirements

### REQ-USERS-001: User Authentication System

The system SHALL implement email/password authentication for all user types. The system SHALL validate email format using RFC 5322 standard and password complexity with minimum 8 characters including uppercase, lowercase, numbers and special characters. Authentication tokens SHALL expire after 24 hours of inactivity.

**Escenarios:**

**Scenario 1: Successful user login**
- **Given** a registered user with valid credentials exists in the system
- **When** the user enters correct email and password
- **Then** the system SHALL authenticate the user and redirect to their role-specific dashboard
- **And** the system SHALL generate a session token valid for 24 hours

**Scenario 2: Invalid credentials provided**
- **Given** a user attempts to login
- **When** the user enters incorrect email or password
- **Then** the system SHALL reject the authentication attempt
- **And** the system SHALL display "Invalid credentials" error message
- **And** the system SHALL NOT reveal whether email or password was incorrect

**Scenario 3: Account lockout after failed attempts**
- **Given** a user has failed authentication 5 consecutive times
- **When** the user attempts to login again
- **Then** the system SHALL lock the account for 15 minutes
- **And** the system SHALL display "Account temporarily locked" message

---

### REQ-USERS-002: Welcome Email Link System

The system SHALL generate unique welcome links for new users and send them via Gmail API integration. Welcome links SHALL be valid for 72 hours and allow first-time password setup. The system SHALL track link usage and prevent reuse of expired or already-used links.

**Escenarios:**

**Scenario 1: Welcome email sent successfully**
- **Given** an Administrator creates a new user account
- **When** the user creation process completes
- **Then** the system SHALL generate a unique welcome link with 72-hour expiration
- **And** the system SHALL send the welcome email via Gmail API within 30 seconds
- **And** the system SHALL log the email delivery status

**Scenario 2: User activates account via welcome link**
- **Given** a user receives a valid welcome email
- **When** the user clicks the welcome link within 72 hours
- **Then** the system SHALL redirect to password setup page
- **And** the system SHALL mark the link as used
- **And** the system SHALL activate the user account upon password creation

**Scenario 3: Expired welcome link accessed**
- **Given** a welcome link has expired after 72 hours
- **When** the user attempts to access the expired link
- **Then** the system SHALL display "Link expired" error message
- **And** the system SHALL provide option to request new welcome email
- **And** the system SHALL NOT allow account activation

---

### REQ-USERS-003: Multi-Role Permission System

The system SHALL implement four distinct user roles: Administrator, Director, Coordinador, and Integrante. Each role SHALL have specific permissions and access restrictions. The system SHALL enforce role-based access control on all endpoints and UI components. Role changes SHALL require Administrator approval and audit logging.

**Escenarios:**

**Scenario 1: Administrator full system access**
- **Given** a user with Administrator role is authenticated
- **When** the user accesses any system functionality
- **Then** the system SHALL grant full access to all modules and data
- **And** the system SHALL display administrative dashboard with system-wide metrics
- **And** the system SHALL allow user management and role assignment operations

**Scenario 2: Coordinador zone-specific access**
- **Given** a user with Coordinador role is authenticated
- **When** the user attempts to access project data
- **Then** the system SHALL grant access only to their assigned zone data
- **And** the system SHALL allow creation of months, milestones and deliverables for their zone
- **And** the system SHALL deny access to other zones' confidential information

**Scenario 3: Integrante limited task access**
- **Given** a user with Integrante role is authenticated
- **When** the user accesses the system
- **Then** the system SHALL display only tasks assigned to them
- **And** the system SHALL allow task status updates for assigned tasks only
- **And** the system SHALL deny access to administrative functions and other users' tasks

**Scenario 4: Unauthorized role escalation attempt**
- **Given** a user with Integrante role is authenticated
- **When** the user attempts to access Administrator functions via direct URL
- **Then** the system SHALL deny access and return 403 Forbidden error
- **And** the system SHALL log the unauthorized access attempt
- **And** the system SHALL redirect user to their authorized dashboard

---

### REQ-USERS-004: User Profile Management

The system SHALL allow users to manage their profile information including name, email, phone number, and zone assignment. Profile changes SHALL require email verification for email updates. The system SHALL maintain profile change history and notify relevant stakeholders of critical updates.

**Escenarios:**

**Scenario 1: Successful profile information update**
- **Given** an authenticated user accesses their profile page
- **When** the user updates their name and phone number
- **Then** the system SHALL validate the input data format
- **And** the system SHALL save the changes immediately
- **And** the system SHALL display success confirmation message

**Scenario 2: Email address change with verification**
- **Given** a user wants to change their email address
- **When** the user submits a new email address
- **Then** the system SHALL send verification email to the new address
- **And** the system SHALL keep the old email active until verification
- **And** the system SHALL update the email only after successful verification within 24 hours

**Scenario 3: Zone assignment change by Administrator**
- **Given** an Administrator wants to reassign a Coordinador to different zone
- **When** the Administrator changes the zone assignment
- **Then** the system SHALL update the user's zone access permissions immediately
- **And** the system SHALL notify the affected user via email
- **And** the system SHALL log the zone change in audit trail

---

### REQ-USERS-005: User Session Management

The system SHALL manage user sessions with automatic timeout after 24 hours of inactivity. The system SHALL support concurrent session detection and provide session termination capabilities. Active sessions SHALL be tracked and displayed to users for security monitoring.

**Escenarios:**

**Scenario 1: Session timeout after inactivity**
- **Given** a user has been inactive for 24 hours
- **When** the user attempts to perform any action
- **Then** the system SHALL terminate the session automatically
- **And** the system SHALL redirect to login page with timeout message
- **And** the system SHALL require re-authentication to continue

**Scenario 2: Multiple concurrent sessions detected**
- **Given** a user logs in from a new device while having active session elsewhere
- **When** the new login is successful
- **Then** the system SHALL allow both sessions to remain active
- **And** the system SHALL display active sessions list in user profile
- **And** the system SHALL provide option to terminate other sessions

**Scenario 3: Manual session termination**
- **Given** a user views their active sessions list
- **When** the user clicks "Terminate" on a specific session
- **Then** the system SHALL immediately invalidate that session
- **And** the system SHALL log the manual termination action
- **And** the system SHALL remove the session from active sessions display

---

### REQ-USERS-006: User Account Status Management

The system SHALL support multiple user account statuses: Active, Inactive, Suspended, and Pending Activation. Status changes SHALL be restricted to Administrator role and require justification comments. The system SHALL enforce access restrictions based on account status and maintain status change history.

**Escenarios:**

**Scenario 1: Administrator suspends user account**
- **Given** an Administrator identifies a user requiring suspension
- **When** the Administrator changes user status to Suspended with justification
- **Then** the system SHALL immediately terminate all active sessions for that user
- **And** the system SHALL prevent future login attempts
- **And** the system SHALL log the suspension with timestamp and reason

**Scenario 2: Suspended user attempts login**
- **Given** a user account has Suspended status
- **When** the user attempts to login with valid credentials
- **Then** the system SHALL deny authentication
- **And** the system SHALL display "Account suspended, contact administrator" message
- **And** the system SHALL NOT create any session or token

**Scenario 3: Account reactivation process**
- **Given** a user account has Suspended or Inactive status
- **When** an Administrator changes status back to Active
- **Then** the system SHALL restore full access permissions immediately
- **And** the system SHALL send reactivation notification email to the user
- **And** the system SHALL log the reactivation with Administrator details

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]