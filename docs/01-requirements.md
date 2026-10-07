# AquaOps AI — Requirements

## 1. Purpose

This document defines the functional and non-functional requirements for the AquaOps AI platform.

The purpose of the requirements is to translate the high-level project overview into clear, testable and traceable system expectations.

The requirements will later guide:

- Architecture
- API design
- Database design
- Backend implementation
- Frontend implementation
- Testing
- Security
- AI integration
- Deployment

---

## 2. Actors

### 2.1 Operations Operator

The Operations Operator is responsible for day-to-day monitoring and investigation of operational incidents.

Primary activities:

- Monitor active incidents
- Review sensor information
- Investigate abnormal conditions
- Update incident status
- Review relevant operational context
- Use the AI assistant for contextual assistance

---

### 2.2 Operations Manager

The Operations Manager oversees operational activity and incident trends.

Primary activities:

- Review incidents
- Monitor operational trends
- Review unresolved incidents
- Review incident history
- Monitor team activity

---

### 2.3 Administrator

The Administrator manages users, roles and system-level configuration.

Primary activities:

- Create and manage users
- Assign roles
- Manage access
- Review administrative information

---

### 2.4 Sensor/Event Source

Sensors or external systems provide operational telemetry and events to the platform.

Examples:

- Flow measurements
- Pressure measurements
- Water-level measurements
- Equipment state
- Sensor health events

---

### 2.5 AI Model Provider

An external or internally hosted language-model capability may provide language-model generation for the AI assistant.

The AI model provider must not be treated as the authoritative source of operational truth.

Relevant application context should be retrieved from trusted application data.

---

# 3. Functional Requirements

## FR-001 — User Authentication

The system shall allow registered users to authenticate securely.

The authentication mechanism shall support:

- Secure credential handling
- Token-based authentication
- Protected application resources
- Appropriate authentication failure handling

---

## FR-002 — Role-Based Authorization

The system shall restrict protected operations according to the authenticated user's role and permissions.

The initial roles are:

- ADMIN
- OPERATIONS_MANAGER
- OPERATIONS_OPERATOR

---

## FR-003 — User Management

Administrators shall be able to manage application users.

The system should support:

- Creating users
- Viewing users
- Updating users
- Activating or deactivating users
- Assigning roles

---

## FR-004 — Sensor Management

Authorized users shall be able to view registered sensors.

Sensor information may include:

- Sensor identifier
- Sensor type
- Location
- Measurement type
- Operational status
- Configuration information

---

## FR-005 — Telemetry Ingestion

The platform shall receive supported sensor telemetry.

Telemetry may include:

- Flow
- Pressure
- Water level
- Equipment state
- Sensor health information

The system shall validate incoming telemetry before processing it.

---

## FR-006 — Event Processing

The platform shall process relevant sensor events asynchronously where asynchronous processing provides a meaningful architectural benefit.

The event-processing workflow should support:

- Event publication
- Event consumption
- Processing failures
- Retry handling where appropriate
- Event observability

---

## FR-007 — Anomaly Detection

The platform shall identify configured abnormal sensor conditions.

Initial anomaly detection may use explainable rules and thresholds.

Examples:

- Pressure outside configured limits
- Flow below a configured threshold
- Sudden unexpected measurement change
- Missing telemetry
- Abnormal equipment state

---

## FR-008 — Incident Creation

The system shall create an operational incident when a configured anomaly meets the criteria for incident creation.

An incident should contain information such as:

- Incident ID
- Type
- Severity
- Status
- Description
- Source sensor
- Detection timestamp
- Last updated timestamp
- Relevant operational context

---

## FR-009 — Incident Lifecycle Management

Authorized users shall be able to manage an incident through its supported lifecycle.

Initial lifecycle states are:

- OPEN
- ACKNOWLEDGED
- INVESTIGATING
- RESOLVED
- CLOSED

The system shall prevent invalid lifecycle transitions where applicable.

---

## FR-010 — Incident Assignment

Authorized users shall be able to assign an incident to an appropriate operations user.

The system should record assignment changes where auditability is required.

---

## FR-011 — Notifications

The platform shall generate notifications for configured important operational events.

Examples:

- New critical incident
- Incident escalation
- Incident assignment
- Important incident state change

Notification delivery mechanisms may evolve during implementation.

---

## FR-012 — Operations Dashboard

The React frontend shall provide an operational dashboard.

The dashboard should provide access to:

- Active incidents
- Incident severity
- Incident status
- Sensor information
- Recent operational information
- Notifications

---

## FR-013 — Incident Investigation

Authorized users shall be able to investigate incidents using relevant operational information.

The investigation view should provide:

- Incident details
- Related sensor information
- Relevant telemetry
- Incident history
- Available related events

---

## FR-014 — AI Assistant

Authorized users shall be able to ask the AI assistant questions related to operational incidents and available system context.

Examples:

- What happened?
- Why was this incident created?
- Which sensor triggered the incident?
- What values changed?
- Are there related recent incidents?
- What operational information should be reviewed?

---

## FR-015 — Context Retrieval for AI

The AI workflow shall retrieve relevant operational context before generating responses where required.

Potential context sources include:

- Incident data
- Sensor information
- Telemetry
- Historical incidents
- Operational documentation

The retrieved context should be relevant to the user's question.

---

## FR-016 — AI Response Grounding

The AI assistant should base operational explanations on retrieved application context rather than relying solely on general model knowledge.

When sufficient context is unavailable, the system should clearly communicate that limitation.

---

## FR-017 — Audit Information

The system should maintain appropriate audit information for important security and operational actions.

Potential audit events include:

- Authentication events
- Role changes
- Incident state changes
- Incident assignment changes
- Administrative actions
- Important AI interactions where appropriate

---

# 4. Non-Functional Requirements

## NFR-001 — Security

Protected resources shall require appropriate authentication and authorization.

Sensitive information shall not be exposed to unauthorized users.

---

## NFR-002 — Reliability

The system should handle expected component failures gracefully.

Failure of an asynchronous processing component should not unnecessarily prevent unrelated system functionality from operating.

---

## NFR-003 — Scalability

The architecture should allow components to scale independently where workload characteristics justify independent scaling.

---

## NFR-004 — Maintainability

Services shall have clearly defined responsibilities.

Interfaces between components should be documented.

---

## NFR-005 — Observability

Important application operations and event-processing workflows should produce sufficient logs and metrics to support troubleshooting.

---

## NFR-006 — Testability

Core business logic should be testable independently from infrastructure where practical.

---

## NFR-007 — Data Integrity

Transactional business data shall maintain appropriate constraints and consistency.

---

## NFR-008 — Performance

Frequently accessed operational information should be retrievable efficiently.

Caching may be used where repeated access to the same operational data would otherwise create unnecessary database load.

---

## NFR-009 — Availability

The system should be designed so that non-critical component failures do not unnecessarily make unrelated functionality unavailable.

---

## NFR-010 — AI Safety

AI-generated information shall be treated as assistance rather than authoritative operational control.

The AI assistant shall not directly execute safety-critical infrastructure actions.

---

# 5. Use Cases

## UC-001 — User Login

**Actor:** Operations Operator, Operations Manager, Administrator

### Goal

Authenticate into the AquaOps platform.

### Basic Flow

1. User opens the login page.
2. User provides credentials.
3. System validates the credentials.
4. System authenticates the user.
5. System provides the appropriate authentication token/session.
6. User accesses authorized functionality.

### Failure Cases

- Invalid credentials
- Disabled account
- Authentication service unavailable

---

## UC-002 — View Active Incidents

**Actor:** Operations Operator

### Goal

View currently active operational incidents.

### Basic Flow

1. Operator authenticates.
2. Operator opens the incident dashboard.
3. System retrieves active incidents.
4. System displays incident severity and status.
5. Operator selects an incident for investigation.

---

## UC-003 — Investigate Incident

**Actor:** Operations Operator

### Goal

Understand the operational context of an incident.

### Basic Flow

1. Operator opens an incident.
2. System displays incident details.
3. System displays related sensor information.
4. System displays relevant telemetry.
5. Operator reviews the available context.
6. Operator may ask the AI assistant for additional explanation.

---

## UC-004 — Create Incident from Anomaly

**Actor:** Sensor/Event Source and System

### Goal

Create an incident when an operational anomaly meets configured criteria.

### Basic Flow

1. Sensor telemetry is received.
2. Telemetry is validated.
3. Relevant event is processed.
4. Anomaly detection evaluates the measurement.
5. Configured anomaly criteria are satisfied.
6. System creates an incident.
7. Incident event is published where required.
8. Notification processing is triggered where configured.

---

## UC-005 — Update Incident

**Actor:** Operations Operator / Operations Manager

### Goal

Update the state of an operational incident.

### Basic Flow

1. Authorized user opens an incident.
2. User selects a valid lifecycle transition.
3. System validates authorization.
4. System validates the lifecycle transition.
5. System updates the incident.
6. System records appropriate audit information.
7. Relevant events/notifications are generated where configured.

---

## UC-006 — Ask AI Assistant

**Actor:** Authorized Operations User

### Goal

Obtain a contextual explanation of an operational incident.

### Basic Flow

1. User opens an incident.
2. User asks a question.
3. AI service validates access to relevant context.
4. Relevant operational information is retrieved.
5. Retrieved context is supplied to the AI workflow.
6. AI generates a response.
7. Response is returned to the user.

### Failure Cases

- Relevant context unavailable
- AI provider unavailable
- Request exceeds allowed limits
- User lacks permission to access requested information

---

# 6. Business Rules

## BR-001 — Authentication

Protected resources must not be accessible without appropriate authentication.

---

## BR-002 — Authorization

A user's permissions must be evaluated before protected operations are performed.

---

## BR-003 — Incident Severity

Incidents shall have a defined severity.

Initial severity levels may include:

- LOW
- MEDIUM
- HIGH
- CRITICAL

---

## BR-004 — Incident Lifecycle

An incident must follow supported lifecycle transitions.

Invalid transitions shall be rejected.

---

## BR-005 — Critical Incidents

Critical incidents shall receive appropriate operational visibility and notification according to configured notification rules.

---

## BR-006 — Incident Ownership

When an incident is assigned to a user, the assignment should be visible to authorized users.

---

## BR-007 — Sensor Data Validation

Invalid or incomplete telemetry shall not be treated as valid operational measurements.

---

## BR-008 — AI Human Oversight

AI-generated responses shall not be treated as authoritative instructions for safety-critical infrastructure operations.

---

## BR-009 — AI Context Access

The AI workflow must respect the same authorization boundaries applicable to the underlying operational information.

---

# 7. Acceptance Criteria

Acceptance criteria will be used to determine whether important requirements have been implemented correctly.

## AC-001 — Authentication

**Given** a registered active user,

**When** valid credentials are submitted,

**Then** the user should be authenticated successfully.

---

## AC-002 — Authorization

**Given** an authenticated user without permission for a protected operation,

**When** the user attempts the operation,

**Then** the system should reject the request.

---

## AC-003 — Sensor Telemetry

**Given** valid telemetry,

**When** the telemetry is submitted to the platform,

**Then** the system should validate and process the telemetry.

---

## AC-004 — Anomaly Detection

**Given** a sensor measurement that violates a configured operational threshold,

**When** the measurement is processed,

**Then** the system should identify the configured anomaly.

---

## AC-005 — Incident Creation

**Given** an anomaly that satisfies incident-creation criteria,

**When** the anomaly is processed,

**Then** an incident should be created with the appropriate severity and initial status.

---

## AC-006 — Incident Update

**Given** an authorized user and a valid incident transition,

**When** the user updates the incident,

**Then** the system should persist the new state.

---

## AC-007 — Invalid Incident Transition

**Given** an incident and an invalid lifecycle transition,

**When** the transition is requested,

**Then** the system should reject the transition.

---

## AC-008 — Notification

**Given** an incident that satisfies configured notification criteria,

**When** the incident event is processed,

**Then** the notification workflow should be triggered.

---

## AC-009 — AI Context

**Given** an authorized user asking a question about an incident,

**When** the AI assistant processes the question,

**Then** relevant operational context should be retrieved where available.

---

## AC-010 — AI Limitation

**Given** insufficient operational context,

**When** the AI assistant receives a question requiring unavailable information,

**Then** the response should communicate the limitation rather than presenting unsupported operational facts as certain.

---

# 8. Data Requirements

The system will manage several categories of data.

## 8.1 User Data

Potential information:

- User ID
- Name
- Email
- Password credential representation
- Role
- Account status
- Created timestamp
- Updated timestamp

---

## 8.2 Sensor Data

Potential information:

- Sensor ID
- Sensor type
- Location
- Measurement type
- Status
- Configuration
- Created timestamp
- Updated timestamp

---

## 8.3 Telemetry Data

Potential information:

- Telemetry ID
- Sensor ID
- Measurement type
- Measurement value
- Unit
- Timestamp
- Quality/status information

---

## 8.4 Incident Data

Potential information:

- Incident ID
- Type
- Severity
- Status
- Description
- Sensor reference
- Assigned user
- Detected timestamp
- Updated timestamp
- Resolution information

---

# 9. Integration Requirements

The platform is expected to integrate with:

- Kafka for asynchronous event processing
- PostgreSQL for transactional data
- MongoDB for suitable telemetry/document data
- Redis for caching
- AI/LLM provider through the AI service
- React frontend through APIs
- Notification mechanisms
- AWS infrastructure during deployment

Integration details will be defined during architecture and implementation phases.

---

# 10. Security Requirements

The system shall consider security throughout development.

Security requirements include:

- Authentication
- Authorization
- Secure credential handling
- JWT/session security
- Protected APIs
- Input validation
- Secure configuration
- Secret management
- Least-privilege access
- Appropriate audit logging
- Protection of operational data
- Authorization-aware AI context retrieval

---

# 11. AI Requirements

The AI assistant should:

1. Support authorized operational users.
2. Use relevant operational context where appropriate.
3. Prefer retrieved application data for operational explanations.
4. Clearly communicate when information is unavailable.
5. Avoid presenting unsupported assumptions as operational facts.
6. Respect user authorization.
7. Avoid direct execution of safety-critical infrastructure actions.
8. Support logging/evaluation of AI interactions where appropriate.
9. Allow future evaluation of response quality.
10. Support future improvements to retrieval and grounding.

---

# 12. Scope

## 12.1 In Scope

- Authentication
- Authorization
- User management
- Sensor management
- Telemetry ingestion
- Event processing
- Anomaly detection
- Incident management
- Incident assignment
- Notifications
- React dashboard
- AI assistant
- Context retrieval/RAG
- PostgreSQL
- MongoDB
- Redis
- Kafka
- Docker
- CI/CD
- AWS deployment
- Testing
- Documentation
- Observability

---

## 12.2 Out of Scope

The initial implementation will not include:

- Direct physical control of pumps
- Direct physical control of valves
- Autonomous safety-critical decisions
- Hardware manufacturing
- Full national-scale utility infrastructure
- Advanced ML research
- Full digital-twin simulation
- Unsupervised autonomous infrastructure operation

---

# 13. Requirements Traceability

The project will progressively connect requirements to implementation.

The intended relationship is:

text
Requirement
     |
     v
Architecture Component
     |
     v
Service
     |
     v
API / Event
     |
     v
Database / Storage
     |
     v
Automated Test

---

# 14. Requirements Change Management
Requirements may evolve during development.
Changes should be evaluated based on:
- Business value
- Technical impact
- Security impact
- Data impact
- Performance impact
- Operational impact
- Testing impact
- AI safety implications
Significant architecture-impacting changes should be documented through an Architecture Decision Record (ADR).


15. Current Requirements Status
Completed
- Project overview
- Initial actor identification
- Functional requirements
- Non-functional requirements
- Use cases
- Business rules
- Acceptance criteria
- Data requirements
- Integration requirements
- Security requirements
- AI requirements
- Scope definition
- Initial requirements traceability