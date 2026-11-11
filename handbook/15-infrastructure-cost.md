# 15 — Infrastructure, Cloud & Cost

Infrastructure should be reproducible, secure, right-sized, and boring. Choose the least operationally demanding platform that meets the requirements, define it as code, and watch the bill like a metric.

**Applies when:** choosing hosting, provisioning resources, containerizing, scaling, managing cost, planning regions or DR.

## Key questions
- What's the simplest platform that meets our requirements and team capacity?
- Can we recreate the entire environment from code?
- What does it cost now, at 10×, and per unit (user/tenant/request)?
- What are we locked into, and what would leaving cost?
- Who has access, and how is that controlled?

## 1. Platform choice [ALL]

| Option | Fits | Trade-offs |
|---|---|---|
| **PaaS** (Render, Railway, Fly.io, Heroku, App Engine, Azure App Service, Vercel/Netlify for frontends) | Small teams, fast start, standard web apps. | Less control; cost at scale; platform limits. |
| **Serverless** (Lambda, Cloud Run, Cloud Functions, Azure Functions, Workers) | Spiky/low traffic, event-driven, minimal ops. | Cold starts, limits, local dev, vendor coupling, cost at sustained load. |
| **Managed containers** (Cloud Run, ECS/Fargate, Azure Container Apps) | Containerized services without cluster management. | Some limits; less ecosystem than k8s. |
| **Kubernetes** (EKS/GKE/AKS or self-managed) | Many services, platform team, portability, complex scheduling needs. | Significant operational and cognitive overhead. |
| **VMs / bare metal** | Specialized hardware, cost optimization at steady load, legacy, on-prem. | Patching, scaling, and HA are on you. |
| **On-prem / customer-hosted / air-gapped** | Enterprise or regulated customers requiring self-hosting. | Packaging (Helm, Docker Compose, installers), upgrades, support matrix ([18](18-saas-enterprise.md)). |

- Prefer **managed services** for databases, queues, caches, object storage unless there's a strong reason (cost at scale, portability, compliance).
- Kubernetes is a means, not a goal: adopt when the operational benefits exceed its cost.

## 2. Infrastructure as Code [MVP → PROD]
- Everything provisioned via code: Terraform/OpenTofu, Pulumi, CDK, Bicep, CloudFormation, Crossplane; app manifests via Helm/Kustomize.
- State stored remotely with locking and encryption; access restricted.
- Modules for repeated patterns; environments as parameterized instances.
- Plan/preview in CI on PR; apply via pipeline, not laptops [PROD].
- Drift detection [SCALE].
- Policy as code (OPA/Conftest, Sentinel, Checkov) for guardrails [SCALE/ENT].
- Tag/label every resource: owner, environment, service, cost center.

## 3. Containers [MVP]
- Minimal, pinned base images; multi-stage builds; non-root; no secrets in layers; `.dockerignore`.
- One process per container; logs to stdout/stderr; config via env.
- Health checks; resource requests/limits; graceful SIGTERM handling ([06](06-reliability.md)).
- Reproducible local stack via Docker Compose (or devcontainers).

## 4. Networking [PROD]
- VPC design: public subnets only for load balancers/edge; private subnets for compute and data.
- Private connectivity to managed services (private endpoints/PrivateLink).
- Egress control (NAT, egress proxies/allow-lists) where SSRF or data exfiltration risk matters.
- DNS managed as code; TLS certificates automated (ACME/managed certs).
- Load balancing (L7 for HTTP routing; L4 for TCP).
- CDN and WAF at the edge ([07](07-performance-scalability.md), [08](08-security.md)).
- Service mesh only when mTLS/traffic policy/observability needs justify it [SCALE].

## 5. Compute scaling [PROD]
- Autoscaling policies with min/max bounds; scale on meaningful signals (latency, concurrency, queue age).
- Right-size instances based on observed utilization.
- Separate scaling for web vs workers.
- Spot/preemptible capacity for interruption-tolerant workloads (batch, workers with checkpointing) [SCALE].

## 6. Access & identity for infra [PROD]
- SSO for cloud console/CLI; MFA; no shared accounts.
- Least privilege IAM; roles over users; short-lived credentials.
- Separate accounts/projects per environment; production access restricted and audited; break-glass procedure [ENT].
- Organization-level guardrails (SCPs/org policies) [ENT].

## 7. Cost management (FinOps) [MVP → SCALE]
- [MVP] Budgets and billing alerts from day one (including anomaly detection).
- [MVP] Know the top cost drivers; watch "surprise" costs: NAT gateway data, cross-AZ/region traffic, egress, logs/telemetry volume, idle resources, over-provisioned DBs, LLM/API usage, snapshots.
- [PROD] Tagging for cost allocation; per-environment cost visibility.
- [SCALE] **Unit economics**: cost per tenant/user/request/job; gross margin awareness for SaaS.
- [SCALE] Commitments (savings plans, reserved instances, committed use) for steady baseline; spot for flexible.
- [SCALE] Storage lifecycle policies (tiering, expiry); log retention limits.
- Shut down/scale-to-zero non-prod environments off-hours; TTL for preview environments.
- Cost reviews as part of architecture decisions (ADR consequences).

## 8. Multi-region & DR [ENT]
- Only when SLOs, latency, or residency require it ([06](06-reliability.md)).
- Options: backup-restore to another region (cheapest, slowest), pilot light, warm standby, active-active (most complex).
- Practise failover; document RTO/RPO achieved.

## 9. Sustainability [SCALE]
- Right-sizing and scale-to-zero reduce both cost and carbon; choose regions with lower carbon intensity where feasible.

## 10. Vendor lock-in [PROD]
- Accept lock-in consciously where the value is high (managed DBs, queues); avoid gratuitous lock-in for commodity features.
- Keep data exportable; use open standards (OCI images, OpenTelemetry, SQL, S3-compatible APIs) where cheap to do so.
- Abstract only where switching is realistic.

## Red flags
- Click-ops production infrastructure with no code.
- Kubernetes for a single small service and a two-person team.
- Shared admin credentials; long-lived cloud keys in CI.
- No budget alerts; nobody knows the monthly bill.
- Public database endpoints.
- Preview/staging environments running forever at production size.
- Unbounded log retention.

## Decisions to record
Hosting platform · IaC tool · network topology · scaling policy · region strategy · cost targets · managed vs self-hosted per component.

## Evidence of done
- Environment can be recreated from code (verified at least once).
- Budgets/alerts configured.
- Resources tagged with owner/env/service.
- No public data stores; IAM least privilege reviewed.

## Related
[06](06-reliability.md) · [08](08-security.md) · [14](14-delivery-cicd.md) · [16](16-operations-incidents.md)
