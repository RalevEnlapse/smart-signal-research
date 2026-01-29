# Stage-gate governance for a multi-agency digital twin portfolio — Deep research

## Executive summary

This document defines an implementable, **risk-tiered (T0–T3)** stage-gate governance model for a **multi-agency digital twin portfolio**. It is designed to:

- preserve agility for discovery work while preventing “pilot forever” dynamics
- integrate **data readiness**, **model validation**, **human-in-the-loop (HITL)** safeguards, and **observability** into every gate
- clarify **decision rights**, **urgency overrides**, and **conflict-of-interest** protections in a multi-agency context
- provide **balanced transparency** and a practical **FOI/public-records workflow** when the legal context is unknown
- close the loop with **outcome measurement** and **anti-gaming** controls
- prioritize **use-existing-tools-first** so teams can start with minimal process and iterate

The framework is built around a small set of required artifacts per gate, standardized checklists, and a portfolio cadence that scales with risk.

---

## 1. Scope and operating context

### 1.1 What is governed

This stage-gate model governs initiatives that create, change, or operationalize any of the following within a digital twin ecosystem:

- data ingestion, fusion, and exchange across agencies
- models (analytics, simulation, optimization, ML) used for decisions or public services
- decision-support workflows and automated actions
- shared platform components (identity, integration, geospatial, event streaming)
- user-facing products (dashboards, APIs, citizen services)

### 1.2 Multi-agency assumptions

- Agencies have distinct mandates, budgets, risk tolerances, and data custodianship responsibilities.
- The portfolio has shared infrastructure and shared reputational risk.
- External vendors/partners may deliver components but must not control gate decisions.

### 1.3 Design principles (use-existing-tools-first)

- **Proportionality**: gate rigor scales with risk tier (T0–T3) and change impact.
- **Evidence over narrative**: decisions rely on artifacts, measures, and test evidence.
- **Transparency by default, with justified exceptions**: publish what can be safely published.
- **Minimal viable governance**: start with templates + checklists using existing tools.
- **Decision traceability**: each gate produces a decision record linking evidence → decision → conditions.

---

## 2. Proportionality policy (risk-tiered rigor: T0–T3)

This section defines the proportionality rules that prevent over-bureaucratization while ensuring high-risk work receives high assurance.

### 2.1 Tier definitions

- **T0 — Discovery / low-risk exploration**
  - Examples: internal analysis spikes on synthetic or fully non-sensitive data; user research; throwaway prototypes.
  - Key constraint: outputs must not be operationalized without tier reassessment.

- **T1 — Controlled prototype / limited pilot**
  - Examples: small pilots in a non-critical operational setting; limited internal users; controlled datasets.
  - Key constraint: any citizen-facing impact or automation triggers reassessment.

- **T2 — Operational / material impact**
  - Examples: supports operational decisions; cross-agency data exchange; significant spend; meaningful service impact.
  - Key constraint: requires formal data readiness, model validation plan, and operational safeguards.

- **T3 — Safety-/rights-critical / high sensitivity**
  - Examples: public safety decisions, enforcement actions, critical infrastructure operations, high-sensitivity personal data, or high automation.
  - Key constraint: strongest assurance (independent review, formal validation, HITL requirements, incident runbooks, rollback readiness).

### 2.2 Tiering inputs (how to assign or change a tier)

Tier is set at Gate 0 and must be revisited at every gate if any of the following changes:

- data sensitivity classification increases
- automation level increases
- scope crosses additional agencies or data domains
- public/citizen impact increases
- spend or vendor lock-in increases

### 2.3 Rigor by tier (minimums)

| Governance element | T0 | T1 | T2 | T3 |
|---|---:|---:|---:|---:|
| Gate package size | 1–2 pages + checklist | ≤5 pages + checklist | ≤10 pages + evidence links | ≤15 pages + evidence links |
| Data readiness evidence | basic | documented | validated | independently reviewed |
| Model validation | n/a or sanity checks | evaluation plan | validated vs acceptance criteria | independent + adversarial testing |
| HITL | optional | recommended | required for high-impact actions | required + escalation/stop conditions |
| Observability/SLOs | basic logging | service metrics | SLOs + alerting | SLOs + audit + safety monitoring |
| Decision forum | single approver | small board | formal board | board + independent assurance |

---

## 3. Governance roles, decision rights, and escalation

### 3.1 Core bodies and roles

- **Portfolio Council (PC)**: cross-agency executive body that owns portfolio priorities, risk appetite alignment, and disputes.
- **Gate Review Board (GRB)**: operational decision body for gates; composition changes by tier.
- **Technical Review Panel (TRP)**: deep technical assurance; provides recommendations, not final decisions.
- **Data Stewardship Group (DSG)**: data custodians and privacy/compliance representatives; required sign-off for data access and sharing.
- **PMO / Portfolio Office (PO)**: runs cadence, templates, training, and auditability.
- **Product Owner / Service Owner**: accountable for value delivery and operational outcomes.
- **Model Owner**: accountable for model behavior, validation evidence, and monitoring.
- **SRE/Operations Lead**: accountable for reliability, observability, and runbooks.

### 3.2 Decision rights (RACI summary)

- **Tier assignment**: GRB proposes, PC arbitrates disputes.
- **Gate approvals (Go / Conditional Go / Hold / Stop)**: GRB decides; PC can override only through formal recorded escalation.
- **Data sharing approval**: DSG required; without DSG approval, gate cannot pass if shared data is involved.
- **Safety and compliance sign-off**: Legal/Compliance must sign for T2/T3 where regulated obligations apply.

### 3.3 Urgency overrides (fast-track without bypassing accountability)

Urgency overrides exist because city operations sometimes require rapid deployment.

**Override rule**: urgency can shorten timelines and defer non-safety work, but cannot waive:

- data access approvals and logging/audit trail
- minimum HITL or rollback controls where risk is material
- incident response runbooks for any operational deployment

**Mechanism**:

- “Emergency Gate” decision by GRB chair + one independent approver (not the project sponsor)
- mandatory decision record with explicit deferred items, owners, and deadlines
- automatic post-decision review within 10 business days

### 3.4 Conflict-of-interest controls

- Vendors/partners may present evidence but must not vote on gate outcomes.
- Any reviewer with procurement, vendor management, or delivery responsibility must declare COI and abstain.
- T3 requires at least one independent reviewer outside the delivering agency.

---

## 4. Stage model (portfolio-friendly)

Stages describe work; **gates control funding, scope, and operational exposure**.

- **Stage 0 — Discovery** (problem framing, feasibility, tiering)
- **Stage 1 — Scoping** (requirements, options, initial architecture)
- **Stage 2 — Business case & assurance plan** (benefits, risks, delivery and assurance)
- **Stage 3 — Build** (implementation, integration, controls)
- **Stage 4 — Validate** (testing, model validation, readiness)
- **Stage 5 — Launch & operate** (go-live, monitoring, incident readiness)
- **Stage 6 — Benefits & learning** (outcomes, optimization, decommission decisions)

---

## 5. Gate-by-gate checklists (implementable, data/AI-aware)

Each gate produces a **Gate Decision Record (GDR)** and an updated **Risk Register** entry.

### Common gate package (all tiers)

- 1-page summary (problem, scope, tier, decision requested)
- evidence links (not attachments where possible)
- updated risk register + mitigations
- decision log entry (what changed since last gate)

### Gate 0 — Intake & tiering (enter Stage 0)

**Decision**: Accept / Reject / Redirect / Merge; assign tier T0–T3.

Checklist:

- **Strategic alignment**: problem statement linked to portfolio outcomes and agency mandates
- **Proportionality**: tier proposed with reasoning; expected change impact documented
- **Data readiness**: initial data sources list; preliminary sensitivity classification; known constraints
- **Model/approach**: whether model(s) are planned; initial validation intent (if applicable)
- **HITL**: anticipated decision points requiring human review
- **Observability**: minimal telemetry plan for any prototype environment
- **Tooling minimalism**: confirm the team can proceed using existing tools (see Section 8)

Minimum evidence by tier:

- T0: concept note + tier rationale
- T1–T3: concept note + initial risk sketch + data access path owner

### Gate 1 — Scope & options (exit Stage 1)

**Decision**: Go / Conditional Go / Hold / Stop.

Checklist:

- **Requirements**: user needs + non-functional requirements drafted (latency, uptime, privacy constraints)
- **Architecture**: high-level architecture and integration approach; data flows identified
- **Data readiness**:
  - data owners identified
  - lineage and collection method described
  - minimum quality expectations defined
- **Model validation**:
  - target use decisions defined
  - evaluation metrics identified (and who owns them)
  - baseline / “do nothing” comparator defined
- **HITL**:
  - where human review occurs and what authority they have
  - escalation path when model output conflicts with operator judgment
- **Observability**:
  - draft SLIs/SLOs for service and model behavior (tier-dependent)
  - logging/audit requirements noted

### Gate 2 — Business case & assurance plan (exit Stage 2)

**Decision**: fund build, constrain scope, or stop.

Checklist:

- **Business case**:
  - costs (capex/opex) and funding source(s)
  - expected benefits with measurement plan (see Section 7)
- **Risk & compliance**:
  - updated risk register with mitigations and residual risk acceptance authority
  - data protection and sharing approach for cross-agency flows
- **Data readiness**:
  - data contract draft (fields, cadence, SLAs/SLOs, retention, access controls)
  - plan for data quality monitoring and remediation ownership
- **Model validation**:
  - validation protocol (datasets, test design, acceptance thresholds)
  - bias/failure-mode considerations appropriate to context
  - plan for drift detection and retraining governance (if ML)
- **HITL**:
  - operating procedures, staffing requirements, and training plan
  - safety interlocks / stop conditions defined for T2/T3
- **Observability**:
  - monitoring plan includes service health + model performance + data pipeline health
  - incident severity definitions drafted

### Gate 3 — Build readiness (exit Stage 3)

**Decision**: proceed to validation, or hold for gaps.

Checklist:

- **Engineering readiness**: backlog prioritized, integration tests planned, security controls designed
- **Data readiness**:
  - pipeline implemented to a test environment
  - data validation checks implemented (schema, completeness, timeliness)
- **Model validation**:
  - model versioning + reproducibility plan
  - offline test harness exists; baseline results recorded
- **HITL**:
  - user workflows implemented; audit trail for overrides exists
- **Observability**:
  - dashboards and alerts created for key SLIs
  - logging includes decision traces (inputs/outputs) consistent with privacy constraints

### Gate 4 — Validation & go-live readiness (exit Stage 4)

**Decision**: authorize production launch, conditional launch, or stop.

Checklist:

- **Testing**:
  - functional + performance testing complete
  - security testing proportional to tier
- **Data readiness**:
  - data quality SLAs/SLOs agreed; monitoring live
  - lineage documented; retention and access confirmed
- **Model validation**:
  - validation evidence against acceptance criteria
  - known limitations documented; “no-go” conditions declared
  - rollback plan for model changes
- **HITL**:
  - runbooks for operator actions
  - training completed; staffing confirmed for operating hours
- **Observability**:
  - SLOs finalized; alert thresholds tested
  - incident response on-call path established

### Gate 5 — Launch review (30–60 days post go-live)

**Decision**: continue, expand, constrain, or roll back.

Checklist:

- **Operational stability**: SLO attainment, incident trend, backlog health
- **Data readiness**: pipeline reliability; quality trends; remediation effectiveness
- **Model validation**: live performance vs expected; drift signals; override rates
- **HITL**: workload, decision quality feedback, any safety events or near misses
- **Observability**: monitoring gaps identified and fixed

### Gate 6 — Benefits & lifecycle decision (quarterly / semiannual)

**Decision**: scale, sustain, redesign, or retire.

Checklist:

- **Outcomes**: benefits realized vs forecast with counterfactual reasoning (as feasible)
- **Anti-gaming controls**: confirm metrics definitions unchanged without approval (see Section 7)
- **Cost & FinOps**: unit cost trends; optimization actions; “keep/kill” economics
- **Risk posture**: residual risk acceptance reaffirmed; new risks captured
- **Transparency**: publishable summary updated; FOI log updated (see Section 6)
- **Lifecycle**: decommission plan and data retention disposition if retiring

---

## 6. Transparency, FOI/public-records workflow (balanced, context-unknown)

Because the FOI/public-records legal context is unknown, apply a two-layer approach:

### 6.1 Two-layer documentation model

- **Public-ready layer** (default publishable):
  - decision outcome (Go/Conditional/Hold/Stop)
  - high-level rationale in plain language
  - high-level risks and mitigations (non-sensitive)
  - intended benefits and how they will be measured

- **Controlled layer** (internal, access-controlled):
  - detailed vendor pricing, security findings, sensitive data specifics
  - detailed model internals where disclosure increases risk
  - personal data elements, credentials, or operational security details

### 6.2 FOI/public-records handling workflow

- Maintain an **FOI Readiness Register** per project that lists:
  - what artifacts exist per gate
  - which layer each artifact belongs to
  - owner for review/redaction
- For any request:
  - log request metadata and deadlines
  - perform a rapid triage (public-ready vs controlled)
  - review/redact controlled items with Legal/Compliance
  - respond with decision record + allowable evidence; note any lawful exemptions when applicable

### 6.3 Transparency defaults by tier

- T0: internal only unless already public by nature
- T1: publish brief summaries after Gate 2 and Gate 5
- T2: publish summaries after each gate (excluding controlled details)
- T3: publish summaries after each gate + independent assurance summary where possible

---

## 7. Outcome measurement and anti-gaming (closing the loop)

### 7.1 Measurement model

Each initiative must define, by Gate 2:

- **Outcome metrics** (what changed in the world): e.g., response time reduction, fewer failures, improved safety indicators
- **Output metrics** (what the system produced): e.g., coverage, prediction accuracy, data completeness
- **Process metrics** (how delivery behaved): e.g., cycle time, defect escape rate

### 7.2 Decision accuracy (how it is adjudicated)

“Decision accuracy” is assessed at Gate 5 and Gate 6 using:

- a pre-declared evaluation horizon (e.g., 60 days, 6 months)
- comparison to baseline (pre-system or “do nothing”)
- qualitative review of unintended consequences

### 7.3 Anti-gaming controls

- Metric definitions are versioned; any change requires GRB approval and a documented rationale.
- Projects cannot redefine “success” after launch without updating forecasts and recording deltas.
- Gate 6 requires an explicit review of any changes to scope, metrics, or benefit claims.
- A sample of Gate 6 outcomes is audited by PO each quarter (especially for T2/T3).

---

## 8. Tooling and minimal viable process (use existing tools first)

### 8.1 Minimal viable implementation (no new tooling)

Start with the tools agencies already have:

- **Work tracking**: Jira / Azure DevOps / Trello (project backlog + gate tasks)
- **Documentation**: SharePoint / Confluence / Google Workspace (gate packages, templates)
- **Approvals**: existing e-sign / email + meeting minutes + decision record template
- **Risk register**: spreadsheet or existing GRC tool
- **Dashboards**: existing BI (Power BI/Tableau) or simple status pages

### 8.2 “Thin layer” governance artifacts

Create a small set of reusable templates:

- Gate Decision Record (GDR)
- Tiering worksheet
- Data readiness checklist + data contract template
- Model validation plan template
- Runbook template
- FOI Readiness Register

### 8.3 Integration patterns

- Link each gate to a milestone in the work tracker.
- Store evidence as links to authoritative sources (test reports, dashboards).
- Use standard naming conventions so audits are trivial.

---

## 9. Cadence, staffing, and operating model

### 9.1 Cadence

- **Monthly**: portfolio triage (Gate 0/1) and prioritization refresh
- **Biweekly (optional)**: technical assurance clinic for teams approaching Gate 3/4
- **Quarterly**: Gate 6 outcomes review + portfolio rebalancing
- **Annual**: governance calibration + tier definitions review

### 9.2 Staffing (indicative)

- **Portfolio Office (PO)**: 1–2 FTE per ~15–30 active initiatives (admin + analytics + audit)
- **Gate Review Board**: rotating membership; protected time blocks per month
- **Technical Review Panel**: part-time SMEs; increased engagement for T2/T3
- **Independent assurance** (T3): internal audit / external reviewer capacity as needed

### 9.3 Meeting discipline

- Gate meetings are time-boxed; decisions are made only on complete packages (or explicitly recorded exceptions).
- Conditional approvals must list conditions, owners, and deadlines.

---

## 10. Runbooks (operationalization)

Runbooks are mandatory for any production deployment (T1+), and must exist by Gate 4.

Minimum runbook sections:

- service overview and dependencies
- operating hours and staffing model
- incident severity levels + response steps
- rollback procedure (including model rollback)
- data pipeline failure handling
- HITL procedures: override, escalation, stop conditions
- communications plan (internal/external) for major incidents

---

## 11. Observability and metrics (SLIs/SLOs and governance metrics)

### 11.1 Service and pipeline observability (system-level)

Minimum indicators (tier-scaled):

- pipeline freshness/latency
- data completeness and schema validity
- service availability and error rate
- cost/unit metrics where meaningful

### 11.2 Model observability (model-level)

Where models are used:

- performance on monitored datasets (as feasible)
- drift indicators
- override rate and reason codes
- calibration/uncertainty monitoring (where relevant)

### 11.3 Governance process metrics (portfolio-level)

- gate cycle time (submission → decision)
- package completeness rate
- conditional approval condition-closure time
- stop rate by stage (signal of selection quality)
- post-launch incident rate by tier
- benefits realization vs forecast (with audit sampling)

---

## 12. Implementation roadmap (phased adoption)

### Phase 1 (0–6 weeks): Minimum viable governance

- adopt tiering policy (Section 2)
- publish templates (Section 8.2)
- run Gate 0–2 for all new initiatives; map existing work to nearest gate

### Phase 2 (6–12 weeks): Assurance integration

- require data readiness + model validation plans by Gate 2
- require observability plans by Gate 3
- runbooks mandatory by Gate 4

### Phase 3 (3–6 months): Portfolio maturity

- start Gate 5 and Gate 6 routines
- implement anti-gaming audits and outcome reviews
- calibrate tiering and checklists based on lessons learned

### Phase 4 (6–12 months): Optimization

- reduce overhead via automation in existing tooling
- standardize cross-agency data contracts
- expand independent assurance patterns for T3

---

## 13. References

- **UK Infrastructure and Projects Authority (IPA)** — *Gate Review 1: Business Justification* (Gateway Review guidance). https://www.gov.uk/government/publications/ogc-gateway-review-1-business-justification-guidance-and-templates
  - Takeaway: Describes the UK government Gateway Review approach, using independent assurance at defined lifecycle decision points (“gates”) anchored to the Five Case Model; useful as a public-sector precedent for stage-gate packages and decision checkpoints.

- **UK Infrastructure and Projects Authority (IPA)** — *Gate 0 Review* (Gateway Review process, early stage). https://assets.publishing.service.gov.uk/media/60f01860e90e0764cb10bd8a/1174-APS-0-CCS0521656666-001_IPA_Gateway_Web.pdf
  - Takeaway: Provides a concrete example of an early “Gate 0” review focused on strategic assessment and readiness to proceed, aligning well to portfolio intake/tiering and evidence expectations at the earliest gate.

- **US Department of Health & Human Services (HHS)** — *EPLC Stage Gate Review Practices Guide* (Enterprise Life Cycle). https://www.hhs.gov/sites/default/files/ocio/eplc/EPLC%20Archive%20Documents/56%20-%20Stage%20Gate%20Reviews/eplc_stage_gate_reviews_practices_guide.pdf
  - Takeaway: Defines stage-gate reviews in a federal IT governance context, emphasizing independent confirmation and standard review questions; helpful for operationalizing gate teams/roles and consistent gate criteria.

- **Victorian Department of Treasury and Finance (DTF), Australia** — *Gateway key decision points, guidance and templates*. https://www.dtf.vic.gov.au/gateway-key-decision-points-guidance-and-templates
  - Takeaway: Demonstrates a government-run gateway review regime and associated templates for decision points, supporting the idea of standardized artifacts and repeatable assurance across a portfolio.

- **UK Government (GOV.UK)** — *Major projects data* (Government Major Projects Portfolio transparency dataset). https://www.gov.uk/government/collections/major-projects-data
  - Takeaway: Shows how a central government publishes portfolio-level information on major projects as “transparency data”, providing a practical model for public reporting at the portfolio layer.

- **UK Infrastructure and Projects Authority (IPA)** — *Annual Report on Major Projects 2023–24* (Government Major Projects Portfolio). https://assets.publishing.service.gov.uk/media/678a4a9869b9b76c761d0574/IPA_Annual_Report_2023-24.pdf
  - Takeaway: A recurring, structured public report on portfolio status and delivery performance; relevant as a template for transparent reporting, cross-portfolio comparability, and explaining assurance approaches.

- **UK Information Commissioner’s Office (ICO)** — *Publication schemes: a guide* (Freedom of Information Act). https://ico.org.uk/for-organisations/foi/publication-schemes-a-guide/
  - Takeaway: Provides authoritative FOI guidance requiring public authorities to proactively publish information via an approved publication scheme, aligning with “transparency by default” patterns and defining what can be routinely disclosed.

- **US Government** — *FOIA.gov (Freedom of Information Act portal)*. https://www.foia.gov/
  - Takeaway: Centralized FOIA guidance and request-handling entry point; useful for defining baseline workflows (logging, routing, response handling) and setting expectations for request intake and public access.

- **US Congressional Research Service (CRS)** — *The Freedom of Information Act (FOIA): A Legal Overview*. https://www.congress.gov/crs-product/R46238
  - Takeaway: Summarizes FOIA’s disclosure framework and exemptions at a high level, helping frame a defensible “public-ready vs controlled” documentation approach and explain lawful limits on transparency.
