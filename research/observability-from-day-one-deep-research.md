# Observability from day one — Deep research

## Executive summary
A City Digital Twin is only useful if leaders and operators can answer one question quickly and defensibly:

> **Is the twin safe to use right now for this decision?**

“Observability from day one” for a city twin therefore goes beyond “services are up.” It must make **data freshness, coverage, trustworthiness, governance compliance, and model validity** visible across a distributed, multi-tenant, multi-vendor system.

This document defines a digital-twin-specific observability program that is:
- **Standards-first** (OpenTelemetry + W3C context propagation; OpenMetrics/Prometheus conventions where applicable).
- **Governed** (telemetry never becomes a backdoor for PII, sensitive locations, or surveillance).
- **Federated** (clear ownership boundaries and escalation that work across agencies and vendors).
- **SLO-driven** (alerting based on error budgets and burn rates, not ad-hoc thresholds).
- **FinOps-aware** (sampling, retention tiers, cardinality control, and cost showback).

Core deliverables:
1. **Twin Observability Contract**: required fields/tags across logs/metrics/traces (dataset IDs, schema versions, run IDs, model versions, tenant/agency, classification tier, correlation IDs).
2. **Twin SLI/SLO catalog**: standard indicators for pipelines, state synchronization, model services, APIs/dashboards, and governance/audit systems.
3. **Ownership + escalation model**: who is on call for source feeds vs pipelines vs platform vs products; who can silence alerts; how audits are logged.
4. **Telemetry governance**: “never log” rules, sanitization enforcement, access tiering (ops/dev/vendor), tiered retention + legal hold.
5. **Testing + rehearsals**: CI observability tests, synthetic transactions/replay, and chaos drills integrated with incident response rehearsals.

This document deepens item 20 in [`kali-task-research.md`](kali-task-research.md:1): *"Observability from day one: Build observability into the digital twin platform from the outset to ensure reliable operations."*

---

## 1) Observability goals for a city digital twin

### 1.1 What you are trying to know
1. **Twin usability**: the twin is fresh enough, complete enough, and not misleading.
2. **Twin integrity**: state and pipelines are not silently corrupt or drifting.
3. **Governance posture**: telemetry and audit logs comply with privacy, access, and retention commitments.
4. **Operational readiness**: cross-agency operations can detect, triage, and resolve incidents.
5. **Cost sustainability**: telemetry volume and query patterns are bounded.

### 1.2 “Safe to use” signals (what dashboards must show)
Every major twin view (internal and public-facing) must surface:
- **Last updated** timestamp per dataset layer (and the dataset criticality tier).
- **Freshness SLO state** (OK / at-risk / violated) with a simple explanation.
- **Coverage / completeness** (what portion of expected sensors/feeds are contributing).
- **Degraded-mode flags** (e.g., “model fallback”, “missing district feeds”, “state-store resync”).
- **Confidence / uncertainty** where models are used.

---

## 2) Twin-specific observability contract (explicit and enforced)

### 2.1 Contract scope (what must conform)
The contract is a hard requirement for:
- all ingestion/pipeline jobs that publish datasets into the twin
- all state-store writers/readers
- all model services used in the twin
- all serving APIs and dashboards
- all governance/audit subsystems

### 2.2 Required identifiers and tags across signals
**All signals** (logs/metrics/traces) must include (where applicable):
- `correlation_id` (request ID or job/run correlation)
- `tenant_id` (agency/department tenant)
- `vendor_id` (if a vendor-operated subsystem)
- `dataset_id` (canonical dataset identifier)
- `dataset_tier` (criticality tier)
- `schema_id` and/or `schema_version`
- `run_id` (pipeline run / batch ID)
- `state_store_id` (if applicable)
- `model_id`, `model_version`, `model_validation_tier` (if applicable)
- `data_classification` (e.g., public / internal / sensitive / restricted)

**Traces** additionally must include:
- `trace_id`, `span_id`

**OpenTelemetry resource attributes** (minimum):
- `service.name`, `service.version`, `deployment.environment`

### 2.3 Canonical event types (minimum)
Standardize event families so operators can work across agencies/vendors:
- `data.ingest`
- `data.validate`
- `data.quarantine`
- `data.publish`
- `state.sync`
- `model.infer`
- `model.drift_check`
- `dashboard.render`
- `access.audit`

### 2.4 Context propagation rules
- Use W3C Trace Context (`traceparent`) for correlation across service boundaries.
- Use W3C Baggage sparingly for **non-sensitive** routing hints only.

Hard rules:
- **Never put PII or precise locations into baggage.**
- Prefer deriving routing/ownership from `dataset_id`/`tenant_id` rather than carrying user identifiers.

### 2.5 Enforcement posture (guardrails-first)
- **Standard libraries/SDKs** enforce required fields and redaction.
- **CI blocks merges** that violate contract and “never log” policies.
- **Collector-level enforcement** can:
  - drop attributes not in allowlists
  - hash/pseudonymize allowed identifiers where necessary
  - drop high-risk payloads (e.g., bodies)

---

## 3) Standard SLI/SLO catalog for city twins

### 3.1 Dataset criticality tiers (example)
Define tiers that drive alerting urgency, retention, and staffing:
- **Tier 0 (safety/critical operations)**: used for time-sensitive operational decisions.
- **Tier 1 (operational)**: affects day-to-day operations; tolerates short outages.
- **Tier 2 (planning/public health)**: analytics and planning; tolerates longer delays.
- **Tier 3 (informational)**: nice-to-have.

### 3.2 SLI/SLO catalog table
Starter catalog; tune per domain and dataset:

| Group | SLI | Definition / calculation | Why it matters in a twin | Example SLO (Tier 0 / Tier 1 / Tier 2) |
|---|---|---|---|---|
| Ingestion/pipelines | Freshness by dataset | `now - last_event_time(dataset_id)` | “Is the layer current?” | 99% < 5m / < 30m / < 24h |
| Ingestion/pipelines | Coverage by dataset | `% expected sources contributing` | Avoid silent partial blindness | ≥ 98% / ≥ 95% / ≥ 90% |
| Ingestion/pipelines | Schema error rate | invalid records / total | Detect breaking changes early | < 0.1% / < 0.5% / < 1% |
| Ingestion/pipelines | Missingness rate | missing required fields / total | Data quality SLO, not infra | dataset-specific error budget |
| Ingestion/pipelines | Outlier rate | outlier flags / total | Detect sensor drift/tampering | dataset-specific guardrails |
| Ingestion/pipelines | Quarantine rate | quarantined / total | Early warning + abuse detection | bounded; spike triggers incident |
| Ingestion/pipelines | Reprocessing backlog age | age of oldest pending reprocess | Detect growing debt | < target window |
| State store/sync | Sync lag | producer timestamp → state-store commit | “How behind is the state?” | p95 < 10s / < 60s / N/A |
| State store/sync | Virtual vs physical divergence | `|twin_state - observed_state|` where defined | Detect stale/incorrect “world state” | divergence p95 < threshold |
| State store/sync | Conflict rate | conflicting updates / total | Detect integration breakage | < threshold |
| Model services | Drift indicators | PSI/KL or domain metric | Prevent silent accuracy loss | investigated within X hours |
| Model services | Calibration health | calibration error/curve | Prevent overconfident outputs | within bounds |
| Model services | Fairness/impact proxy | outcome parity/impact ratios where relevant | Avoid uneven harm | monitored + guardrail triggers |
| Model services | Version distribution | % traffic by model_version | Know what’s actually running | 99% on approved versions |
| Serving/APIs | Availability | success responses / total | User-facing reliability | 99.9% |
| Serving/APIs | Latency | p95/p99 response time | Usability and trust | p95 < target |
| Serving/APIs | “Truthfulness indicators present” | % views with last-updated + degraded flags | Prevent misleading dashboards | 100% |
| Public dashboards | Staleness disclosure | disclosure shown when stale | Trust and compliance | 100% when SLO violated |
| Governance/audit | Audit log completeness | % privileged actions logged | Compliance + investigations | 100% |
| Governance/audit | Break-glass usage | count and rate | Detect misuse | reviewed within 24h |
| Governance/audit | Alert silencing audit | % silences with ticket + expiry | Prevent hiding incidents | 100% |

**Integration note:** dataset-level SLIs should align to data quality definitions and quarantine policy in [`research/data-quality-slas-deep-research.md`](research/data-quality-slas-deep-research.md:1).

---

## 4) Telemetry governance (privacy + access + retention)

### 4.1 “Never log / never emit” rules
Telemetry must never include:
- Direct identifiers: names, phone/email, national IDs, patient identifiers.
- Credentials/secrets: tokens, API keys, session IDs, private keys.
- Raw payloads from sensitive systems (case files, health records, policing notes).
- **Precise residential location** or person-level location traces.
- Raw images/audio/video frames unless explicitly approved for a bounded debugging window.
- Full request/response bodies by default.

### 4.2 Enforcement mechanisms (what actually works)
**At the code boundary** (preferred):
- Standard logging wrapper that only accepts structured objects.
- Field allowlists for each log event type.
- Secret scrubbing (regex + known key patterns).

**In CI**:
- “Never log” linters (blocklist common anti-patterns: `console.log(req.body)`, payload dumps).
- Schema validation for log events.
- Unit tests for redaction.

**At the collector**:
- Attribute allowlists.
- Drop/transform processors.
- Rate limits and backpressure to avoid cascading failures.

### 4.3 Access tiering (ops vs dev vs vendor)
Observability data is production data. Apply least privilege:

- **Ops (SRE/on-call)**: access to operational telemetry needed to restore service.
- **Developers / data engineers**: access scoped to owned services; time-bounded elevated access.
- **Vendors**: ticket-based, expiring access; no blanket access across tenants.

RBAC dimensions to enforce:
- `tenant_id`
- `service.name`
- `dataset_id`
- `data_classification`

Break-glass access requires:
- incident ID
- explicit approval per policy
- immutable audit trail of queries and exports

### 4.4 Retention tiers + legal hold
Tiered retention controls cost and supports investigations:

- **Hot (7–14 days)**: high-cardinality traces/logs for active debugging.
- **Warm (30–90 days)**: reduced-cardinality logs + key metrics.
- **Cold (6–24 months)**: compliance/audit logs + aggregated metrics.

Legal hold:
- immutable retention override tied to case/incident number
- explicit export controls and audit logging

---

## 5) Alert design, ownership, and escalation (federated)

### 5.1 SLO-driven paging strategy
- Page on **error budget burn** (multiwindow burn-rate) for customer-impacting SLOs.
- Use symptom alerts (fast detection) and cause signals (diagnosis).
- Every page must have:
  - a clear owner
  - a runbook
  - a safe-mode/rollback

### 5.2 On-call boundaries (data source vs pipeline vs platform vs product)
- **Data source owners**: sensor networks, partner feeds, vendor APIs.
- **Pipeline owners**: ingest/validate/quarantine/publish jobs.
- **Platform SRE**: shared infra (collectors, storage, query, IAM, networking).
- **Product/domain ops**: dashboards and operational workflows.

### 5.3 Escalation paths (minimum)
- Freshness/coverage SLO breach → pipeline on-call → source owner if upstream.
- Schema errors/quarantine spike → pipeline on-call + data steward → source owner.
- Observability platform outage → platform SRE → vendor (if managed service).
- Suspected PII leakage → security/privacy on-call + platform SRE (immediate).

### 5.4 Alert fatigue controls
- Burn-rate alerts instead of hard thresholds for SLOs.
- Paging policies: severity gating, aggregation/grouping, maintenance windows.
- Weekly review: top noisy alerts, precision/recall proxies, paging load.

### 5.5 “Who can silence alerts” + audit logging
Only:
- on-call leads
- incident commander

All silences require:
- reason
- duration
- ticket/incident reference

Silences must be written to immutable audit logs.

---

## 6) Cost governance for observability (FinOps integration)

### 6.1 Sampling strategies (practical defaults)
- **Traces**: tail-based sampling (keep errors; sample successful requests).
- **Logs**: default INFO/WARN; restrict DEBUG to bounded time windows.
- **Metrics**: prefer pre-aggregated metrics for high-volume questions.

### 6.2 Retention tiers by signal class and dataset tier
- Tier 0: longer retention for key metrics + audit logs; traces remain short.
- Tier 2/3: shorter retention and heavier sampling.

### 6.3 Query guardrails and cardinality controls
- Ban unbounded labels (user IDs, raw addresses, full URLs with query params).
- Collector allowlists for attributes.
- Per-metric cardinality limits and dashboards for top offenders.

### 6.4 Showback/chargeback
Attribute telemetry cost by:
- `service.name`
- `tenant_id`
- `vendor_id`

Publish monthly reports:
- telemetry ingest/storage/query cost per team/service
- top cardinality offenders

### 6.5 Metrics vs logs vs traces (when to use what)
- “Are we meeting SLO?” → metrics.
- “What failed?” → traces + structured logs.
- “Who did what?” → audit logs.

---

## 7) Testing observability (CI + synthetic + chaos)

### 7.1 Observability tests in CI/CD
Minimum tests:
- Required span/log fields exist (contract conformance).
- Correlation works across boundaries (trace + log correlation).
- Forbidden fields never appear (PII/secrets).

### 7.2 Synthetic transactions and replay tests
- Synthetic end-to-end flows: ingest → validate → publish → dashboard render.
- Replay-based tests for pipelines using fixed snapshots.

### 7.3 Chaos experiments (validate signal quality under failure)
Run drills that confirm operators still have diagnosable signals:
- drop a source feed
- corrupt schema
- slow a pipeline stage
- break context propagation across a vendor boundary

Integrate with [`research/incident-response-rehearsals-deep-research.md`](research/incident-response-rehearsals-deep-research.md:1) and require AAR evidence.

---

## 8) Operational runbooks (minimum set)

### 8.1 Dataset freshness breach
- Triage: confirm scope (`dataset_id`, `tenant_id`), check ingestion lag and upstream status.
- Mitigation: mark dashboards degraded; show “last updated”; switch to fallback.
- Escalation: source owner if upstream; product ops for user communications.

### 8.2 Suspected PII leakage into logs
- Contain: restrict access, stop exports, rotate credentials if needed.
- Identify: search patterns, find emitting `service.name`/`service.version`.
- Remediate: patch + deploy; purge within retention policy; document.
- Notify: privacy/legal per policy.

### 8.3 Cardinality/cost explosion
- Contain: drop offending attributes at collector; tighten sampling.
- Identify: top label keys/services.
- Prevent: add CI checks; improve dashboards; update standards.

### 8.4 Drift in key model outputs
- Validate: confirm data shift vs model bug.
- Mitigate: fallback model; disable feature; increase HITL sampling.
- Escalate: model owner + domain ops.

### 8.5 Vendor-operated subsystem telemetry outage
- Identify: vendor boundary vs platform.
- Mitigate: mark degraded; use vendor status feed.
- Escalate: vendor on-call via contract; require post-incident report.

---

## 9) Key metrics (program-level)
- SLO compliance by domain and dataset tier.
- Alert quality: page volume per on-call, noisy alert rate, actionability rate.
- Time-to-detect and time-to-diagnose.
- Telemetry cost per service/team; top cardinality offenders.
- Governance: break-glass frequency; audit log completeness.

---

## 10) Implementation roadmap

### 0–3 months: contract + minimal SLIs + safe logging + on-call routing
- Publish the **Twin Observability Contract** and ship shared libraries.
- Define dataset tiers and minimal SLIs (freshness, coverage, schema errors).
- Establish safe logging rules (“never log”) and basic RBAC.
- Implement on-call routing boundaries and escalation contacts.

### 3–12 months: SLO-based paging + cost controls + CI tests + chaos drills
- Add error budgets and burn-rate paging.
- Implement retention tiers + sampling policies.
- Add CI observability tests and synthetic transactions.
- Introduce cardinality dashboards + showback.

### 12–24 months: federation maturity + cross-domain dashboards + audit readiness
- Cross-domain “safe to use” dashboards.
- Continuous audit readiness (legal hold workflows, immutable logs).
- Regular chaos drills and rehearsal integration.

---

## References (high-signal sources; use as starting points)

### OpenTelemetry and modern observability conventions
- OpenTelemetry context propagation: [`opentelemetry.io/docs/concepts/context-propagation/`](https://opentelemetry.io/docs/concepts/context-propagation/) — Explains correlating traces/metrics/logs via propagated context across services.
- OpenTelemetry specification — environment variable propagation carriers: [`opentelemetry.io/docs/specs/otel/context/env-carriers/`](https://opentelemetry.io/docs/specs/otel/context/env-carriers/) — Recommends consistent context propagation using W3C Trace Context and Baggage across carriers.
- W3C Baggage specification: [`www.w3.org/TR/baggage/`](https://www.w3.org/TR/baggage/) — Defines a standard format for propagating request context properties (must be used carefully to avoid leaking sensitive data).

### SLO-based alerting / burn-rate alerting
- Google SRE Workbook — alerting on SLOs: [`sre.google/workbook/alerting-on-slos/`](https://sre.google/workbook/alerting-on-slos/) — Practical guidance for turning SLOs into alerts using burn-rate and multiwindow patterns.

### Privacy-safe logging guidance and audit log integrity norms
- OWASP Logging Cheat Sheet: [`cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html`](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) — Secure logging guidance with emphasis on avoiding sensitive data exposure.

### Cost control practices for high-cardinality telemetry
- OpenTelemetry metric best practices (cardinality): [`opentelemetry.io/docs/languages/dotnet/metrics/best-practices/`](https://opentelemetry.io/docs/languages/dotnet/metrics/best-practices/) — Notes cardinality limits and suggests using configuration (Views) to manage attribute explosion.
