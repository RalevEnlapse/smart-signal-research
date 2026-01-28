# Automated compliance checks — Deep research (implementation-grade for municipal hybrid environments)

## Executive summary
Automated compliance checks turn compliance from periodic audits into an **operational control loop**: policies are encoded, continuously evaluated across environments (cloud + on‑prem + vendors), and produce **tamper-evident evidence** that links to deployments, data products, ADRs, and risk acceptances.

For a city digital twin program, this is essential because:
- data is often location-linked and politically sensitive,
- systems span multiple agencies and vendors,
- public dashboards introduce accessibility obligations,
- drift is constant across infrastructure, data pipelines, and analytics/modeling.

Defaults assumed (overrideable):
- deployment_context: hybrid (cloud + on-prem) with vendors/partners
- initial_scope_level: minimum-viable-controls
- enforcement_posture: guardrails-first
- evidence_immutability_level: strong
- policy_stack_preference: portable
- public_dashboard_in_scope: yes
- privacy_regime: generic

---

## High-signal external sources (used to anchor concrete choices)
- **NIST SP 800-53 Rev. 5 — Security and Privacy Controls for Information Systems and Organizations (NIST)**
  - URL: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
  - Takeaway: provides a widely used control catalog (security + privacy) and emphasizes auditability/assurance; useful as a municipal control vocabulary and for mapping to a control catalog.

- **OSCAL version of NIST 800-53 controls (NIST GitHub)**
  - URL: https://github.com/usnistgov/oscal-content/tree/v1.4.0/src/nist.gov/SP800-53
  - Takeaway: controls are available in a machine-readable format; useful for building a control catalog and evidence mappings without vendor lock-in.

- **OPA for Kubernetes Admission Control (Open Policy Agent)**
  - URL: https://openpolicyagent.org/docs/kubernetes
  - Takeaway: shows how to enforce policies at admission time (create/update/delete) — a reference pattern for “guardrails-first” enforcement in platform layers.

- **SLSA Provenance specification (slsa.dev)**
  - URL: https://slsa.dev/spec/v0.1/provenance
  - Takeaway: defines a standard provenance model describing where/when/how an artifact was built; useful for linking compliance evidence to build/deploy lineage.

- **WCAG 2.2 (W3C)**
  - URL: https://www.w3.org/TR/WCAG22/
  - Takeaway: normative accessibility requirements for web content; automated checks can cover a subset, with mandatory manual verification for others.

---

## 1) Scope: compliance domains for a city digital twin (v0)
This section defines what you can realistically automate first.

### 1.1 In-scope domains (minimum viable controls)
1) **Privacy & data protection**
- purpose limitation, minimization, retention, access tiers
- geolocation precision controls
- public release of aggregates (re-identification risk checks)

2) **Security baseline**
- IAM hygiene, encryption at rest/in transit, logging/auditability
- vulnerability management signals and patch posture

3) **Critical infrastructure data handling tiers**
- tiering of datasets/systems; stricter controls for higher tiers

4) **Accessibility (public dashboards/services)**
- automated scanning plus manual verification gates

### 1.2 Out of scope (until governance is stable)
- “Full compliance automation” across every vendor product with bespoke rules.
- Automatic changes to OT/SCADA settings (twin remains advisory for OT).
- Fully automated legal determinations (policy-as-code supports decisions; does not replace legal review).

---

## 2) Initial control catalog (v0) — pragmatic and enforceable
Use a small, named control set with stable IDs, mapped to your policies and evidence.

### 2.1 Control catalog table (v0)
Legend: enforcement levels = `detect-only` | `guardrails-first` | `enforce-blocking`.

| Control ID | Domain | Intent | Automatically testable | Manual / human gate | Default enforcement |
|---|---|---|---|---|---|
| PRIV-01 | Privacy | Data minimization (collect only what’s needed) | Schema checks: prohibited fields; payload size; sampling | DPIA/PIA review for new collection | guardrails-first |
| PRIV-02 | Privacy | Purpose limitation (allowed uses/joins/exports) | Dataset policy tags + export gate rules | Data governance approval for new joins | guardrails-first |
| PRIV-03 | Privacy | Retention & deletion | TTL policy present; lifecycle rules; deletion job SLI | Legal hold approvals | enforce-blocking (for high tiers) |
| PRIV-04 | Privacy | Geolocation precision control | enforce rounding/tiling policy; deny precise export | approval for exceptions | guardrails-first |
| SEC-01 | Security | IAM baseline | no public admin; MFA required; least privilege lint | periodic access recertification | enforce-blocking (critical paths) |
| SEC-02 | Security | Encryption baseline | encryption at rest + in transit; key policy checks | crypto exceptions approval | enforce-blocking |
| SEC-03 | Security | Logging/auditability | audit logs enabled; log sink configured; retention | sampling review of log completeness | guardrails-first |
| SEC-04 | Security | Vulnerability management | SBOM presence; image scanning results | risk acceptance for deferrals | guardrails-first |
| CI-01 | Critical infra tiers | Tier classification applied | presence of `ci_tier` tag; routing to correct env | tier assignment approval | guardrails-first |
| CI-02 | Critical infra tiers | Restricted access for higher tiers | access policy checks; network segmentation checks | periodic tabletop exercises | enforce-blocking |
| A11Y-01 | Accessibility | WCAG automated coverage | automated scans (axe/pa11y/lighthouse); regression diffs | manual keyboard/screen-reader tests | guardrails-first |
| A11Y-02 | Accessibility | Accessible release process | build gate requires A11Y report attached | sign-off for known gaps | guardrails-first |

Practical rule: keep “v0” small, but make each control have:
- a testable portion,
- a human gate portion,
- a clear evidence artifact.

---

## 3) Evidence architecture (canonical model + auditability mechanics)
The goal is to make evidence:
- **reproducible** (same inputs → same evaluation),
- **tamper-evident**,
- **traceable** to decisions (ADRs) and accepted risks.

### 3.1 Canonical evidence schema (minimum fields)
Canonical entity: `ComplianceEvidenceRecord`.

Required fields (minimum):
- `evidence_id` (uuid)
- `control_id` (e.g., `PRIV-03`)
- `check_id` (e.g., `retention.ttl.present`)
- `check_version` (semver)
- `policy_repo_ref` (git commit/sha)
- `time_window_start`, `time_window_end`
- `environment` (dev/test/prod + on-prem/vendor)
- `system_component` (service, pipeline, cluster, dashboard)
- `dataset_ids[]` (if applicable)
- `model_ids[]` (if applicable)
- `build_id` / `deploy_id` (pipeline run + artifact digest)
- `result` (pass/fail/error/not_applicable)
- `severity` (low/med/high/critical)
- `finding_summary`
- `raw_artifacts[]` (URIs to configs/log extracts/reports)
- `hashes[]` (hash per artifact)
- `signer` (service identity)
- `signed_at` (timestamp)
- `links`:
  - `adr_ids[]`
  - `risk_acceptance_ids[]`
  - `exception_ids[]`

### 3.2 Provenance and linkage rules
- Every evidence record must link to **the exact policy version** and **the exact target snapshot**.
- Every “fail” must link to a **ticket/work item** with an owner and SLA.
- Every exception must link to:
  - the evidence it overrides,
  - compensating controls evidence,
  - expiry.

### 3.3 Evidence storage and immutability options
Evidence immutability level: **strong** (default).

| Level | Storage pattern | Tamper resistance | When to use |
|---|---|---|---|
| basic | append-only DB + backups | moderate | low-risk internal controls |
| strong | WORM/object-lock storage + signed records + immutable log | high | audit-ready municipal programs |

Recommended strong pattern:
- Write raw artifacts to **WORM-capable object storage** (object lock).
- Store a hash manifest and sign evidence records.
- Maintain an append-only “evidence ledger” (tamper-evident log) for evidence metadata.

### 3.4 Retention tiers + legal hold
- Define evidence retention tiers:
  - short (operational triage)
  - medium (audit cycles)
  - long (critical infrastructure / investigations)
- Implement legal hold:
  - freeze evidence deletion,
  - capture who initiated hold and justification,
  - export pack procedure.

---

## 4) Exception governance (make it hard to game)
Exceptions are inevitable; unmanaged exceptions are compliance debt.

### 4.1 Exception lifecycle
1) request (with business justification)
2) review (security/privacy/data governance)
3) approval (threshold-based)
4) time-boxing (expiry)
5) compensating controls (required)
6) renewal or closure (mandatory)

### 4.2 Required exception metadata
- `exception_id`
- `control_id` / `check_id`
- `scope` (systems/datasets/models)
- `justification`
- `risk_rating`
- `compensating_controls[]` (with evidence links)
- `expiry_date`
- `renewal_limit` (e.g., max 2 renewals)
- `approvers[]` with roles
- `linked_evidence_ids[]`

### 4.3 Approver thresholds (including exec sign-off)
- High/critical severity exceptions, or any affecting public release / critical infra tiers:
  - require security + privacy + accountable exec delegate.
- Exceptions longer than 90 days (or >2 renewals):
  - require executive sign-off and a remediation plan.

### 4.4 Exception debt metrics + escalation automation
Metrics:
- exception count by control/severity
- total “exception-days” outstanding
- renewals per exception
- % exceptions with missing compensating controls evidence

Automation:
- reminders 14/7/1 days before expiry
- automatic escalation on expiry (page/ticket to owning director)
- monthly exception review board agenda auto-generated

### 4.5 Emergency exceptions
- allow emergency exceptions for incident response with short expiry (e.g., 7–14 days)
- mandatory post-hoc review within next governance cadence
- require retrospective and decision log entry (ADR or incident decision record)

---

## 5) Policy-as-code: choices + portability (hybrid + vendor reality)
Policy-as-code must run in multiple places and remain portable across cloud/on‑prem.

### 5.1 Portable reference design (recommended)
Where policies run:
- **CI**: validate IaC, container images, data pipeline definitions, dashboard builds.
- **Deploy gates / admission**: enforce platform rules (e.g., Kubernetes admission control patterns).
- **Runtime**: continuous drift detection and periodic re-evaluation.

How to version and test policies:
- store policies in git with semantic versioning
- unit tests for policies + golden test cases
- policy change PRs require review from policy owners

Mapping across environments:
- define an environment abstraction layer (labels: `cloud_provider`, `on_prem`, `vendor`)
- keep policy intent stable; implement environment-specific adapters (collectors) for facts.

### 5.2 Technology choices (portable)
Design principles:
- choose a policy language/runtime that is:
  - embeddable (CI/runtime),
  - testable,
  - not tied to a single cloud vendor.

Recommended approach (portable):
- use a general policy engine for decisions + separate collectors for system facts.
- keep evidence schema independent from the policy engine.

---

## 6) Remediation realism (public-sector change control)
Automated findings are useless without a realistic remediation path.

### 6.1 Risk-based remediation taxonomy
| Category | Description | Examples | Default workflow |
|---|---|---|---|
| Safe auto-remediation | low-risk, reversible, well-tested | add missing tag, tighten S3 bucket public flag, rotate expiring certs | auto PR + auto-apply with rollback |
| Ticket-only remediation | needs human judgment | retention policy conflicts, join policy changes | ticket + owner SLA |
| Change-board required | impacts production stability or contracts | encryption/key rotation for legacy systems, network segmentation | CAB approval + maintenance window |
| Deferred with compensating controls | cannot remediate quickly | vendor limitation | time-boxed exception + compensating controls |

### 6.2 Runbook essentials (for each category)
- pre-checks and blast radius
- rollback plan
- maintenance window constraints
- verification step produces new evidence record

---

## 7) Data-layer compliance (analytics + modeling)
City twins are analytics-heavy; compliance must apply to datasets and models.

### 7.1 Dataset policies
- classification tags (public/internal/restricted + critical infra tier)
- allowed joins/exports (purpose limitation)
- retention automation (TTL + deletion jobs)
- export controls (public release gates + aggregation thresholds)

### 7.2 Model governance hooks
Evidence must include:
- training dataset lineage (dataset IDs + versions)
- reproducibility artifacts (code commit, parameters)
- bias checks and evaluation summaries
- release gates (who approved; what changed)

### 7.3 Controls for public release of aggregates
- re-identification risk checks (minimum group size, suppression rules)
- geolocation precision checks (rounding/tiling)
- disclosure review for sensitive infrastructure layers

---

## 8) Operations: SLIs/SLOs + ownership (RACI)

### 8.1 SLIs/SLOs (examples)
- check coverage: % of in-scope systems evaluated per cadence
- evidence freshness: max age of evidence for critical controls
- policy drift: policy version lag across environments
- MTTR by severity for violations
- exception hygiene: % expired exceptions; % missing compensating controls

### 8.2 RACI (minimal)
| Area | Responsible | Accountable |
|---|---|---|
| Policy definitions | security/privacy/data governance | compliance steering lead |
| Check implementations | platform engineering | platform lead |
| Evidence store | platform + security | CISO delegate |
| Exceptions | risk owners | exec delegate |
| Remediation execution | service owners / vendors | delivery leads |

---

## 9) Implementation roadmap

### 0–3 months (minimum viable compliance)
- define control catalog v0 and IDs
- stand up evidence schema + storage (strong immutability baseline)
- implement CI checks for 5–10 high-impact controls (IAM/logging/retention/accessibility)
- implement exception workflow with expiry automation

### 3–12 months (guardrails-first maturity)
- deploy gates/admission controls for key platform rules
- integrate evidence to deployments and ADR/risk acceptance links
- add runtime drift detection for production
- publish internal compliance dashboard (and public dashboard summary if in scope)

### 12–24 months (federation across agencies/vendors)
- expand control coverage and environment adapters
- cross-agency evidence federation and standardized audit packs
- mature data/model compliance gates (lineage, reproducibility, bias)

---

## 10) Risks & mitigations (with detection + response playbooks)

| Abuse case | Detection signals | Response playbook |
|---|---|---|
| Suppressed checks | missing evidence intervals; pipeline step skipped | block merges; incident ticket; rotate credentials; governance escalation |
| Exception abuse | repeated renewals; high exception-days | enforce renewal limits; require exec sign-off; publish exception debt reports |
| Evidence tampering | hash mismatch; missing ledger entries | isolate evidence store; forensics; legal hold; re-run checks |
| Vendor non-compliance | missing evidence from vendor systems | contract enforcement; require attestations; restrict integration |

---

## References
### Workspace source
- [`kali-task-research.md`](kali-task-research.md:1) — Item 19: Automated compliance checks

### External references (retrieved via Firecrawl MCP)
- NIST SP 800-53 Rev. 5: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- OSCAL content (NIST): https://github.com/usnistgov/oscal-content/tree/v1.4.0/src/nist.gov/SP800-53
- OPA for Kubernetes: https://openpolicyagent.org/docs/kubernetes
- SLSA provenance spec: https://slsa.dev/spec/v0.1/provenance
- WCAG 2.2 (W3C): https://www.w3.org/TR/WCAG22/
