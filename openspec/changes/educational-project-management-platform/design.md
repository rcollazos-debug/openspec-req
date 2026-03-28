# Design: educational-project-management-platform

## Overview

La plataforma educativa de gestión de proyectos implementa una arquitectura web moderna basada en microservicios con separación clara por dominios OpenSpec. El sistema utiliza PostgreSQL como base de datos principal con esquemas normalizados para 11 dominios (auth, users, roles, notifications, projects, tasks, risks, files, reports, audit, api), integración completa con Google APIs (Drive y Gmail) mediante OAuth 2.0, y comunicación en tiempo real vía WebSockets para la pantalla de proyección.

La arquitectura sigue el patrón MVC con una API REST como capa de servicios, frontend web responsive, y un motor de riesgos probabilístico que simula escenarios realistas de gestión de proyectos. El sistema maneja 30 usuarios concurrentes distribuidos en 5 zonas de construcción, cada una representando 20% del proyecto total, con progresión temporal acelerada donde 10 minutos reales equivalen a 1 mes del proyecto simulado.

El diseño prioriza la escalabilidad, seguridad mediante roles granulares, y trazabilidad completa de todas las acciones del usuario para fines educativos y de auditoría. La integración con Google Drive proporciona almacenamiento centralizado por zonas, mientras que Gmail API automatiza las notificaciones críticas del flujo de trabajo educativo.

## Architecture Decisions

### Decision 1: PostgreSQL como Base de Datos Principal
- **Context:** Necesidad de manejar relaciones complejas entre usuarios, roles, proyectos, tareas y auditoría con integridad transaccional
- **Decision:** Utilizar PostgreSQL con esquemas separados por dominio OpenSpec
- **Rationale:** PostgreSQL ofrece ACID compliance, soporte robusto para JSON, capacidades de auditoría avanzadas, y escalabilidad probada para 30+ usuarios concurrentes
- **Consequences:** Requiere configuración de connection pooling, pero proporciona consistencia de datos y capacidades de reporting avanzadas

### Decision 2: Arquitectura de Microservicios por Dominio
- **Context:** Sistema complejo con 11 dominios diferentes que requieren separación de responsabilidades
- **Decision:** Implementar arquitectura modular donde cada dominio OpenSpec tiene su propio módulo con interfaces bien definidas
- **Rationale:** Facilita mantenimiento, testing independiente, y escalabilidad granular por funcionalidad
- **Consequences:** Mayor complejidad inicial pero mejor mantenibilidad a largo plazo y capacidad de evolución independiente

### Decision 3: WebSockets para Comunicación en Tiempo Real
- **Context:** La pantalla de proyección requiere actualizaciones cada 10 segundos y los dashboards necesitan reflejar cambios inmediatamente
- **Decision:** Implementar WebSockets para comunicación bidireccional en tiempo real
- **Rationale:** WebSockets proporcionan latencia mínima, soporte nativo en navegadores modernos, y capacidad de push desde servidor
- **Consequences:** Requiere manejo de reconexión automática y estado de conexión, pero elimina polling innecesario

### Decision 4: OAuth 2.0 para Integración con Google APIs
- **Context:** Integración crítica con Google Drive y Gmail API para funcionalidad core del sistema
- **Decision:** Utilizar OAuth 2.0 con service accounts para autenticación automatizada
- **Rationale:** OAuth 2.0 es el estándar de la industria, proporciona tokens con expiración automática, y permite acceso programático sin intervención del usuario
- **Consequences:** Requiere manejo de refresh tokens y rate limiting, pero garantiza seguridad y cumplimiento de políticas de Google

### Decision 5: Motor de Riesgos Basado en Probabilidades
- **Context:** Necesidad de simular eventos aleatorios realistas para el aprendizaje de gestión de proyectos
- **Decision:** Implementar motor de riesgos con algoritmos probabilísticos y ejecución programada cada 10 minutos
- **Rationale:** Proporciona experiencia educativa realista, permite configuración flexible de escenarios, y mantiene impredecibilidad necesaria para el aprendizaje
- **Consequences:** Requiere generación de números aleatorios criptográficamente seguros y validación de integridad probabilística

## Component Design

### Authentication Service (Domain: auth)

**Responsabilidad:** Maneja autenticación de usuarios, generación de tokens JWT, y integración con Gmail para enlaces de bienvenida
**Interfaz:**
```typescript
interface AuthService {
  authenticate(email: string, password: string): Promise<AuthResult>
  generateWelcomeLink(userId: string): Promise<WelcomeLink>
  validateToken(token: string): Promise<TokenValidation>
  refreshToken(refreshToken: string): Promise<AuthResult>
  sendWelcomeEmail(email: string, welcomeLink: string): Promise<EmailResult>
}
```

### User Management Service (Domain: users)

**Responsabilidad:** Gestiona perfiles de usuario, asignación de zonas, y estados de cuenta
**Interfaz:**
```typescript
interface UserService {
  createUser(userData: CreateUserRequest): Promise<User>
  updateProfile(userId: string, updates: ProfileUpdate): Promise<User>
  assignToZone(userId: string, zoneId: string): Promise<Assignment>
  getUsersByZone(zoneId: string): Promise<User[]>
  updateAccountStatus(userId: string, status: AccountStatus): Promise<User>
}
```

### Role Management Service (Domain: roles)

**Responsabilidad:** Controla permisos basados en roles y valida acceso a recursos
**Interfaz:**
```typescript
interface RoleService {
  validatePermission(userId: string, resource: string, action: string): Promise<boolean>
  assignRole(userId: string, role: UserRole): Promise<RoleAssignment>
  getRolePermissions(role: UserRole): Promise<Permission[]>
  checkZoneAccess(userId: string, zoneId: string): Promise<boolean>
}
```

### Project Management Service (Domain: projects)

**Responsabilidad:** Gestiona creación de proyectos, zonas, meses, hitos y cálculo de porcentajes de cumplimiento
**Interfaz:**
```typescript
interface ProjectService {
  createProject(projectData: CreateProjectRequest): Promise<Project>
  createMonth(zoneId: string, monthData: MonthData): Promise<Month>
  createMilestone(monthId: string, milestoneData: MilestoneData): Promise<Milestone>
  calculateZoneCompletion(zoneId: string): Promise<CompletionPercentage>
  getConsolidatedProgress(): Promise<ProjectProgress>
}
```

### Task Management Service (Domain: tasks)

**Responsabilidad:** Maneja flujo completo de tareas desde creación hasta finalización con validación de estados
**Interfaz:**
```typescript
interface TaskService {
  createTask(taskData: CreateTaskRequest): Promise<Task>
  updateTaskStatus(taskId: string, newStatus: TaskStatus): Promise<Task>
  assignTask(taskId: string, assigneeId: string): Promise<Task>
  getTasksByZone(zoneId: string, filters?: TaskFilters): Promise<Task[]>
  validateStateTransition(currentStatus: TaskStatus, newStatus: TaskStatus): boolean
}
```

### Risk Engine Service (Domain: risks)

**Responsabilidad:** Ejecuta eventos de riesgo aleatorios y aplica impactos a zonas según probabilidades configuradas
**Interfaz:**
```typescript
interface RiskService {
  configureRisk(riskData: RiskConfiguration): Promise<Risk>
  executeRiskEvaluation(): Promise<RiskExecutionResult[]>
  applyRiskImpact(riskId: string, zoneId: string): Promise<ImpactResult>
  getRiskHistory(filters?: RiskFilters): Promise<RiskEvent[]>
}
```

### Notification Service (Domain: notifications)

**Responsabilidad:** Envía notificaciones por email y en aplicación con integración Gmail API
**Interfaz:**
```typescript
interface NotificationService {
  sendEmailNotification(recipient: string, template: string, data: any): Promise<EmailResult>
  createInAppNotification(userId: string, notification: NotificationData): Promise<Notification>
  sendBulkNotifications(recipients: string[], notification: NotificationData): Promise<BulkResult>
  markAsRead(notificationId: string): Promise<void>
}
```

### File Management Service (Domain: files)

**Responsabilidad:** Gestiona archivos en Google Drive con estructura de carpetas por zona y control de versiones
**Interfaz:**
```typescript
interface FileService {
  uploadFile(zoneId: string, fileData: FileUpload): Promise<DriveFile>
  createZoneFolders(projectId: string, zoneCount: number): Promise<FolderStructure>
  generateMeetingMinutes(meetingData: MeetingData): Promise<GoogleDoc>
  syncWithDrive(): Promise<SyncResult>
  searchFiles(query: string, zoneId?: string): Promise<DriveFile[]>
}
```

### Projection Screen Service (Domain: api)

**Responsabilidad:** Proporciona datos en tiempo real para pantalla de proyección con WebSocket
**Interfaz:**
```typescript
interface ProjectionService {
  broadcastUpdate(data: ProjectionData): Promise<void>
  getProjectionState(): Promise<ProjectionState>
  triggerAutoPrint(): Promise<PrintResult>
  handleClientConnection(clientId: string): Promise<void>
  updateCountdown(remainingTime: number): Promise<void>
}
```

## Data Model

### Users Table
| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| id | UUID | Identificador único del usuario | PRIMARY KEY, NOT NULL |
| email | VARCHAR(255) | Email del usuario | UNIQUE, NOT NULL, RFC 5322 |
| password_hash | VARCHAR(255) | Hash bcrypt de contraseña | NOT NULL, min 12 salt rounds |
| name | VARCHAR(100) | Nombre completo | NOT NULL |
| role | ENUM | Rol del usuario | NOT NULL, (Administrador, Director, Coordinador, Integrante) |
| zone_id | UUID | Zona asignada (si aplica) | FOREIGN KEY zones(id) |
| status | ENUM | Estado de la cuenta | NOT NULL, (Active, Inactive, Suspended, Pending) |
| created_at | TIMESTAMP | Fecha de creación | NOT NULL, DEFAULT NOW() |
| updated_at | TIMESTAMP | Última modificación | NOT NULL, DEFAULT NOW() |

### Projects Table
| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| id | UUID | Identificador único del proyecto | PRIMARY KEY, NOT NULL |
| name | VARCHAR(200) | Nombre del proyecto | UNIQUE, NOT NULL |
| duration_weeks | INTEGER | Duración en semanas | NOT NULL, CHECK (duration_weeks BETWEEN 1 AND 52) |
| zone_count | INTEGER | Número de zonas | NOT NULL, CHECK (zone_count BETWEEN 1 AND 10) |
| current_month | INTEGER | Mes actual de simulación | NOT NULL, DEFAULT 1 |
| status | ENUM | Estado del proyecto | NOT NULL, (Planning, Active, Paused, Completed, Archived) |
| start_date | TIMESTAMP | Fecha de inicio | NULL |
| completion_percentage | DECIMAL(5,2) | Porcentaje de cumplimiento | DEFAULT 0.00, CHECK (completion_percentage BETWEEN 0 AND 100) |

### Tasks Table
| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| id | UUID | Identificador único de tarea | PRIMARY KEY, NOT NULL |
| title | VARCHAR(200) | Título de la tarea | NOT NULL |
| description | TEXT | Descripción detallada | NULL |
| assignee_id | UUID | Usuario asignado | FOREIGN KEY users(id) |
| zone_id | UUID | Zona de la tarea | FOREIGN KEY zones(id), NOT NULL |
| status | ENUM | Estado actual | NOT NULL, (Creada, Asignada, En_curso, En_revision, Finalizada) |
| due_date | TIMESTAMP | Fecha límite | NOT NULL |
| created_by | UUID | Usuario creador | FOREIGN KEY users(id), NOT NULL |
| created_at | TIMESTAMP | Fecha de creación | NOT NULL, DEFAULT NOW() |
| completed_at | TIMESTAMP | Fecha de finalización | NULL |

### Audit_Logs Table
| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| id | BIGSERIAL | Identificador secuencial | PRIMARY KEY, NOT NULL |
| user_id | UUID | Usuario que ejecutó la acción | FOREIGN KEY users(id) |
| action_type | VARCHAR(50) | Tipo de acción | NOT NULL |
| resource_type | VARCHAR(50) | Tipo de recurso afectado | NOT NULL |
| resource_id | UUID | ID del recurso afectado | NULL |
| old_values | JSONB | Valores anteriores | NULL |
| new_values | JSONB | Valores nuevos | NULL |
| ip_address | INET | Dirección IP del usuario | NOT NULL |
| timestamp | TIMESTAMP | Momento de la acción | NOT NULL, DEFAULT NOW() |
| session_id | UUID | ID de sesión | NOT NULL |

## Flow Diagrams

### User Authentication and Welcome Link Flow

```
Usuario → Frontend → Auth API → Database → Gmail API → Google Drive
   |         |          |          |           |            |
   |--login->|          |          |           |            |
   |         |--POST--->|          |           |            |
   |         |          |--verify->|           |            |
   |         |          |<-user----|           |            |
   |         |          |--token-->|           |            |
   |         |<--JWT----|          |           |            |
   |<-redirect|         |          |           |            |
   |         |          |          |           |            |
   |         |          |--welcome>|           |            |
   |         |          |          |--create-->|            |
   |         |          |          |           |--send----->|
   |         |          |          |           |<-confirm---|
   |         |          |<-success-|           |            |
```

### Task Creation and Status Update Flow

```
Coordinador → Task API → Database → Notification Service → Gmail API → Integrante
     |           |          |              |                  |           |
     |--create-->|          |              |                  |           |
     |           |--validate>|              |                  |           |
     |           |<-zone-ok--|              |                  |           |
     |           |--save---->|              |                  |           |
     |           |<-task-id--|              |                  |           |
     |           |--notify-->|              |                  |           |
     |           |           |--trigger---->|                  |           |
     |           |           |              |--email---------->|           |
     |           |           |              |                  |--deliver->|
     |           |           |              |<-delivered-------|           |
     |<-success--|           |              |                  |           |
```

### Risk Execution and Impact Application Flow

```
Scheduler → Risk Engine → Random Generator → Database → Projection Screen → Coordinadores
    |           |              |               |              |                |
    |--trigger->|              |               |              |                |
    |           |--evaluate--->|               |              |                |
    |           |<-probability-|               |              |                |
    |           |--check------>|               |              |                |
    |           |<-execute-----|               |              |                |
    |           |--apply------>|               |              |                |
    |           |<-updated-----|               |              |                |
    |           |--broadcast-->|               |              |                |
    |           |              |               |              |--display------>|
    |           |--notify----->|               |              |                |
    |           |              |               |--send------->|                |
    |           |              |               |              |                |--alert->|
```