# Benefits Realization Tracking (Deep Research)

## Purpose
Benefits Realization Tracking ensures the **City Digital Twin program produces measurable public value** and that claims about value are **auditable, reproducible, and hard to game**.

This document defines:
- A **tiered benefits-claim system** with publication thresholds.
- A **benefit lineage** model (claim → intervention → decisions/ADRs → datasets/models/pipelines → evidence).
- **Causal rigor** standards (baselines, counterfactuals, pre-analysis plans, sensitivity checks).
- Governance to prevent Goodhart’s Law failures and to separate **internal operational learning** from **public reporting**.

## Scope
Applies to benefits claimed from:
- Digital twin products (dashboards, simulations, scenario tools, 3D/4D/5D models)
- Data platforms and integrations that feed the twin
- Operational interventions enabled by the twin (signal timing changes, maintenance prioritization, inspections, dispatch routing)

## Operating principles
1. **Claims are tiered**: early signals are not presented as impact.
2. **Evidence is linked**: every metric in a report must be traceable to raw data + processing + decision.
3. **Rigor matches stakes**: higher spend / higher visibility requires stronger methods.
4. **Net value**: benefits are evaluated **against total cost** (CapEx, OpEx, change management, risk).
5. **Anti-gaming by design**: metrics are paired with guardrails, audits, and independent validation.

---

## Definitions
- **Benefit**: a measurable improvement attributable (to some degree) to program activities, expressed in outcome terms (service levels, risk reduction, cost avoidance, emissions, equity).
- **Output**: a delivered artifact (dataset, model, dashboard). Outputs are not benefits.
- **Outcome**: real-world change in performance or conditions.
- **Claim**: a formal statement that a benefit occurred, at a defined confidence tier.
- **Baseline**: the “no-change” reference trajectory.
- **Counterfactual**: what would have happened without the intervention.

---

## Benefits claim tiers (publication and decision thresholds)
All benefits must be recorded as claims with one of the tiers below.

### Tier 0 — Activity / Delivery
**What it is**: work completed (datasets onboarded, model deployed, training delivered).
- Acceptable evidence: completion checklists, pipeline run logs, change tickets.
- Not allowed: public statements implying impact.

### Tier 1 — Operational signal (plausible association)
**What it is**: observed metric movement aligned with expectations, without a defensible counterfactual.
- Requirements:
  - Metric definition + owner + data lineage
  - Baseline description (historical average or forecast)
  - Confounder register (known external drivers)
- Allowed messaging: “early signal / monitoring indicator”.

### Tier 2 — Supported contribution (quasi-experimental)
**What it is**: evidence supports that the intervention contributed meaningfully.
- Requirements (minimum):
  - Documented intervention timing and scope
  - Baseline and comparison group or pre/post with adjustments
  - Sensitivity checks and robustness narrative
  - Peer review by independent analytics reviewer
- Examples of methods:
  - Difference-in-differences, interrupted time series, matched controls

### Tier 3 — High-confidence impact (causal, audit-ready)
**What it is**: strong causal attribution suitable for high-stakes decisions and public accountability.
- Requirements (minimum):
  - Pre-analysis plan (PAP) approved before outcome data is inspected
  - Strong counterfactual strategy (RCT where feasible; otherwise best-available quasi-experimental)
  - Full reproducibility package (code, data snapshots, parameterization, run logs)
  - Independent validation and sign-off

### Tier selection rules
- Tier is set at claim creation and may only **increase** with additional evidence.
- High-visibility claims (press releases, budget submissions) require **Tier 2+**, and material fiscal claims require **Tier 3**.

---

## Benefit lifecycle
1. **Identify** benefit hypothesis (who benefits, mechanism, expected magnitude, risks).
2. **Define** metrics + baselines + guardrails.
3. **Register** a claim candidate with tier target and evaluation plan.
4. **Collect & validate** data with lineage.
5. **Analyze** per the evaluation plan.
6. **Review & approve** per governance.
7. **Report** internally and/or publicly with tier-appropriate language.
8. **Retire** claim (benefit achieved, invalidated, or superseded).

---

## City digital twin benefit taxonomy (minimum set)
Benefits should be tagged to support portfolio roll-up without misleading aggregation.

### Service performance
- Travel time reliability, on-time performance
- Incident response time, clearance time
- Maintenance backlog reduction

### Cost and productivity
- Avoided site visits / optimized inspections
- Reduced rework in capital projects
- Lower energy consumption / operational cost

### Risk and resilience
- Flood risk exposure reduction
- Critical asset failure probability reduction
- Improved recovery time objectives for key services

### Environmental outcomes
- Emissions reductions, air quality proxies
- Noise reduction
- Modal shift indicators

### Equity and inclusion
- Distributional impacts by neighborhood / demographic proxy (where lawful)
- Access improvements (time to services, barrier reduction)

### Governance and compliance
- Faster permitting with maintained quality
- Reduced audit findings
- Improved data quality / fewer critical incidents

---

## Evidence & benefit lineage (audit-ready model)
Every claim must have a **lineage chain** linking from decision to data to analysis.

### Lineage chain
- Claim ID → benefit statement → intervention(s) → decision record(s) → system changes → data sources → transformations → metrics → analysis runs → reviewer approvals.

### Required evidence artifacts (per claim)
- **Claim record** (metadata)
- **Metric specification** (definition, inclusion/exclusion rules, units)
- **Data contract references** (source systems, schema versions)
- **Provenance** (pipeline run IDs, hashes, timestamps)
- **Analysis package** (notebook/report, code, parameter files)
- **Review log** (who reviewed, what was challenged, what changed)

### Claim record template (minimum fields)
| Field | Description |
|---|---|
| Claim ID | Unique ID (e.g., BR-2026-001) |
| Tier | 0–3 |
| Benefit statement | Plain-language statement |
| Owner | Accountable lead |
| Reviewers | Independent reviewers |
| Intervention | What changed, where, when |
| Affected population | Who/what is impacted |
| Primary metric(s) | Outcome metrics |
| Guardrail metrics | Anti-gaming / safety metrics |
| Baseline method | Historical/forecast/control |
| Counterfactual method | RCT / DiD / ITS / matched controls / none |
| Data lineage | Links to datasets and pipeline runs |
| Cost basis | TCO inputs used |
| Uncertainty | CI/ranges, sensitivity summary |
| Approval status | Draft/Reviewed/Approved/Published |
| Publication level | Internal only / Public summary / Public detailed |

---

## Causal rigor requirements (baseline, counterfactual, uncertainty)
### Baselines
- Define the baseline period and justify representativeness.
- Address seasonality and policy changes.
- Prefer baselines that can be reproduced from archived data snapshots.

### Counterfactual strategies
Choose the strongest feasible approach:
1. **Randomized / phased rollout** (best)
2. **Matched controls** (comparable areas/assets)
3. **Difference-in-differences**
4. **Interrupted time series** with covariates
5. **Before/after** (Tier 1 only)

### Pre-analysis plan (PAP)
Required for Tier 3 and recommended for Tier 2.
PAP must include:
- Primary/secondary outcomes, hypothesis direction
- Data sources and inclusion criteria
- Model specification and covariates
- Multiple testing controls (if many metrics)
- Stopping rules and handling of missing data

### Uncertainty and sensitivity
- Report ranges (confidence intervals where possible).
- Run sensitivity checks for key assumptions (baseline window, control selection, outliers).

---

## Anti-gaming controls (Goodhart-resistant design)
### Metric design patterns
- Pair each primary metric with at least one **guardrail** (e.g., faster response time paired with safety outcomes).
- Use **balanced scorecards** for multi-objective systems.
- Avoid single-metric incentives for teams/vendors.

### Operational controls
- **Independent analytics review** for Tier 2–3.
- **Data quality gates**: lineage completeness, anomaly checks, schema drift monitoring.
- **Audit sampling**: random claim audits each quarter.
- **Perverse incentive tests**: explicit review of how a metric could be manipulated.

### Vendor and partner anti-gaming
- Contractual requirement to provide evidence artifacts and raw logs for audits.
- Clear penalties for metric manipulation and incentives for transparent reporting.

---

## Governance: roles, cadence, and approvals
### Roles (minimum)
- **Benefit Owner (Accountable)**: business/agency lead responsible for benefit definition and operational change.
- **Analytics Lead (Responsible)**: evaluation design and analysis execution.
- **Data Steward (Responsible)**: data contracts, lineage, and quality.
- **Independent Reviewer (Consulted)**: challenge function (internal audit, PMO analytics QA, or external evaluator).
- **Governance Board (Approver)**: approves Tier 2–3 claims and public reporting.

### Cadence
- Monthly: portfolio review (Tier 0–1), data quality and leading indicators.
- Quarterly: claim audits, Tier upgrades, benefits roll-up with uncertainty.
- Annual: benefits strategy refresh; alignment with budget cycle.

### Approval matrix
| Activity | Tier 0 | Tier 1 | Tier 2 | Tier 3 |
|---|---:|---:|---:|---:|
| Internal reporting | ✅ | ✅ | ✅ | ✅ |
| Public summary | ❌ | ⚠️ (signal only) | ✅ | ✅ |
| Material fiscal claim | ❌ | ❌ | ⚠️ | ✅ |
| Governance board sign-off | ❌ | ❌ | ✅ | ✅ |

---

## Reporting: internal vs public
### Internal reporting
- Can include Tier 0–3, but must clearly label tiers.
- Can include operational detail needed for improvement.

### Public reporting
- Must avoid disclosing sensitive operational details (security) and personal data.
- Must present:
  - Tier, uncertainty, and limitations
  - Clear narrative separating outputs from outcomes
  - Equity/distributional notes where relevant and lawful

---

## Cost and TCO coupling
All Tier 2–3 claims must include a cost basis:
- Implementation cost (build/buy)
- Operating cost (cloud, licenses, support)
- Change management and training
- Data acquisition and governance overhead
- Risk costs (privacy/security mitigation)

Net benefit should be expressed as:
- Net present value (when applicable)
- Cost per unit improvement (e.g., $/minute reduced)

---

## Tooling and implementation architecture
### Evidence store
- Append-only or object-lock storage for evidence artifacts.
- Versioned datasets and models with immutable identifiers.

### Benefits registry
- A structured registry (YAML/JSON/DB) backing the claim record template.
- Integration with:
  - ADR repository
  - Issue tracker / change management
  - Data catalog / lineage tools

### Automation hooks
- Automated generation of claim evidence packs from pipeline runs.
- Continuous monitoring of primary and guardrail metrics.

---

## Risks and mitigations
| Risk | Example | Mitigation |
|---|---|---|
| Goodhart’s Law | Teams optimize metric, harm service | Guardrails, audits, balanced scorecards |
| Attribution overclaim | External factors drive improvement | Tiering + counterfactual standards |
| Data drift | Metric meaning changes | Data contracts, schema versioning |
| Political pressure | Publish weak claims | Approval thresholds, independent review |
| Privacy exposure | Too granular public reporting | Aggregation, suppression, disclosure control |

---

## Minimum viable rollout (90 days)
1. Stand up benefits registry and claim template.
2. Define portfolio taxonomy + 10 core metrics with guardrails.
3. Pilot 3 claims (one per major domain: mobility, resilience, buildings).
4. Implement lineage capture for those claims.
5. Establish monthly review + quarterly audit.

---

## Sources
- HM Treasury, **The Green Book (Appraisal and Evaluation in Central Government)**. GOV.UK publication page (2020). https://www.gov.uk/government/publications/the-green-book-appraisal-and-evaluation-in-central-government/the-green-book-2020
