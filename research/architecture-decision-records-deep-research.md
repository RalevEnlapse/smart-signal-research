# Architecture Decision Records (ADRs) — Deep research (decision-grade + operational)

## Executive summary (for a multi-agency city digital twin)
Architecture Decision Records (ADRs) are small, durable documents that capture **what was decided**, **why**, **who decided**, **what was considered**, and **what operational/compliance consequences follow**.

For a municipal-scale city digital twin program (multi-agency, multi-vendor, long-lived), ADRs are not “extra documentation”—they are a **governance control** and a **delivery accelerator**:

- Reduce **architectural drift** across procurement cycles, vendor changes, and staff turnover.
- Create **audit-ready rationale** for privacy/security/interoperability choices.
- Make change safer by connecting deployments, schema changes, and operational runbooks to explicit decisions.

This document is optimized for:
- Target audience: city twin program architects + delivery leads + security/privacy + procurement/PMO
- Org scale: multi-agency, multi-vendor
- Process weight: lightweight-by-default
- Tooling preference: docs-as-code-first
- Integration targets: GitHub/GitLab + CI/CD + Jira/ADO + Confluence/SharePoint
- Compliance context: public-sector procurement + audit readiness + privacy/security reviews

---

## 1) What ADRs are (and what they are not)

### 1.1 ADRs are
- A **decision log** with rationale, alternatives, and consequences.
- A **contract between teams and vendors** (“this is the chosen interface/standard; deviations are exceptions”).
- A **traceability anchor** to implementation evidence (PRs, release notes, test plans) and governance artifacts (DPIAs, risk acceptances, SoWs).

### 1.2 ADRs are not
- A general knowledge base, meeting minutes, or a design doc dumping ground.
- A substitute for reusable guidance (patterns), canonical examples (reference implementations), or detailed operational docs.

Boundary rules are defined explicitly in [“Knowledge management boundaries”](research/architecture-decision-records-deep-research.md:376).

---

## 2) Right-sized ADR operating model (anti over-tooling)
The most common failure modes are:
- **Over-tooling** (a platform is bought before the practice works)
- **Under-capturing** (decisions are made in chats/meetings and never recorded)

### 2.1 Decision framework: repo markdown vs index UI vs dedicated platform
Use this **right-sizing matrix** (start simple; scale only when pain is proven).

| Option | When it’s enough | What you gain | What it costs/risks |
|---|---|---|---|
| **A. Markdown ADRs in repo (docs-as-code)** | One program repo or a few repos; cross-agency visibility is manageable via search; decisions mostly public/internal | Lowest friction; PR-based review; audit trail “for free” | Discoverability across many repos can degrade; classification/redaction needs discipline |
| **B. Markdown ADRs + generated index/search UI** (static site + metadata extraction) | Many teams/vendors; need cross-repo discoverability but can keep ADRs in Git | Better findability; consistent metadata; basic dashboards | Build/maintain extraction + publishing; need conventions |
| **C. Dedicated ADR platform** | You require strict RBAC on decision content; complex cross-portfolio reporting; heavy decision graph dependency management; regulators/auditors require structured exports | Central view, RBAC, reporting, workflow automation | Procurement + admin overhead; risks vendor lock-in; teams stop writing unless UX is excellent |

**Rule for a city twin program:** default to **A**, move to **B** when search/index pain becomes real, consider **C** only when mandated by RBAC/reporting needs you cannot meet with Git + static publishing.

### 2.2 Start simple, scale if needed (practical path)
- Phase 1 (weeks): **A** in the program repo with a template + PR checks.
- Phase 2 (1–2 quarters): **B** by extracting front-matter into an index and publishing an internal static site.
- Phase 3 (only if justified): evaluate **C** with explicit requirements: RBAC, export format, API access, portability, and cost.

### 2.3 Minimum viable ADR setup (MVA)
Folder + conventions (example; adjust to your repo structure):
- `docs/adr/` — ADR markdown files
- `docs/adr/README.md` — how to write/approve/supersede
- `docs/adr/templates/` — template variants
- `docs/adr/index.md` — generated or curated index

Naming:
- `ADR-####-short-title.md` (e.g., `ADR-0042-canonical-crs.md`)

Review flow (lightweight-by-default):
- Draft via PR.
- Required reviewers depend on category (see RACI in [Section 4](research/architecture-decision-records-deep-research.md:168)).
- Merge = “accepted” **only when acceptance criteria are present**.

Scale-up options (only if needed):
- Publish ADRs to an internal portal (static site) + tag filters.
- Add a simple “decision graph” by parsing `supersedes/relates-to/depends-on` links.
- Add sensitive decision handling (restricted repo or restricted folder + redacted public summary).

---

## 3) ADR trigger conditions (what is ADR-worthy in city twin work)
The purpose of triggers is to keep ADRs:
- **Complete** (important decisions captured)
- **Lean** (routine work stays in PRs/design docs)

### 3.1 City-twin ADR trigger checklist
Write an ADR when **any** of the following is true.

#### A) Identity, entity resolution, and linkage
- [ ] Choosing an identity model (resident identity vs device identity vs place identity).
- [ ] Entity resolution strategy (probabilistic vs deterministic; confidence thresholds).
- [ ] Cross-agency linkage rules (what can be linked, by whom, under what legal basis).

Example: “Link parking permit holders to vehicle telemetry for enforcement” → ADR required (privacy + consent + access tiers).

#### B) Canonical geospatial representation
- [ ] Canonical CRS selection (including vertical datum if relevant).
- [ ] Tiling/index strategy (vector tiles, quadkeys, H3/hex, grid) and precision rules.
- [ ] Linear referencing / network referencing choices (roads, utilities) and conflation strategy.

Example: “Standardize on EPSG:4326 for APIs, store EPSG:3857 tiles for visualization” → ADR required.

#### C) Data contracts + schema evolution
- [ ] Introducing/changing canonical entity schemas or shared event topics.
- [ ] Breaking change policy (compat window, versioning scheme, deprecation timelines).
- [ ] Data contract ownership and compatibility testing approach.

Example: “Move from per-sensor payloads to standardized Observations schema” → ADR required.

#### D) Privacy posture (operational rules)
- [ ] Aggregation thresholds (k-anonymity / minimum group size) for publishing.
- [ ] Retention policy, archival tiering, deletion SLAs.
- [ ] Redaction rules and secondary-use constraints.
- [ ] Access tier model (public/internal/restricted; purpose limitation).

Example: “Publish footfall heatmaps only above N=30 and 15-min bins” → ADR required.

#### E) Interoperability standards selection and profiles
- [ ] Selecting standards (e.g., OGC SensorThings, NGSI-LD, CityGML/CityJSON, IFC, GTFS-RT) and **defining your profile**.
- [ ] Conformance claims and test suite strategy.
- [ ] Cross-vendor API contract finalization.

Example: “Adopt NGSI-LD with program-wide JSON-LD context governance” → ADR required.

#### F) Model governance (analytics/AI) decisions
- [ ] Validation gates (pre-release checks, drift thresholds, rollback triggers).
- [ ] Human-in-the-loop (HITL) tiers: when a human must approve outputs.
- [ ] Bias/equity checks and acceptable error bounds by use case.

Example: “Automated anomaly detection can trigger operator alerts without human approval” → ADR required (risk + ops).

#### G) Vendor/procurement lock-in and exit strategies
- [ ] Decisions that constrain portability (proprietary formats, closed APIs, managed service dependencies).
- [ ] Exit strategy: data export format, migration tools, escrow, transition assistance.

Example: “Use vendor X’s proprietary digital twin scene format” → ADR required with explicit exit terms.

### 3.2 Not ADR-worthy (use normal artifacts)
- Routine refactors without architecture impact.
- Minor UI changes.
- Non-breaking dependency upgrades.

### 3.3 “Sometimes ADR-worthy” (use judgment)
- Switching orchestration tools where interfaces are unchanged.
- Changing observability implementation but not SLOs or operational responsibilities.
- New internal library adoption.

Rule of thumb: if it changes **interfaces, guarantees, risk posture, or procurement obligations**, it’s ADR-worthy.

---

## 4) Decision rights, overrides, and emergency capture
City programs need explicit decision authority so vendors and agencies know what is binding.

### 4.1 Decision authority model (RACI)
Minimum roles:
- **ADR Author**: drafts; ensures options and consequences are captured.
- **Decision Owner**: accountable for outcome and adoption.
- **Reviewers**: security, privacy, data governance, operations/SRE, domain leads.
- **Architecture Review Board (ARB)**: approves high-impact decisions; arbitrates conflicts.
- **Procurement/Legal liaison (PMO)**: ensures vendor obligations and data-sharing terms align.

RACI (adapt as needed):

| Activity | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Identify ADR-worthy decision | Tech lead / architect | Delivery lead / product owner | Security/Privacy/Ops | ARB |
| Draft ADR | ADR Author | Decision Owner | Domain leads + vendors (if impacted) | Program |
| Review (standard) | Required reviewers by category | Decision Owner | Procurement/Legal | Stakeholders |
| Approve (major decisions) | ARB | ARB chair / city CTO delegate | Security/Privacy/Data Gov | All delivery streams |
| Approve (minor decisions) | Delegated approver | Decision Owner | ARB optional | Stakeholders |
| Override an accepted ADR | ARB chair (or named delegate) | City CTO delegate | Security/Privacy/Legal | Program |
| Accept risk/exception | Security risk owner | CISO delegate | Legal/Privacy | ARB |
| Supersede/deprecate | Decision Owner | ARB (for major) | Affected owners | Program |

### 4.2 Who can accept/override decisions (explicit rules)
- **Acceptance**:
  - Minor/local ADRs: delegated approver (document the delegation).
  - Program-wide standards, identity/privacy models, interoperability profiles, retention policies: **ARB approval required**.
- **Override**:
  - Only ARB chair (or named delegate) can override, and only via:
    - a superseding ADR, or
    - an Emergency ADR (temporary) that must be ratified.

### 4.3 Emergency ADR workflow (incident-driven)
Use when an outage/security incident forces an architectural choice faster than normal governance.

**Emergency ADR rules**
- Timebox: create within **48 hours** of the incident.
- Status: `accepted-temporary`.
- Mandatory fields: incident/ticket link, scope of impact, security/privacy notes, rollback plan, expiry.

**Ratification + expiry**
- Expiry default: **30–60 days**.
- Must schedule a ratification review within the next **two ARB cadences**.
- Outcomes:
  - ratify as permanent (`accepted`) with full operational consequences, or
  - supersede/revert with a durable design.

**Mandatory retrospective**
- Attach a “decision regret” note: what we wish we had decided earlier, and what governance/monitoring changes follow.

---

## 5) City-twin ADR taxonomy (discoverable, program-aligned)
Use categories that match the actual decision surface area of a city twin.

| Category | Definition (what belongs here) | Examples |
|---|---|---|
| **GEO-REP (Geospatial representation)** | Canonical spatial/temporal representation and precision rules | CRS, tiling/index, linear referencing, conflation |
| **SENS-STD (Sensors & telemetry standards)** | Sensor models, observation semantics, units, calibration metadata | OGC SensorThings profile, event schemas |
| **ID-CONSENT (Identity + consent model)** | Identity linkage, consent/legal basis, access tiers | pseudonymization, linkage rules |
| **RET-REDACT (Retention + redaction rules)** | Retention schedules, deletion SLAs, redaction/aggregation | aggregation thresholds, retention tiers |
| **INTEROP-PROFILE (Interoperability profile)** | Standards selection and local profiles + conformance testing | NGSI-LD context governance, API profiles |
| **DATA-CONTRACT (Data contracts & schema evolution)** | Canonical entity/event contracts and versioning policies | breaking change policy, compatibility window |
| **SEC-BOUNDARY (Security boundaries & trust)** | AuthN/Z, segmentation, KMS, admin model | privileged access, tenant isolation |
| **RISK-EXCEPT (Risk acceptance / security exceptions)** | Time-boxed exceptions, compensating controls, approvals | temporary bypass, compensating monitoring |
| **MODEL-GOV (Model governance & HITL)** | Validation gates, HITL tiers, drift monitoring | rollback triggers, equity checks |
| **VENDOR-EXIT (Vendor lock-in & exit strategy)** | Portability constraints, exit plans, procurement clauses | export formats, transition support |
| **OPS-RELIABILITY (Ops, SLOs, runbooks)** | Operational guarantees and response patterns | degraded mode, DR, SLOs |

---

## 6) Templates (city-twin-specific) + required “operational consequence” sections
A usable ADR forces operational reality: monitoring, migration, rollback, cost, and compliance impacts.

### 6.1 Required sections (for all templates)
Minimum sections (do not omit; use “N/A + justification”):
- **Decision at a glance** (1–5 bullets)
- **Context & problem statement**
- **Decision**
- **Options considered** (at least one credible alternative)
- **Operational consequences**
  - Monitoring/SLIs affected
  - Runbook changes
  - Degraded-mode behavior
- **Migration + rollback**
- **Cost/FinOps impact** (who pays, expected growth, cost controls)
- **Privacy impact** (DPIA link or “not required because …”)
- **Security impact** (threat considerations; risk acceptance link if needed)
- **Procurement/vendor impact** (SoW/SLA/DPA/data sharing links)
- **Acceptance criteria** (verifiable)
- **References/evidence** (PRs, tickets, test results)

### 6.2 Template variant A — Data Contract ADR (`DATA-CONTRACT`)
Use when defining/changing canonical schemas, event topics, or contract tests.

```markdown
---
adr_id: ADR-####
title: Data contract for <entity/event>
status: proposed
date: YYYY-MM-DD
category: [DATA-CONTRACT]
decision_owner: <name/role>
security_classification: internal
affected_systems: [<systems>]
affected_vendors: [<vendors>]
related_artifacts:
  - jira: <ticket>
  - pr: <link>
  - contract_catalog: <link/id>
---

## Decision at a glance
- Decision: …
- Impacted consumers/producers: …
- Compatibility window: …
- Rollback trigger: …

## Context
…

## Decision
…

## Schema (summary)
- Entity/event name: …
- Key fields: …
- Units/CRS/timezone conventions: …

## Versioning + evolution policy
- Versioning scheme: …
- Breaking change definition: …
- Deprecation timeline: …
- Contract tests: …

## Operational consequences
- Monitoring: …
- Runbooks: …

## Migration + rollback
…

## Cost/FinOps impact
…

## Privacy impact
- DPIA required? yes/no + justification
- Retention/redaction implications: …

## Security impact
…

## Procurement/vendor impact
- SoW/SLA implications: …

## Acceptance criteria
- [ ] Compatibility tests pass for all registered consumers
- [ ] Backfill/migration completed with <error rate>
- [ ] Observability dashboards updated

## References
…
```

### 6.3 Template variant B — Security Exception / Risk Acceptance ADR (`RISK-EXCEPT`)
Use for time-boxed exceptions, compensating controls, and formal risk acceptance.

```markdown
---
adr_id: ADR-####
title: Security exception — <what is being excepted>
status: accepted-temporary
date: YYYY-MM-DD
category: [RISK-EXCEPT]
decision_owner: <security risk owner>
security_classification: restricted
expiry_date: YYYY-MM-DD
related_artifacts:
  - security_review: <link>
  - risk_register: <id>
  - incident: <id/link if applicable>
  - compensating_controls: <link>
---

## Decision at a glance
- Exception: …
- Why now: …
- Compensating controls: …
- Expiry: …

## Context
…

## Decision
…

## Compensating controls (mandatory)
- Control: … / Owner: … / Evidence: …

## Monitoring + alerts (mandatory)
- Signals: …
- Alert thresholds: …

## Migration to compliance (mandatory)
- Plan: …
- Date: …

## Procurement/vendor impact
- Contractual obligations impacted: …

## Acceptance criteria
- [ ] Controls implemented and verified
- [ ] Expiry review scheduled

## References
…
```

### 6.4 Template variant C — Model Governance ADR (`MODEL-GOV`)
Use for analytics/AI deployment rules, validation gates, and HITL tiers.

```markdown
---
adr_id: ADR-####
title: Model governance for <model/use case>
status: proposed
date: YYYY-MM-DD
category: [MODEL-GOV]
decision_owner: <role>
security_classification: internal
related_artifacts:
  - model_card: <link>
  - validation_protocol: <link>
  - human_in_loop: <link>
---

## Decision at a glance
- Deployment class: (advisory / automated / safety-critical)
- HITL tier: …
- Rollback trigger: …

## Context
…

## Decision
…

## Validation gates
- Offline metrics: …
- Online monitoring/drift: …
- Equity/bias checks: …

## HITL + escalation
- When humans approve: …
- Escalation path: …

## Operational consequences
- Monitoring: …
- On-call expectations: …

## Privacy impact
…

## Security impact
…

## Acceptance criteria
- [ ] Monitoring dashboards and alerts exist
- [ ] Rollback is tested

## References
…
```

---

## 7) Link ADRs to compliance + procurement (audit-ready by design)
ADRs should directly support procurement governance and audit readiness.

### 7.1 Compliance linkages (privacy/security)
For any decision affecting personal data, surveillance capability, or access control, ADRs must link to:
- **DPIA / PIA** (data protection/privacy impact assessment) outcome and mitigation actions.
- **Security review** outcomes.
- **Risk acceptance / exception** records when residual risk remains.

Practical rule:
- If the ADR changes **data purpose, linkage, retention, disclosure**, or **access tiers**, then it must include a DPIA link or “not required because …”.

### 7.2 Procurement linkages (SoWs, SLAs, DPAs, data-sharing agreements)
When vendors are impacted, ADRs must connect to procurement artifacts:
- **SoW** clauses: standards conformance, interface obligations, test evidence, delivery gates.
- **SLA** obligations: availability, response times, support boundaries.
- **DPA / data processing addendum** (or equivalent) and **data-sharing agreements**: purpose limitation, onward sharing, retention, breach notification.
- **Exit strategy**: data export formats, migration tools, transition support, escrow (where applicable).

### 7.3 Audit evidence: required metadata
To support audits without heroics, ensure every ADR captures:
- `decision_owner`, `decision_makers`, and approval record (ARB minutes link or PR approval).
- `security_classification`.
- Links to DPIA/security review/risk acceptance where applicable.
- Implementation evidence (PRs/releases/tests) once accepted.
- If superseded: explicit `superseded_by` link and reason.

---

## 8) Integrations (GitHub/GitLab + CI/CD + Jira/ADO + Confluence/SharePoint)
Keep integrations lightweight and enforcement-oriented.

### 8.1 Pull requests
- PR template checkbox: “ADR required?” with link to [trigger checklist](research/architecture-decision-records-deep-research.md:116).
- If ADR required: PR must include ADR link or create one in same PR.

### 8.2 CI/CD checks (docs-as-code-first)
Minimal checks that deliver outsized value:
- ADR metadata lint (YAML front matter presence + allowed statuses/categories).
- Broken link checking (supersession links, DPIA links, SoW references).
- Index generation verification.

### 8.3 Jira/ADO
- Add a field “Governing ADR” on epics/features that change contracts/standards.
- Automate reminders: if category is `RISK-EXCEPT`, create an expiry task.

### 8.4 Confluence/SharePoint
Use for:
- Human-friendly browsing (published static ADR index or curated views).
- Cross-agency communications.

Do not move the authoritative ADR content out of Git unless tooling preference changes.

---

## 9) Quality gates and acceptance criteria (harder to game)
Avoid “checkbox completeness” as the main signal; it incentivizes verbose low-value ADRs.

### 9.1 ADR-level quality gate (merge criteria)
An ADR is mergeable when:
- Context states constraints and decision pressure.
- Options include at least one realistic alternative.
- Operational consequences include **monitoring + migration/rollback**.
- Compliance/procurement impacts are addressed (or justified N/A).
- Acceptance criteria are verifiable.

### 9.2 Program-level metrics (higher signal)
Use metrics that correlate with operational outcomes.

| Metric | Why it’s hard to game | How to measure |
|---|---|---|
| **Reversal rate + reasons** | Forces root cause classification, not volume | % ADRs superseded within 6–12 months + reason codes (wrong assumptions / vendor constraint / cost / security incident / interoperability failure) |
| **Decision regret postmortems** | Requires narrative learning + corrective actions | Quarterly sample of “top impact” ADRs; record regret + action items |
| **Adoption indicators** | Measures actual use in delivery | Count references to ADR IDs in PRs/design docs/runbooks; ARB attendance rate for relevant streams |
| **Time-to-find + time-to-decide** | Focuses on operational efficiency | Time to locate governing ADR during incident drills; time from trigger to accepted decision |
| **Operational impact accuracy** | Ties to reality, not prose | Compare predicted consequences to incidents/SLO violations; track mismatches |

Reason codes (recommended for reversals):
- `assumption_invalid`
- `vendor_constraint`
- `interop_breakdown`
- `privacy_or_legal_change`
- `security_incident`
- `cost_overrun`
- `ops_unworkable`

---

## 10) Knowledge management boundaries (interfaces to other artifacts)
ADRs are one node in a broader “docs-as-code” ecosystem.

### 10.1 Clear boundaries
- **ADRs**: record **a specific decision** + rationale + consequences.
- **Patterns**: reusable “how we do X” guidance (e.g., streaming ingestion pattern).
- **Reference implementations**: runnable examples demonstrating the pattern.
- **Data contract catalog**: canonical schemas, versions, owners, compatibility tests.

### 10.2 Linking conventions (simple and enforceable)
- ADRs link outward using stable IDs:
  - `pattern: PAT-####`
  - `reference_impl: RI-####`
  - `data_contract: DC-####`
- Patterns/reference implementations must link back to the ADR(s) that authorized them.

Practical rule: **never duplicate** reusable guidance inside an ADR; link to the canonical artifact.

---

## 11) Runbooks (operationalization)

### 11.1 Runbook: “major schema change proposed”
Trigger: breaking change to canonical entity/data contract.

Checklist:
- [ ] Confirm ADR trigger (`DATA-CONTRACT`).
- [ ] Draft ADR includes migration plan + compatibility window.
- [ ] Identify affected systems, agencies, and vendors.
- [ ] Define rollout phases and rollback plan.
- [ ] Approval gate: data governance + ARB (if cross-program).
- [ ] Update contract catalog and conformance tests.

### 11.2 Runbook: “emergency architecture change during incident”
Trigger: outage/security incident requires immediate architectural modification.

Checklist:
- [ ] Implement emergency change with incident ticket reference.
- [ ] Create Emergency ADR within 48 hours (`accepted-temporary`).
- [ ] Set expiry date + ratification meeting.
- [ ] Mandatory retrospective + decision regret note.

### 11.3 Runbook: “ADR drift / outdated decision discovered”
Trigger: implementation diverges from ADR, or assumptions become invalid.

Checklist:
- [ ] Decide: update (clarification) vs supersede (new decision).
- [ ] If superseding: create new ADR; link both directions.
- [ ] Update patterns/reference implementations/data contracts that depend on it.
- [ ] Notify affected agencies/vendors.

---

## 12) Implementation roadmap + operating cadence

### 12.1 Roadmap
**0–3 months (foundation)**
- Adopt taxonomy + templates (Sections 5–6).
- Implement trigger checklist in PR templates.
- Define decision rights + emergency workflow (Section 4).
- Start minimal index (`docs/adr/index.md`) and enforce metadata lint.

**3–12 months (scale + automation)**
- Add metadata extraction to generate a searchable ADR index.
- Wire CI to require links for DPIA/security review where applicable.
- Implement expiry automation for `accepted-temporary` ADRs (tickets/reminders).
- Start quarterly reversal + decision-regret reviews.

**12–24 months (federation + dashboards)**
- Cross-agency federation: shared taxonomy, shared IDs, and cross-repo discovery.
- Dashboards for adoption/metrics (PR references, time-to-find drills, reversal reasons).
- Mature compliance exports (audit packs per domain/release).

### 12.2 Operating cadence (lightweight-by-default)
- Weekly/biweekly ARB: review proposed ADRs and ratify expiring emergency ADRs.
- Monthly: exception/expiry review (security + PMO).
- Quarterly: decision regret review + incident drill “time-to-find ADR”.

---

## 13) Risks & mitigations (detect + respond)
| Risk | How we detect | How we respond |
|---|---|---|
| Low adoption | few ADRs for trigger events; PRs without ADR references | enforce PR prompt; coaching; ARB sampling |
| Over-documentation | long cycle times; ADRs with low adoption | tighten triggers; require acceptance criteria + operational consequences |
| Gameable metrics | “complete” ADRs but no usage | shift to adoption + reversal/regret metrics; audit samples |
| Vendor drift | vendor implements different patterns | tie SoW conformance to ADR IDs; contract tests |
| Emergency decisions become permanent silently | expired `accepted-temporary` ADRs | automated expiry tasks; ARB ratification queue |

---

## 14) References
### 14.1 Workspace source
- [`kali-task-research.md`](kali-task-research.md:1) — Item 18: Architecture decision records

### 14.2 External references
- ADR Community — https://adr.github.io/
- AWS Prescriptive Guidance ADR process — https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html
