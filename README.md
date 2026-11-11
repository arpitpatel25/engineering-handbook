# Engineering Handbook

A project-agnostic decision handbook for building production-grade software — written to be read by **both humans and AI coding agents**.

It is not a framework, a stack, or a set of mandates. It is a structured map of **everything worth considering** when building software that should survive contact with real users, real load, real attackers, real auditors, and real maintainers. Each area is laid out as questions, options, trade-offs, red flags, and evidence of done — so whoever is building (person or agent) can make a deliberate, recorded decision instead of an accidental one.

> **Most of this will not apply to any given project. That is by design.**
> The value is in having *consciously decided* something is out of scope, not in doing everything.

---

## Who this is for

- **AI coding agents** (Claude Code, Codex, Cursor, Copilot, etc.) working on any repository. Start at [`AGENTS.md`](AGENTS.md).
- **Developers** directing AI agents who want consistent, high-quality decisions without re-explaining standards every session.
- **Reviewers** assessing whether a codebase is production- or enterprise-ready.

## How it is organized

| Path | What it holds |
|---|---|
| [`AGENTS.md`](AGENTS.md) | Operating instructions for agents: how to triage relevance, how to decide, how to record. **Start here.** |
| [`PRINCIPLES.md`](PRINCIPLES.md) | The small set of cross-cutting principles that everything else derives from. |
| [`handbook/`](handbook/) | Deep-dive guides, one per area (architecture, data, security, reliability, …). |
| [`checklists/`](checklists/) | Condensed, tickable checklists by project maturity stage and by activity (design review, PR review, launch). |
| [`templates/`](templates/) | Drop-in templates: ADR, NFR sheet, feature spec, threat model, runbook, postmortem, project `AGENTS.md`, PR template, etc. |

### Handbook index

| # | Area | Guide |
|---|---|---|
| 00 | Project profiling & relevance triage | [handbook/00-project-profiling.md](handbook/00-project-profiling.md) |
| 01 | Problem framing & requirements (incl. NFRs) | [handbook/01-requirements.md](handbook/01-requirements.md) |
| 02 | Architecture & system design | [handbook/02-architecture.md](handbook/02-architecture.md) |
| 03 | Data: modeling, storage, lifecycle | [handbook/03-data.md](handbook/03-data.md) |
| 04 | APIs, contracts & integrations | [handbook/04-apis-and-integrations.md](handbook/04-apis-and-integrations.md) |
| 05 | Async processing, queues, events & workflows | [handbook/05-async-messaging-workflows.md](handbook/05-async-messaging-workflows.md) |
| 06 | Reliability & resilience | [handbook/06-reliability.md](handbook/06-reliability.md) |
| 07 | Performance, latency & scalability | [handbook/07-performance-scalability.md](handbook/07-performance-scalability.md) |
| 08 | Security | [handbook/08-security.md](handbook/08-security.md) |
| 09 | Identity, access & multi-tenancy | [handbook/09-identity-access-tenancy.md](handbook/09-identity-access-tenancy.md) |
| 10 | Privacy, compliance & governance | [handbook/10-privacy-compliance.md](handbook/10-privacy-compliance.md) |
| 11 | Code design & repository structure | [handbook/11-code-design.md](handbook/11-code-design.md) |
| 12 | Testing & quality | [handbook/12-testing-quality.md](handbook/12-testing-quality.md) |
| 13 | Observability | [handbook/13-observability.md](handbook/13-observability.md) |
| 14 | Delivery: CI/CD, environments, releases | [handbook/14-delivery-cicd.md](handbook/14-delivery-cicd.md) |
| 15 | Infrastructure, cloud & cost | [handbook/15-infrastructure-cost.md](handbook/15-infrastructure-cost.md) |
| 16 | Operations & incident management | [handbook/16-operations-incidents.md](handbook/16-operations-incidents.md) |
| 17 | Frontend & client applications | [handbook/17-frontend-clients.md](handbook/17-frontend-clients.md) |
| 18 | SaaS & enterprise product capabilities | [handbook/18-saas-enterprise.md](handbook/18-saas-enterprise.md) |
| 19 | AI / LLM-powered features | [handbook/19-ai-llm-features.md](handbook/19-ai-llm-features.md) |
| 20 | Dependencies, supply chain & licensing | [handbook/20-dependencies-supply-chain.md](handbook/20-dependencies-supply-chain.md) |
| 21 | Documentation & repository hygiene | [handbook/21-documentation.md](handbook/21-documentation.md) |
| 22 | Open source & community | [handbook/22-open-source.md](handbook/22-open-source.md) |
| 23 | Working with AI agents | [handbook/23-working-with-ai-agents.md](handbook/23-working-with-ai-agents.md) |
| 24 | Project-type addenda (libraries, CLIs, mobile, data/ML, desktop, embedded, games, internal tools) | [handbook/24-project-type-addenda.md](handbook/24-project-type-addenda.md) |
| 25 | Glossary of patterns & terms | [handbook/25-glossary.md](handbook/25-glossary.md) |

## Using it from another project

Pick whichever fits:

1. **Reference it globally.** Clone it somewhere stable (e.g. `~/engineering-handbook`) and add one line to your global agent instructions (e.g. `~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`):
   ```
   For any software design, architecture, or quality decision, consult ~/engineering-handbook/AGENTS.md first and follow its process.
   ```
2. **Reference it per project.** Copy [`templates/project-AGENTS.md`](templates/project-AGENTS.md) into the target repo as `AGENTS.md` / `CLAUDE.md` and fill it in.
3. **Vendor it.** Add as a git submodule (`git submodule add <url> .handbook`) so agents in CI/cloud sandboxes can read it.

## Stance

- **Checklists, not commandments.** Options are presented with trade-offs. Context decides.
- **Proportionality.** A weekend prototype and a bank's ledger deserve different rigor. See [00 — Project profiling](handbook/00-project-profiling.md).
- **Deviation is allowed — silence is not.** Rejecting a recommendation is fine; doing so without recording why is the failure mode.
- **Living document.** When a project teaches a lesson this handbook lacked, add it here.
