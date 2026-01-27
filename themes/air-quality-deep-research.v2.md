# Air quality — Deep research (v2, decision-grade + buildable)

## Executive summary (for municipal operators, platform teams, policy analysts)
An urban air-quality “twin” is an operational system that converts **imperfect measurements + meteorology + emissions knowledge** into **decision-ready** estimates of pollution concentrations, exposure, and intervention impacts—across **real-time operations**, **daily advisory planning**, and **annual planning/evaluation**.

This v2 strengthens the original layered architecture by adding:
- A clear **governance boundary** between advisory and regulatory uses (labels, publish gates, misuse prevention).
- Practical **model class decision tree** by spatial scale and use case (street canyon ↔ neighborhood ↔ regional background).
- Low-cost sensor **calibration + drift** strategy and a policy for **community sensors**.
- A standard **causal evaluation playbook** for intervention claims (minimum data, pre-registration-lite, confounding controls).
- Concrete **uncertainty computation + UX patterns** (probability-of-exceedance, “what would change my mind”).
- Concrete **data contracts** (canonical schemas, QA flags, lineage/provenance, spatial/temporal standards) and interoperability mappings.
- **Runbooks**, **SLIs/SLOs**, validation gates, ownership, and an implementation roadmap suitable for limited budgets.

Scope assumptions (defaults): city-wide deployment; mixed reference monitors + dense low-cost sensors; limited budget; generic region/standards; advisory-first posture; options-not-prescriptions; high equity focus; public dashboard yes; time horizon real-time + daily + annual planning; no citations unless provided.

---

## 0. What this system is (and is not)
### 0.1 What it is
A system that can:
- **Nowcast** (0–2 hours) and **forecast** (2–72 hours) PM2.5/NO2/O3 at neighborhood scale, with uncertainty.
- Detect and characterize episodes (wildfire smoke, inversions, incidents).
- Provide **planning-grade** scenario comparisons (e.g., LEZ options) with traceable assumptions.
- Estimate **population-weighted exposure** and equity-focused distributional impacts.

### 0.2 What it is not
- Not a compliance determination engine. If outputs are used for enforcement, it must meet **regulatory-grade** requirements (out of scope by default).
- Not a substitute for a legal emissions inventory or certified monitoring network.

---

## 1. Stakeholders, ownership, and decision rights
### 1.1 Primary stakeholders
- Environmental agency / public health
- Transportation operations + planning
- Urban planning / zoning
- Emergency management / resilience
- Communities and EJ partners
- Platform + data engineering / security

### 1.2 RACI (minimum)
| Capability | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Sensor network ops + maintenance | Env ops | Env director | Platform | Public dashboard team |
| Calibration models + QA rules | Data science | Env director | Academia/community advisory panel | Operators |
| Forecast/nowcast production | Data science | Env ops lead | Platform | Public comms |
| Public advisories (issuance/rollback) | Public health comms | Public health officer | Env ops | Public |
| Scenario outputs for planning | Policy analytics | Planning director | Data science | Council/public |
| Access control + audit logging | Platform security | CIO/CISO | Env ops | Governance board |

Decision rights that must be explicit:
- Who can change thresholds for alerts?
- Who can publish a new model version to public dashboards?
- Who can label an output as planning-grade vs policy-evaluation-grade?

---

## 2. Governance boundary: advisory vs regulatory (must-have)
### 2.1 Labeling taxonomy (fitness-for-purpose)
All outputs—maps, timeseries, scenario results—MUST carry a **label**. Labels are not marketing; they are controls.

| Label | Allowed uses | Forbidden uses | Typical validation burden |
|---|---|---|---|
| **Informational** | Public awareness, education, exploratory dashboards | Any policy claims, ranking neighborhoods for enforcement | Basic QA, sanity checks |
| **Planning-grade** | Operational planning, hotspot triage, prioritizing monitoring/inspections, screening scenarios | Claiming intervention caused measured change; compliance | Routine cross-validation; uncertainty bands |
| **Policy evaluation-grade** | Comparative evaluation of interventions; reporting estimated benefits with caveats | Enforcement, compliance determinations | Formal evaluation design; confounder control; sensitivity analysis |
| **Regulatory-grade** | (Only if explicitly commissioned) support formal compliance processes under standards | Any use outside specified regulatory protocol | Certified methods, auditability, documented acceptance criteria |

Rule: **Default label is Planning-grade** unless explicitly upgraded by a governance gate.

### 2.2 Publish/approval gates (public releases and scenario outputs)
A “release” includes: dashboard updates, public reports, scenario comparisons, and model updates.

**Gate 0 — Internal (operators)**
- Data freshness/coverage SLOs met.
- Sensor QA status acceptable (no critical drift incidents unresolved).
- Model validation checks pass (see Section 9).

**Gate 1 — Public advisory (real-time/daily)**
- Output label is **Planning-grade** (or Informational).
- Model/version is **pinned** (immutable model_id + config hash).
- Uncertainty shown (probability-of-exceedance or CI).
- Public comms approves the advisory text and disclaimers.

**Gate 2 — Scenario/policy outputs (annual planning)**
- Output label is **Policy evaluation-grade** ONLY if:
  - Analysis plan template completed (Section 7.3).
  - Baseline and counterfactual assumptions are versioned and reviewable.
  - Sensitivity checks documented (weather normalization, background trends).

**Gate 3 — Regulatory-adjacent requests**
- Default response: provide planning-grade outputs with disclaimers and redirect to certified pathways.
- Any upgrade requires formal commissioning, legal review, and a documented regulatory protocol.

Release metadata required (for every published artifact):
- `release_id`, `release_time_utc`
- `label` (taxonomy above)
- `model_run_id` (immutable)
- `model_version`, `config_hash`, `code_ref`
- `data_cutoff_time_utc`
- `sensor_snapshot_id` (which sensors + QA state)

### 2.3 Misuse prevention mechanics (UI + access + audit)
- **Disclaimers + UI cues**:
  - Always show label badge (Informational / Planning-grade / Policy evaluation-grade).
  - “Not for compliance/enforcement” banner when not regulatory-grade.
  - Hover/expand for “What this means” and “What it does NOT mean”.
- **Access controls**:
  - Scenario engine write-access limited to authorized analysts.
  - Threshold changes require two-person approval for public triggers.
- **Audit trails**:
  - Log who viewed/exported policy-evaluation outputs.
  - Log who changed model configs, thresholds, or sensor inclusion rules.

---

## 3. Threat model / abuse cases (updated)
### 3.1 Assets
- Integrity of sensor measurements and **calibration lineage**
- Integrity of QA flags and sensor inclusion rules
- Integrity of model run configurations and scenario assumptions
- Credibility of public advisories and policy evaluation outputs

### 3.2 Key abuse/failure cases
- Data poisoning via community sensors without QA (false hotspots)
- Cherry-picked time windows to exaggerate intervention benefits
- Model output presented as compliance evidence
- Silent model degradation (e.g., seasonal shift, wildfire smoke regime) leading to wrong advisories

### 3.3 Controls (mapped to sections)
- Data contracts + provenance (Section 6)
- Calibration rotation + drift detection (Section 5)
- Publish gates + labeling taxonomy (Section 2)
- Validation gates + runbooks (Sections 9–10)

---

## 4. Layered reference architecture (buildable)
### 4.1 Components (keep layered, add concrete interfaces)
1. **Ingestion**
   - Reference monitors, low-cost sensors, mobile campaigns
   - Meteorology (stations + gridded forecasts)
2. **QA/QC + calibration**
   - Automated QA flags, drift detection, calibration models
3. **Canonical observation store**
   - Contracted schema with provenance (Section 6)
4. **Emissions layer (versioned)**
   - Inventory + activity-based updates (traffic, heating proxies)
5. **Modeling layer (multi-scale)**
   - Regional background + urban neighborhood + street/near-road modules
6. **Fusion/assimilation**
   - Combine model + observations, propagate uncertainty
7. **Exposure + equity analytics**
   - Population-weighted exposure, disparity tracking
8. **Scenario evaluation**
   - Parameterized interventions + causal evaluation workflows
9. **Serving + comms**
   - Operator console, public dashboard, APIs
10. **Governance + observability**
   - Gates, audit logs, SLOs

### 4.2 Core data flows
- Sensors + met → QA/calibration → observations (with QA + lineage)
- Emissions inventory + activity signals → emissions fields (versioned)
- Emissions + met + boundary conditions → model fields
- Observations + model → fusion/assimilation → best-estimate + uncertainty
- Concentrations + population/time-activity → exposure metrics
- Scenario config → scenario runs → labeled outputs + release metadata

---

## 5. Low-cost sensor network realities (calibration, drift, community governance)
### 5.1 Calibration strategy (concrete, budget-aware)
Low-cost sensors are valuable for spatial detail but require **continuous governance**.

**Co-location plan (reference monitors)**
- Goal: fit/refresh calibration models across seasons and regimes.
- Start-up recommendation (city-wide):
  - Co-locate **10–20% of low-cost units** at reference sites at any time (or a minimum of 5 units if network is small).
  - Co-location duration: **4–8 weeks** per unit per season (minimum), longer if pollution regimes are variable.
  - Rotation schedule: quarterly rotation for a subset; prioritize units with high drift risk or high decision impact.

**Calibration model options (choose based on complexity + explainability needs)**
- Simple: multivariate regression with temp/RH corrections
- Moderate: gradient-boosted trees with monotonic constraints (where appropriate)
- Advanced: Bayesian hierarchical calibration (borrowing strength across sensors)

**Calibration outputs required**
- Calibrated value + uncertainty
- Calibration model version + training window
- Fit diagnostics by regime (low/high concentration, humidity ranges)

### 5.2 Drift detection and sensor health scoring
Drift is not an edge case; it is normal.

**Drift detectors (layered)**
- Rule-based: sudden offsets, stuck-at values, unrealistic variance
- Peer comparison: compare to nearby sensors (robust median) controlling for wind regime
- Reference comparison: for co-located units, monitor residuals over time

**Sensor health score** (0–1) should be computed and stored for every sensor-hour:
- coverage score × calibration recency × drift likelihood × siting quality

### 5.3 Required metadata (non-negotiable)
Each sensor installation must record:
- `sensor_model`, `firmware_version`, `serial_number`
- `inlet_height_m`, `distance_to_road_m`, `micro_siting_type` (near-road/background/industrial)
- `installation_time_utc`, `maintenance_events[]` (cleaning, replacement, moves)
- `power_source` and known outage windows

### 5.4 Governance for community-deployed sensors
Community sensors can increase equity and trust, but must be governed.

**Representativeness rules**
- Label all community sensors as `deployment_type = community` and `siting_confidence` (self-reported vs verified).
- Default behavior:
  - Include in **Informational** layer with clear caveats.
  - Promote to Planning-grade only after meeting QA criteria (minimum uptime, calibration, siting verification).

**Inclusion/exclusion policy**
- Exclude from automated alerts unless:
  - calibration lineage exists,
  - drift score acceptable,
  - siting meets minimum standard.

**Transparency**
- Public dashboard should show a legend: reference monitor vs calibrated low-cost vs community-unverified.

---

## 6. Data architecture specifics (contracts + provenance + interoperability)
This section defines **canonical internal contracts**. You can map external standards (OGC SensorThings, NGSI-LD) into these.

### 6.1 Spatial + temporal indexing standards
- Spatial index: choose one primary grid for analytics and serving:
  - Option A: H3 hexes (e.g., res 8–10 for neighborhood; res 11–12 for near-road overlays)
  - Option B: Web tiles (XYZ) for visualization + a separate analysis grid
- Temporal standard:
  - Raw observations: as-received timestamps
  - Canonical aggregation: 1-min (ops), 1-hr (planning), 24-hr (annual)
- Time: always store `time_utc` + local timezone offset for reporting.

### 6.2 Canonical schema: observations
**Entity:** `Observation`
- Identity
  - `observation_id` (uuid)
  - `sensor_id`
  - `site_id` (optional for fixed sites)
- Time
  - `time_utc`
  - `time_received_utc`
- Location
  - `lat`, `lon`, `elevation_m`
  - `spatial_index` (e.g., h3)
- Measurement
  - `pollutant` (pm25, no2, o3, ...)
  - `value_raw`, `unit`
  - `value_calibrated` (nullable)
  - `value_uncertainty` (e.g., stddev or CI width)
- QA + health
  - `qa_flags[]` (see 6.5)
  - `sensor_health_score`
  - `calibration_status` (current/expired/unknown)
- Lineage/provenance
  - `calibration_lineage_id` (nullable)
  - `ingest_pipeline_version`
  - `source_system` (reference_monitor/vendor_x/community)
  - `data_rights` (public/internal/restricted)

### 6.3 Canonical schema: calibration lineage
**Entity:** `CalibrationLineage`
- `calibration_lineage_id`
- `sensor_id`
- `method` (regression/gbm/bayesian)
- `trained_on` (site_ids, time windows)
- `features_used` (temp, RH, etc.)
- `model_version`, `config_hash`
- `performance_summary` (bias/RMSE by regime)
- `valid_from_utc`, `valid_to_utc`

### 6.4 Canonical schema: model fields (gridded outputs)
**Entity:** `ModelField`
- `model_run_id`
- `model_class` (eulerian_grid/gaussian_plume/lur/ml_emulator/hybrid)
- `pollutant`
- `time_utc` (or time range)
- `spatial_index` (grid cell id / h3)
- `value` (concentration)
- `uncertainty` (stddev/quantiles)
- `inputs_snapshot_id` (met/emissions boundary conditions)
- `code_ref`, `config_hash`

### 6.5 Canonical QA flags (minimum set)
| Flag | Meaning | Typical action |
|---|---|---|
| `MISSING` | no data | gap-fill only if allowed |
| `OUT_OF_RANGE` | physically implausible | exclude |
| `STUCK_SENSOR` | low variance | exclude |
| `CLOCK_SKEW` | timestamp issues | correct/reject |
| `DRIFT_SUSPECT` | drift likely | down-weight/route to runbook |
| `SITING_POOR` | siting not representative | label + restrict uses |
| `CALIBRATION_EXPIRED` | calibration too old | down-weight/exclude from alerts |
| `COMMUNITY_UNVERIFIED` | unverified deployment | informational only |

### 6.6 Emissions inventory versioning contract
**Entity:** `EmissionsInventoryVersion`
- `inventory_version_id`
- `sector` (traffic/heating/industry/...)
- `spatial_resolution`
- `temporal_profile_version`
- `emission_factors_version`
- `assumptions_doc_ref`
- `valid_years`

### 6.7 Interoperability mappings
**OGC SensorThings → Canonical**
- `Thing`/`Datastream` → `sensor_id` + `pollutant`
- `Observation.result` → `value_raw`
- `Observation.phenomenonTime` → `time_utc`
- `Location` → `lat/lon/elevation`
- Add: QA flags + calibration lineage as extensions in canonical store.

**NGSI-LD → Canonical**
- Entity `AirQualityObserved` → `Observation`
- `observedAt` → `time_utc`
- `location` → `lat/lon`
- Add: `calibration_lineage_id`, `qa_flags`, `sensor_health_score`.

---

## 7. Model choice and resolution strategy (with decision tree)
### 7.1 The multi-scale requirement
You will not get good answers with a single model everywhere.
- **Regional background** determines the “floor” (transboundary pollution, wildfire smoke).
- **Neighborhood scale** captures urban gradients.
- **Street/near-road** captures hotspots and canyon effects.

### 7.2 Decision tree: selecting model classes by use case
Use this to choose a practical stack without overbuilding.

**Step 1 — What decision are you supporting?**
- **Real-time advisory/alerts** → prioritize robustness + uncertainty + latency
- **Operational planning (next 1–7 days)** → forecast skill and episode detection
- **Annual planning/scenario comparison** → interpretable attribution + causal evaluation

**Step 2 — What spatial scale matters?**
1) **Street canyon / near-road (10–100 m)**
- If you have building geometry and need canyon effects:
  - Option: street-canyon parameterizations + simplified near-road dispersion
- If you mostly need ranking hotspots:
  - Option: land-use regression / ML downscaling with strong QA + uncertainty

2) **Neighborhood (100 m–2 km)**
- If you have decent emissions + met and need transport patterns:
  - Option: urban dispersion/hybrid
- If budget/compute limited:
  - Option: statistical spatiotemporal models trained on observations + met covariates

3) **Regional background (2–20+ km)**
- Option: Eulerian grid chemistry-transport model outputs as boundary/background fields
- If not feasible: use external background products + statistical correction

**Step 3 — Source type focus**
- Point sources (stacks): Gaussian plume/puff is often sufficient for planning-grade screening.
- Area/line sources (traffic): hybrid (activity-based emissions + downscaling/fusion) is typical.

**Step 4 — Choose one of the following model classes (options)**
| Model class | Best for | Typical pitfalls |
|---|---|---|
| **Land-use regression (LUR)** | stable spatial patterns, annual averages | weak for day-to-day episodes |
| **Gaussian plume/puff** | point sources, screening, incident response | steady-state assumptions, building effects |
| **Eulerian grid (chemistry transport)** | regional background, secondary PM/O3 | too coarse for street hotspots |
| **Hybrid (CTM + urban downscaling + fusion)** | city-wide operational mapping | integration complexity |
| **ML emulator of physics model** | faster scenario sweeps | risk of out-of-domain failure |

### 7.3 Boundary conditions and transboundary pollution
Every city system needs a background strategy.
- Maintain a **BackgroundField** input with its own versioning:
  - wildfire smoke episodes,
  - regional transport,
  - seasonal baseline.
- When background dominates (e.g., wildfire smoke), switch decision logic:
  - focus on protective actions (advisories) rather than local source attribution.

### 7.4 Error targets by use case (and validation burden)
Define targets up front; they determine how strict your gates are.

| Use case | Primary metric | Suggested target (planning-grade) | Validation burden |
|---|---|---|---|
| Real-time alerts | alert precision/recall | ≥ 0.8 precision at chosen trigger | frequent, automated |
| Daily advisories | probability calibration | reliable prob-of-exceedance | automated + weekly review |
| Annual planning maps | bias + RMSE by neighborhood | low bias (near 0) and stable RMSE | seasonal stratification |
| Policy evaluation | effect estimate stability | robustness across sensitivity checks | formal analysis plan |

---

## 8. Fusion / assimilation options (practical)
Pick the simplest approach that delivers stable improvements and uncertainty.

### 8.1 Assimilation/fusion choices
| Option | When to use | Notes |
|---|---|---|
| **Bias correction by station** | early phase | low effort, limited spatial generalization |
| **Kalman filter bias state** | nowcast/forecast | captures time-varying bias |
| **Bayesian hierarchical fusion** | mixing sensor qualities | principled weighting + uncertainty |
| **Ensemble methods** | multiple model sources | supports prob outputs |
| **ML residual correction** | dense covariates | must guard against overfit |

### 8.2 Uncertainty propagation (what to compute)
Track uncertainty components and propagate as far as feasible:
- Emissions uncertainty (activity, factors)
- Meteorology uncertainty (wind, stability)
- Model structural uncertainty (class limitations)
- Observation uncertainty (sensor noise, calibration)

Minimum viable output uncertainty:
- **Quantiles** (e.g., P10/P50/P90) per grid cell/time
- Or mean + standard deviation, with a documented interpretation

---

## 9. Causal evaluation of interventions (decision-grade claims)
### 9.1 Minimum data requirements for causal claims
To claim “intervention X reduced PM2.5 by Y”, require at minimum:
- **Pre period**: 8–12 weeks (or a full season for annual claims)
- **Post period**: 8–12 weeks
- **Control areas**: comparable areas not treated (or synthetic control)
- **Confounders**: meteorology, seasonality, background trends, events (wildfire)
- **Spillover assessment**: traffic diversion, boundary shifts

If these cannot be met: label as **Planning-grade** (correlational) and avoid causal language.

### 9.2 Recommended designs (choose based on feasibility)
- Difference-in-differences with matched controls
- Synthetic control (when few treated units)
- Interrupted time series with covariate adjustment (weaker)

### 9.3 Pre-registration-lite “analysis plan” template
Store this as a versioned artifact before looking at outcomes.

**Intervention**
- name, dates, geography, implementation fidelity indicators

**Outcomes**
- primary pollutant metrics and aggregation windows
- secondary outcomes (e.g., NO2 near schools)

**Controls**
- control areas selection criteria
- exclusion criteria (wildfire days, sensor outages)

**Modeling**
- weather normalization approach
- background field handling
- spillover strategy

**Sensitivity checks (required)**
- alternate control sets
- alternate weather adjustment
- alternate time windows

**Reporting rules**
- state label (planning vs policy evaluation-grade)
- show uncertainty and limitations

---

## 10. Uncertainty communication UX (operators + public)
### 10.1 Compute + expose uncertainty consistently
At minimum, each displayed value should have:
- central estimate (e.g., P50)
- uncertainty band (P10–P90)
- probability-of-exceedance for relevant thresholds

### 10.2 UI patterns
- **Probability-of-exceedance tiles**: “30% chance PM2.5 > threshold in next 24h”
- **Confidence bands** on time series and forecasts
- **What would change my mind** indicators:
  - “If 2 reference monitors confirm rising trend”
  - “If wind shifts to X direction”

### 10.3 Decision thresholds under uncertainty
- Use **risk-based triggers** (not single deterministic values).
- Document:
  - threshold definition (probability and consequence)
  - roles authorized to change thresholds
  - two-person approval for public advisory thresholds

---

## 11. Operational SLIs/SLOs and validation gates
### 11.1 Operational SLIs
- Ingestion freshness (p95)
- Data completeness (% expected observations received)
- Calibration coverage (% sensors with current calibration)
- Fusion latency (end-to-end)
- API uptime

### 11.2 Suggested SLOs (starter)
| SLO | Target |
|---|---|
| Observation processing latency | p95 < 5 minutes |
| Completeness (reference monitors) | ≥ 98% hourly coverage |
| Completeness (low-cost) | ≥ 90% hourly coverage |
| Calibration currency | ≥ 85% sensors “current” |
| Public dashboard uptime | ≥ 99.5% |

### 11.3 Model validation gates (automated + reviewed)
Automated daily checks:
- Bias/RMSE/MAE by regime (low vs high concentrations)
- Probability calibration for exceedance events
- Episode performance tags (wildfire, inversion)

Weekly review:
- Spatial residual maps (where errors cluster)
- Sensor network health (drift incidents)
- Changes in background field behavior

Release gate requirement:
- No unresolved critical drift incidents.
- Model metrics within control limits (define per city after 4–8 weeks baseline).

---

## 12. Operational runbooks (minimum set)
### 12.1 Runbook: sensor anomaly/drift incident
**Trigger**: `DRIFT_SUSPECT` spikes; peer disagreement; co-location residual trend.

Checklist:
- [ ] Confirm affected sensor IDs + geography
- [ ] Check maintenance events and power/network logs
- [ ] Compare to nearest reference/healthy sensors controlling for wind
- [ ] If confirmed drift: set `CALIBRATION_EXPIRED` or exclude from alerts
- [ ] Dispatch maintenance or rotate to co-location site
- [ ] Document incident: cause, duration, impacted products, corrective actions
- [ ] Backfill policy: mark historical data with updated QA flags; do not silently rewrite public history without a release note

### 12.2 Runbook: forecast model degradation/drift
**Trigger**: sustained drop in forecast skill; calibration failure; regime change.

Checklist:
- [ ] Identify which regime failed (wildfire, inversion, high humidity)
- [ ] Freeze public model version (pin) and fall back to simpler baseline if needed
- [ ] Run diagnostics: boundary/background fields, met inputs, emissions inputs
- [ ] Retrain/update bias correction or fusion weights
- [ ] Validate on hold-out monitors; record new `model_version` + `config_hash`
- [ ] Publish via Gate 1 with release notes and updated uncertainty

### 12.3 Runbook: public advisory issuance + rollback
**Trigger**: probability-of-exceedance above trigger; operator confirmation.

Checklist:
- [ ] Confirm label is Planning-grade (not regulatory)
- [ ] Confirm latest data cutoff time + model run id
- [ ] Verify no critical sensor incidents affecting area
- [ ] Draft advisory text (what, where, when, who at risk, protective actions)
- [ ] Two-person approval for thresholds override (if needed)
- [ ] Publish advisory + record `release_id`
- [ ] Rollback: if later data invalidates, publish rollback with explanation and pinned artifact ids

---

## 13. Key metrics (operations, model, equity)
### 13.1 Operational metrics
- Freshness p95/p99
- Completeness by sensor class
- Latency by pipeline stage
- Uptime (API/dashboard)

### 13.2 Model metrics
- Bias, RMSE/MAE by pollutant and regime
- Calibration of probabilities (reliability diagrams)
- Alert precision/recall for exceedance events
- Spatial error hotspots

### 13.3 Equity metrics (high focus)
- Coverage representativeness by neighborhood burden (ensure sensors are not only in affluent areas)
- Disparity tracking: exposure percentiles by demographic proxies
- Bias audits: does model error correlate with neighborhood characteristics?
- Community participation: number of community sensors promoted to planning-grade after verification

---

## 14. Implementation roadmap (0–3, 3–12, 12–24 months)
### 0–3 months: stand up minimum operational capability
- Stand up ingestion + canonical observation store (Section 6)
- Implement QA flags + basic drift detectors
- Implement calibration co-location program plan + metadata enforcement
- Launch operator console + internal planning-grade dashboard with uncertainty
- Define labeling taxonomy + publish gates (Section 2)

### 3–12 months: multi-scale modeling + fusion + public dashboard
- Add background field strategy and boundary condition handling
- Implement fusion/assimilation (start with bias correction → hierarchical)
- Add public dashboard with uncertainty UX patterns
- Add scenario engine v1 for planning-grade comparisons
- Establish weekly model review and runbooks

### 12–24 months: policy evaluation-grade workflows and maturity
- Build formal causal evaluation workflow + pre-registration-lite
- Add robust scenario libraries (LEZ, fleet electrification, heating)
- Expand equity analytics and bias audits
- Mature governance: audit trails, two-person approvals, periodic external review

---

## 15. Risks & mitigations (updated)
| Risk | What it looks like | Mitigation |
|---|---|---|
| Low-cost sensor bias/drift | phantom hotspots, unstable trends | calibration rotation + drift runbook + QA flags |
| Misuse as compliance evidence | outputs cited in enforcement | labeling + UI disclaimers + access control + audit logs |
| Cherry-picked intervention results | selective windows, no controls | analysis plan template + gate for policy evaluation-grade |
| Background dominance (wildfire) | local attribution wrong | explicit background field + regime switching |
| Equity blind spots | sparse sensors in burdened areas | coverage metrics + targeted deployments + community sensor governance |

---

## 16. Deliverables and readiness checklists
### 16.1 Deliverables
- Canonical observation + calibration lineage store
- Planning-grade nowcast/forecast + uncertainty
- Public dashboard with label badges and prob-of-exceedance
- Scenario engine with versioned assumptions
- Runbooks + SLOs + validation gates

### 16.2 Readiness checklist (v2)
- [ ] Labeling taxonomy implemented across all outputs
- [ ] Publish gates operational (Gate 0/1/2)
- [ ] Canonical schema and provenance fields enforced
- [ ] Calibration rotation plan running; drift incidents tracked
- [ ] Model validation gates automated; weekly review in calendar
- [ ] Public dashboard shows uncertainty and misuse disclaimers
- [ ] Causal evaluation template available for policy teams

---

## Appendix A — Data contract examples (JSON sketches)
These are illustrative; implement as your platform standard.

```json
{
  "observation_id": "uuid",
  "sensor_id": "S-123",
  "time_utc": "2026-01-27T12:00:00Z",
  "lat": 54.7104,
  "lon": 20.4522,
  "spatial_index": "h3:8:...",
  "pollutant": "pm25",
  "value_raw": 18.4,
  "value_calibrated": 16.9,
  "value_uncertainty": {"p10": 12.0, "p50": 16.9, "p90": 23.1},
  "qa_flags": ["CALIBRATION_EXPIRED"],
  "sensor_health_score": 0.62,
  "calibration_lineage_id": "C-999",
  "ingest_pipeline_version": "ingest@1.4.2",
  "source_system": "low_cost_network",
  "data_rights": "public"
}
```

---

## Appendix B — Glossary
- **Nowcast**: near-term estimate using latest observations.
- **Background field**: regional baseline concentrations used as boundary conditions.
- **Fusion/assimilation**: combining model fields and observations to reduce error.
- **Planning-grade**: decision support with uncertainty; not compliance.

