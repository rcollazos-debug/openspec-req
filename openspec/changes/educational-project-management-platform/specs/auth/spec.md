# Delta Spec: auth — educational-project-management-platform

## ADDED Requirements

### REQ-AUTH-001: User Authentication with Email and Password

The system SHALL implement email and password authentication for all user types. The system SHALL validate email format according to RFC 5322 standard and password complexity requirements. The system MUST store passwords using bcrypt hashing with minimum salt rounds of 12. The system SHALL implement account lockout after 5 consecutive failed login attempts for 15 minutes.

**Escenarios:**

**Scenario 1: Successful user login**
- **Given** a registered user with valid credentials exists in the system
- **When** the user submits correct email and password
- **Then** the system SHALL authenticate the user and redirect to role-appropriate dashboard
- **And** the system SHALL create a session token valid for 8 hours

**Scenario 2: Invalid credentials provided**
- **Given** a user attempts to login
- **When** the user submits incorrect email or password
- **Then** the system SHALL reject the authentication attempt
- **And** the system SHALL increment failed login counter for that email
- **And** the system SHALL display generic error message "Invalid credentials"

**Scenario 3: Account lockout after multiple failures**
- **Given** a user has failed login 4 times consecutively
- **When** the user attempts to login with incorrect credentials for the 5th time
- **Then** the system SHALL lock the account for 15 minutes
- **And** the system SHALL display message "Account temporarily locked due to multiple failed attempts"

---

### REQ-AUTH-002: Welcome Email Link Authentication

The system SHALL generate unique welcome links sent via Gmail API for new user registration. The system MUST create time-limited tokens valid for 24 hours embedded in welcome URLs. The system SHALL allow users to set their password through the welcome link without prior authentication. The welcome link MUST expire after first use or 24 hours, whichever comes first.

**Escenarios:**

**Scenario 1: Successful welcome link access**
- **Given** an Administrator has created a new user account
- **When** the new user clicks the welcome link within 24 hours
- **Then** the system SHALL display password creation form
- **And** the system SHALL validate the token and allow password setup

**Scenario 2: Expired welcome link access**
- **Given** a welcome link was generated more than 24 hours ago
- **When** the user attempts to access the expired link
- **Then** the system SHALL display "Link expired" error message
- **And** the system SHALL redirect to contact administrator page

**Scenario 3: Already used welcome link**
- **Given** a user has already set password using welcome link
- **When** the same user attempts to access the welcome link again
- **Then** the system SHALL display "Link already used" message
- **And** the system SHALL redirect to login page

---

### REQ-AUTH-003: Role-Based Access Control System

The system SHALL implement four distinct user roles: Administrador, Director, Coordinador, and Integrante. The system MUST enforce role-specific permissions for all system functionalities. The system SHALL prevent privilege escalation and unauthorized access to restricted features. Each user MUST be assigned exactly one role during account creation.

**Escenarios:**

**Scenario 1: Administrador accessing all system features**
- **Given** a user is authenticated with Administrador role
- **When** the user attempts to access any system functionality
- **Then** the system SHALL grant access to all features including user management, reports, and system configuration
- **And** the system SHALL display full administrative dashboard

**Scenario 2: Coordinador accessing zone-specific features**
- **Given** a user is authenticated with Coordinador role
- **When** the user attempts to access zone management features
- **Then** the system SHALL grant access only to their assigned zone functionalities
- **And** the system SHALL restrict access to other zones and administrative features

**Scenario 3: Unauthorized role access attempt**
- **Given** a user is authenticated with Integrante role
- **When** the user attempts to access Coordinador-only features via direct URL
- **Then** the system SHALL deny access and return HTTP 403 Forbidden
- **And** the system SHALL log the unauthorized access attempt with user ID and timestamp

---

### REQ-AUTH-004: Session Management and Security

The system SHALL implement secure session management with automatic timeout after 8 hours of inactivity. The system MUST generate cryptographically secure session tokens using UUID v4 standard. The system SHALL implement session invalidation on logout and concurrent session limits of 2 per user. The system MUST store session data server-side with secure HTTP-only cookies.

**Escenarios:**

**Scenario 1: Automatic session timeout**
- **Given** a user is authenticated and inactive for 8 hours
- **When** the user attempts to perform any system action
- **Then** the system SHALL invalidate the session and redirect to login page
- **And** the system SHALL display "Session expired, please login again" message

**Scenario 2: Successful user logout**
- **Given** a user is authenticated with active session
- **When** the user clicks logout button
- **Then** the system SHALL invalidate the current session token
- **And** the system SHALL clear all session cookies and redirect to login page

**Scenario 3: Concurrent session limit exceeded**
- **Given** a user already has 2 active sessions
- **When** the user attempts to login from a third device
- **Then** the system SHALL terminate the oldest session
- **And** the system SHALL create new session for current login attempt
- **And** the system SHALL notify user about session displacement

---

### REQ-AUTH-005: Password Security and Recovery

The system SHALL enforce password complexity requirements: minimum 8 characters, at least one uppercase, one lowercase, one number, and one special character. The system MUST implement password recovery via email with time-limited reset tokens valid for 1 hour. The system SHALL prevent password reuse of last 5 passwords per user. The system MUST force password change every 90 days for Administrador role.

**Escenarios:**

**Scenario 1: Password complexity validation success**
- **Given** a user is setting or changing password
- **When** the user submits password "MySecure123!"
- **Then** the system SHALL accept the password as it meets all complexity requirements
- **And** the system SHALL hash and store the password securely

**Scenario 2: Password complexity validation failure**
- **Given** a user is setting password
- **When** the user submits password "simple"
- **Then** the system SHALL reject the password
- **And** the system SHALL display specific error messages for each missing requirement

**Scenario 3: Password recovery process**
- **Given** a user has forgotten their password
- **When** the user submits password recovery request with valid email
- **Then** the system SHALL send password reset link via Gmail API
- **And** the system SHALL create time-limited reset token valid for 1 hour

---

### REQ-AUTH-006: Integration with Gmail API for Authentication Emails

The system SHALL integrate with Gmail API to send authentication-related emails including welcome links, password resets, and security notifications. The system MUST use OAuth 2.0 service account authentication for Gmail API access. The system SHALL implement email template management for consistent branding and messaging. The system MUST handle Gmail API rate limits and implement retry logic with exponential backoff.

**Escenarios:**

**Scenario 1: Successful welcome email delivery**
- **Given** an Administrator creates a new user account
- **When** the system attempts to send welcome email via Gmail API
- **Then** the system SHALL successfully deliver email with welcome link
- **And** the system SHALL log successful email delivery with timestamp

**Scenario 2: Gmail API rate limit exceeded**
- **Given** the system has reached Gmail API hourly rate limit
- **When** the system attempts to send authentication email
- **Then** the system SHALL implement exponential backoff retry strategy
- **And** the system SHALL queue email for delivery when rate limit resets

**Scenario 3: Gmail API authentication failure**
- **Given** the Gmail API service account credentials are invalid
- **When** the system attempts to send any authentication email
- **Then** the system SHALL log authentication error and alert system administrator
- **And** the system SHALL display user-friendly error message about email delivery delay

---

### REQ-AUTH-007: Multi-Factor Authentication for Administrator Role

The system SHALL implement optional Time-based One-Time Password (TOTP) multi-factor authentication for Administrador role users. The system MUST support standard TOTP applications like Google Authenticator and Authy. The system SHALL provide QR code generation for easy MFA setup. The system MAY allow MFA bypass codes for emergency access with single-use tokens.

**Escenarios:**

**Scenario 1: Successful MFA setup**
- **Given** an Administrador user wants to enable MFA
- **When** the user accesses MFA setup in security settings
- **Then** the system SHALL generate QR code with TOTP secret
- **And** the system SHALL require verification of first TOTP code before enabling MFA

**Scenario 2: MFA verification during login**
- **Given** an Administrador user has MFA enabled
- **When** the user provides correct email/password and TOTP code
- **Then** the system SHALL authenticate user and grant full access
- **And** the system SHALL create session with MFA-verified flag

**Scenario 3: Invalid TOTP code provided**
- **Given** an Administrador user with MFA enabled attempts login
- **When** the user provides correct credentials but invalid TOTP code
- **Then** the system SHALL reject authentication attempt
- **And** the system SHALL increment failed MFA counter and display "Invalid authentication code" error

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]