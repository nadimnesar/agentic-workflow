# Performance Checklist Reference

Quick reference for performance in Java/Spring Boot + Angular + PostgreSQL stack.
Use alongside `performance-optimization` skill.

## Backend: Spring Boot

### JPA / Hibernate
- [ ] N+1 query detected and fixed (use `@EntityGraph` or `JOIN FETCH`)
- [ ] `spring.jpa.open-in-view=false` (disables OSIV pattern)
- [ ] Batch fetching: `hibernate.jdbc.batch_size=50`
- [ ] `@Transactional(readOnly=true)` on read-only query methods
- [ ] Pagination: use `Pageable` + `Slice` instead of `List` for large datasets
- [ ] Lazy loading vs eager loading: prefer lazy, fetch eagerly only when needed

### Connection Pool (HikariCP)
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10        # Rule: cores * 2 + effective_disk_count
      minimum-idle: 5
      idle-timeout: 300000
      connection-timeout: 5000
      max-lifetime: 1800000
```

### Caching
- [ ] Redis cache for frequently-read, rarely-changed data (configurations, reference tables)
- [ ] Spring Cache annotations: `@Cacheable`, `@CacheEvict`, `@CachePut`
- [ ] Cache-aside pattern: check cache → DB → populate cache
- [ ] Local cache (Caffeine) for ultra-low-latency, small datasets

### API Performance
- [ ] Response times < 200ms (p95)
- [ ] Async processing for long-running tasks (`@Async`, CompletableFuture, SQS/Kafka)
- [ ] Compression: `server.compression.enabled=true` (gzip)
- [ ] HTTP/2 enabled (if behind ALB/TLS)
- [ ] Static resources served via Nginx/CDN, not Spring Boot

### JVM Tuning
```yaml
# Common flags
-XX:+UseZGC                    # Low-latency GC (Java 17+)
-Xms2g -Xmx2g                  # Heap size (match to prevent resizing)
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/dumps
-Dspring.jmx.enabled=false     # Disable if not needed
```

## Frontend: Angular

### Core Web Vitals Targets
| Metric | Good | Needs Improvement | Poor |
|---|---|---|---|
| LCP | <= 2.5s | <= 4.0s | > 4.0s |
| INP | <= 200ms | <= 500ms | > 500ms |
| CLS | <= 0.1 | <= 0.25 | > 0.25 |

### Checklist
- [ ] Lazy-loaded route modules (not eager imports)
- [ ] `OnPush` change detection strategy on all components
- [ ] TrackBy function on *ngFor to reduce DOM operations
- [ ] Images: `loading="lazy"`, `ngSrc` with Angular Image directive
- [ ] Bundle budget: `< 200KB gzipped initial JS`
- [ ] No `setInterval` or `setTimeout` (use Angular `timer` from rxjs for cleanup)
- [ ] Unsubscribe from observables in `ngOnDestroy` (or use `takeUntilDestroyed`)
- [ ] Server-side rendering (Angular Universal) for SEO-critical pages

## Database: PostgreSQL

### Indexing
- [ ] Indexes on all foreign keys and frequently-filtered columns
- [ ] Composite indexes match query WHERE clause order (leftmost prefix rule)
- [ ] Partial indexes for filtered conditions: `CREATE INDEX ... WHERE status = 'ACTIVE'`
- [ ] Covering indexes: `CREATE INDEX ... INCLUDE (name, email)` to avoid heap lookups
- [ ] No unused indexes (check `pg_stat_user_indexes`)
- [ ] Full-text search: GIN index with `to_tsvector('english', column)`

### Query Optimization
```sql
-- Check slow queries
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Missing index detection
SELECT schemaname, tablename, seq_scan, seq_tup_read
FROM pg_stat_user_tables
WHERE seq_scan > 1000;
```

### Connections
```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50
          fetch_size: 100
        generate_statistics: true   # enable in dev only
```

## Messaging & Async

### Kafka
- [ ] Idempotent producer: `enable.idempotence=true`
- [ ] Batch size: `batch.size=16384`, `linger.ms=5`
- [ ] Consumer: `max.poll.records=500`, process within `max.poll.interval.ms`
- [ ] Records partitioned by key for ordering guarantees

### RabbitMQ
- [ ] Prefetch count set to 1-3 (not unlimited)
- [ ] Dead letter queue configured with TTL and retry policy
- [ ] Publisher confirms enabled

## Measurement Tools

```bash
# JVM profiling
jcmd <pid> Thread.print
jstat -gcutil <pid> 1s          # GC stats every second
-XX:StartFlightRecording        # JDK Flight Recorder

# API profiling in code
long start = System.nanoTime();
// ... code ...
log.info("query took {}ms", TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - start));

# Database
EXPLAIN ANALYZE <query>;        # Query plan + actual times
pg_stat_statements              # Track query performance

# Angular
ng build --stats-json           # Analyze bundle
esbuild --bundle --analyze      # Bundle analysis
```
