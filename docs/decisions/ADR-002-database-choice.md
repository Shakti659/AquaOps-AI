# ADR-002: Database Choice

## 1. Document Information

| Field | Value |
|---|---|
| ADR ID | ADR-002 |
| Title | Database Choice |
| Status | Accepted with Conditional Decision |
| Decision Date | 2026-11-16 |
| Decision Owner | AquaOps AI Project |
| Related Document | `docs/02-architecture.md` |

## 2. Context

AquaOps AI processes and manages different categories of data.

These include:

- User accounts and roles.
- Sensor registration and configuration.
- Sensor telemetry readings.
- Incident records and assignments.
- Notification records.
- Audit and operational history.

These categories have different data structures, relationships, query patterns, and consistency requirements.

The architecture must provide reliable transactional storage for business records while supporting the ingestion and retrieval of sensor telemetry.

It must also avoid introducing unnecessary database infrastructure.

## 3. Decision

PostgreSQL will be the authoritative relational database for transactional business data.

MongoDB will remain a candidate for selected telemetry documents and will be introduced only if workload and query requirements justify its operational overhead.

Redis will be used as an optional, rebuildable cache rather than an authoritative database.

The initial data responsibilities are:

| Data category | Initial storage decision |
|---|---|
| Users and roles | PostgreSQL |
| Sensor metadata | PostgreSQL |
| Incidents and assignments | PostgreSQL |
| Notification records | PostgreSQL |
| Audit records | PostgreSQL |
| Selected telemetry documents | MongoDB candidate |
| Frequently accessed cached data | Redis |

## 4. Alternatives Considered

### Alternative A: PostgreSQL Only

All authoritative business records and sensor telemetry would be stored in PostgreSQL.

Advantages:

- One primary database technology.
- Strong relational integrity.
- Transaction support.
- SQL querying and indexing.
- Fewer infrastructure components.
- Simpler local development, backup, and monitoring.

Disadvantages:

- Telemetry schema and query patterns require careful design.
- High-volume ingestion may require partitioning, retention policies, and specialized indexing.
- Flexible and variable-shaped telemetry documents may need additional modeling effort.

This is a valid approach for the initial implementation.

### Alternative B: MongoDB Only

All business records and telemetry would be stored in MongoDB.

Advantages:

- Flexible document structures.
- Nested data can be represented naturally.
- Document-oriented queries.

Disadvantages:

- Relational business data and relationships require deliberate modeling.
- Transactional and integrity requirements must be carefully designed.
- The project would lose some of the advantages of a relational model for its core business records.

This option is not selected because PostgreSQL is a better fit for the initial transactional business data requirements.

### Alternative C: PostgreSQL and MongoDB Together

PostgreSQL would store transactional business records, while MongoDB would store selected telemetry documents.

Advantages:

- Clear distinction between transactional business data and selected telemetry workloads.
- Flexible document storage for variable telemetry structures.
- Independent optimization of database workloads.

Disadvantages:

- Two database technologies to deploy and maintain.
- Additional backup, monitoring, security, and operational responsibilities.
- Cross-database consistency requires careful handling.
- More complex local development and deployment.

This option remains conditional on evidence that MongoDB provides a meaningful advantage.

## 5. PostgreSQL Rationale

PostgreSQL is selected as the authoritative database for business data because it provides:

- Relational data modeling.
- Primary and foreign key constraints.
- Transaction support.
- SQL querying.
- Indexing capabilities.
- Data integrity controls.
- Mature ecosystem and tooling.

Examples include users, roles, sensor metadata, incidents, assignments, notification history, and audit records.

The Incident Service must retain authoritative incident information in PostgreSQL.

Cached or event-streamed copies must not replace this source of truth.

## 6. MongoDB Evaluation Criteria

MongoDB is a candidate for selected telemetry data because sensor readings may contain nested or evolving document structures.

However, its adoption must be supported by measurable requirements.

The following factors will be evaluated:

1. Expected telemetry ingestion volume.
2. Telemetry document structure and variability.
3. Required query patterns.
4. Indexing and retrieval performance.
5. Data retention and archival requirements.
6. Aggregation and reporting requirements.
7. Backup and recovery requirements.
8. Operational complexity and infrastructure cost.
9. Comparison with PostgreSQL-based telemetry storage.
10. Potential use of specialized time-series database capabilities.

If PostgreSQL satisfies the requirements with acceptable performance and operational simplicity, MongoDB may be omitted from the initial implementation.

## 7. Redis Responsibilities

Redis will be used as an optional caching layer.

Potential use cases include:

- Frequently accessed sensor metadata.
- Dashboard summary information.
- Short-lived application data.
- Other read-heavy data where caching provides measurable benefit.

The initial caching approach will use the cache-aside pattern.

PostgreSQL remains the authoritative source of business records.

Redis failure must not cause loss of authoritative data. Supported read operations should fall back to their authoritative data source.

Cache expiration and invalidation must be considered when business data changes.

## 8. Data Ownership and Service Boundaries

Each service owns its authoritative business data.

Initial ownership:

| Service | Data responsibility |
|---|---|
| User Service | Users, roles, and account information |
| Sensor Service | Sensor metadata and telemetry ingestion responsibility |
| Incident Service | Incidents, assignments, and incident lifecycle |
| Notification Service | Notification records and delivery status |
| AI Service | AI investigation and response processing; no separate authoritative business store initially |

A service must not directly modify another service's database tables.

Cross-service data requirements must be handled through service APIs or asynchronous events.

Redis cache entries must be treated as rebuildable copies, not authoritative records.

## 9. Consequences

### Positive Consequences

- PostgreSQL provides a reliable foundation for transactional business records.
- Data ownership boundaries are clearly documented.
- MongoDB adoption remains evidence-based rather than mandatory.
- Redis can improve read performance where justified.
- The design avoids treating Kafka or Redis as authoritative business databases.

### Negative Consequences

- Adding MongoDB introduces another database technology and operational workload.
- Multiple data stores create additional backup and monitoring responsibilities.
- Cross-service workflows require eventual consistency considerations.
- Cache invalidation and stale-data handling require deliberate design.
- Telemetry retention and high-volume querying may require future database optimization.

## 10. Review Conditions

The database decision should be reviewed when:

- Telemetry ingestion volume increases.
- Query performance becomes unacceptable.
- Data retention requirements change.
- Database storage or infrastructure costs increase.
- MongoDB provides a measurable advantage over PostgreSQL.
- A time-series extension or specialized storage option becomes more appropriate.

## 11. Final Decision Statement

PostgreSQL is accepted as the authoritative relational database for AquaOps AI transactional business data.

Redis is accepted as an optional, rebuildable caching layer.

MongoDB remains a conditional candidate for selected telemetry documents and will be adopted only when workload, query, performance, and operational evidence justify its use.

The project will prioritize data integrity and operational simplicity over unnecessary database diversity.

## 12. References

- `docs/00-project-overview.md`
- `docs/01-requirements.md`
- `docs/02-architecture.md`
- `docs/decisions/ADR-001-project-architecture.md`

## 13. Status

PostgreSQL decision: Accepted.

Redis caching decision: Accepted as an optional capability.

MongoDB decision: Conditional; subject to workload validation and implementation testing.