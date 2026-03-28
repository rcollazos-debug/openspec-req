# Delta Spec: audit — educational-project-management-platform

## ADDED Requirements

### REQ-AUDIT-001: Activity Logging System

The system SHALL log all user activities including authentication events, role changes, project modifications, task state transitions, file operations, and administrative actions. Each log entry MUST include timestamp, user identifier, action type, affected resource, previous state, new state, and IP address. The system SHALL store audit logs in a dedicated audit table with immutable records that CANNOT be modified or deleted by any user role.

**Escenarios:**

**Scenario 1: Successful activity logging**
- **Given** a user is authenticated in the system
- **When** the user performs any action (create task, modify deliverable, upload file, etc.)
- **Then** the system SHALL create an audit log entry with all required fields
- **And** the log entry SHALL be stored with a unique sequential identifier

**Scenario 2: System action logging**
- **Given** the system executes an automated process (risk execution, notification sending, percentage calculation)
- **When** the automated process completes
- **Then** the system SHALL log the action with "SYSTEM" as the user identifier
- **And** the log SHALL include the specific process name and affected resources

**Scenario 3: Failed action attempt logging**
- **Given** a user attempts an unauthorized action
- **When** the system denies the action due to insufficient permissions
- **Then** the system SHALL log the failed attempt with "PERMISSION_DENIED" status
- **And** the log SHALL include the attempted action and reason for denial

---

### REQ-AUDIT-002: Change History Tracking for Deliverables

The system SHALL maintain a complete change history for all deliverables showing who made changes, what was changed, when the change occurred, and the previous and new values. The change history MUST be accessible to Coordinators for their zone deliverables and to Directors for all zones. The system SHALL display change history in chronological order with the most recent changes first.

**Escenarios:**

**Scenario 1: Deliverable modification tracking**
- **Given** a Coordinator modifies a deliverable in their zone
- **When** the deliverable is saved with changes
- **Then** the system SHALL create a change history record showing the field modified, old value, new value, and timestamp
- **And** the change SHALL be visible in the deliverable's history view

**Scenario 2: Multiple field changes tracking**
- **Given** a user modifies multiple fields of a deliverable in a single save operation
- **When** the changes are committed
- **Then** the system SHALL create separate history records for each modified field
- **And** all records SHALL share the same timestamp and transaction identifier

**Scenario 3: Change history access control**
- **Given** a Coordinator attempts to view change history for a deliverable outside their zone
- **When** they access the history view
- **Then** the system SHALL deny access and display "Insufficient permissions" message
- **And** the access attempt SHALL be logged in the audit system

---

### REQ-AUDIT-003: User Session Tracking

The system SHALL track all user sessions including login time, logout time, session duration, IP address, and user agent. The system MUST automatically log session termination events whether initiated by user logout, session timeout, or system shutdown. Session data SHALL be retained for audit purposes and accessible only to Administrator role.

**Escenarios:**

**Scenario 1: Successful login session creation**
- **Given** a user provides valid credentials
- **When** authentication is successful
- **Then** the system SHALL create a session record with login timestamp, IP address, and user agent
- **And** the session SHALL be assigned a unique session identifier

**Scenario 2: Session timeout tracking**
- **Given** a user session has been inactive for the configured timeout period
- **When** the session expires
- **Then** the system SHALL update the session record with logout timestamp and "TIMEOUT" termination reason
- **And** the system SHALL log the session termination event

**Scenario 3: Manual logout tracking**
- **Given** an authenticated user clicks the logout button
- **When** the logout process completes
- **Then** the system SHALL update the session record with logout timestamp and "USER_LOGOUT" termination reason
- **And** the session SHALL be invalidated immediately

---

### REQ-AUDIT-004: Administrative Action Auditing

The system SHALL log all administrative actions performed by Administrator role including user management, role assignments, system configuration changes, risk parameter modifications, and report generation. Each administrative action log MUST include the specific action performed, affected user or resource, timestamp, and justification field when provided. The system SHALL send email notifications to a configured audit email address for critical administrative actions.

**Escenarios:**

**Scenario 1: User role modification auditing**
- **Given** an Administrator changes a user's role
- **When** the role change is saved
- **Then** the system SHALL log the action with previous role, new role, and affected user identifier
- **And** an email notification SHALL be sent to the audit email address

**Scenario 2: System configuration change auditing**
- **Given** an Administrator modifies risk parameters or system settings
- **When** the configuration is updated
- **Then** the system SHALL log each modified parameter with old and new values
- **And** the change SHALL include a mandatory justification field

**Scenario 3: Bulk administrative action auditing**
- **Given** an Administrator performs a bulk operation affecting multiple users
- **When** the bulk operation completes
- **Then** the system SHALL create individual audit records for each affected user
- **And** all records SHALL reference the same bulk operation identifier

---

### REQ-AUDIT-005: File Operation Auditing

The system SHALL log all file operations including uploads, downloads, deletions, and Google Drive synchronization events. Each file operation log MUST include the file name, operation type, user performing the action, timestamp, file size, and Google Drive file identifier when applicable. The system SHALL track file access patterns and generate alerts for unusual file activity.

**Escenarios:**

**Scenario 1: File upload auditing**
- **Given** a user uploads a file to their zone's Google Drive folder
- **When** the upload completes successfully
- **Then** the system SHALL log the upload with file name, size, Google Drive ID, and user identifier
- **And** the log SHALL include the zone associated with the upload

**Scenario 2: File download tracking**
- **Given** a user downloads a file from the system
- **When** the download is initiated
- **Then** the system SHALL log the download attempt with file identifier and user
- **And** the system SHALL update the log with success or failure status upon completion

**Scenario 3: Suspicious file activity detection**
- **Given** a user performs more than 10 file operations within 1 minute
- **When** the threshold is exceeded
- **Then** the system SHALL generate an alert and log the suspicious activity
- **And** the system SHALL temporarily restrict file operations for that user

---

### REQ-AUDIT-006: Data Integrity Verification

The system SHALL implement data integrity checks to verify that audit logs have not been tampered with. The system MUST generate cryptographic hashes for audit log entries and store them separately from the main audit table. The system SHALL provide an integrity verification function accessible only to Administrator role that validates all audit logs against their stored hashes.

**Escenarios:**

**Scenario 1: Audit log hash generation**
- **Given** a new audit log entry is created
- **When** the entry is saved to the database
- **Then** the system SHALL generate a SHA-256 hash of the log entry content
- **And** the hash SHALL be stored in a separate integrity verification table

**Scenario 2: Integrity verification execution**
- **Given** an Administrator initiates an integrity check
- **When** the verification process runs
- **Then** the system SHALL validate each audit log entry against its stored hash
- **And** the system SHALL generate a report showing any integrity violations found

**Scenario 3: Integrity violation detection**
- **Given** an audit log entry has been modified outside the application
- **When** the integrity verification runs
- **Then** the system SHALL identify the compromised entry and log the integrity violation
- **And** the system SHALL send an immediate alert to the Administrator

---

### REQ-AUDIT-007: Audit Log Retention and Archival

The system SHALL retain audit logs for a minimum of 1 year from the date of creation. The system MUST implement an automated archival process that moves logs older than 90 days to a separate archive table while maintaining accessibility for reporting purposes. The system SHALL provide export functionality for audit logs in CSV and JSON formats with date range filtering capabilities.

**Escenarios:**

**Scenario 1: Automated log archival**
- **Given** audit logs are older than 90 days
- **When** the daily archival process runs
- **Then** the system SHALL move the old logs to the archive table
- **And** the archived logs SHALL remain accessible through the audit interface

**Scenario 2: Audit log export**
- **Given** an Administrator requests audit log export for a specific date range
- **When** the export process is initiated
- **Then** the system SHALL generate a file containing all logs within the specified range
- **And** the export SHALL include both active and archived logs

**Scenario 3: Log retention policy enforcement**
- **Given** audit logs are older than 1 year
- **When** the annual cleanup process runs
- **Then** the system SHALL permanently delete logs older than the retention period
- **And** the deletion process SHALL be logged as an administrative action

## MODIFIED Requirements

[No existing requirements to modify]

## REMOVED Requirements

[No existing requirements to remove]