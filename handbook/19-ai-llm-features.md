# 19 — AI / LLM-Powered Features

LLM features are probabilistic components inside deterministic systems. They need the same engineering discipline as any dependency — timeouts, cost control, security — plus things unique to them: evaluation, prompt/version management, grounding, and defence against prompt injection.

**Applies when:** calling LLMs or other ML models, building RAG, agents, tool use, classification/extraction with models, or AI copilots.

## Key questions
- What exactly should the model do, and how will we measure if it's doing it well?
- Is an LLM the right tool, or would deterministic code, search, or a small model suffice?
- What happens when the model is wrong, slow, unavailable, or manipulated?
- What data is sent to which provider, and under what terms?
- What does each call cost, and how does that scale?

## 1. Problem fit [ALL]
- Use LLMs for language understanding/generation, fuzzy classification, extraction, summarization, reasoning over unstructured data. Prefer deterministic code for exact logic, math, permissions, and anything with a precise spec.
- Define success criteria and failure tolerance up front (e.g. "extraction F1 ≥ 0.95; human review for low confidence").
- Design the UX for imperfection: show sources, allow edits/undo, confirm before consequential actions, set expectations.

## 2. Model & provider selection [MVP]
- Criteria: quality on **your** eval set, latency, cost, context window, tool-use/structured output support, multimodality, data retention terms, region availability, rate limits, reliability.
- Use the smallest/cheapest model that passes evals per task; route tasks to different models where useful.
- Pin model versions; re-run evals before upgrading.
- Provider abstraction (thin) for fallback or switching — balanced against using provider-specific strengths.
- Self-hosted/open-weight models when data control, cost at scale, or offline needs justify the ops burden.

## 3. Prompt & context engineering [MVP]
- Prompts are code: versioned in the repo, reviewed, tested by evals, with change history.
- Clear system instructions, task description, constraints, output format, examples (few-shot) where helpful.
- **Structured outputs** (JSON schema / tool calling) for machine-consumed responses; validate and handle parse failures.
- Context assembly is the main quality lever: relevant, minimal, well-ordered context; explicit delimiters between instructions and data.
- Prompt caching for stable prefixes to cut cost/latency.
- Templates parameterized; no ad-hoc string concatenation of untrusted input into instructions.

## 4. Retrieval-augmented generation (RAG) [MVP if applicable]
- Ingestion: parsing, cleaning, chunking strategy (by structure/semantics, with overlap where useful), metadata (source, tenant, permissions, timestamps).
- Retrieval: embeddings + vector search, keyword/BM25, **hybrid** search, reranking; filters by tenant/permissions **at query time**.
- Freshness: re-indexing on source changes; deletion propagation ([10](10-privacy-compliance.md)).
- Grounding: instruct to answer from context, cite sources, say "I don't know".
- Evaluate retrieval separately (recall@k, MRR) from generation (faithfulness, answer relevance).
- Simpler alternatives first: long-context stuffing, structured search, or SQL may beat a vector DB.

## 5. Agents & tool use [MVP if applicable]
- Start with the simplest pattern: single call → chain/workflow → routing → agent loop. Use autonomous loops only when the path genuinely can't be predetermined.
- Tools: narrow, well-described, typed inputs, idempotent where possible, permission-checked **as the end user** (not a superuser).
- Guardrails: max steps/iterations, time and token budgets, loop detection.
- **Human-in-the-loop** approval for irreversible or high-impact actions (payments, deletions, sending messages, code execution).
- Sandboxing for code execution and browsing; network egress control.
- Full trace of agent steps for debugging and audit.
- Interoperability standards (e.g. MCP for tools/context) where they reduce integration work.

## 6. Security [ALL]
- **Prompt injection** (direct and indirect via retrieved docs, web pages, emails, tool outputs) is unsolved — design assuming the model can be manipulated:
  - Least privilege for tools; never give the model capabilities the current user doesn't have.
  - Separate trusted instructions from untrusted data; treat model output as untrusted input.
  - Confirmation for sensitive actions; restrict data exfiltration channels (e.g. rendering arbitrary URLs/images, outbound requests).
- Output handling: escape/sanitize before rendering (XSS), never `eval` model output, validate before executing SQL/commands.
- Data leakage: tenant isolation in retrieval and memory; no cross-user context mixing; system prompt may leak — don't put secrets in it.
- Abuse: rate limits, spend caps per user/tenant, content moderation where user-facing.
- Reference: OWASP Top 10 for LLM Applications.

## 7. Privacy & compliance [MVP]
- Provider data terms: retention, training usage, zero-data-retention options, region; DPAs ([10](10-privacy-compliance.md)).
- Minimize/redact PII in prompts and logs; disclose AI use to users where required.
- EU AI Act and sector rules: risk classification, transparency, human oversight, documentation [ENT].
- Customer controls: opt-out of AI features, choice of provider/region [ENT].

## 8. Evaluation [MVP]
- **Evals are the tests of AI features.** Build a dataset of representative inputs + expected outputs/criteria, including edge cases and adversarial examples; grow it from production failures.
- Methods: exact/structured match, heuristics, reference-based metrics, **LLM-as-judge** (calibrated against human labels), human review.
- Run evals in CI on prompt/model/retrieval changes; track scores over time; block regressions.
- Online evaluation: user feedback (thumbs, edits, acceptance rate), sampled human review, A/B tests [PROD].

## 9. Reliability & performance [MVP]
- Timeouts, retries with backoff on rate limits/5xx, fallbacks (alternate model/provider, cached answer, graceful "unavailable" state) ([06](06-reliability.md)).
- **Streaming** responses for perceived latency in interactive UIs.
- Async/background processing for long tasks ([05](05-async-messaging-workflows.md)).
- Handle non-determinism: temperature settings appropriate to task; validation and retry on malformed output.
- Semantic or exact caching where safe.
- Batch APIs for offline bulk workloads (cheaper).

## 10. Cost management [MVP]
- Track tokens and cost per request, feature, user, and tenant ([15](15-infrastructure-cost.md)).
- Budgets, alerts, and hard caps; plan-based quotas ([18](18-saas-enterprise.md)).
- Levers: smaller models, shorter prompts, prompt caching, retrieval precision, output length limits, batching, caching.

## 11. Observability [PROD]
- Log/trace each call: model, version, prompt template version, token counts, latency, cost, tool calls, outcome — with PII redaction ([13](13-observability.md)).
- LLM observability tools (Langfuse, LangSmith, Helicone, Arize Phoenix, Braintrust, OpenTelemetry GenAI conventions).
- Dashboards for quality signals, error/refusal rates, latency, and spend.

## 12. ML systems beyond LLMs
See [24](24-project-type-addenda.md) — data pipelines, training, feature stores, model registry, drift monitoring.

## Red flags
- No eval set; "it looked good on a few examples".
- Model output executed, rendered as HTML, or used in queries without validation.
- Agents with broad credentials and no approval step for destructive actions.
- Retrieval without tenant/permission filtering.
- Unpinned model versions in production.
- No per-user cost limits on a public AI feature.
- Prompts scattered as inline strings across the codebase.
- Sending customer data to providers without reviewing terms.

## Decisions to record
Model(s) per task & why · provider data terms · prompt versioning approach · eval methodology & thresholds · agent permission model · cost limits.

## Evidence of done
- Eval suite exists, runs in CI, and passes thresholds.
- Prompt injection considered for every untrusted input path; tool permissions scoped.
- Cost and latency per call observable; limits configured.
- Fallback behaviour on provider failure tested.

## Related
[05](05-async-messaging-workflows.md) · [06](06-reliability.md) · [08](08-security.md) · [10](10-privacy-compliance.md) · [23](23-working-with-ai-agents.md)
