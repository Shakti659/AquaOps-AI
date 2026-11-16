# ADR-003: Kafka Choice

## 1. Document Information

| Field | Value |
|---|---|
| ADR ID | ADR-003 |
| Title | Kafka Choice |
| Status | Accepted |
| Decision Date | 2026-11-16 |
| Decision Owner | AquaOps AI Project |
| Related Document | `docs/02-architecture.md` |

## 2. Context

AquaOps AI receives sensor telemetry and processes it through several business responsibilities.

The telemetry workflow includes:

1. Sensor telemetry ingestion.
2. Telemetry validation and persistence.
3. Anomaly evaluation.
4. Incident creation.
5. Notification processing.
6. Dashboard updates and AI-assisted investigation.

These activities do not all require immediate request/response communication.

If every service directly called the next service through REST, the system would develop additional runtime dependencies.

For example, temporary Notification Service unavailability should not prevent an already accepted incident from remaining recorded.

The architecture therefore requires an asynchronous communication mechanism that can distribute events to independent consumers.

## 3. Decision

Apache Kafka is selected as the initial event-streaming platform for AquaOps AI.

The system will use:

- REST for synchronous request/response communication.
- Kafka for asynchronous event distribution.

Initial topic candidates:

| Topic | Responsibility |
|---|---|
| `telemetry.received` | Distribute accepted telemetry events |
| `incident.created` | Distribute newly created incident events |
| `notification.requested` | Distribute notification delivery requests |

The final topic configuration, partition count, replication settings, retention policies, and event schemas will be established during implementation and deployment.

## 4. Alternatives Considered

### Alternative A: REST-Only Communication

All services would communicate through synchronous HTTP requests.

Advantages:

- Simple request/response model.
- Familiar development approach.
- Straightforward API testing.
- Fewer infrastructure dependencies.

Disadvantages:

- Direct runtime dependencies between services.
- A downstream service outage may affect upstream requests.
- Producers must manage downstream communication.
- Multiple consumers require additional integration logic.
- Long-running processing can increase request latency.

REST remains appropriate for synchronous business operations but is not selected as the only communication mechanism.

### Alternative B: RabbitMQ

RabbitMQ is a message broker that supports queues, acknowledgements, routing, and asynchronous message processing.

Advantages:

- Flexible message routing.
- Suitable for task-oriented message processing.
- Supports acknowledgement and retry-oriented workflows.
- Can be a good fit for background jobs and work queues.

Disadvantages:

- Introduces broker infrastructure and operational responsibilities.
- Its messaging and replay model differs from Kafka's retained event-stream model.
- Additional evaluation would be required for the project's event distribution and replay requirements.

RabbitMQ is a valid alternative but is not selected for the initial architecture.

### Alternative C: Kafka

Kafka provides distributed event streams that producers publish to and independent consumers process.

Advantages:

- Decouples event producers from consumers.
- Supports multiple independent consumer groups.
- Retains events according to configured retention policies.
- Supports replay by resetting consumer offsets within retained data.
- Supports partition-based parallelism.
- Fits telemetry and incident event distribution.

Disadvantages:

- Additional infrastructure and operational complexity.
- Requires topic, partition, retention, and consumer-group configuration.
- Consumers must handle duplicate processing.
- Monitoring consumer lag and broker health is necessary.
- Ordering guarantees are limited to individual partitions.

Kafka is selected because its event-streaming model fits the asynchronous telemetry and incident workflows.

### Alternative D: Database Polling

Services could periodically query database tables to discover new work.

Advantages:

- Simple initial implementation.
- No separate message broker required.
- Can be suitable for small background workloads.

Disadvantages:

- Repeated polling creates database overhead.
- Processing may be delayed until the next polling cycle.
- Coordinating multiple workers requires additional logic.
- Event distribution becomes less cleanly separated from database storage.

Database polling is not selected as the primary event-distribution mechanism.

## 5. Rationale

Kafka is selected to support asynchronous event distribution across AquaOps AI business services.

The telemetry workflow can publish an event without requiring the producer to know every downstream consumer.

For example:

1. Sensor Service validates incoming telemetry.
2. The accepted telemetry is persisted according to the chosen data ownership design.
3. A telemetry event is published to Kafka.
4. Incident processing consumes the event and evaluates deterministic anomaly rules.
5. Incident Service records an incident when a configured rule is violated.
6. An incident-created event is published.
7. Notification Service consumes the event and processes delivery.

This approach reduces direct runtime coupling between event producers and consumers.

Kafka also supports independent consumers and replay of retained events when configured appropriately.

## 6. Kafka Concepts Used in AquaOps AI

### 6.1 Producer

A producer publishes events to Kafka topics.

Example: Sensor Service publishes accepted telemetry events.

### 6.2 Topic

A topic is a named stream of events.

Initial topics include:

- `telemetry.received`
- `incident.created`
- `notification.requested`

### 6.3 Broker

A broker is a Kafka server responsible for storing and serving topic partitions.

A production deployment may use multiple brokers for availability and capacity.

### 6.4 Partition

A topic can contain multiple partitions.

Partitions allow event storage and processing to be distributed.

Ordering is guaranteed within a partition rather than globally across all partitions.

The sensor identifier may be used as an event key when preserving per-sensor ordering is important.

### 6.5 Consumer

A consumer reads events from a topic and processes them.

Examples include incident processing and notification delivery.

### 6.6 Consumer Group

A consumer group allows multiple consumer instances to share partition processing.

Different consumer groups can independently consume the same topic.

### 6.7 Offset

An offset identifies an event's position within a partition.

Consumers track processing progress through committed offsets.

## 7. Reliability and Failure Handling

### 7.1 Kafka Unavailability

If Kafka becomes unavailable, event publication may fail.

The system must not report an event as successfully published when it has not been durably accepted according to the configured delivery guarantees.

Where reliable database-to-event publication is required, the Transactional Outbox Pattern will be considered.

The outbox record is committed alongside the authoritative business transaction. A publisher subsequently sends pending events to Kafka.

Pending events can be retried after broker recovery.

### 7.2 Duplicate Event Processing

Kafka consumers must be designed to handle duplicate event delivery.

Consumers should use event identifiers and idempotent processing logic to prevent duplicate business effects.

For example, repeated processing of the same telemetry event must not create multiple identical incidents when the business rule requires only one incident.

### 7.3 Consumer Failure

If a consumer instance fails, another instance in its consumer group may take over its assigned partitions after rebalancing.

The consumer must resume from an appropriate committed offset and safely handle any event that may be processed again.

### 7.4 Consumer Lag

Consumer lag occurs when consumers fall behind the events produced to a topic.

Consumer lag must be monitored because excessive lag can delay incident detection and notification delivery.

### 7.5 Notification Failure

Notification delivery failures must not roll back authoritative incident records.

The Notification Service will use controlled retry and backoff policies.

Repeated failures may be routed to a dead-letter mechanism for investigation and later recovery.

## 8. Event Design Principles

Kafka events must follow consistent design principles:

- Each event must have a unique event identifier.
- Event names must represent meaningful business events.
- Event payloads must include the information required by consumers.
- Events should carry timestamps and relevant entity identifiers.
- Event schemas must be versioned and evolved carefully.
- Sensitive information must not be unnecessarily included in event payloads.
- Consumers must not assume events arrive exactly once.
- Consumers must be designed for idempotent processing.
- Event ownership and producer responsibilities must be clear.

The exact event schema and serialization format will be finalized during implementation.

## 9. Consequences

### Positive Consequences

- Producers and consumers are more loosely coupled.
- Multiple independent services can consume business events.
- Asynchronous processing reduces unnecessary synchronous dependencies.
- Retained events support replay within configured retention.
- Partition-based processing supports parallelism.
- Telemetry and incident workflows become easier to extend with additional consumers.

### Negative Consequences

- Kafka introduces infrastructure and operational overhead.
- Eventual consistency must be considered.
- Duplicate event handling is required.
- Consumer offsets and rebalancing require understanding.
- Broker availability and consumer lag must be monitored.
- Event schemas and compatibility require governance.

## 10. Review Conditions

The Kafka decision should be reviewed if:

- The event-processing workload becomes very small.
- Operational complexity becomes disproportionate.
- A different messaging system better fits the workload.
- Event ordering or delivery requirements change.
- Throughput or retention requirements change.
- Deployment and infrastructure constraints make Kafka unsuitable.

## 11. Final Decision Statement

Apache Kafka is accepted as the initial asynchronous event-streaming platform for AquaOps AI.

REST remains the communication mechanism for synchronous API operations.

Kafka will be used for asynchronous telemetry, incident, and notification event distribution.

The architecture must account for duplicate delivery, consumer failures, event retention, consumer lag, and reliable event publication.

## 12. References

- `docs/00-project-overview.md`
- `docs/01-requirements.md`
- `docs/02-architecture.md`
- `docs/decisions/ADR-001-project-architecture.md`
- `docs/decisions/ADR-002-database-choice.md`

## 13. Status

Accepted for the initial AquaOps AI architecture.

Detailed topic configuration, event schemas, consumer groups, retry policies, and delivery guarantees will be established during implementation and deployment.