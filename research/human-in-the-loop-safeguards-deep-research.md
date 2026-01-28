# Human-in-the-loop safeguards — Deep research

## Executive summary
Human-in-the-loop (HITL) safeguards are **socio-technical controls** that keep **accountable humans** in the loop for high-impact recommendations produced by analytics/optimization/ML/AI in a City Digital Twin.

HITL is not “add an approve button.” To be defensible, it must make **meaningful human control operational**:

- **Workload-aware:** approval queues have capacity models, SLOs, prioritization, and overload behavior (no silent backlog).
- **Auditable:** every recommendation/decision has immutable provenance (data snapshot IDs, model version, run ID), reason codes, and outcome linkage.
- **Contestable:** internal staff and (where enabled) affected residents/organizations can appeal decisions on a defined timeline with defined artifacts.
- **Safe under incidents:** incident mode enables **bounded autonomy** via pre-approved playbooks, time-boxed exceptions, and mandatory post-hoc review.

This document deepens item 23 in [`kali-task-research.md`](kali-task-research.md:1): *“Human-in-the-loop safeguards: Keep humans in approval loops for high-impact decisions to reduce automation risk.”*

---

## 1) Scope and definitions

### 1.1 What HITL governs in a City Digital Twin
HITL applies anywhere the twin influences actions that can cause harm, rights impacts, or major spend, including:
- dispatch and emergency recommendations
- traffic control and pricing
- inspection prioritization and enforcement targeting
- public communications and warnings
- resource allocation (shelter beds, outreach)
- infrastructure actuation via partners (utility demand response)

### 1.2 “Meaningful human control” (MHC)
MHC means that a human decision-maker:
- has **time and capacity** to review
- has **understandable evidence** with uncertainty
- has **real authority** to approve/deny/override
- has **traceability** (what produced the recommendation)
- is embedded in a system with **post-action monitoring** and **recourse**

---

## 2) Acceptance criteria for “meaningful human control” (testable)

Treat MHC as acceptance criteria, not principles.

### 2.1 Capacity and training
- Approver role has documented training/certification for the decision class.
- On-call/coverage ensures Tier 2–3 decisions have reachable approvers.

### 2.2 Evidence sufficiency and comprehensibility
- Every recommendation includes an evidence packet meeting the required contract (§4).
- UI renders confidence/uncertainty and applicability constraints.

### 2.3 Authority and override reality
- Approvers can reject/override without penalty; override reasons are collected.
- Override/deny rates are monitored; very low rates trigger review for rubber-stamping.

### 2.4 Traceability and auditability
- Immutable decision logs include data lineage IDs, model version, and run ID.
- A sampled audit can reproduce the recommendation from the logged inputs.

### 2.5 Post-decision monitoring exists
- Outcome monitoring and harm metrics are defined per decision domain (§6).
- Periodic sampling audits occur at a defined cadence.

---

## 3) Decision tiering and decision rights (Tier 0–3)

### 3.1 Tier model (keep consistent across the twin)
- **Tier 0 — Informational:** no actuation; dashboards and insights.
- **Tier 1 — Low impact:** reversible operational tweaks; single approver.
- **Tier 2 — High impact:** significant operational impact; multi-party approval; tighter constraints.
- **Tier 3 — Exceptional/high stakes:** potential safety/civil liberties/large spend impacts; **two-person rule** + incident commander/executive approval.

### 3.2 Decision rights and separation of duties
- Tier 2+: separation of duties between model owner and approver.
- Tier 3: two-person approval (distinct roles) + explicit authority basis recorded.

### 3.3 Tier eligibility (coupled to validation)
A model can only support Tier 2–3 decisions when validation gates are met (§7). Otherwise the same workflow is downgraded (Tier 1) or forced to informational (Tier 0).

---

## 4) Evidence packet + provenance contract (required)

### 4.1 Evidence packet contents (minimum)
Every recommendation must include:

**A) Decision context**
- decision type + tier
- affected geography/assets/populations (at allowed granularity)
- proposed action + constraints + expected timeline

**B) Inputs and lineage**
- input sources + timestamps
- **data snapshot IDs** / dataset versions
- feature set ID/version (if applicable)

**C) Model provenance**
- model name + **model version**
- training data snapshot ID
- run ID / inference job ID

**D) Confidence and uncertainty**
- confidence intervals/credible intervals or calibrated probabilities
- sensitivity to key assumptions (where available)
- applicability constraints (“do not use when X”) and known failure modes

**E) Rationale**
- top contributing factors (appropriate to model type)
- “what would change the recommendation?” counterfactual cues

**F) Operator action affordances**
- approve / deny / request more evidence
- override with reason code
- mark as “insufficient evidence” (safe stop)

### 4.2 Handling conflicting evidence
When signals conflict:
- show conflict explicitly (source disagreement, stale inputs)
- default to **insufficient evidence** for Tier 2–3 unless a human records a justification
- require a second reviewer for Tier 3 conflict overrides

### 4.3 UI uncertainty communication requirements
- Never show a single-point output without uncertainty.
- Use consistent uncertainty vocabulary (“high/medium/low confidence” tied to quantitative thresholds).
- Surface data freshness and missingness.

---

## 5) Workload-aware approval design

### 5.1 Queue model and SLOs
Define per tier:
- **time-to-review SLO** (e.g., Tier 2: 5 min; Tier 3: 2 min during incidents)
- **max queue age** (hard cap; beyond this, route to escalation)
- **review bandwidth model**: expected arrivals/hour vs reviewer capacity/hour

Track:
- backlog size by tier
- p50/p90/p99 time-to-review
- % items breaching max queue age

### 5.2 Prioritization and batching
- Priority order: Tier 3 → Tier 2 → Tier 1.
- Support batching for low-risk Tier 1 (time windows) but forbid batching for Tier 3.

### 5.3 Approver unavailability and escalation
If approver unavailable:
- escalate to on-call role after a fixed timeout
- if still unavailable, degrade to safe state:
  - Tier 3: block actuation; require incident commander replacement
  - Tier 2: block or downgrade to Tier 1 depending on playbook

### 5.4 Rubber-stamping detection signals
Monitor and alert on:
- very fast approvals (below minimum read time)
- low variance in decisions (always approve)
- high reversal rate post-approval
- approvals without opening evidence packet

When detected:
- require refresher training; temporarily raise tier; increase audit sampling.

---

## 6) Outcome monitoring, harm metrics, and feedback-loop detection

### 6.1 Harm metric categories (define per domain)
At minimum define:
- **safety harm:** incidents, near misses, injuries
- **service harm:** denials/delays of service, missed dispatches
- **rights/legitimacy harm:** complaints, appeals upheld, policy violations
- **equity harm:** disparate impact metrics by approved strata

### 6.2 Feedback-loop detection
Identify spirals such as:
- enforcement allocation loops (more presence → more detected incidents → more presence)
- service denial loops (missed neighborhoods → worsening outcomes → reduced trust)

Signals:
- widening geographic concentration not explained by exposure/need
- persistent disparities increasing after deployment
- policy “success” metrics improving while complaint/harm metrics worsen

### 6.3 Post-decision monitoring and audits
- Link decisions to outcomes where feasible.
- Sample audits (e.g., weekly Tier 2; every Tier 3; monthly Tier 1 sample).
- Audit findings must have an owner and closure SLA.

---

## 7) Integration with model validation and release management

### 7.1 Coupling principles
- A model’s **validation status** determines the maximum tier it can support.
- HITL policies are versioned and approved like code (link to ADRs).

### 7.2 Required validation artifacts to change tiers
To increase automation or allow a higher tier, require:
- validation report (offline metrics + bias checks)
- stress tests / red-team scenarios
- calibration results and failure modes
- monitoring plan and rollback criteria

### 7.3 Release patterns
- shadow mode → limited pilot → canary → broad rollout
- feature flags for instant disable
- rollback runbook (who can trigger; what evidence)

---

## 8) Incident mode and bounded autonomy (enabled)

### 8.1 Activation criteria and authority
- incident mode can be activated only by an authorized incident commander role.
- activation is logged with scope, start time, and expected end time.

### 8.2 Pre-approved playbooks
For each decision type, define:
- what can be auto-executed (if anything)
- what remains human-approved
- constraints and blast-radius limits

### 8.3 Time-boxed exceptions
- exceptions expire automatically
- compensating controls: increased logging, narrower scope, higher audit sampling

### 8.4 Post-incident review
- mandatory retrospective
- audit sample of incident-mode decisions
- decisions to keep/retire playbook changes

---

## 9) Recourse and contestability (internal + public)

### 9.1 Who can appeal
- internal staff/operators
- affected residents/organizations (public recourse enabled)

### 9.2 Appeals workflow
1) intake (portal/phone/email; accessibility supported)
2) triage (category, urgency)
3) evidence review (decision record + relevant policies)
4) decision (uphold/modify/reverse) with rationale
5) communicate outcome with appropriate explanation level

### 9.3 Timelines and artifacts
- acknowledge within 5 business days
- substantive response within 30 days (or interim update)

Artifacts:
- appeal case ID
- decision log excerpt
- policy version used
- outcome and remediation actions

### 9.4 Explanation policy (internal vs external)
- internal: full evidence packet subject to role-based access
- external/public: redacted explanation that protects sensitive data and security details

---

## 10) Emergency exception handling (bounded autonomy)

Emergency exceptions are not ad-hoc.
- must map to a pre-approved playbook
- must be time-boxed
- must include post-hoc review and sampling audits

---

## 11) Operational runbooks (minimum)

### 11.1 Approver unavailable / queue overload
- activate escalation policy
- downgrade or block decisions per tier
- switch to incident staffing plan

### 11.2 Disagreement between approver and recommendation
- override with reason code
- require evidence packet review confirmation
- route to periodic model review if overrides exceed threshold

### 11.3 Suspected model malfunction
- switch model to shadow/off via feature flag
- notify model owner and governance
- initiate rollback if criteria met

### 11.4 Emergency exception activation and closure
- record authority and scope
- enforce expiry
- perform post-incident audit

### 11.5 Public appeal / contested decision
- capture request
- apply explanation policy
- route to review panel

---

## 12) Key metrics

### 12.1 Workload and timeliness
- time-to-review p50/p90/p99 by tier
- backlog age distribution
- % breaching max queue age

### 12.2 Decision quality signals
- approval/deny/override rates by tier and model version
- reversal rate and root causes
- rubber-stamping indicators (§5.4)

### 12.3 Audit outcomes
- audit finding rate by tier
- closure time for findings

### 12.4 Incident mode
- activations count/duration
- post-hoc compliance rate

### 12.5 Fairness/harm indicators
- domain-specific disparate impact metrics
- complaints and appeals upheld

---

## 13) Implementation roadmap

### 0–3 months
- finalize tiering policy and decision rights
- implement evidence packet spec + provenance logging
- deploy immutable decision log
- basic approval queue + escalation runbooks

### 3–12 months
- recourse/appeals workflow and explanation policy
- workload metrics and staffing/on-call coverage
- validation coupling (shadow/canary/rollback)
- incident mode drills

### 12–24 months
- mature harm monitoring and feedback-loop detection
- continuous audits and policy refinement
- cross-agency federation of tiering + recourse standards

---

## References (retrieved via Firecrawl MCP)

### Meaningful human control / human factors
- TechDispatch — Human oversight of automated decision-making (European Data Protection Supervisor) — https://www.edps.europa.eu/system/files/2025-09/25-09-15_techdispatch-human-oversight_en.pdf
  - Takeaway: Defines “meaningful/effective” human oversight and how it relates to practical governance requirements for automated decision-making.
- Human Factors Guidance for the Integration of Artificial Intelligence / Machine Learning into Transportation Systems (US DOT) — https://rosap.ntl.bts.gov/view/dot/86694/dot_86694_DS1.pdf
  - Takeaway: Human factors considerations for integrating AI/ML into operational decision contexts, including operator workload and interface concerns.

### Auditability and contestability / recourse
- Artificial Intelligence Risk Management Framework (AI RMF 1.0) (NIST) — https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf
  - Takeaway: Provides a cross-sector risk management framing emphasizing documentation, transparency, human oversight, and lifecycle governance.
- Challenging the Machine: Contestability in Government AI Systems (Lawfare) — https://www.lawfaremedia.org/article/challenging-the-machine-contestability-in-government-ai-systems
  - Takeaway: Discusses design choices that enable effective contestability of government AI systems and practical barriers to appeal.

### Safety case / change control patterns
- Canary Release: Deployment Safety and Efficiency (Google SRE Workbook) — https://sre.google/workbook/canarying-releases/
  - Takeaway: Practical release pattern to limit blast radius and validate changes before full rollout; useful for coupling automation tiers to deployment safety.
- AI Risk Management Framework | NIST — https://www.nist.gov/itl/ai-risk-management-framework
  - Takeaway: Landing page and pointers for NIST AI RMF as an operational governance reference.
