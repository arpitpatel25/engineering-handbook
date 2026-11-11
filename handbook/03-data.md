# 03 — Data: Modeling, Storage & Lifecycle

Data outlives code. Schemas, identifiers, and storage choices are among the hardest decisions to reverse — give them disproportionate care.

**Applies when:** designing schemas, choosing databases, writing migrations, adding indexes, handling large data, defining retention or deletion.

## Key questions
- What are the core entities, their relationships, and invariants?
- What are the access patterns (queries, frequency, latency needs)?
- What consistency and transactional guarantees are needed?
- How big will it get, how fast, and how long do we keep it?
- Who/what may read or modify each piece of data?
- How do we change the schema safely while running?
- How do we back up, restore, export, and delete it?

## 1. Storage selection [ALL]

| Store type | Strengths | Watch out for |
|---|---|---|
| **Relational (Postgres, MySQL, SQL Server, SQLite)** | Transactions, constraints, joins, mature tooling, flexible queries. A sensible default for most business data. | Horizontal write scaling needs planning; schema migrations at scale. |
| **Document (MongoDB, DynamoDB*, Firestore)** | Flexible schema, natural for aggregate-shaped data, horizontal scale. | Cross-document consistency, ad-hoc queries, schema drift. |
| **Key-value (Redis, Valkey, DynamoDB, etcd)** | Very low latency, simple access. | Limited querying; persistence/durability semantics vary. |
| **Wide-column (Cassandra, ScyllaDB, Bigtable)** | Massive write throughput, time-series-ish access. | Query-driven modeling; limited transactions. |
| **Search (OpenSearch/Elasticsearch, Meilisearch, Typesense, Postgres FTS)** | Full-text, faceting, relevance. | Not a source of truth; index sync. |
| **Time-series (TimescaleDB, InfluxDB, Prometheus, ClickHouse)** | Metrics/events by time, downsampling. | Cardinality limits. |
| **Analytical / OLAP (ClickHouse, BigQuery, Snowflake, DuckDB, Redshift)** | Aggregations over huge datasets. | Not for transactional workloads. |
| **Graph (Neo4j, Neptune)** | Deep relationship traversal. | Niche ops/tooling; often a relational model suffices. |
| **Vector (pgvector, Qdrant, Pinecone, Weaviate)** | Similarity search for embeddings. | Index tuning, recall/latency trade-offs; often pgvector suffices. |
| **Object storage (S3, GCS, Azure Blob, R2)** | Large files, cheap, durable. | Eventual semantics on some ops, access control, egress cost. |
| **Embedded (SQLite, RocksDB, DuckDB)** | Local-first, single-node, edge, mobile, tests. | Concurrency limits; replication tooling. |

Considerations:
- Prefer **fewer** data stores; each one adds ops, backup, security, and consistency surface.
- A general-purpose relational DB can often cover search, queues, JSON documents, and vectors at small-to-moderate scale — evaluate before adding specialized stores.
- Managed vs self-hosted: managed costs money; self-hosted costs people and risk.

## 2. Data modeling [ALL]
- Model the **domain**, not the UI. Name tables/fields in the ubiquitous language.
- Enforce invariants **in the database** where possible: `NOT NULL`, `UNIQUE`, `FOREIGN KEY`, `CHECK`, exclusion constraints. Application-only validation will eventually be bypassed.
- Normalize by default; denormalize deliberately for measured read performance, with a plan to keep copies consistent.
- **Identifiers**: decide internal vs external IDs. Options: auto-increment (compact, leaks counts, enumerable), UUIDv4 (random, index fragmentation), **UUIDv7/ULID** (time-ordered, index-friendly), snowflake-style. Consider prefixed public IDs (`usr_…`) for debuggability.
- **Timestamps**: store in UTC with timezone-aware types; `created_at`, `updated_at`; consider `deleted_at` for soft delete.
- **Money**: integer minor units or exact decimal + currency code. Never floats.
- **Enums/status fields**: model state machines explicitly; document allowed transitions.
- **Polymorphism**: be careful with polymorphic associations (no FK integrity); consider separate tables or a supertype table.
- **JSON columns**: good for genuinely schemaless/extension data; bad as a way to avoid modeling.
- **Multi-tenancy**: tenant key on every tenant-owned row; composite indexes beginning with tenant id; consider row-level security. See [09](09-identity-access-tenancy.md).
- **Soft vs hard delete**: soft delete complicates every query and uniqueness constraint; hard delete complicates audit. Decide per entity; privacy law may require hard deletion.
- **Optimistic concurrency**: version column / ETag for concurrent edits [MVP].
- **Audit/history**: history tables, temporal tables, CDC, or event log when "who changed what when" matters [PROD].

## 3. Querying & performance [MVP]
- Index for actual query patterns; check with `EXPLAIN (ANALYZE)`.
- Composite index column order matters (equality first, then range; tenant first in multi-tenant).
- Avoid **N+1 queries**; batch/eager load deliberately.
- **Always bound result sets**: pagination (cursor/keyset for large or live data; offset only for small sets), limits on exports.
- Avoid `SELECT *` on hot paths; fetch what you need.
- Watch long-running transactions, lock contention, and hot rows (counters → sharded counters or async aggregation).
- Connection pooling (app-side and/or proxy such as PgBouncer) [PROD]; size pools to DB limits across all instances.
- Statement timeouts and lock timeouts configured [PROD].
- Slow query logging and query performance insights enabled [PROD].
- Read replicas for read-heavy workloads — beware replication lag and read-your-writes [SCALE].
- Partitioning (by time/tenant) for very large tables [SCALE]; sharding as a last resort, with a carefully chosen shard key [SCALE].

## 4. Transactions & consistency [MVP]
- Understand your DB's isolation levels (read committed vs repeatable read vs serializable) and choose per operation.
- Keep transactions short; never call external services inside a DB transaction.
- Use `SELECT … FOR UPDATE`, advisory locks, or optimistic concurrency for read-modify-write races.
- Uniqueness and idempotency enforced by constraints, not by check-then-insert.
- Cross-store consistency (DB + queue, DB + search index): outbox pattern / CDC ([05](05-async-messaging-workflows.md)).

## 5. Schema migrations [ALL]
- Migrations as versioned code, applied automatically and identically in every environment.
- Never edit an applied migration; add a new one.
- [PROD] **Zero-downtime migrations** via expand → migrate/backfill → contract:
  1. Add new nullable column/table (compatible with old code).
  2. Deploy code that writes both / reads new with fallback.
  3. Backfill in batches (throttled, resumable, idempotent).
  4. Switch reads; then stop writing old.
  5. Drop old column in a later release.
- [PROD] Know which DDL locks tables (e.g. adding a column with default on older engines, creating indexes non-concurrently, changing column types). Use online/concurrent variants.
- [PROD] Migrations must be compatible with the **previous** app version (for rollbacks and rolling deploys).
- Down migrations / rollback plan, or explicit "forward-fix only" policy.
- Large data backfills run as jobs, not inside the deploy migration step.
- Test migrations against production-like data volume [SCALE].

## 6. Data lifecycle [MVP]
- **Classification**: public, internal, confidential, restricted/PII/PHI/PCI. Drives access, encryption, logging, retention ([10](10-privacy-compliance.md)).
- **Retention**: how long each data class is kept; automated purge jobs.
- **Deletion**: user/tenant deletion that cascades through DB, object storage, search indexes, caches, analytics, backups (or documented backup expiry), and third-party processors.
- **Archival**: cold storage for old data; restore path.
- **Export / portability**: user/tenant data export in a usable format [ENT].
- **Data residency**: where data physically lives; region pinning [ENT].
- **Seed & fixture data**: realistic, non-production, no real PII in dev/test.
- **Production data in lower environments**: avoid; if needed, anonymize/mask.

## 7. Backup & restore [MVP]
- Automated backups with point-in-time recovery where available.
- RPO/RTO defined ([01](01-requirements.md)) and backups configured to meet them.
- Backups encrypted, stored in a separate account/region, access-restricted, immutable/locked where ransomware is a concern [PROD].
- **Restore is tested regularly** — an untested backup is not a backup [PROD].
- Object storage: versioning and lifecycle rules; protection against accidental deletion.

## 8. Analytics & reporting [SCALE]
- Don't run heavy analytics on the primary OLTP database; use replicas, CDC to a warehouse, or ETL.
- Define event/tracking schema deliberately; version it.
- Separate product analytics (behaviour) from business reporting (financials) and from operational metrics.
- Data quality checks, lineage, and ownership for pipelines ([24](24-project-type-addenda.md)).

## 9. Caching data [MVP]
See [07](07-performance-scalability.md). Key data concerns: source of truth is always the DB; cache keys include tenant/user/version; invalidation strategy explicit; no sensitive data cached where it can leak across users.

## 10. Files & blobs [MVP]
- Store in object storage, metadata in DB.
- Direct-to-storage uploads via pre-signed URLs for large files.
- Validate type (magic bytes, not just extension), size limits, malware scanning where risk warrants [PROD].
- Serve user content from a separate domain or with safe `Content-Disposition`/`Content-Type` to avoid XSS.
- Image/video processing asynchronously.

## Red flags
- Floats for money; local time stored without zone.
- No constraints in the DB; uniqueness only checked in code.
- Unbounded queries / exports; offset pagination over millions of rows.
- Migrations that lock large tables during deploy; destructive migration in same release as code change.
- Backups never restored.
- Multiple services writing the same table.
- Search index or cache treated as source of truth.
- Real customer PII in dev/staging/test fixtures.
- Sequential IDs exposed in URLs where enumeration is a risk.

## Decisions to record
Primary store(s) and why · ID strategy · tenancy data model · soft/hard delete policy · retention per data class · migration policy · backup RPO/RTO · consistency/isolation choices for critical operations.

## Evidence of done
- Schema has constraints matching invariants.
- Migrations are reversible or explicitly forward-only, and backward-compatible.
- Hot queries have supporting indexes verified with query plans.
- Backup restore tested (date recorded).
- Retention & deletion jobs exist for data classes that require them.

## Related
[02](02-architecture.md) · [05](05-async-messaging-workflows.md) · [07](07-performance-scalability.md) · [09](09-identity-access-tenancy.md) · [10](10-privacy-compliance.md)
