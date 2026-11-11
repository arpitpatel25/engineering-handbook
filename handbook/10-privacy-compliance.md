# 10 — Privacy, Compliance & Governance

Privacy is about respecting people's data; compliance is about proving you do. Both are easiest when designed in: collect less, know where data goes, and make deletion and access real operations.

> This guide is engineering guidance, not legal advice. Confirm obligations with qualified counsel for regulated contexts.

**Applies when:** handling personal data, operating in regulated industries/regions, selling to enterprises, or processing data on behalf of customers.

## Key questions
- What personal or sensitive data do we collect, why, and is each field necessary?
- Where does it flow (services, logs, analytics, backups, vendors, AI providers)?
- Who can access it, and is that access logged?
- How long do we keep it, and can we really delete it?
- Which regulations and contractual obligations apply?

## 1. Data inventory & classification [MVP]
- Maintain a **data map**: data element → classification → purpose → storage location(s) → processors/vendors → retention → legal basis.
- Classes: public · internal · confidential · restricted (PII, sensitive personal data, health/PHI, payment/PCI, credentials, children's data, biometrics, government IDs).
- Classification drives encryption, access, logging, retention, and residency rules.

## 2. Privacy by design [ALL → MVP]
- **Data minimization**: don't collect what you don't need; don't keep it longer than needed.
- **Purpose limitation**: use data only for stated purposes.
- **Pseudonymization/anonymization** for analytics and lower environments.
- **No PII in logs, URLs, analytics events, error trackers, or LLM prompts** unless necessary and governed; redact/mask by default ([13](13-observability.md), [19](19-ai-llm-features.md)).
- Defaults favour privacy (opt-in for non-essential processing).
- Privacy review for features that add new data uses.

## 3. Data subject rights [MVP for consumer/EU users]
Engineering must support:
- **Access/export** (portable, machine-readable).
- **Rectification**.
- **Erasure** — across primary DB, replicas, object storage, search indexes, caches, analytics, data warehouse, logs (or bounded retention), backups (documented expiry), and **third-party processors**.
- **Restriction/objection** of processing; marketing opt-out.
- Identity verification for requests; request tracking and SLA (e.g. GDPR: one month).

## 4. Consent & transparency [MVP]
- Privacy policy and terms reflect actual data flows.
- Cookie/tracking consent where required (EU ePrivacy, etc.); no non-essential trackers before consent.
- Consent records (what, when, version).
- Honour Global Privacy Control / Do Not Track where required (e.g. some US states).

## 5. Regulatory landscape (check which apply)
| Regime | Scope (summary) | Engineering implications |
|---|---|---|
| **GDPR / UK GDPR** | Personal data of people in EU/UK | Lawful basis, DSRs, DPAs with processors, breach notice ≤72h, DPIAs, transfers (SCCs), records of processing. |
| **CCPA/CPRA & US state laws** | California & other states' residents | Notice, opt-out of sale/sharing, DSRs, sensitive data limits. |
| **India DPDP Act** | Digital personal data in India | Consent, notice, rights, breach reporting, children's data. |
| **LGPD (Brazil), PIPEDA (Canada), APPI (Japan), PDPA (Singapore), etc.** | Regional | Similar rights-based obligations. |
| **HIPAA** (US health) | PHI via covered entities/business associates | BAAs, access controls, audit logs, encryption, minimum necessary. |
| **PCI DSS** | Cardholder data | Avoid scope: use a payment processor's hosted fields/tokenization so card data never touches your servers. |
| **SOC 2** | Trust Services Criteria (security, availability, confidentiality, PI, privacy) | Evidence of controls: access reviews, change management, monitoring, vendor management, incident response. |
| **ISO 27001 / 27701** | ISMS / privacy management | Policies, risk assessment, controls (Annex A), audits. |
| **FedRAMP / StateRAMP / IL levels** | US government cloud | Strict controls, US-person/region constraints. |
| **COPPA / age-appropriate design codes** | Children | Parental consent, minimization. |
| **EU AI Act** | AI systems in EU | Risk classification, transparency, documentation ([19](19-ai-llm-features.md)). |
| **Accessibility laws** (ADA, EAA, Section 508) | Digital accessibility | WCAG conformance ([17](17-frontend-clients.md)). |
| **Sector-specific** (FINRA, SOX, GLBA, DORA, NIS2, FERPA…) | Finance, education, critical infra | Retention, audit, resilience, reporting. |

## 6. Vendors & sub-processors [PROD]
- Inventory of third parties that receive data (cloud, email, analytics, support, error tracking, LLM providers).
- DPAs / BAAs in place; sub-processor list published [ENT].
- Vendor security review for critical vendors.
- Data transfer mechanisms for cross-border transfers.
- Configure vendors for minimization (IP anonymization, PII scrubbing, zero-data-retention for AI APIs where available).

## 7. Data residency & sovereignty [ENT]
- Regional deployments or region-pinned storage per tenant.
- Ensure backups, logs, analytics, support tools, and AI providers also respect residency.
- Document which data may leave the region (e.g. telemetry) and why.

## 8. Retention & deletion [MVP]
- Retention schedule per data class (operational need vs legal minimums vs maximums).
- Automated purge jobs with monitoring.
- Legal hold capability [ENT].
- Deletion verifiable and logged (without retaining the deleted personal data in the log).

## 9. Governance & evidence [ENT]
- Policies: information security, access control, acceptable use, incident response, BC/DR, vendor management, data retention, change management, SDLC.
- Controls mapped to frameworks; evidence collected continuously (compliance automation tools: Vanta, Drata, Secureframe, etc.).
- Periodic access reviews; employee onboarding/offboarding checklists; security training.
- Records of processing activities; DPIAs for high-risk processing.
- Trust center / security page; answers ready for security questionnaires (SIG, CAIQ).

## 10. Breach readiness [PROD]
- Incident response plan with privacy branch: assess scope, contain, notify regulators/customers within mandated windows, document.
- Logs sufficient to determine what data was accessed and by whom.

## Red flags
- Collecting data "in case it's useful".
- PII in logs, analytics, or third-party error trackers by default.
- "Delete account" that only flips a flag forever.
- Production data copied to dev/staging.
- Sending customer data to AI providers without terms review and disclosure.
- Card numbers touching application servers.
- No list of where personal data lives.

## Decisions to record
Data classification · retention schedule · deletion approach · residency approach · applicable regimes and scope boundaries · vendor list.

## Evidence of done
- Data map exists and matches code.
- Export and deletion flows implemented and tested end-to-end (including processors).
- Log/telemetry redaction verified.
- Retention jobs running and monitored.
- DPAs in place for processors (production with personal data).

## Related
[03](03-data.md) · [08](08-security.md) · [13](13-observability.md) · [18](18-saas-enterprise.md) · [19](19-ai-llm-features.md)
