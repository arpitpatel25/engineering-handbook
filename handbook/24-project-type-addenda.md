# 24 — Project-Type Addenda

The core guides assume a typical networked service. Other project types have additional or different concerns. Apply the sections matching your project type(s) from [00](00-project-profiling.md).

---

## A. Libraries, SDKs & frameworks
- **Public API surface** is the product: minimal, consistent, hard to misuse; everything else private/internal.
- **Semantic versioning** strictly; breaking changes only in majors; deprecation warnings a version ahead; migration guides.
- **Compatibility matrix**: supported language/runtime versions, platforms, frameworks; tested in CI matrix.
- **Dependency footprint** minimal; wide version ranges for dependencies; avoid pulling heavy transitive trees; optional dependencies/extras for integrations.
- **No global side effects** on import; no unsolicited logging to stdout; configurable logging via the host's logger; no telemetry without opt-in.
- **Errors**: typed, documented, stable; never `exit()` the host process.
- **Thread/async safety** documented.
- **Packaging**: correct metadata, types shipped (`.d.ts`, `py.typed`), ESM/CJS handling (JS), wheels for platforms (Python native), tree-shakeable.
- **Docs**: API reference generated from code, getting started, examples tested in CI, changelog.
- **Bundle size** budgets for frontend libraries.
- **Trusted publishing** and provenance; 2FA on registry accounts ([20](20-dependencies-supply-chain.md)).
- **SDKs for APIs**: retries/backoff, idempotency keys, pagination helpers, timeouts, typed models, configurable base URL, user-agent with version.

## B. CLI & developer tools
- Follow platform conventions (POSIX flags, `--help`, `--version`, subcommands); see clig.dev guidelines.
- **Exit codes** meaningful and documented; errors to stderr, data to stdout.
- **Machine-readable output** (`--json`) for scripting; stable output formats treated as API.
- Human-friendly defaults: colors (respect `NO_COLOR`, TTY detection), progress indicators, helpful errors with suggestions.
- Non-interactive mode for CI (`--yes`, env vars); never block on prompts without a TTY.
- Configuration precedence: flags > env > project config > user config > defaults (XDG paths).
- Safe destructive operations: confirmation, `--dry-run`, undo where possible.
- Distribution: single binary or package managers (Homebrew, apt, winget, Scoop, npm/pipx); signed binaries; auto-update or update notifications (opt-out).
- Shell completions; man pages.
- Fast startup; offline behaviour.
- Credentials stored in OS keychain, not plaintext.

## C. Mobile apps (native & cross-platform)
- **Platform choice**: native (Swift/Kotlin) vs cross-platform (React Native, Flutter, KMP) — trade performance/platform fidelity vs shared code.
- **Offline & flaky networks**: local persistence, sync strategy, conflict resolution, queued mutations, retry.
- **App lifecycle**: backgrounding, process death, state restoration, deep links/universal links.
- **Release process**: store review times; staged rollouts; **you cannot hotfix instantly** — feature flags and remote config are essential; minimum supported version enforcement / forced upgrade path.
- **API compatibility**: old app versions live for years — backend APIs must remain backward compatible longer ([04](04-apis-and-integrations.md)).
- **Security**: secure storage (Keychain/Keystore), certificate pinning considerations, no secrets in the binary, jailbreak/root considerations for high-risk apps, OWASP MASVS.
- **Performance**: startup time, frame rate, memory, battery, app size, network usage.
- **Permissions**: request in context, minimal, with rationale; privacy manifests/labels (App Store, Play data safety).
- **Push notifications**: token management, opt-in, payload privacy.
- **Crash reporting & analytics** with symbolication.
- **Accessibility** (VoiceOver/TalkBack, dynamic type).
- **Device matrix** testing; OS version support policy.
- Store compliance (in-app purchase rules, review guidelines).

## D. Desktop apps
- Framework (native, Electron, Tauri, Qt, .NET MAUI, Flutter) — trade footprint, performance, and dev speed.
- **Auto-update** with signed updates and rollback; code signing & notarization (macOS), Authenticode (Windows).
- **Electron security**: context isolation, sandbox, no `nodeIntegration` in renderers, strict CSP, IPC validation.
- OS integration: file associations, menus, shortcuts, tray, notifications, per-platform conventions.
- Local data: storage location conventions, migrations of local schema, backups, encryption of sensitive data.
- Offline-first behaviour; crash reporting; installer/uninstaller hygiene; enterprise deployment (MSI, MDM, config policies) [ENT].

## E. Data pipelines, ETL & analytics
- **Idempotent, re-runnable** jobs; partition-based processing; backfill capability.
- **Orchestration** (Airflow, Dagster, Prefect, dbt, cloud-native) with dependencies, retries, SLAs, alerting.
- **Data contracts** and schema evolution between producers and consumers; schema registry.
- **Data quality** checks (Great Expectations, dbt tests, Soda): freshness, volume, nulls, uniqueness, referential integrity, distribution drift.
- **Lineage** and catalog (OpenLineage, DataHub, Unity Catalog) [SCALE].
- Storage formats (Parquet, Iceberg/Delta/Hudi table formats); partitioning and file sizing.
- Batch vs streaming (Kafka/Flink/Spark Structured Streaming) per latency need.
- Late-arriving and out-of-order data handling; event time vs processing time.
- PII handling, masking, access controls on datasets; retention ([10](10-privacy-compliance.md)).
- Cost control for warehouse queries (partition pruning, clustering, quotas).
- Semantic layer / metrics definitions to keep numbers consistent.

## F. Machine learning systems (non-LLM or custom models)
- **Problem framing**: baseline (heuristic) first; offline metrics tied to business metrics.
- **Data**: versioning (DVC, lakeFS, table snapshots), labeling quality, train/validation/test splits without leakage, representativeness, bias audits.
- **Reproducibility**: tracked experiments (MLflow, W&B), seeded runs, pinned environments, recorded data versions.
- **Model registry** with versioning, lineage, approval stages.
- **Serving**: batch vs online; latency/throughput targets; feature consistency between training and serving (feature store if needed); model packaging; GPU/CPU cost.
- **Monitoring**: data drift, prediction drift, performance decay, feedback loops; retraining triggers.
- **Safe rollout**: shadow mode, canaries, A/B tests; rollback to previous model.
- **Responsible AI**: fairness, explainability where required, model cards, documentation for regulators ([10](10-privacy-compliance.md)).
- LLM-specific concerns: [19](19-ai-llm-features.md).

## G. Embedded, IoT & firmware
- Resource constraints (memory, CPU, power, storage) drive language and design; static allocation; deterministic timing.
- **Safety**: watchdogs, fail-safe states, brown-out handling; safety standards where applicable (IEC 61508, ISO 26262, IEC 62304).
- **OTA updates**: signed images, A/B partitions, rollback, power-loss resilience, staged rollouts.
- **Security**: secure boot, hardware root of trust, unique per-device credentials, encrypted comms, disabled debug ports in production, SBOM; regulations (EU Cyber Resilience Act, ETSI EN 303 645).
- **Connectivity**: intermittent networks, store-and-forward, backoff; protocols (MQTT, CoAP, BLE, LoRaWAN).
- **Fleet management**: provisioning, telemetry, remote config, decommissioning.
- Hardware-in-the-loop testing; simulators; long-duration soak tests.
- Long support lifetimes — plan for toolchain and dependency longevity.

## H. Games
- Game loop architecture, frame budget (16.6 ms @ 60 fps), deterministic simulation where networked.
- Engine choice and asset pipeline; asset versioning (LFS/Perforce).
- Multiplayer: authoritative server, lag compensation, cheat prevention (never trust the client), matchmaking, scaling sessions.
- Save data versioning and migration; cloud saves.
- Performance profiling on target hardware; memory budgets; loading times.
- Platform certification requirements (console TRCs/XRs); store policies; age ratings.
- Live ops: remote config, events, telemetry, economy balancing, anti-fraud for purchases.
- Accessibility options (remapping, subtitles, colour-blind modes, difficulty).

## I. Internal tools & back-office
- Often the right call to use low-code/admin frameworks (Retool, Appsmith, Django admin, React Admin) over custom builds.
- Still needs: SSO, role-based access, audit logging of actions on production data, input validation, confirmation for destructive bulk actions.
- Least privilege DB access (avoid superuser connections); read replicas for reporting.
- Lower polish requirements, not lower security requirements.

## J. Browser extensions & plugins
- Minimal permissions (host permissions especially); justify each in store listing.
- Manifest V3 constraints (service workers, no remote code).
- Content script isolation; message passing validation; never inject untrusted HTML.
- Privacy: data handling disclosure; no excessive collection.
- Store review processes and update cadence.
- Plugins for host apps: respect host APIs/lifecycle, versioned compatibility, sandboxing.

## K. Infrastructure / platform tooling
- Declarative, idempotent operations; dry-run/plan modes.
- Safe defaults and guardrails; blast-radius limits; confirmation for destructive operations.
- Backward-compatible configuration schemas with validation and migrations.
- Extensive integration tests against real (ephemeral) infrastructure.
- Clear upgrade paths and compatibility matrices; deprecation policies.

## L. Real-time & collaborative systems
- Transport (WebSockets, SSE, WebRTC); connection scaling (sticky sessions, pub/sub fan-out, presence).
- Consistency for collaboration: OT or CRDTs (Yjs, Automerge); conflict UX.
- Backpressure for slow clients; reconnection with resume/state sync; message ordering per channel.
- Authorization on subscribe and on each message; per-channel rate limits.

## M. Fintech, healthcare & other high-risk domains
- Double-entry ledgers for money movement; immutable transaction records; reconciliation; idempotency everywhere; precise decimal arithmetic; regulatory reporting.
- Four-eyes approval for sensitive operations; segregation of duties.
- Strong audit trails, retention per regulation, and evidence for auditors ([10](10-privacy-compliance.md)).
- Formal change management, validation, and documentation (e.g. FDA software validation, SOX controls).
- Clinical/safety risk management where applicable.
