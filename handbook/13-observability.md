# 13 — Observability

Observability is the ability to answer "what is happening and why?" in production — including for questions you didn't anticipate. Without it, every incident is guesswork and every performance claim is a guess.

**Applies when:** anything deployed. Rigor scales with stage.

## Key questions
- How do we know the system is healthy right now — from the user's perspective?
- Given a user complaint, can we find their request and see what happened end-to-end?
- What will page someone, and is every page actionable?
- What does telemetry cost, and does it leak sensitive data?

## 1. Foundations [MVP]
- **Structured logs** (JSON or logfmt) with consistent fields: timestamp (UTC, ISO 8601), level, service, environment, version/commit, message, `request_id`/`trace_id`, `tenant_id`, `user_id` (if allowed), and event-specific fields.
- **Correlation**: accept or generate a request ID at the edge; propagate through all services, jobs, and messages (W3C Trace Context `traceparent`).
- **Error tracking** (Sentry, Rollbar, Bugsnag, Honeybadger, or OpenTelemetry-based) with release tagging, grouping, and source maps.
- **Uptime/synthetic checks** for key endpoints from outside.
- Centralized log aggregation with retention and search.

## 2. The signals [PROD]
- **Logs**: discrete events with context. Log at boundaries and decisions, not every line.
- **Metrics**: aggregated numeric time series — cheap to store, fast to query, basis for alerts.
- **Traces**: request flow across services with timing per span — essential for latency debugging in distributed systems.
- **Profiles** (continuous profiling: Pyroscope, Parca, cloud profilers) [SCALE].
- **Events / wide structured events**: high-cardinality events per request enabling ad-hoc slicing (Honeycomb-style) [SCALE].
- **Real User Monitoring (RUM)** & frontend errors [PROD for web/mobile].

## 3. Standards & tooling [PROD]
- **OpenTelemetry** (OTel) as the vendor-neutral standard for traces, metrics, and logs; auto-instrumentation for HTTP, DB, queues, plus manual spans for business operations.
- Backends: Grafana stack (Loki/Tempo/Mimir/Prometheus), Datadog, New Relic, Honeycomb, Elastic, cloud-native (CloudWatch, Cloud Monitoring, Azure Monitor), SigNoz, etc.
- Collector/agent layer (OTel Collector) for routing, sampling, redaction.

## 4. What to measure [PROD]
- **RED** for every service/endpoint: **R**ate, **E**rrors, **D**uration (histograms → p50/p95/p99).
- **USE** for resources: **U**tilization, **S**aturation, **E**rrors (CPU, memory, disk, connections, pools, threads).
- **Four golden signals**: latency, traffic, errors, saturation.
- Queues: depth, oldest message age, processing rate, failures, DLQ size, consumer lag ([05](05-async-messaging-workflows.md)).
- Dependencies: per external service latency/errors/timeouts/circuit-breaker state.
- Business/product metrics: signups, orders, payments succeeded/failed, active tenants — often the fastest indicator something is broken.
- SLIs for SLOs ([06](06-reliability.md)).
- Cost signals: telemetry volume, LLM tokens, third-party API usage [SCALE].

## 5. Logging discipline [MVP]
- Levels used consistently: `error` (needs attention), `warn` (unexpected but handled), `info` (significant business events, lifecycle), `debug` (off in prod by default).
- **Never log**: secrets, tokens, passwords, full card numbers, session cookies, and — by default — PII or request/response bodies. Redaction at the logger/collector level as defence in depth ([10](10-privacy-compliance.md)).
- Log once per error at the boundary that handles it (avoid duplicate stack traces at every layer).
- Include identifiers to join context, not entire objects.
- Control volume: sampling for high-volume success logs; rate-limit noisy logs.
- Separate **audit logs** (tamper-evident, long retention, compliance) from operational logs ([18](18-saas-enterprise.md)).

## 6. Metrics hygiene [PROD]
- Use histograms for latency (not averages, not gauges).
- **Cardinality control**: no unbounded label values (user IDs, request IDs, raw URLs) in metrics — use route templates; put high-cardinality data in traces/events.
- Consistent naming and units (`http_server_request_duration_seconds`); follow OTel semantic conventions.

## 7. Tracing [PROD]
- Propagate context across HTTP, gRPC, queues, scheduled jobs, and async boundaries.
- Span names = operation (route template, job type), attributes = details (tenant, entity IDs where allowed).
- Sampling strategy: head-based percentage + tail-based to keep errors and slow traces [SCALE].
- Link logs to traces (trace_id in every log line).

## 8. Alerting [PROD]
- **Alert on symptoms users feel** (SLO burn rate, error rate, latency, failed payments), not every cause (CPU 80%).
- Every page must be **actionable, urgent, and owned**, with a linked runbook ([16](16-operations-incidents.md)).
- Severity levels: page (wake someone) vs ticket (next business day) vs dashboard only.
- Multi-window burn-rate alerts for SLOs reduce noise [SCALE].
- Alert on absence: scheduled jobs that didn't run, no traffic when there should be, heartbeats.
- Review alert noise regularly; delete alerts nobody acts on.
- Route alerts to the owning team (on-call tooling: PagerDuty, Opsgenie, incident.io, Grafana OnCall).

## 9. Dashboards [PROD]
- One overview per service: RED, saturation, dependencies, deploy markers.
- One per critical user journey / SLO.
- Business KPI dashboard.
- Dashboards as code (Grafana JSON/Terraform) versioned in repo [SCALE].

## 10. Debuggability [MVP]
- Request ID shown to users in error pages/API errors so support can look it up.
- Version/commit exposed in a health/info endpoint and in all telemetry.
- Ability to raise log level dynamically or enable debug for a tenant/request [SCALE].
- Admin/support tooling to inspect entity state and job history ([18](18-saas-enterprise.md)).

## 11. Cost & retention [SCALE]
- Telemetry can rival infra cost — set retention per signal, sample, drop debug logs in prod, aggregate.
- Longer retention for audit and SLO data; shorter for debug-level detail.

## Red flags
- `console.log`/`print` debugging left in production paths; unstructured logs.
- No way to correlate a user's error report with backend logs.
- Alerts that fire daily and are ignored.
- PII/tokens in logs or error trackers.
- User IDs as metric labels.
- Dashboards only of CPU/memory; nothing user-centric.
- Average latency as the headline metric.

## Decisions to record
Telemetry stack · log schema · sampling strategy · SLOs & alert policy · retention · redaction approach.

## Evidence of done
- Every new endpoint/job emits RED metrics and traces automatically (instrumentation verified).
- Errors land in the error tracker with release tag.
- Alerts exist for new critical paths, each with a runbook link.
- Logs verified free of secrets/PII for new code paths.

## Related
[06](06-reliability.md) · [07](07-performance-scalability.md) · [10](10-privacy-compliance.md) · [16](16-operations-incidents.md)
