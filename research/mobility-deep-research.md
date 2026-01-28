# Mobility — Deep research

## Executive summary
A mobility “digital twin” is a **decision-support system**, not a single model. It combines:
- **canonical mobility data products** (network, observed conditions, incidents, curb rules/enforcement, OD demand)
- **calibrated/validated models** (assignment/simulation, forecasting, pricing/curb policy evaluation)
- **governed decision workflows** (HITL approvals, incident mode, rollback)

This document makes mobility **decision-grade** for both operations and planning by defining:
- fitness-for-purpose labels so outputs are not misused
- data product contracts with owners and SLAs
- calibration/validation playbooks with regime-based acceptance thresholds
- governance for pricing and curb enforcement with equity constraints and feedback-loop monitoring
- interoperability and vendor portability requirements, including exit strategies

This document deepens item 3 in [`kali-task-research.md`](kali-task-research.md:1): *“Mobility: Model multimodal traffic dynamics to optimize flows, pricing, and curb access under changing demand.”*

---

## 1) Scope, decisions, and fitness-for-purpose labels

### 1.1 Operational decision support (ops)
Examples:
- incident detection/triage support and detour candidates
- signal timing candidate identification (not autonomous retiming unless separately governed)
- transit priority candidates and performance monitoring
- curb operations during special events (loading zone reallocations, temporary restrictions)

Ops outputs must support **time-bounded decisions**, have **incident-mode** behavior, and integrate with approvals.

### 1.2 Planning / policy evaluation (planning)
Examples:
- congestion pricing and road user charging evaluation
- curb allocation policy and pricing design
- lane reallocation / bus priority / Vision Zero changes

Planning outputs are inherently uncertain and must not be treated as enforcement evidence.

### 1.3 Fitness-for-purpose labels (publishability)
Label every output product:
- **O-Label (Operational):** suitable for internal operations; must have freshness SLOs and degraded-mode rules.
- **P-Label (Planning):** suitable for scenario evaluation; not for real-time control.
- **E-Label (Evidence):** suitable for regulatory / enforcement / legal evidentiary use (rare; requires extra governance).

Default posture:
- most analytics are O or P.
- **E-Label is opt-in** with explicit approval gates.

---

## 2) Canonical mobility data products + ownership (explicit)

All mobility capabilities depend on well-owned, versioned data products.

| Data product | Description (canonical) | Typical owner | SLA/SLO targets (examples) | Schema/versioning expectations | Privacy class |
|---|---|---|---|---|---|
| Road network representation | links/nodes, lane groups, speed limits, control devices; linear referencing | DOT GIS / TMC engineering | monthly refresh; emergency updates < 24h | semantic version; stable IDs; change log | P0–P1 |
| Observed speed/flow/occupancy time series | per segment/corridor time-series from sensors/probes | TMC + platform data | freshness p95 < 3 min (ops); completeness > 98% | schema registry; event-time; backfill rules | P1–P2 (probe higher) |
| Incidents/events | CAD/911 extracts, DOT events, work zones, special events | TMC + public safety liaison | ingest p95 < 60s for ops; revision policy | event taxonomy; severity; geometry; versioned updates | P1–P2 |
| OD demand estimates | OD matrices by time slice + class; versioned | planning analytics | weekly/monthly; reproducibility required | snapshot IDs; model inputs recorded | P1–P2 |
| Curb inventory + rules | curb segments, allowed uses, time windows, permits | parking/curb agency | quarterly refresh; urgent updates < 72h | stable curb IDs; rule versioning | P0–P1 |
| Enforcement events (curb) | citations/warnings/observations; location/time | enforcement ops + parking adjudication | daily batch; integrity checks | immutable event log; appeal linkage | P2–P3 |

Notes:
- Privacy class should align to the governance/privacy program in [`research/governance-privacy-deep-research.md`](research/governance-privacy-deep-research.md:1).
- Probe/CV-derived products are typically higher sensitivity due to re-identification risk.

---

## 3) Calibration & validation playbook (concrete)

### 3.1 Regime-based evaluation (avoid “average looks good”)
Define regimes and evaluate separately:
- peak vs off-peak
- weekdays vs weekends
- weather (rain/snow/heat)
- special events
- incidents (crashes, closures, major disruptions)

### 3.2 Evaluation splits (avoid leakage)
Use **time-based** and **geography-based** splits:
- time split: train on months 1–9, validate on 10–11, test on 12 (rolling)
- geography split: hold out corridors or districts (to test generalization)
- incident holdouts: maintain a library of incident days not used for calibration

### 3.3 Acceptance criteria by use case
Define thresholds by corridor and regime. Examples:

**Operational ETA / travel time nowcasting (O-Label):**
- corridor travel time MAPE: ≤ 10–15% in normal regimes; ≤ 20% in incident regimes
- bias: near zero across peak/off-peak (no systematic underprediction)
- freshness: meets SLOs; degraded mode if not

**Corridor planning scenarios (P-Label):**
- reproduce baseline volumes/speeds within defined tolerance bands
- queue/spillback plausibility checks at critical bottlenecks

**Pricing policy evaluation (P-Label; politically sensitive):**
- behavioral response sensitivity bounds published
- spillover monitoring plan included

### 3.4 Rare events and incident-mode validation
Define “incident mode”:
- triggered by incident feed severity, abnormal speed drops, or operator activation

Validation must include:
- performance on incident days (holdout)
- fallback behavior when data degrades
- decision workflow correctness (HITL tiering escalates)

### 3.5 Safety surrogate metrics validation (if used)
If surrogate safety measures (SSMs) are used:
- validate correlation against crash history at corridor/intersection level
- quantify drift and false positives
- never optimize solely on surrogates without governance approval

Reference modeling guidance sources should be used for calibration practices (see References).

---

## 4) Pricing and equity (in scope)

### 4.1 Policy objectives and constraints
Define objectives explicitly (who sets them, and how conflicts are resolved):
- congestion reduction and reliability targets
- revenue constraints (if any)
- emissions/air quality co-benefits
- transit performance protection

Constraints:
- max toll by time/zone
- geographic caps (avoid burden concentration)
- protected exemptions (disability, emergency vehicles)

### 4.2 Equity mitigations (guardrails)
Require at least one of:
- income-based discounts or rebates
- exemptions for essential trips/vehicles (governed)
- distributional caps (e.g., no decile sees > X% increase in travel cost burden)
- reinvestment rules (revenue earmarks for affected communities)

### 4.3 Governance for politically sensitive levers
- publish objectives, weights, and constraints used in optimization
- approval gates: technical review → equity review → executive approval
- communications plan that separates what is measured vs what is value judgment

### 4.4 Evaluation design (pricing / curb interventions)
Use quasi-experimental designs:
- before/after with control corridors/areas
- synthetic controls where feasible
- measure spillovers (diversion routes, adjacent neighborhoods)
- measure behavior adaptation (mode shift, time shift, trip suppression)
- publish uncertainty and limitation statements

---

## 5) Curb management + enforcement feedback loop (in scope)

### 5.1 Operational integration
Define workflow:
- curb rule change proposal → approval → signage/comms → enforcement posture
- enforcement dispatch priorities during events
- citation lifecycle and appeals integration

### 5.2 Biased enforcement risk + mitigation
Risks:
- enforcement concentrates where data is richer or complaints are louder
- feedback loops: more tickets → “hotspot” model → more patrols

Mitigations:
- monitor enforcement allocation vs need indicators
- require periodic random sampling patrols to reduce data bias
- separate “complaint volume” from “violation prevalence” in models
- establish oversight review for any automated prioritization

### 5.3 Feedback-loop detection metrics
- geographic concentration indices over time
- disparity ratios by approved strata
- divergence between modeled violation risk and observed enforcement intensity

---

## 6) Interoperability + vendor portability strategy

### 6.1 Canonical schema mapping layer
Even if external standards are used (DATEX II, TPEG, OpenLR, NGSI-LD), the city must own canonical internal entities:
- Link, Node, LaneGroup
- CurbSegment, CurbRule
- Observation (speed/flow/occupancy)
- Incident, WorkZone, SpecialEvent

Maintain:
- mapping tables from vendor schemas → canonical schema
- versioned transformations and test fixtures

### 6.2 Portability requirements for probe/CV vendors
Contracts should require:
- derived products at minimum (segment travel times with confidence/coverage)
- metadata: methodology summary, coverage bias notes, timestamps, revision rules
- retention and deletion guarantees
- auditability: ability to explain major discontinuities

### 6.3 Exit strategy and data escrow
- data escrow for key derived products + historical backfills
- transition plan to new provider with comparability methodology
- keep canonical KPIs stable across vendor changes

---

## 7) Operational runbooks (minimum)

### 7.1 Incident surge / special event mode
- activation criteria
- staffing and decision cadence
- temporary policy constraints (no major pricing changes)

### 7.2 Feed degradation and degraded-mode operations
- detect data dropouts; label confidence
- switch to fallback estimates
- freeze sensitive decisions if evidence insufficient

### 7.3 Pricing change lifecycle
- proposal → equity/technical review → approval → rollout
- monitoring plan and rollback criteria

### 7.4 Curb rule + enforcement posture change
- change control; comms; enforcement guidance
- monitor disparities; adjust

### 7.5 Contested results / public challenge process
- intake process
- publish redacted methodology and uncertainty
- review panel decision and revision protocol

---

## 8) Key metrics

### 8.1 Data SLIs
- freshness/coverage by product (probe, sensors, incidents)
- schema change and backfill rates

### 8.2 Model metrics
- bias/RMSE/MAPE by regime
- calibration drift over time
- incident-mode performance

### 8.3 Equity metrics
- distribution of travel time and cost burdens
- enforcement disparities and feedback-loop signals

### 8.4 Safety metrics
- crash trends and severity
- surrogate validation drift (if used)

### 8.5 Operational outcomes
- incident clearance time
- bus reliability (if coupled)
- corridor reliability improvements

---

## 9) Implementation roadmap

### 0–3 months
- publish data product catalog + owners + SLAs
- build baseline validation harness; define pilot corridors
- implement schema registry and conformance checks

### 3–12 months
- regime-based calibration playbooks + incident library
- curb/enforcement integration and bias monitoring
- equity reporting baseline for pricing/curb proposals

### 12–24 months
- pricing evaluation maturity (spillovers + behavior adaptation)
- cross-domain coupling (transit/energy/emissions)
- governance hardening and portability drills

---

## References (retrieved via Firecrawl MCP)

### Calibration / validation practices
- Guidelines for Applying Traffic Microsimulation Modeling Software (FHWA) — https://highways.dot.gov/media/6916
  - Takeaway: Practical guidance for calibrating/validating microsimulation models against observed travel time, delay, and queue measures.
- Texas DOT microsimulation calibration criteria — https://www.txdot.gov/manuals/des/tsp/chapter-13-microsimulation-analysis/13-5-calibration.html
  - Takeaway: Example calibration criteria (including travel time tolerances) that can be adapted into corridor/regime acceptance thresholds.

### Congestion pricing / equity mitigations
- Congestion Relief Zone Toll: Discounts and Exemptions (MTA) — https://www.mta.info/fares-tolls/tolls/congestion-relief-zone/discounts-exemptions
  - Takeaway: Concrete examples of discount/exemption structures for congestion pricing that can inform equity mitigations.
- Income-Based Equity Impacts of Congestion Pricing (FHWA) — https://rosap.ntl.bts.gov/view/dot/760/dot_760_DS1.pdf
  - Takeaway: Discusses distributional impacts of congestion pricing by income and considerations for mitigation.

### Privacy practices for mobility/probe data
- Investigation into the collection and use of de-identified mobility data (Office of the Privacy Commissioner of Canada) — https://www.priv.gc.ca/en/opc-actions-and-decisions/investigations/investigations-into-federal-institutions/2022-23/pa_20230529_phac/
  - Takeaway: Practical regulatory discussion of de-identified mobility data risks and governance expectations.
- Privacy Principles for Mobility Data — https://www.mobilitydataprivacyprinciples.org/
  - Takeaway: Principles for responsible mobility data use; useful as a baseline governance posture for minimization and transparency.
