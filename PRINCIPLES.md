# Principles

Everything in this handbook derives from a handful of principles. When a specific guide doesn't cover a situation, reason from these.

## 1. Decisions over code
Code is cheap to produce; the decisions it encodes are what matter. A good codebase makes its decisions **visible** (ADRs, specs, named concepts) and **reversible where possible**.

## 2. Proportionality
Rigor should match risk and stage. Over-engineering is a defect just as under-engineering is. Ask: *what is the cost of being wrong here, and when would we find out?*

## 3. Simplicity is a feature
Choose the simplest design that satisfies the **actual** requirements. Every component, dependency, abstraction, and configuration option is a liability that must earn its place. Boring technology is usually the right technology.

## 4. Make the right thing easy and the wrong thing hard
Encode standards in tooling (types, linters, CI gates, templates, generators) rather than in memory or prose. A rule that isn't enforced will decay.

## 5. Design for failure
Networks partition, disks fill, dependencies time out, messages duplicate, humans misconfigure, deploys go bad. Assume every external call can fail, be slow, or happen twice, and decide what should happen.

## 6. Secure and private by default
Least privilege, deny by default, validate at boundaries, minimize data collected and retained, never trust input. Security is not a stage — it applies from the first commit.

## 7. Observability is part of the feature
If you can't tell whether it's working in production, it isn't done. Every meaningful flow should be traceable, measurable, and alertable proportionate to its importance.

## 8. Explicit contracts at boundaries
Between modules, services, teams, and customers: typed, versioned, documented contracts. Inside a boundary: freedom. Changes to contracts are deliberate and backward-compatible by default.

## 9. Fast, safe feedback loops
Short paths from change → test → deploy → observe → rollback. Small batches. Automated verification. The speed of a team is bounded by the speed and trustworthiness of its feedback.

## 10. Consistency beats local optimality
One way of doing each thing across the codebase. A worse-but-consistent pattern is usually better than a mix of better ones. Change conventions deliberately and everywhere, or not at all.

## 11. Optimize for the reader
Code is read far more than written — by humans and agents. Clear names, cohesive modules, colocated related things, comments that explain *why*. Avoid cleverness.

## 12. Measure, then optimize
Performance, cost, and reliability work starts from data (profiles, traces, load tests, bills), not intuition.

## 13. Own the whole lifecycle
Build → deploy → operate → evolve → decommission. Data has a lifecycle; so do features, APIs, and dependencies. Plan for change, migration, and deletion.

## 14. Write it down
Assumptions, decisions, runbooks, and lessons. Undocumented knowledge is a single point of failure — and invisible to AI agents.
