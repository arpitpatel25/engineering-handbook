# 04 — APIs, Contracts & Integrations

APIs are promises. Internal ones bind modules and teams; public ones bind customers for years. Design them deliberately, document them as contracts, and evolve them without breaking consumers.

**Applies when:** adding/changing endpoints, SDKs, webhooks, event schemas, or integrating with third-party services.

## Key questions
- Who are the consumers (internal UI, partners, public developers, other services)?
- What style fits: resource-oriented, RPC, query language, streaming?
- How will it evolve without breaking consumers?
- How are errors, pagination, filtering, and limits expressed consistently?
- How are callers authenticated, authorized, and rate limited?
- What happens when a third party we depend on is down, slow, or wrong?

## 1. API style

| Style | Fits | Trade-offs |
|---|---|---|
| **REST / HTTP+JSON** | Public APIs, CRUD-ish resources, broad client compatibility, caching. | Over/under-fetching; conventions vary. |
| **GraphQL** | Many clients with differing data needs; aggregating many sources. | Caching, authorization per field, query cost limits, N+1 resolvers. |
| **gRPC / Protobuf** | Internal service-to-service, streaming, strict contracts, performance. | Browser support needs proxy; less human-debuggable. |
| **tRPC / typed RPC** | Full-stack TypeScript monorepo, single team. | Couples client to server language; not for public APIs. |
| **WebSockets / SSE** | Real-time server→client (SSE) or bidirectional (WS). | Connection management, scaling, auth refresh, reconnection. |
| **Webhooks (outbound)** | Notifying external systems of events. | Delivery guarantees, retries, security, ordering. |
| **Async messaging / events** | Internal decoupling. See [05](05-async-messaging-workflows.md). | Eventual consistency. |

## 2. Contract-first design [MVP]
- Machine-readable spec: **OpenAPI** (REST), GraphQL SDL, `.proto` (gRPC), **AsyncAPI** / JSON Schema / Avro (events).
- Generate docs, clients, server stubs, and validators from the spec — or generate the spec from code — but have **one** source of truth.
- Validate requests and responses against the contract in tests (contract tests) [PROD].
- Lint the spec (e.g. Spectral) for consistent conventions [PROD].

## 3. Conventions (pick once, apply everywhere) [MVP]
- **Naming**: plural resource nouns, consistent case (snake_case or camelCase in JSON — one), no verbs in REST paths except for explicit actions (`POST /invoices/{id}:void` or `/invoices/{id}/void`).
- **HTTP semantics**: correct methods (GET safe & idempotent, PUT/DELETE idempotent, POST not), correct status codes (400 vs 401 vs 403 vs 404 vs 409 vs 422 vs 429 vs 5xx).
- **Error format**: one structured format everywhere (e.g. RFC 9457 Problem Details): machine-readable `type`/`code`, human `title`/`detail`, field-level errors, correlation/request ID. Never leak stack traces or internals.
- **Pagination**: cursor-based for large/live collections; consistent params (`limit`, `cursor`) and response envelope (`next_cursor`, `has_more`); enforce max page size.
- **Filtering, sorting, field selection**: consistent syntax; allow-listed fields only.
- **Dates/times**: ISO 8601 / RFC 3339 in UTC. **Money**: minor units + currency. **IDs**: opaque strings.
- **Partial updates**: PATCH semantics defined (JSON Merge Patch vs JSON Patch) — null vs absent distinguished.
- **Bulk operations**: explicit bulk endpoints with per-item results and limits.
- **Long-running operations**: return `202 Accepted` + operation resource to poll, or callback/webhook.
- **Request/correlation ID** accepted and echoed (`X-Request-Id` / `traceparent`).

## 4. Idempotency & concurrency [MVP]
- Support **Idempotency-Key** header on non-idempotent operations with side effects (payments, orders, sends). Store key + request fingerprint + response for a retention window; return the same response on replay; reject same key with different payload.
- Optimistic concurrency via `ETag` + `If-Match` (or version field) to prevent lost updates.
- Conditional GET (`If-None-Match`) for caching where useful.

## 5. Versioning & evolution [MVP for public; PROD for internal]
- Strategy: URI (`/v1`), header, or date-based versions (`API-Version: 2025-01-01`). Pick one.
- **Backward-compatible changes**: adding optional fields/endpoints/enum values (if clients tolerate unknown values).
- **Breaking changes**: removing/renaming fields, changing types/semantics, new required inputs, tightening validation, changing error codes, changing default behaviour.
- Clients must be **tolerant readers** (ignore unknown fields); servers should be conservative in what they send.
- Deprecation policy: announcement, `Deprecation`/`Sunset` headers, timeline, usage tracking per consumer, migration guide.
- Changelog for API consumers.

## 6. Security of APIs [ALL]
See [08](08-security.md) & [09](09-identity-access-tenancy.md). API-specific:
- Authenticate every non-public endpoint; authorize every object access (**BOLA/IDOR** is the #1 API vulnerability — check that the caller may access *this* resource, not just *this type*).
- Avoid mass assignment: allow-list writable fields.
- Input validation on every field (type, length, range, format); reject unknown fields where appropriate.
- Rate limits and quotas per key/user/tenant/IP; return `429` with `Retry-After`.
- Request size limits; timeouts; pagination caps; GraphQL depth/complexity limits.
- CORS restricted to known origins; no wildcard with credentials.
- API keys: hashed at rest, prefixed for identification and secret scanning, scoped, rotatable, expirable, last-used tracked.
- Never put secrets/tokens in URLs (they leak to logs).

## 7. Outbound webhooks [PROD]
- Signed payloads (HMAC with timestamp) to allow verification; replay window.
- At-least-once delivery with retries (exponential backoff), delivery log, manual redelivery, auto-disable after sustained failure with notification.
- Event IDs for consumer deduplication; ordering not guaranteed unless documented.
- Thin events (ID + type, consumer fetches) vs fat events (full payload) — trade-off: freshness/security vs fewer calls.
- SSRF protection on customer-supplied URLs (block internal IP ranges, metadata endpoints, re-check after DNS resolution and redirects).
- Timeouts on delivery; don't let slow consumers block others (per-endpoint queues).

## 8. Inbound webhooks & third-party integrations [MVP]
- Verify signatures; reject stale timestamps; respond fast (2xx) and process async.
- Deduplicate by provider event ID.
- Don't trust payload content blindly — refetch authoritative state from the provider when it matters.
- Wrap each vendor in an adapter/anti-corruption layer; map their model to yours.
- **Timeouts, retries with backoff, circuit breakers** on all outbound calls ([06](06-reliability.md)).
- Respect provider rate limits; queue and throttle.
- Sandbox/test modes and recorded fixtures for tests; never hit real third parties in unit tests.
- Monitor third-party error rates & latency separately.
- Know each vendor's SLA, data processing terms, and what happens to your product when they're down.

## 9. SDKs & client libraries [SCALE / public APIs]
- Generated from spec or hand-written idiomatically per language; versioned with semver.
- Built-in retries with backoff for safe operations, idempotency key support, pagination helpers, typed errors, timeouts, telemetry hooks.
- See [24](24-project-type-addenda.md) for library concerns.

## 10. Developer experience (public APIs) [ENT]
- Reference docs + guides + quickstart + runnable examples.
- API explorer / Postman collection.
- Test mode / sandbox with test keys.
- Status page and changelog.
- Clear limits documentation (rate limits, payload sizes, retention).

## Red flags
- Different error shapes across endpoints.
- 200 OK with `{"error": …}` bodies.
- Authorization checked by type only (any logged-in user can fetch any `/orders/{id}`).
- Unbounded list endpoints.
- Breaking changes deployed without versioning.
- Business logic in controllers/handlers (should be in domain/service layer).
- Calling third parties without timeouts.
- Webhook handlers doing heavy work synchronously.
- Exposing database models directly as API responses.

## Decisions to record
API style · versioning strategy · error format · pagination style · auth mechanism(s) · idempotency approach · webhook delivery guarantees · deprecation policy.

## Evidence of done
- Spec exists and matches implementation (contract test or generation).
- Every endpoint has authn/authz tests including cross-tenant/other-user access denial.
- Pagination limits and rate limits enforced.
- Third-party calls have timeouts and failure handling tested.
- API changelog updated for public changes.

## Related
[05](05-async-messaging-workflows.md) · [06](06-reliability.md) · [08](08-security.md) · [09](09-identity-access-tenancy.md) · [18](18-saas-enterprise.md)
