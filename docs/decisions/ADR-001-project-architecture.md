# ADR-001: Project Architecture

## 1. Document Information

| Field | Value |
|---|---|
| ADR ID | ADR-001 |
| Title | Project Architecture |
| Status | Accepted |
| Decision Date | 2026-11-16 |
| Decision Owner | AquaOps AI Project |
| Related Document | `docs/02-architecture.md` |

## 2. Context

AquaOps AI is an Intelligent Water Operations and Incident Management Platform.

The platform receives sensor telemetry, validates incoming readings, identifies abnormal operating conditions, creates incidents, sends notifications, and provides operators with an AI-assisted investigation experience.

The system contains several distinct business responsibilities:

- User and role management.
- Sensor registration and telemetry ingestion.
- Incident detection and lifecycle management.
- Notification delivery.
- AI-assisted incident investigation.

These responsibilities have different processing requirements and may need to evolve independently.

The project also aims to demonstrate practical knowledge of backend engineering, distributed systems, event-driven architecture, database design, security, and cloud deployment.

Therefore, an appropriate application architecture must provide clear business boundaries while keeping development and operational complexity manageable.

## 3. Decision

We will use a small, domain-oriented microservices architecture.

The initial system will contain five backend business services and one API Gateway:

1. User Service
2. Sensor Service
3. Incident Service
4. Notification Service
5. AI Service
6. API Gateway

Each business service will own its responsibilities and authoritative data.

Services will communicate through REST APIs when immediate request/response behavior is required and Kafka events when asynchronous communication is appropriate.

The architecture will avoid creating unnecessary services for small internal responsibilities.

## 4. Alternatives Considered

### Alternative A: Monolithic Architecture

A monolithic architecture would place the application's business functionality inside one deployable backend application.

Advantages:

- Simpler initial development.
- Easier local setup.
- Fewer infrastructure requirements.
- Simpler deployment.
- Easier transaction management within one application.

Disadvantages:

- Business responsibilities may become tightly coupled.
- Independent deployment of individual business areas is not straightforward.
- Scaling individual workloads independently is difficult.
- Large applications may become harder to maintain if module boundaries are weak.

### Alternative B: Modular Monolith

A modular monolith would keep one deployable application while separating business responsibilities into well-defined internal modules.

Advantages:

- Clearer module boundaries than an unstructured monolith.
- Simpler deployment than microservices.
- Easier local development.
- Lower infrastructure overhead.
- Suitable for many small and medium-sized applications.

Disadvantages:

- Modules still share the same deployment lifecycle.
- Independent scaling is limited.
- Module boundaries must be enforced through application design.

### Alternative C: Microservices Architecture

A microservices architecture separates business responsibilities into independently deployable services.

Advantages:

- Independent service development and deployment.
- Clear business responsibility boundaries.
- Independent scaling where necessary.
- Better isolation of some service failures.
- Demonstrates distributed communication and event-driven integration.

Disadvantages:

- More infrastructure and configuration.
- Network communication failures.
- Distributed data consistency challenges.
- More complicated testing and debugging.
- Additional security and observability requirements.

## 5. Rationale

Microservices were selected because AquaOps AI has distinct business workflows and the project aims to demonstrate practical distributed-system engineering.

For example:

- Sensor Service handles telemetry ingestion.
- Incident Service owns incident creation and lifecycle operations.
- Notification Service processes notification requests.
- AI Service provides advisory investigation capabilities.
- User Service manages identity and user roles.

These boundaries allow each service to focus on a specific business responsibility.

The architecture also allows Kafka to distribute events without requiring every participating service to communicate directly through synchronous API calls.

However, the number of services will remain deliberately limited to prevent unnecessary complexity.

## 6. Consequences

### Positive Consequences

- Business responsibilities have clear service boundaries.
- Services can be developed and deployed independently.
- Kafka enables asynchronous event distribution.
- Service-specific scaling is possible when required.
- The architecture demonstrates practical distributed-system concepts.
- Failures in optional capabilities, such as AI assistance, can be isolated from the core incident workflow when properly designed.

### Negative Consequences

- Local development requires multiple application components.
- Service communication introduces network failure scenarios.
- Distributed workflows require careful consistency handling.
- Kafka introduces broker and consumer-management responsibilities.
- Testing across services requires additional integration effort.
- Deployment, monitoring, and troubleshooting become more complex.

## 7. Constraints and Mitigation

The project environment has limited local hardware resources.

Therefore:

- Services will be started gradually during development.
- Docker Compose will manage infrastructure dependencies.
- Unnecessary services and infrastructure will be avoided.
- Database and cache choices will be validated against actual requirements.
- Observability and troubleshooting will be introduced incrementally.

The architecture will not create a separate microservice for every small feature.

Anomaly detection will initially remain part of the Incident Service's logical responsibilities rather than becoming another independent service without justification.

## 8. Review Conditions

This decision should be reviewed if:

- The number of services creates excessive operational overhead.
- Service boundaries become difficult to maintain.
- Performance testing shows that the architecture is unsuitable.
- Deployment complexity becomes disproportionate to project requirements.
- A modular monolith or alternative architecture becomes more appropriate.

## 9. Final Decision Statement

A small, domain-oriented microservices architecture is accepted for AquaOps AI.

The decision prioritizes clear business boundaries, independent service responsibilities, event-driven integration, and practical distributed-system learning while controlling unnecessary architectural complexity.

The decision does not imply that microservices are universally superior to monolithic architectures.

## 10. References

- `docs/00-project-overview.md`
- `docs/01-requirements.md`
- `docs/02-architecture.md`

## 11. Status

Accepted for the initial AquaOps AI architecture.

Implementation and architecture reviews may result in future revisions.