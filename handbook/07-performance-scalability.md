# 07 — Performance, Latency & Scalability

Performance is a feature users feel directly. Scalability is the ability to keep that performance as load grows, at acceptable cost. Both start with **targets and measurement**, not guesses.

**Applies when:** setting latency targets, something is slow, expecting growth, adding caching, designing hot paths, choosing data structures/stores.

## Key questions
- What are the latency targets (p50/p95/p99) for each interaction?
- Where does time go today (traces, profiles)?
- What resource saturates first under load (CPU, memory, DB, I/O, network, a third party)?
- What's the growth dimension (users, tenants, data, requests, fan-out)?
- What's the cost per unit of scale?

## 1. Measure first [ALL]
- Define targets ([01](01-requirements.md)); use **percentiles**, not averages — tail latency is what users notice and what compounds in fan-out.
- Instrument: request duration histograms, DB query timings, external call timings, queue lag ([13](13-observability.md)).
- **Latency budget**: break the target down by component (e.g. 300 ms = 20 ms edge + 50 ms app + 80 ms DB + 150 ms external).
- Profile before optimizing (CPU, memory/allocation, I/O, lock contention). Optimize the measured bottleneck only.
- Know the numbers: memory access ~100 ns; SSD read ~100 µs; same-datacenter round trip ~0.5 ms; cross-continent round trip ~100–150 ms.

## 2. Common bottlenecks & remedies [MVP]
| Symptom | Likely causes | Remedies to consider |
|---|---|---|
| Slow endpoint, many DB calls | N+1 queries | Batch/eager loading, dataloaders |
| Slow single query | Missing/wrong index, large scan, bad plan | Index, rewrite, `EXPLAIN ANALYZE`, denormalize/materialize |
| Slow under concurrency | Lock contention, pool exhaustion, hot rows | Shorter transactions, pool sizing, sharded counters, queueing |
| Large responses | Over-fetching, no pagination, no compression | Field selection, pagination, gzip/brotli |
| Slow third-party calls on path | Synchronous external dependency | Async, cache, parallelize, timeouts |
| High p99 only | GC pauses, cold caches, noisy neighbours, retries, cold starts | Tune runtime, warmup, isolation, hedged requests [SCALE] |
| Memory growth | Leaks, unbounded caches/buffers, loading whole files | Bounded caches, streaming |
| Sequential work | Serial I/O | Concurrency/parallelism with limits |

## 3. Caching [MVP]
Layers: browser/HTTP cache → CDN/edge → reverse proxy → application (in-memory, distributed like Redis) → DB buffer cache/materialized views.

Decide for each cache:
- **What** to cache (expensive, frequently read, tolerably stale).
- **Key design**: include every dimension that changes the result (tenant, user, locale, permissions, version). Missing a dimension = data leak.
- **TTL** and staleness tolerance.
- **Invalidation**: TTL-only, explicit on write, event-driven, versioned keys. The hardest part — keep it simple.
- **Patterns**: cache-aside (most common), read-through, write-through, write-behind (risk of loss), refresh-ahead.
- **Stampede protection**: request coalescing / single-flight, locks, jittered TTLs, stale-while-revalidate.
- **Failure mode**: system must work (slower) if cache is down; avoid cache becoming an implicit source of truth.
- **Memory bounds & eviction** policy (LRU/LFU).
- HTTP caching: `Cache-Control`, `ETag`, `Vary`; never cache personalized responses at shared layers without correct `Vary`/private.

## 4. Data access performance [MVP]
See [03](03-data.md): indexes, pagination, connection pooling, read replicas, partitioning. Also:
- Precompute/materialize expensive aggregates (materialized views, summary tables, counters updated async).
- Search engine for full-text/faceted search instead of `LIKE '%…%'` at scale.
- Batch writes; bulk inserts; upserts.

## 5. Application-level performance [MVP]
- Efficient algorithms/data structures on hot paths (watch accidental O(n²)).
- Stream large payloads (files, exports, LLM output) instead of buffering.
- Concurrency model appropriate to workload (async I/O for I/O-bound; worker processes/threads for CPU-bound).
- Avoid blocking calls on event loops.
- Serialization costs (large JSON) — consider pagination, compression, binary formats internally [SCALE].
- Reuse connections (keep-alive, HTTP/2, pooled clients); avoid creating clients per request.
- Startup time & cold starts (serverless, autoscaling) [PROD].

## 6. Network & edge [MVP]
- CDN for static assets and cacheable content; compression (brotli/gzip); HTTP/2 or HTTP/3.
- Minimize round trips (batching, request coalescing, preconnect).
- Place compute near data; avoid chatty cross-region calls.
- Edge compute for latency-critical, simple logic [SCALE].

## 7. Frontend performance [MVP]
See [17](17-frontend-clients.md): Core Web Vitals (LCP, INP, CLS), bundle size budgets, code splitting, image optimization, SSR/SSG/streaming where helpful, avoiding waterfalls.

## 8. Scalability strategies [SCALE]
- **Vertical** first (bigger machine) — simple, often enough longer than expected.
- **Horizontal** for stateless tiers; requires externalized state.
- **Async/queues** to absorb bursts ([05](05-async-messaging-workflows.md)).
- **Read scaling**: replicas, caches, CDNs.
- **Write scaling**: batching, partitioning, sharding, append-only logs.
- **Data partitioning** by tenant/time/key; plan for hot partitions.
- **Autoscaling** on meaningful signals (latency, queue age, concurrency), with limits and cooldowns.
- Identify the **first bottleneck** — scaling app servers past what the DB can handle just moves the problem.

## 9. Load & performance testing [PROD]
- Tools: k6, Locust, Gatling, JMeter, Artillery, vegeta.
- Test types: baseline, load (expected peak), stress (find breaking point), soak (hours — leaks, degradation), spike.
- Realistic data volumes and traffic mix; test against production-like environments.
- Performance budgets/regression checks in CI for critical paths (micro-benchmarks, bundle size, Lighthouse) [SCALE].
- Record results and the capacity model (e.g. "1 instance ≈ 400 rps at p95 120 ms").

## 10. Cost-performance trade-off [SCALE]
- Track cost per request/user/tenant ([15](15-infrastructure-cost.md)).
- Over-provisioning is a latency fix that costs money — make it a conscious choice.
- Caches and CDNs often reduce both latency and cost.

## Red flags
- Optimizing without profiling/traces.
- Averages only in dashboards.
- Caches without invalidation strategy or with keys missing tenant/user dimension.
- Unbounded in-memory caches.
- Horizontal scaling of app tier while the DB is the bottleneck.
- Microservice fan-out on every request without parallelization or timeouts.
- Loading entire files/result sets into memory.
- Premature sharding / exotic data stores for modest load.

## Decisions to record
Latency targets & budgets · caching layers, keys, invalidation · scaling strategy · capacity model · load test results.

## Evidence of done
- Latency histograms for critical endpoints with p95/p99 visible.
- Load test results for production-stage launches, with breaking point known.
- Cache invalidation & stampede behaviour tested.
- No N+1 or unbounded queries on hot paths (verified by tests/query logs).

## Related
[03](03-data.md) · [06](06-reliability.md) · [13](13-observability.md) · [15](15-infrastructure-cost.md) · [17](17-frontend-clients.md)
