# Non-Functional Requirements — <System / Feature>

Write numbers. If something genuinely doesn't matter at this stage, write "Not a concern at <stage> because …".

| Attribute | Target | Measured how | Notes / assumptions |
|---|---|---|---|
| Latency (interactive API) | p95 < ___ ms, p99 < ___ ms | | |
| Latency (page load / LCP) | p75 < ___ s | | |
| Background job completion | ___ % within ___ | | |
| Throughput | avg ___ rps, peak ___ rps | | |
| Availability | ___ % per ___ (journey: ___) | | |
| Durability / RPO | max ___ data loss | | |
| RTO | restore within ___ | | |
| Consistency | strong for ___; eventual (≤ ___ s) for ___ | | |
| Data volume | ___ now, ___ in 12 months | | |
| Users / tenants | ___ now, ___ in 12 months | | |
| Security | data classes: ___; auth strength: ___ | | |
| Privacy / compliance | regimes: ___; residency: ___ | | |
| Accessibility | WCAG ___ level ___ | | |
| Localization | locales: ___ | | |
| Supported platforms | browsers/OS/devices: ___ | | |
| Cost | ≤ $___/month; ≤ $___ per ___ | | |
| Maintainability | owner: ___; lifespan: ___ | | |
| Observability | must be able to answer: ___ | | |

## Back-of-envelope
- DAU × actions/day ÷ 86,400 × peak factor = ___ rps
- Records/day × size × retention = ___ storage
- Fan-out per user action: ___
