# Climate risk — Deep research (decision-anchored, reproducible)

## Executive summary
Climate risk stress-testing for a City Digital Twin is the capability to **support high-stakes municipal decisions** with **reproducible** hazard–exposure–vulnerability–interdependency analyses.

This document upgrades the prior “concept menu” into an implementable playbook that is:
- **Decision-anchored**: common city decisions are mapped to required model outputs, uncertainty products, equity slices, and sign-offs.
- **Scenario-governed**: a standard scenario set is required for recurring decisions, with versioning and anti-cherry-picking reporting.
- **Calibration-aware**: fragility/vulnerability functions are localized and validated, with explicit epistemic vs aleatory uncertainty.
- **Interdependency-deep**: dependency semantics include capacity/thresholds and restoration logistics (crew/access constraints).
- **Engineering-pragmatic**: caching, deterministic pipelines, containerized runs, and cost controls are first-class.

This document deepens item 8 in [`kali-task-research.md`](kali-task-research.md:1): “Climate risk: Stress-test city systems against heat, storms, sea level rise, and compound hazards for adaptation planning.”

---

## Scope and defaults
Defaults (unless overridden):
- Hazards: flood + extreme heat + wildfire smoke + wind
- Planning horizons: 5y + 20y + 50y
- Uncertainty posture: explicit-and-published
- Equity focus: high
- Scenario governance posture: standard-set-required
- Interdependency depth: medium
- Compute constraints: moderate

---

## 1) Decision templates (must be concrete)
### 1.1 Decision-to-output table (minimum set)
Use this table to prevent “maps without decisions”. Every climate risk product must declare which decision it supports.

| Municipal decision | Required outputs | Spatial/temporal resolution | Uncertainty product | Equity slice | Governance sign-off |
|---|---|---|---|---|---|
| Annual / multi-year capital hardening prioritization | risk reduction per $; avoided outage minutes; avoided population exposure; project ranking under standard scenarios | asset-level; annual + event-based | scenario fan chart; sensitivity to fragility assumptions; percentile bands | distribution of risk reduction by neighborhood; critical customers affected | Resilience + Capital Planning + Utility/Dept owner |
| Update design standards (IDF curves, freeboard, heat thresholds) | updated design values; exceedance probability vs horizon; “regret” analysis | citywide + sub-basin; seasonal | confidence bands; epistemic notes; method provenance | impacts on vulnerable areas; affordability considerations | Engineering Standards Board + Resilience |
| Cooling centers siting + trigger thresholds | candidate sites; catchment coverage; trigger thresholds based on heat index forecast; staffing/staging plan | neighborhood; hourly/daily | forecast uncertainty + historical exceedance; decision thresholds with false pos/neg costs | access equity (walk/transit time); medically vulnerable proxies | Public Health + Emergency Mgmt + Facilities |
| Pre-storm resource staging | staging locations; predicted access constraints; expected outage minutes; crew routing constraints | grid/route-level; 0–72h | ensemble spread; worst-plausible bounds | priority for critical facilities and disadvantaged neighborhoods | Emergency Mgmt + Public Works + Utilities |
| Critical facility backup power investments | facility criticality; outage risk; cost-effectiveness; prioritized list | facility-level; event-based | outage probability ranges; dependency sensitivity | critical customers affected; equity-weighted benefit | Utilities + Facilities + Resilience |
| Vegetation mgmt / wildfire smoke response | smoke exposure forecasts; indoor shelter prioritization; filtration deployment plans | neighborhood; hourly/daily | forecast uncertainty; exposure percentile bands | sensitive populations (schools, elderly); access constraints | Public Health + Emergency Mgmt |

### 1.2 Minimum decision artifact
Every decision support package must include:
- Decision being supported (from table)
- Standard scenario set used (see §4)
- Run IDs and pinned inputs (see §6)
- Uncertainty products and assumptions
- Equity disclosure controls (see §5)

---

## 2) Fragility/vulnerability calibration and uncertainty
Fragility/vulnerability functions convert hazard intensity (depth, temperature, smoke PM2.5, wind speed) into consequences (damage, outage probability, health impacts).

### 2.1 Sourcing fragility/vulnerability functions
Options (often combined):
- Published defaults from established frameworks/tools (e.g., loss estimation libraries)
- Utility/agency internal historical data (outage logs, service requests, asset failures)
- Post-event assessments (inspections, damage surveys)
- Expert elicitation (documented assumptions)

### 2.2 Localization with limited local data
Use a tiered localization strategy:
1. **Transfer + parameter tuning**: start from published/default curves and tune key parameters with local covariates (asset age, soil type, elevation, maintenance state).
2. **Bayesian updating**: treat default curves as priors and update with observed local events (even if sparse).
3. **Segmented curves**: separate by asset class and context (e.g., underground vs overhead lines; older masonry vs newer).
4. **Elicitation with audit**: when data is sparse, use structured elicitation with an assumptions register and planned validation.

### 2.3 Validation: historical and near-miss events
Minimum validation practices:
- Hindcast at least 2–3 known local events (where data exists): do predicted impacts match observed outages/damages within uncertainty?
- Near-miss analysis: validate that the model does not predict failures when none occurred, within confidence.
- Cross-source consistency checks: reconcile outage logs, work orders, and citizen reports.

### 2.4 Aleatory vs epistemic uncertainty (explicit)
- **Aleatory** (inherent randomness): event-to-event variability (e.g., storm track, microbursts).
- **Epistemic** (knowledge uncertainty): uncertainty in fragility parameters, missing asset attributes, incomplete exposure.

Required communication:
- publish both where feasible:
  - aleatory as scenario ensemble spread
  - epistemic as parameter uncertainty ranges / alternative model structures
- do not collapse both into a single number without explanation.

---

## 3) Interdependency model semantics + restoration logistics
### 3.1 Dependency graph semantics (minimum)
Model critical services as a dependency graph with explicit semantics.

**Nodes**: assets, facilities, service areas, and lifeline functions.
**Edges**: directed dependencies with:
- `type`: power→water pump, road access→repair crew, comms→SCADA
- `capacity`: max service level supported
- `threshold`: hazard intensity or upstream service level below which downstream fails or degrades
- `latency`: delay before failure propagates
- `degradation_function`: continuous (graceful) or discrete (step)

### 3.2 Failure propagation rules
Minimum rules:
- define what “failure” means per node (binary outage vs degraded capacity)
- propagate along directed edges with thresholds and lags
- include substitution where applicable (generators, alternate routes)

### 3.3 Restoration dynamics and logistics coupling
Restoration is not just “time-to-repair”; it is constrained by logistics.

Model components:
- repair time distributions per asset class (mean + variance)
- crew availability by skill type
- access constraints (flooded roads, debris)
- priority policies (critical facilities first)
- resource constraints (spares, fuel)

Outputs:
- restoration curves (service level over time)
- outage minutes and population affected over time

### 3.4 “Critical service” definitions and KPIs
Define services in survivor-centric terms (alignable to lifelines):
- power availability
- water systems
- communications
- transportation access
- health/medical access

KPIs:
- outage minutes (total and per critical customer)
- population affected (count and person-hours)
- time-to-restore-to-80% service
- critical facility downtime (hospitals, shelters, schools)

---

## 4) Scenario governance (anti-cherry-picking)
### 4.1 Standard scenario set (required)
For recurring decisions, the following scenario set is mandatory unless an exception is approved.

Minimum standard set structure:
- **Baseline (historical)**: recent climatology reference period
- **Near-term (5y)**: current trend + plausible extremes
- **Mid-term (20y)**: at least two pathways (moderate + higher warming)
- **Long-term (50y)**: at least two pathways
- **Compound event**: e.g., heat + outage; storm + high tide

Each scenario must specify:
- hazard dataset(s) and downscaling method
- spatial resolution
- time horizon
- ensemble size / representative quantiles

### 4.2 Scenario registry and versioning rules
Maintain a scenario registry with:
- `scenario_id`, `version`, `created_by`, `approved_by`, `valid_from`, `valid_to`
- rationale (“why this scenario”)
- compatibility notes (what decisions it supports)

### 4.3 Governance to add/retire scenarios
- Additions require:
  - written rationale
  - impact on decision comparability (what changes in reporting)
  - backfill plan for prior-year comparability
- Retirements require:
  - deprecation notice
  - mapping to replacement scenarios

### 4.4 Anti-cherry-picking reporting standard
Every decision report must include:
- results for **all scenarios in the standard set** (not a subset)
- a consistent comparative table:
  - baseline vs each future scenario
  - best estimate + uncertainty bands
  - equity slice summary
- disclosure of any exceptions (with approval reference)

---

## 5) Equity integration without harm
### 5.1 Distributional metrics (minimum)
Report how risk and benefits distribute across communities.

Metrics (examples):
- risk exposure person-hours by neighborhood
- critical facility downtime by service area
- adaptation benefit distribution (risk reduction) by neighborhood
- “least advantaged quintile” benefit share

### 5.2 Community review with real influence
Implement a review loop that can change prioritization:
- publish an understandable summary for review
- accept community-submitted corrections (missing assets, access constraints)
- document how feedback changed:
  - inputs (exposure, critical facilities)
  - weights (equity weighting)
  - project selection

### 5.3 Disclosure controls for sensitive vulnerability data
To avoid harm:
- tiered access:
  - internal detailed layers (critical infrastructure, sensitive vulnerabilities)
  - public aggregated summaries
- aggregation + suppression:
  - minimum population thresholds
  - avoid pinpointing sensitive facilities

---

## 6) Compute/data engineering pragmatics (moderate constraints)
### 6.1 Caching strategy
Cache expensive components:
- hazard tiles by scenario and resolution
- exposure joins (asset-to-tile mapping)
- precomputed vulnerability indices by asset class

### 6.2 Deterministic, reproducible pipelines
Requirements:
- containerized runs (image digest pinned)
- deterministic pipeline steps (fixed seeds where relevant)
- pinned data versions (scenario registry + exposure snapshot IDs)
- run manifests saved per run:
  - inputs, parameters, code version, container digest, outputs hashes

### 6.3 Cost controls
- “fast vs detailed” modes:
  - fast: coarse resolution + reduced ensemble for planning cycles
  - detailed: full ensemble + fine resolution for final decisions
- budget per run and per decision cycle
- precompute annual baseline products

---

## 7) Operational runbooks
### 7.1 Annual scenario refresh + publication cycle
1. Propose updates to scenario registry (rationale, versioning).
2. Run standard scenario set for core decisions.
3. Publish comparative reporting package (anti-cherry-picking).
4. Conduct independent review and sign-off.

### 7.2 Post-event model update + calibration
1. Ingest event observations (hazard footprints, outages, damages).
2. Validate hindcast.
3. Update fragility priors (Bayesian update) and document changes.
4. Re-run standard scenarios if calibration materially changes outputs.

### 7.3 Contested-result challenge process
1. Log challenge with specific claim (input, method, output).
2. Reproduce run from manifest.
3. If disagreement persists: run sensitivity cases and publish deltas.
4. Governance decision: accept revision or document rationale for no change.

---

## 8) Key metrics
- reproducibility: % runs reproducible from pinned artifacts
- scenario coverage: % decisions supported by standard scenario set
- calibration: hindcast fit measures; uncertainty calibration (does observed fall within bands?)
- interdependency realism: % critical dependencies modeled with restoration constraints
- equity: distributional impact tracking + privacy-safe publication coverage

---

## 9) Implementation roadmap
### 0–3 months
- decision templates and sign-off model
- baseline hazard/exposure registry
- scenario registry MVP (IDs, versioning, approvals)

### 3–12 months
- localized fragility/vulnerability approach with explicit uncertainty
- standard scenario set operationalized
- deterministic pipelines + caching

### 12–24 months
- interdependency semantics + restoration logistics integration
- mature equity governance + disclosure controls
- portfolio optimization integrated into capital planning

---

## Sources (high-signal anchors)
- RAND — Robust Decision Making (RDM). https://www.rand.org/global-and-emerging-risks/centers/pardee/dmdu-decision-making-under-deep-uncertainty/robust-decision-making.html — DMDU approach emphasizing robustness across many plausible futures rather than prediction; useful for scenario governance and comparative reporting.
- HM Treasury Green Book supplementary guidance — Accounting for the effects of climate change (PDF). https://assets.publishing.service.gov.uk/media/6830434ce9440506ee953a33/Accounting_for_the_effects_of_climate_change_Green_Book_supplementary_guidance.pdf — Public-sector appraisal guidance for ensuring options are resilient to climate change and for structuring analysis and sensitivity.
- FEMA — Community Lifelines Implementation Toolkit v2.1 (PDF). https://www.fema.gov/sites/default/files/documents/fema_lifelines-toolkit-v2.1_2023.pdf — Defines “lifelines” as outcome-based critical services and supports consistent reporting across interdependent systems.
- FEMA — Hazus Flood Model Technical Manual (Hazus 7.0) (PDF). https://www.fema.gov/sites/default/files/documents/fema_rsl_hazus-7-fltm_06272025_0.pdf — Illustrates use of default damage/fragility functions and how loss estimation models structure vulnerability functions.
