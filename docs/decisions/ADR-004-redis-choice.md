# ADR-004: Redis Choice

## 1. Document Information

| Field | Value |
|---|---|
| ADR ID | ADR-004 |
| Title | Redis Choice |
| Status | Accepted as an Optional Capability |
| Decision Date | 2026-11-16 |
| Decision Owner | AquaOps AI Project |
| Related Document | `docs/02-architecture.md` |

## 2. Context

AquaOps AI provides operational dashboards and APIs that may repeatedly retrieve the same information.

Examples include:

- Dashboard incident summaries.
- Frequently accessed sensor metadata.
- Operational statistics.
- Other read-heavy application data.

Repeatedly executing similar database queries can increase database load and response time as application usage grows.

The architecture therefore needs an optional caching mechanism for frequently accessed information.

However, caching must not compromise the integrity of authoritative business records.

## 3. Decision

Redis is selected as an optional, rebuildable caching layer for AquaOps AI.

The initial caching strategy will use the Cache-Aside Pattern.

PostgreSQL remains the authoritative source of transactional business data.

Redis must not be treated as the permanent or authoritative storage location for users, sensors, incidents, assignments, or other business records.

Caching will be introduced only where it provides measurable performance or database-load benefits.

## 4. Alternatives Considered

### Alternative A: Direct Database Access

The application would retrieve all requested information directly from PostgreSQL.

Advantages:

- Simpler architecture.
- No additional cache infrastructure.
- No cache invalidation complexity.
- Data is read from the authoritative database.

Disadvantages:

- Repeated requests may execute similar queries.
- Database load may increase with application usage.
- Frequently requested summaries may require repeated computation.

This approach is appropriate for data that does not benefit meaningfully from caching.

### Alternative B: Application-Level In-Memory Cache

Individual application instances could store frequently accessed data in their own memory.

Advantages:

- No separate cache server required.
- Fast access to locally cached data.
- Useful for small application-level caching requirements.

Disadvantages:

- Cache data is not automatically shared between application instances.
- Cache contents may differ between service instances.
- Restarting an instance removes its in-memory cache.
- Cache invalidation across multiple instances requires additional coordination.

This option may be useful for limited local caching but is not selected as the shared caching strategy.

### Alternative C: Redis

Redis provides a shared in-memory data store that application instances can access.

Advantages:

- Fast access to frequently requested data.
- Shared cache across multiple application instances.
- Configurable expiration through TTL.
- Supports multiple data structures.
- Can reduce repeated database reads.

Disadvantages:

- Additional infrastructure and operational overhead.
- Cache invalidation complexity.
- Cached values may become stale.
- Memory usage must be monitored.
- Application behavior must account for Redis unavailability.

Redis is selected as the optional shared caching layer.

## 5. Rationale

Redis is appropriate for AquaOps AI because selected operational information may be requested repeatedly by dashboards and backend services.

For example, dashboard summary information may be calculated from incident records and requested frequently.

Instead of repeatedly calculating the same summary from PostgreSQL, the application can temporarily cache the result.

The application checks Redis first and retrieves the authoritative information from PostgreSQL when the cache entry is unavailable.

This can reduce repeated database work and improve response times for suitable read-heavy operations.

## 6. Cache-Aside Pattern

The initial implementation will use the Cache-Aside Pattern.

The workflow is:

1. The application receives a data request.
2. The application checks Redis for the corresponding cache key.
3. If the key exists, the application returns the cached value.
4. If the key does not exist, the application queries PostgreSQL.
5. The application stores the retrieved result in Redis.
6. The application returns the result to the requester.

This pattern allows the application to control which information is cached.

Cache keys and expiration periods must be designed according to the data's purpose and freshness requirements.

## 7. Potential Caching Use Cases

Initial candidates include:

| Data | Caching consideration |
|---|---|
| Dashboard summaries | Cache frequently requested calculated summaries |
| Sensor metadata | Cache frequently accessed, relatively stable sensor information |
| Operational statistics | Cache results where a small amount of staleness is acceptable |
| User permissions | Cache only with careful expiration and invalidation controls |
| Incident details | Evaluate according to access patterns and freshness requirements |

Passwords and sensitive credentials must not be stored as ordinary application cache entries.

Authoritative incident and user records remain in PostgreSQL.

## 8. Cache Expiration and Invalidation

Cached information may become outdated when its authoritative data changes.

The application must define how cache entries are refreshed or invalidated.

Potential mechanisms include:

- Time To Live (TTL).
- Explicit cache eviction after a successful business update.
- Refreshing a cached value after retrieving updated data.
- Event-driven invalidation where justified.

The exact expiration periods and invalidation mechanisms will be decided during implementation.

The application must not assume that cached values are always current.

## 9. Redis Failure Handling

Redis is an optional performance component and must not become a single point of failure for authoritative business data.

If Redis becomes unavailable:

1. The application should handle the cache connection failure safely.
2. Supported reads should fall back to PostgreSQL.
3. The application should continue its core business operations where possible.
4. Cache errors should be recorded without exposing sensitive information.
5. Redis can be repopulated after recovery.

The application must not silently treat a failed cache write as a successful authoritative data write.

Redis failure may reduce performance but must not cause loss of authoritative business records.

## 10. Consequences

### Positive Consequences

- Reduced repeated database reads for suitable workloads.
- Faster responses for frequently accessed cached information.
- Shared caching across application instances.
- Configurable expiration.
- Better separation between authoritative storage and temporary performance data.

### Negative Consequences

- Additional infrastructure and operational overhead.
- Cache invalidation complexity.
- Potentially stale information.
- Additional memory requirements.
- More failure scenarios to test and monitor.

## 11. Review Conditions

The Redis decision should be reviewed if:

- Performance testing shows little benefit from caching.
- Application usage changes significantly.
- Database query performance becomes a concern.
- Cache invalidation becomes too complicated.
- Infrastructure cost outweighs performance benefits.
- A simpler application-level caching approach becomes sufficient.

## 12. Final Decision Statement

Redis is accepted as an optional, rebuildable caching layer for AquaOps AI.

The initial caching approach will use Cache-Aside.

PostgreSQL remains the authoritative source of transactional business data.

Redis will be introduced selectively, based on measured performance needs, and application behavior must remain safe when the cache is unavailable.

## 13. References

- `docs/00-project-overview.md`
- `docs/01-requirements.md`
- `docs/02-architecture.md`
- `docs/decisions/ADR-001-project-architecture.md`
- `docs/decisions/ADR-002-database-choice.md`
- `docs/decisions/ADR-003-kafka-choice.md`

## 14. Status

Accepted as an optional caching capability for the initial AquaOps AI architecture.

Detailed cache keys, TTL values, invalidation rules, and Redis configuration will be established during implementation and performance testing.