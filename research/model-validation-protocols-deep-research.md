# Model validation protocols — Deep research

## Executive summary
City digital twins use many kinds of “models”: physics simulators (flood/energy), statistical forecasting (demand/ETA), ML perception (CV defect detection), anomaly detection (data integrity), and optimization/recommendation systems (signal timing candidates, resource allocation).

A validation program for a city twin must therefore:
- validate **by model type and failure mode** (not one generic checklist)
- connect results to **decision enablement** (what the model is allowed to do)
- connect results to **Human-in-the-loop (HITL) tiering** and evidence packets
- define **operational rollback** and incident-mode disabling
- govern **ground truth/labels** and measurement uncertainty
- operationalize realistic municipal threats (poisoning, extraction, inversion) with practical controls

This document deepens item 22 in [`kali-task-research.md`](kali-task-research.md:1): *“Model validation protocols: validate predictive and optimization models before they inform high-impact decisions.”*

---

## 1) City-twin model typology (validation pathways + rollouts)

Not all models validate the same way.

| Model type | City-twin examples | Primary failure modes | Recommended validation approach | Recommended rollout pattern |
|---|---|---|---|---|
| Physics-based simulators | flood model; district energy simulation; air dispersion | wrong boundary conditions; parameter miscalibration; non-identifiability; brittle to rare regimes | calibration vs historical events; sensitivity analysis; uncertainty quantification; domain expert review | offline validation → limited pilot use; do not “A/B test” physics; publish uncertainty |
| Statistical forecasting | ETA; demand forecasts; hospital/EMS surge proxy | non-stationarity; leakage; poor calibration; regime failures | time-based splits; regime-based metrics; calibration checks; backtesting library | offline → shadow → canary (if used operationally) |
| ML classifiers / vision | pothole detection; illegal dumping; infrastructure defect detection | label noise; domain shift (lighting/weather); spurious cues; fairness harms | labeled holdouts; inter-rater reliability; stress tests; slice-based eval | offline → shadow (compare to human) → canary with HITL |
| Anomaly detection | pipeline drift detection; sensor spoofing detection | alert fatigue; missed rare events; feedback loops | precision/recall on historical incidents; synthetic injection tests; alert quality monitoring | shadow → canary; require runbooks and throttling |
| Optimization / recommender systems | signal timing candidates; curb allocation; pricing proposals | unstable feedback; objective gaming; equity harms; safety constraint violations | offline simulation + constrained optimization tests; counterfactual evaluation; guardrail verification | offline → decision-support only (Tier 1) → expand tiers only with evidence + governance |

---

## 2) Risk-tiered validation gates (decision enablement)

Use a Tier 0–3 scheme consistent with HITL.

- **Tier 0 (informational):** dashboards; no direct action.
- **Tier 1 (low impact):** reversible tweaks; single approval.
- **Tier 2 (high impact):** significant operational/policy impact; multi-party approval.
- **Tier 3 (exceptional):** safety/rights/large spend impacts; two-person rule + executive/IC approval.

### 2.1 Validation burden by tier

| Tier | Minimum validation artifacts | Acceptance thresholds (examples) | What happens if not met |
|---|---|---|---|
| 0 | model card; data lineage; baseline eval report | basic accuracy + sanity checks | restrict to internal exploration; no public claims |
| 1 | + regime breakdown; monitoring plan; rollback plan | pass thresholds in normal regimes; known limitations | require explicit caveats; downgrade outputs |
| 2 | + independent review; robustness tests; fairness/impact checks; incident-mode tests | pass thresholds across regimes incl. stress; calibrated uncertainty | block Tier 2 enablement; stay Tier 1/HITL-only |
| 3 | + external/independent validation; red-team; formal safety constraints; audit plan | strongest thresholds; demonstrated safe failover | strict blocking; require executive acceptance of residual risk |

### 2.2 Gating controls (platform enforcement)
Validation results must control:
- whether the model can run in production
- whether outputs can appear in operator UIs
- whether recommendations can be executed automatically (rare; Tier 2–3 only)

Couple gates to feature flags and release controls.

---

## 3) Ground truth and label governance

Validation is only as good as the labels.

### 3.1 Label generation pipeline options
- human labeling with double review (for vision)
- field audits / sampling inspections
- sensor calibration campaigns
- administrative records (with known biases)

### 3.2 Label governance requirements
- label definitions and guidelines versioned
- inter-rater reliability tracked (where human labels)
- audit discrepancy rate measured
- periodic relabeling triggers (concept drift, new devices, new neighborhoods)

### 3.3 Label error budgets and uncertainty
- define acceptable label error budgets per use case
- represent measurement uncertainty in evaluation (confidence intervals, sensitivity)
- do not treat labels as perfect truth when they are proxies

---

## 4) Monitoring and drift (operational SLIs/SLOs)

### 4.1 Performance by regime
Track performance by:
- time-of-day/week
- weather
- special events
- incident mode

### 4.2 Calibration and uncertainty metrics
- calibration curves; Brier score (for probabilistic outputs)
- coverage of prediction intervals

### 4.3 Fairness and impact metrics
Where relevant:
- disparate impact / error parity across approved strata
- outcome harm signals (complaints, appeals upheld)

### 4.4 Data quality dependencies
Tie to data product SLAs (see [`research/data-quality-slas-deep-research.md`](research/data-quality-slas-deep-research.md:1)):
- if upstream freshness/completeness degrades, downgrade model tier or disable

---

## 5) Security and privacy threat model (practical)

### 5.1 Likelihood in municipal contexts
- extraction/inversion: moderate when APIs are public or vendor-shared; lower when internal-only.
- poisoning: plausible via compromised data pipelines or vendor feeds.

### 5.2 Practical mitigations
- provenance and signing for model artifacts and key datasets
- rate limits and authentication on model APIs
- strict access controls for training data
- redaction/aggregation for outputs; DP where needed
- protected environments for sensitive training (enclaves only when justified)

### 5.3 Incident response hooks
If poisoning/drift suspected:
- switch to safe baseline; disable recommendations
- preserve evidence (data snapshots, run IDs)
- initiate investigation and revalidation

---

## 6) Artifacts and auditability (minimum required)

Every production model must have:
- city-twin model card (intended use, exclusions, regimes)
- dataset lineage + version pinning
- evaluation report with regime breakdown
- approval record + change log
- rollback plan + “known bad” signature list

---

## 7) Integration with HITL (explicit coupling)

Map validation → decision tier and operator experience.

- Validation pass level determines max tier eligibility.
- Evidence packet shown to approvers must include:
  - model version and validation status
  - confidence/uncertainty
  - known failure modes and regimes

Align audit sampling:
- higher tier → higher sampling, mandatory reversal analysis.

See HITL evidence packet contract in [`research/human-in-the-loop-safeguards-deep-research.md`](research/human-in-the-loop-safeguards-deep-research.md:81).

---

## 8) Operational runbooks (minimum set)

- pre-release validation & sign-off
- post-release drift detection and rollback
- contested outcome investigation (model blamed for harm)
- emergency incident-mode model disabling (switch to safe baseline)

---

## 9) Key metrics

- % models with complete artifacts
- gating pass rate; rollback frequency and root causes
- drift detection MTTA/MTTR
- label quality (inter-rater reliability, audit discrepancy)
- regime coverage score

---

## 10) Implementation roadmap

### 0–3 months
- model inventory + typology
- minimal artifact standard
- baseline monitoring and drift dashboards

### 3–12 months
- risk-tiered gates + shadow/canary pipelines
- label governance program and audits

### 12–24 months
- mature robustness/security controls
- cross-agency audit readiness
- domain exemplars (mobility, health, resilience)

---

## References (retrieved via Firecrawl MCP)

### Model risk management / validation governance
- Artificial Intelligence Risk Management Framework (AI RMF 1.0) (NIST) — https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf
  - Takeaway: Provides lifecycle risk management guidance emphasizing test/evaluation/verification/validation and documentation for transparency and accountability.
- AI Risk Management Framework | NIST — https://www.nist.gov/itl/ai-risk-management-framework
  - Takeaway: Landing page for the AI RMF with supporting resources and context.

### Monitoring / rollout patterns
- Deployment (AWS Prescriptive Guidance for MLOps) — https://docs.aws.amazon.com/prescriptive-guidance/latest/ml-operations-planning/deployment.html
  - Takeaway: Describes deployment patterns including shadow mode and progressive rollouts that can be adapted to municipal model governance.

### Label governance / reliability
- Assessing Data Reliability (GAO-20-283G) — https://www.gao.gov/assets/gao-20-283g.pdf
  - Takeaway: Practical approach to assessing data reliability and documenting limitations—useful for label/ground-truth governance.

### Security and integrity anchors
- Security and Privacy Controls for Information Systems and Organizations (NIST SP 800-53r5) — https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf
  - Takeaway: Control catalog useful for mapping provenance, integrity, access control, and monitoring requirements into model validation programs.
