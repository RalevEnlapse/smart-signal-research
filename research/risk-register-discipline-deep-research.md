# Risk register discipline (city digital twin) — Deep research

## Executive summary
A city digital twin program combines cross-agency data sharing, real-time sensing, analytics/ML, and operational decision support. This creates a risk surface that is broader than a typical IT system: privacy and civil liberties risks (including linkability/inference), model and automation harms, multi-vendor dependency and lock-in, and cross-agency governance failure modes.

A risk register is not “a spreadsheet”; it is the auditable operating mechanism that ensures risks are (1) continuously identified, (2) scored consistently against a municipal risk appetite, (3) mitigated with verified evidence (not paper compliance), and (4) integrated with governance decisions, stage-gates, observability, and incident rehearsal outcomes.

This document defines a twin-specific risk taxonomy/library, a conservative semi-quantitative scoring model with escalation thresholds suitable for multi-agency programs, evidence requirements for risk treatment, and feedback loops from SLOs/leading indicators and incident response into the risk register.

This document deepens item 15 in [`kali-task-research.md`](../kali-task-research.md:1): “Risk register discipline: Maintain a living risk log with likelihood/impact scoring, mitigations, and clear escalation paths.”

---

## 1. Scope and operating principles

### 1.1 Scope
The risk register covers:
- **Program risks**: funding, political continuity, procurement, cross-agency alignment.
- **Product/system risks**: platform reliability, data quality, security, privacy, model performance.
- **Operational risks**: on-call readiness, change management, incident response, vendor operations.
- **Legal/ethical risks**: civil liberties, discrimination, surveillance creep, due process/recourse.

Out of scope: individual project task trackers (those feed into mitigations and are linked).

### 1.2 Principles (auditable, conservative posture)
- **Conservative by default**: when uncertainty is high, score likelihood/impact upward and require stronger evidence.
- **Evidence-gated closure**: a risk is “treated/closed” only with verifiable artifacts.
- **Decision traceability**: material risk decisions are documented via ADRs and recorded in meeting minutes.
- **Multi-agency comparability**: scoring definitions are consistent across departments, even when impact manifests differently.
- **Feedback loops**: SLO/SLA breaches, model validation results, and incident rehearsal findings create/adjust risks.

---

## 2. Roles, decision bodies, and RACI

### 2.1 Core roles
- **Risk owner**: accountable for treatment plan execution and evidence submission.
- **Risk steward (program risk manager)**: maintains taxonomy/library, facilitates reviews, ensures scoring consistency, manages audit trail.
- **Service owner**: accountable for operational performance, SLOs, and incident processes.
- **Data owner (agency)**: accountable for data source quality, sharing constraints, and change notifications.
- **Model owner**: accountable for model validation, monitoring, and rollback.

### 2.2 Decision bodies (example municipal setup)
- **Digital Twin Steering Committee**: prioritization, funding, risk acceptance above appetite.
- **Data Governance Council**: data sharing agreements, data access tiering, privacy constraints.
- **Security & Privacy Review Board**: security exceptions, privacy risk acceptance, DPIA outcomes.
- **Model Risk/Ethics Panel**: approval for high-impact models, validation exceptions, harm mitigations.
- **Architecture Review Board (ARB)**: design decisions, technical standards; ADR approvals.

### 2.3 Minimum RACI expectations
- **Score and propose treatment**: Risk owner (R), steward (C), relevant board (C).
- **Approve within appetite**: Steering Committee delegate (A) or board per category.
- **Approve above appetite / exceptions**: Steering Committee (A) + relevant board (A/C).
- **Verify evidence and close**: independent verifier (R) (internal audit, security, or peer review), steward (A for register state).

---

## 3. Twin-specific risk taxonomy and library

### 3.1 Taxonomy (categories and examples)
Use categories below as required fields in every risk entry.

1) **Privacy, civil liberties, and linkability**
- Linkage across datasets enables re-identification of individuals/households.
- Function creep: repurposing sensors/data for enforcement beyond original mandate.
- Inference risk from aggregate outputs (e.g., predicting sensitive attributes).

2) **Data governance & cross-agency coordination**
- Misaligned legal bases or data sharing agreements across agencies.
- Ambiguous data ownership and change control, causing “silent schema drift.”
- Conflicting definitions/semantics (e.g., “occupancy,” “incident,” “asset”).

3) **Data quality drift & sensing integrity**
- Sensor calibration decay, time sync issues, missingness, bias by geography.
- Data pipeline changes introducing drift in distributions or labels.
- Lack of provenance/lineage leading to un-auditable insights.

4) **Model risk and automation harm**
- Model degradation, concept drift, brittle performance in rare events.
- Disparate impacts across neighborhoods or demographic proxies.
- Over-trust/automation bias; insufficient human-in-the-loop controls.

5) **Cybersecurity and platform resilience**
- Attack surface expansion through IoT, APIs, and multi-tenant platforms.
- Credential compromise, lateral movement, ransomware affecting city services.
- Single points of failure in streaming/geo services.

6) **Vendor dependency and lock-in**
- Proprietary formats/SDKs, lack of escrow, limited exit paths.
- Vendor outage/insolvency affecting critical operations.
- Contract terms preventing effective audit/telemetry access.

7) **Operational readiness and incident response**
- Insufficient on-call coverage, unclear runbooks, weak escalation.
- Incident rehearsal gaps leading to delayed containment.
- SLOs not tied to risk posture, causing recurring degradations.

8) **Regulatory, reputational, and legitimacy**
- Public backlash from perceived surveillance.
- Noncompliance with procurement, privacy, or records retention rules.
- Loss of trust due to opaque decision-making.

### 3.2 Risk library (starter entries)
Maintain a reusable library (templates) so teams don’t reinvent risks and controls.

Each library entry should include:
- Category, description pattern, typical causes
- Default leading indicators
- Common mitigations (prevent/detect/correct)
- Minimum evidence set for closure
- Typical owner(s) and governance body

Example library entry (privacy/linkability):
- **Description pattern**: “Combining [dataset A] with [dataset B] enables re-identification or sensitive inference beyond approved purpose.”
- **Leading indicators**: new joins in analytics jobs; access scope expansion; number of datasets in views; anomaly in query patterns.
- **Mitigations**: purpose limitation; access tiering; aggregation thresholds; differential privacy where appropriate; red-team inference tests.
- **Evidence**: approved DPIA; access policy + enforcement logs; inference test report; ADR for approved joins.

---

## 4. Risk register data model (fields) and audit trail

### 4.1 Required fields
- Risk ID (immutable), title, description
- Category (from taxonomy), affected services/data products
- Risk owner, service owner, data owner, model owner (as applicable)
- Likelihood (1–5), Impact (1–5), Score (L×I)
- Confidence (High/Med/Low) + rationale (why scored this way)
- Risk appetite classification (Within / At / Above appetite)
- Treatment decision (Avoid / Reduce / Transfer / Accept)
- Treatment plan: controls, milestones, due dates
- Evidence checklist (links to artifacts)
- Escalation status (None / In review / Escalated)
- Decision records: ADR link(s) + meeting minutes references
- Observability mapping: SLOs/SLIs, leading indicators, alert routing
- Incident linkage: incident IDs, rehearsal IDs, postmortems
- Last reviewed date, next review date, review cadence

### 4.2 Audit trail requirements
- All changes to likelihood/impact, treatment decision, or closure status must be logged with:
  - who changed it, when, and why
  - link to evidence or decision record
- “Re-open” requires reason and triggers (e.g., drift, incident, vendor change).

---

## 5. Semi-quantitative scoring, thresholds, and escalation

### 5.1 Likelihood scale (municipal, 1–5)
Score likelihood over a 12-month horizon unless stated.
- **1 — Rare**: would require multiple unlikely failures; no known precedent locally.
- **2 — Unlikely**: plausible but not observed; strong controls exist.
- **3 — Possible**: has occurred in comparable cities/programs or precursors exist.
- **4 — Likely**: has occurred locally or control gaps are present.
- **5 — Almost certain**: recurring issue, active exploitation, or imminent change makes it near-certain.

### 5.2 Impact scale (municipal, 1–5)
Impact considers service disruption, safety, privacy/civil liberties, financial cost, and public trust.
- **1 — Minor**: localized inconvenience; no sensitive data; low remediation cost.
- **2 — Moderate**: limited service degradation; minor privacy exposure; manageable operational burden.
- **3 — Major**: multi-department operational impact; credible privacy/civil liberties concern; significant rework cost.
- **4 — Severe**: citywide service degradation or safety-relevant impairment; regulatory breach likely; major reputational harm.
- **5 — Critical**: loss of life/safety risk, systemic rights harm, or major unlawful disclosure; prolonged outage; severe political/financial consequences.

### 5.3 Risk score bands (conservative posture)
Compute **Score = Likelihood × Impact** and classify:
- **1–4 (Low)**: manage within team; review quarterly.
- **5–9 (Moderate)**: management attention; review monthly.
- **10–14 (High)**: requires governance body review; review biweekly.
- **15–25 (Critical)**: immediate escalation; treatment plan within 10 business days; weekly review until reduced.

### 5.4 Risk appetite and acceptance rules (multi-agency)
Because digital twins can affect civil liberties and critical services, set a conservative baseline:
- **Default appetite**: avoid accepting **High/Critical** risks unless a decision body formally approves with documented rationale.
- **Automatic “Above appetite” triggers** (escalate regardless of score):
  - any credible **privacy/linkability/inference** risk involving sensitive attributes or vulnerable populations
  - any risk impacting **public safety operations** or emergency response
  - any risk with **Low confidence** in scoring *and* potential impact ≥ 4
  - any risk requiring a **security/privacy exception**

### 5.5 Escalation criteria and SLA
- **High (10–14)**: submit to relevant board within 15 business days; decision recorded via ADR or minutes.
- **Critical (15–25) or Above appetite**: notify steward + Steering Committee within 2 business days; interim controls within 5 business days.
- If mitigation due date slips by > 14 days for High/Critical, auto-escalate.

---

## 6. Risk treatment verification (evidence and anti-paper controls)

### 6.1 Evidence requirements by control type
A mitigation is only “complete” when evidence demonstrates it is **implemented and effective**.

**Preventive controls** (examples: access controls, data minimization, architecture changes)
- Evidence: configuration exports; policy-as-code results; access logs; ADR approving design; penetration test or privacy review sign-off.

**Detective controls** (examples: monitoring, anomaly detection, audit logs)
- Evidence: dashboards; alert rules; test alerts (synthetic); on-call routing proof; log retention settings.

**Corrective controls** (examples: rollback plans, incident runbooks)
- Evidence: runbook links; rehearsal results; time-to-recover metrics; post-rehearsal action items closed.

**Model controls** (examples: validation, bias testing, drift monitoring)
- Evidence: validation report; model card; drift thresholds and alerting; shadow deployment results; rollback decision record.

### 6.2 Anti-paper-compliance controls
To prevent “checkbox” mitigation completion:
- **Independent verification**: closure requires review by a party not responsible for implementation (security, privacy office, SRE, internal audit, or peer review rotation).
- **Control effectiveness tests**: at least one of:
  - tabletop or game day proving the control works under realistic conditions
  - automated checks (CI/CD policy checks, configuration drift detection)
  - sampling audits (access review sampling, log review sampling)
- **Expiry on evidence**: evidence older than a defined window (e.g., 180 days) triggers re-verification for ongoing controls.

### 6.3 Closure criteria
A risk can be moved to “treated” when:
- residual score is recalculated and within appetite
- evidence checklist is complete and verified
- decision record exists for any explicit risk acceptance

---

## 7. Integration with stage-gates and ADRs (governance alignment)

### 7.1 Stage-gate integration (minimum)
At each stage-gate, the risk register is a required input and output.

- **Gate 0 — Discovery**
  - Output: initial risk identification using taxonomy; privacy/linkability screening; vendor lock-in assessment.
- **Gate 1 — Design approval**
  - Output: ADRs for key architectural choices (data storage, identity, join patterns, vendor dependencies) linked to risks.
  - No-go if any Above appetite risk lacks treatment plan.
- **Gate 2 — Pilot / limited rollout**
  - Output: evidence of observability (SLIs/SLOs), incident runbooks, rehearsal schedule; model validation plan.
- **Gate 3 — Production**
  - Output: verified controls; SLOs adopted; on-call and escalation in place; DPIA complete where applicable.
- **Gate 4 — Scale / multi-agency expansion**
  - Output: cross-agency governance agreements; periodic audits; exit plan validated for critical vendors.

### 7.2 ADR linkage rules
Create an ADR when a decision materially changes risk posture, including:
- accepting Above appetite risks
- selecting a vendor/platform with lock-in implications
- enabling new data joins or expanding access scopes
- deploying or updating high-impact models

Each ADR must link to:
- affected risk IDs
- considered alternatives and why rejected
- monitoring/rollback commitments (operational constraints)

---

## 8. Observability, leading indicators, and SLO feedback loops

### 8.1 Map risks to leading indicators
For each High/Critical risk, define at least 2 leading indicators and 1 lagging indicator.

Examples (twin-specific):
- **Data quality drift**
  - Leading: missingness %, schema change frequency, sensor calibration age, outlier rate by district.
  - Lagging: model performance drop, increase in manual overrides, incident count tied to bad data.

- **Privacy/linkability**
  - Leading: number of datasets joined per view; growth in access grants; unusual query patterns; new external sharing endpoints.
  - Lagging: privacy incidents; audit findings; citizen complaints.

- **Vendor lock-in**
  - Leading: proprietary API usage count; % workloads not portable; time to recreate environment from IaC; escrow/extract tests.
  - Lagging: exit cost estimates rising; inability to meet audit/telemetry requirements.

### 8.2 SLO integration
- Maintain SLOs for critical twin services (ingest, geo services, analytics APIs, dashboards).
- Define how SLO burn rates affect risk scoring:
  - repeated SLO breaches in a quarter increase Likelihood by 1 for related operational risks
  - “near misses” in rehearsal increase Likelihood or reduce Confidence
- Use error budgets as governance signals: when error budget is exhausted, changes requiring elevated risk can be frozen pending review.

### 8.3 Incident response integration
- When an incident occurs:
  - link the incident/postmortem to existing risk IDs
  - create new risks for systemic causes not previously captured
  - update likelihood/impact and add new controls with evidence
- When an incident rehearsal occurs:
  - record rehearsal ID and outcomes as evidence
  - open risks for gaps discovered (e.g., missing runbook, unclear ownership)

---

## 9. Operating cadence and reporting

### 9.1 Cadence
- **Weekly**: Critical/Above appetite risks; overdue mitigations; recent incidents and SLO breaches.
- **Monthly**: full register review; scoring calibration across agencies.
- **Quarterly**: risk appetite review, trend analysis, and board reporting.

### 9.2 Reporting artifacts (auditable)
- Risk dashboard with score distribution, top risks, overdue mitigations
- Change log summary (what changed in the register)
- Evidence completion rates for High/Critical risks
- Cross-agency dependency map (risks spanning multiple departments)

---

## 10. Deliverables (what “done” looks like)
- A twin-specific taxonomy and reusable risk library entries (templates)
- A risk register with required fields and change logging
- Scoring guide with appetite thresholds and escalation SLAs
- Evidence checklists and independent verification process
- Integrations: links to ADRs, stage-gates, SLOs/dashboards, incidents and rehearsals

---

## References

### Workspace source
- Item 15 in [`kali-task-research.md`](../kali-task-research.md:1)

### External references
- **COSO — Enterprise Risk Management (ERM) Guidance** — https://www.coso.org/guidance-erm
  - Takeaway: Canonical COSO landing page for ERM guidance and supporting materials; useful for aligning a municipal digital program’s risk governance and decision traceability to a recognized ERM framework.

- **ISO — ISO 31000: Risk management (guidelines) (PUB100426)** — https://www.iso.org/files/live/sites/isoorg/files/store/en/PUB100426.pdf
  - Takeaway: High-level ISO primer explaining ISO 31000’s principles, framework, and process; good for public-sector programs that need a non-prescriptive, enterprise-wide approach adaptable to digital services.

- **NIST — SP 800-37 Rev. 2: Risk Management Framework for Information Systems and Organizations (PDF)** — https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-37r2.pdf
  - Takeaway: Defines the RMF life-cycle steps (Prepare, Categorize, Select, Implement, Assess, Authorize, Monitor) that map cleanly to evidence-gated risk treatment and ongoing monitoring in public-sector digital systems.

- **UK National Audit Office — Managing risks in government (PDF)** — https://www.nao.org.uk/wp-content/uploads/2011/06/managing_risks_in_government.pdf
  - Takeaway: Public-sector audit perspective on making risk management practical and decision-relevant (not “paper compliance”); supports governance cadence and escalation design for government programs.

- **Queensland Audit Office — Using risk appetite in the public sector** — https://www.qao.qld.gov.au/blog/using-risk-appetite-public-sector
  - Takeaway: Explains how public entities define and use risk appetite/tolerance to drive decisions and control effort; a useful reference for drafting municipal risk appetite statements tied to strategic objectives.

- **Charters Towers Regional Council (Queensland) — Risk Appetite Statement (PDF)** — https://www.charterstowers.qld.gov.au/files/assets/public/v/1/forms-publications/documents/policies/risk-appetite-statement-strat0024.pdf
  - Takeaway: Example municipal risk appetite statement artifact that can be adapted as a template for categorizing appetite across risk types and setting practical boundaries for acceptance/escalation.

- **U.S. GAO — Government Auditing Standards (Yellow Book), 2024 Revision (PDF)** — https://www.gao.gov/assets/d24106786.pdf
  - Takeaway: Defines expectations for sufficiency/appropriateness of evidence and rigorous testing; supports anti-gaming closure by requiring evidence that is not only documented but credible and fit for audit objectives.

- **U.S. GAO — Standards for Internal Control in the Federal Government (Green Book) (PDF)** — https://www.gao.gov/assets/gao-14-704g.pdf
  - Takeaway: Internal-control principles (including monitoring) that help structure “risk closure” as validated control design + operating effectiveness, with periodic reassessment rather than one-time sign-off.

- **Google SRE Workbook — Chapter 2: Implementing SLOs** — https://sre.google/workbook/implementing-slos/
  - Takeaway: Practical guidance for deriving error budgets from SLOs and using policies when budgets are burned; a strong pattern for integrating risk registers with observability, incident response, and governance decision gates.
