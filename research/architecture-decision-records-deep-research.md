# Architecture decision records (ADRs) — Deep research (decision-grade + operational)

## Executive summary
Architecture Decision Records (ADRs) are small, durable documents that capture **what was decided**, **why**, **who decided**, **what was considered**, and **what the operational consequences are**.

For a municipal “city twin” program—multi-team, multi-vendor, long-lived—ADRs are a core governance and delivery mechanism:
- They reduce **architectural drift** across years of procurement cycles and staff turnover.
- They create **audit-ready rationale** for privacy/security/interoperability choices.
- They make change safer by connecting deployments, schema changes, and operational runbooks to explicit decisions.

This revision strengthens implementation realism by:
- Right-sizing “docs-as-code” vs “ADR platform” choices.
- Defining **ADR-worthy triggers** for city twin work, with decision authority and emergency pathways.
- Adding a **city-twin-specific taxonomy** and templates.
- Connecting ADRs to **procurement/compliance** artifacts (SoW clauses, DPIAs, risk acceptance).
- Replacing gameable metrics with higher-signal measures and acceptance criteria.

Assumptions (defaults): public-sector procurement, multi-vendor delivery, limited staffing; advisory-first governance; options-not-prescriptions; high equity focus; balanced transparency.

---

## 1. Why this theme matters (and how it helps you run the twin)
A city digital twin is a long-lived system-of-systems: data contracts, geospatial representations, identity/consent models, and security postures evolve over time. Without decision records, the program becomes a collection of integrations held together by personal memory.

ADRs make the platform operable:
- **Safer operations**: incident response teams can see why a tradeoff exists (e.g., eventual consistency, buffering, edge processing) and what failure modes were accepted.
- **Controlled change**: you can pin changes to decisions, define rollout gates, and enforce review for high-impact modifications.
- **Cross-department alignment**: privacy, security, and data-sharing agreements become traceable to explicit, reviewable decisions.

---

## 2. Scope: ADR practice vs ADR tooling (avoid over-tooling)
### 2.1 Decision: when “docs-as-code” is enough
Use a simple Git-based approach when:
- ≤ 5–8 teams actively changing architecture
- You can enforce ADR templates via pull request (PR) checks
- Search needs are satisfied by repo search + a static docs site
- You don’t need cross-repo linking at scale

**Minimum viable ADR system (“docs-as-code”)**
- ADRs stored as markdown under a single folder (example: `docs/adr/`)
- A required template + lightweight CI check
- A generated index page (by status/category/tags)
- PR template includes an “ADR impacted?” prompt

### 2.2 Decision: when an ADR platform is warranted
Consider an ADR “platform” only when:
- Many vendors/repos need a single cross-program view
- Decision relationships matter (decision graph, dependencies, supersession)
- You need role-based access for sensitive decisions (security exceptions)
- You must produce audit reports across portfolios quickly

Right-sizing rule: **prove the practice first**, then scale tooling.

---

## 3. Stakeholders, decision authority, and RACI
### 3.1 Roles (minimum)
- **ADR Author**: writes draft, captures options and consequences
- **Decision Owner**: accountable for outcome; ensures implementation alignment
- **Reviewers**: security, privacy, data governance, operations, domain leads
- **Architecture Review Board (ARB)**: approves high-impact decisions
- **Procurement/Legal liaison**: ensures vendor obligations and data-sharing terms align

### 3.2 RACI for ADR lifecycle
| Activity | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Identify ADR-worthy decision | Tech lead / architect | Product/platform owner | Ops + security | ARB |
| Draft ADR | ADR Author | Decision Owner | Domain leads | Platform teams |
| Review | Reviewers | ARB (for major) | Procurement/legal | Stakeholders |
| Approve | ARB or delegated approver | ARB chair / CTO equivalent | Security/privacy | Teams |
| Implement + verify | Delivery team | Decision Owner | Ops | ARB |
| Supersede/deprecate | Decision Owner | ARB | Affected owners | Program |

### 3.3 Emergency / incident-driven decisions
For incidents (security events, major outages), you need a fast path:
- Create an **Emergency ADR** within 48 hours of the incident.
- Mark it `status: accepted-temporary` with an explicit expiry date.
- Require a follow-up ADR within 30–60 days to either:
  - confirm as permanent, or
  - revert/supersede with a durable design.

---

## 4. “ADR-worthy” trigger conditions for city twins
The most common failure is teams not knowing what deserves an ADR. Use the triggers below.

### 4.1 Always ADR-worthy (must write)
- **Canonical data contracts** and breaking schema changes
- **Identity/consent model** (citizen identity linkage, pseudonymization approach)
- **Privacy posture** (retention, redaction, secondary-use constraints)
- **Security boundaries** (network segmentation, key management, admin access model)
- **Interoperability profile** choices (OGC/NGSI-LD profiles, conformance claims)
- **Geospatial representation** (tiling vs hex; CRS standards; accuracy targets)
- **Eventing/streaming design** for core telemetry (ordering, dedupe, replay)
- **Model governance** decisions (versioning, validation gates, human-in-loop)
- **Vendor selection constraints** that lock in interfaces or formats

### 4.2 Sometimes ADR-worthy (use judgment)
- Switching ETL orchestration approach
- Introducing a new “golden record” entity
- Changes to observability standards and SLOs

### 4.3 Not ADR-worthy (use normal docs)
- Routine refactors with no architecture impact
- Minor UI changes
- Non-breaking dependency bumps

---

## 5. City-twin ADR taxonomy (domain-specific)
Use a taxonomy that maps to city twin reality so decisions are discoverable.

| Category | Examples |
|---|---|
| **Data contracts & semantics** | canonical entity definitions, versioning policy, schema evolution rules |
| **Geospatial & time** | spatial index choice, CRS, tiling strategy, temporal aggregation standards |
| **Interoperability** | OGC SensorThings profile, NGSI-LD context strategy, API conformance |
| **Security & identity** | authn/authz, tenant separation, privileged access model |
| **Privacy & ethics** | DPIA outcomes, de-identification, retention/redaction, public transparency posture |
| **Integration patterns** | batch vs streaming, idempotency, replay, edge vs cloud partitioning |
| **Reliability & ops** | SLOs, degraded mode behavior, DR strategy |
| **Procurement & vendor management** | data-sharing terms, exit strategy, escrow, SLA obligations |
| **Analytics/model governance** | validation requirements, release gates, bias/equity checks |

---

## 6. ADR template (operationally actionable)
A good ADR is implementable and testable.

### 6.1 Required fields (minimum)
- `adr_id` (unique)
- `title`
- `status` (see 6.3)
- `date`
- `decision_owner`
- `decision_makers`
- `context`
- `decision`
- `options_considered` (with tradeoffs)
- `consequences` (positive/negative)
- `operational_impact` (runbooks/SLOs/degraded mode)
- `security_privacy_notes` (or explicit “N/A” with justification)
- `procurement_compliance_links` (or “N/A”)
- `implementation_plan` (phased)
- `acceptance_criteria`
- `references` (PRs, issues, contracts)

### 6.2 “Decision at a glance” block (for non-technical stakeholders)
Include a short block near the top:
- What is the decision?
- What changes for residents/operators?
- What risks are accepted?
- What would make us reverse this decision?

### 6.3 Status lifecycle (expanded)
- `proposed`
- `accepted`
- `accepted-temporary` (emergency)
- `rejected`
- `deprecated`
- `superseded` (must link to superseding ADR)

---

## 7. Integration into decision moments (how it actually gets used)
### 7.1 Workflow integration points
- **PR template**: “Does this change require an ADR?” (link to triggers)
- **Architecture review gate**: ARB agenda is driven by proposed ADRs
- **CI checks**:
  - validate ADR front-matter fields
  - ensure `superseded` includes a link
  - require `acceptance_criteria` for `accepted`
- **Release governance**: major releases include an ADR delta summary

### 7.2 Minimal “ADR checks” for CI/CD (docs-as-code)
- Schema lint for ADR metadata (YAML front matter or markdown headings)
- Broken link checking for referenced ADRs/PRs
- Index generation verification

### 7.3 Publishing and access
Balance transparency with security:
- Public or cross-department ADRs should have a redacted summary version when needed.
- Security exception ADRs may be restricted; still require an internal audit trail.

---

## 8. Data requirements: ADR metadata and decision graph (practical)
### 8.1 Canonical entity: `ADR`
Minimum contract fields (store in markdown + extractable metadata):
- `adr_id`, `title`, `status`, `date`
- `category[]`, `tags[]`
- `system_scope` (which subsystems)
- `affected_contracts[]` (data contract IDs)
- `security_classification` (public/internal/restricted)
- `links` (PRs, issues, SoWs, DPIAs)

### 8.2 Canonical entity: `DecisionLink`
To avoid overbuilding a graph database early, start as extracted edges:
- `from_adr_id`, `to_adr_id`, `relationship` (depends-on/supersedes/relates-to)

---

## 9. Validation & acceptance criteria (make it “good enough to use”)
### 9.1 Acceptance criteria for the ADR practice (program-level)
- ≥ 80% of **ADR-worthy triggers** produce an ADR within 5 business days.
- ≥ 95% of ADRs have a valid owner and status.
- ≥ 90% of accepted ADRs include explicit operational impact + acceptance criteria.
- New joiners can answer “why did we choose X?” within 10 minutes using ADR search.

### 9.2 Acceptance criteria for each ADR (quality gate)
An ADR is “acceptable” when:
- Context explains the decision pressure and constraints.
- Options include at least one credible alternative.
- Consequences include operational impacts and failure modes.
- Reversal conditions are explicit.
- Links exist to implementation evidence (PRs/issues) after acceptance.

### 9.3 Backtesting / historical replay (for ADR quality)
Quarterly, sample a set of incidents and major changes:
- Can responders find the governing ADR in < 5 minutes?
- Did the ADR predict the failure mode?
- Was a reversal/supersession captured?

---

## 10. Operational SLIs/SLOs and metrics (avoid gameable metrics)
### 10.1 Operational SLIs
- **Decision capture latency**: decision date → ADR created
- **Review cycle time**: draft → accepted
- **Adoption signal**: ADR referenced in PRs/design docs for affected areas
- **Supersession hygiene**: percent of superseded ADRs correctly linked

### 10.2 Suggested SLOs
| Metric | SLO |
|---|---|
| ADR created after decision | ≤ 5 business days |
| Review completed | ≤ 10 business days |
| Superseded ADRs properly linked | ≥ 98% |
| ADR references in major PRs | ≥ 70% of major changes |

### 10.3 Higher-signal quality measures
- **Decision reversal rate with root causes** (good programs reverse sometimes; measure why)
- **Decision regret postmortems** (explicit learning loop)
- **Operational impact accuracy** (did reality match predicted consequences?)

---

## 11. Linking ADRs to procurement and compliance (public-sector reality)
### 11.1 Procurement linkages
For vendor-involved decisions, ADRs should reference:
- SoW requirements that depend on the decision (APIs, standards conformance)
- Exit/portability clauses (data export formats, IP rights)
- Vendor SLAs and audit obligations

### 11.2 Compliance linkages
ADRs should link to (or summarize outcomes of):
- DPIA / privacy impact assessments
- Security risk acceptance and exceptions
- Data-sharing agreements and retention schedules

### 11.3 Exception handling
When teams request exceptions (e.g., storage in a non-standard location):
- Create an ADR with `status: accepted-temporary`.
- Include expiry date and remediation plan.
- Track exceptions as a register driven by ADRs.

---

## 12. Runbooks (operationalization)
### 12.1 Runbook: “major schema change proposed”
Trigger: breaking change to canonical entity/data contract.

Checklist:
- [ ] Confirm ADR-worthy trigger
- [ ] Draft ADR includes migration plan + compatibility window
- [ ] Identify affected systems and vendors
- [ ] Define rollout phases and rollback plan
- [ ] Approval gate: ARB + data governance
- [ ] Release notes prepared (internal + external)

### 12.2 Runbook: “emergency architecture change during incident”
Trigger: outage/security incident requires immediate architectural modification.

Checklist:
- [ ] Implement emergency fix with incident ticket reference
- [ ] Create Emergency ADR within 48 hours
- [ ] Set expiry date + follow-up owner
- [ ] Post-incident review: decide to revert or formalize

### 12.3 Runbook: “ADR drift / outdated decision discovered”
Trigger: implementation diverges from ADR, or ADR assumptions invalid.

Checklist:
- [ ] Determine whether to update or supersede
- [ ] If superseding: create new ADR, link both ways
- [ ] Communicate to affected teams and vendors
- [ ] Update operational docs/runbooks impacted

---

## 13. Implementation roadmap (0–3, 3–12, 12–24 months)
### 0–3 months (foundation, low staffing)
- Adopt standard ADR template and taxonomy (Sections 5–6)
- Define triggers and ARB decision rights (Section 4)
- Implement docs-as-code repo structure and CI checks
- Pilot with 1–2 high-change areas (data contracts + security boundary)

### 3–12 months (scale across vendors)
- Roll out ADR practice to all delivery streams
- Add extraction for metadata + index site
- Introduce quarterly “decision regret” review
- Implement procurement/compliance link fields and exception register

### 12–24 months (maturity)
- If justified: add cross-repo ADR discovery and decision graph views
- Integrate ADRs into stage gates and release governance
- Formalize external transparency posture (public summaries, redactions)

Dependencies:
- Clear ARB charter and meeting cadence
- A shared contract catalog (even if minimal)
- Program-level security/privacy governance processes

---

## 14. Risks & mitigations (with detect + respond)
| Risk | How we detect | How we respond |
|---|---|---|
| Low adoption | few ADRs for trigger events | enforce PR prompts; ARB checks; coaching |
| Over-documentation | cycle time increases; teams complain | tighten trigger list; lightweight templates |
| Gameable metrics | “complete” ADRs with low usefulness | shift to adoption + regret metrics; sampling audits |
| Vendor drift | vendor implements different patterns | tie SoW conformance to ADR references; enforce interface tests |
| Emergency decisions become permanent silently | accepted-temporary with no follow-up | expiry alerts; ARB review queue |

---

## 15. References
### 15.1 Workspace source
- [`kali-task-research.md`](../kali-task-research.md:1) — Item 18: Architecture decision records

### 15.2 External references (retrieved via Firecrawl MCP)
- ADR Community — https://adr.github.io/
- AWS Prescriptive Guidance ADR process — https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html
