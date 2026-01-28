# Energy — Deep research

## Executive summary

An urban energy “twin” is a **decision-support system** that represents energy supply/demand, distribution constraints, electrification trajectories, and emissions impacts to guide city policy and investment—**without pretending the city has utility-grade operational visibility or control**.

The core question remains:

> What happens to reliability, costs, and emissions if we electrify buildings, add EV charging, deploy DER/storage, and implement demand response—under realistic coincident peaks and outage scenarios?

This revision makes the theme operational by adding:

- A concrete **utility–city collaboration operating model** (joint steering, custodianship, dispute resolution, publish approvals, procurement boundary).
- A **fitness-for-purpose labeling scheme** and a decision tree for model fidelity under limited topology access, including uncertainty quantification and misuse prevention.
- Equity and affordability guardrails for DR/managed charging and electrification scenarios with auditable reporting.
- A clear cyber-physical boundary: advisory-by-default, partner-mediated actuation only, with security/safety constraints.
- Emissions accounting governance (average vs marginal factors, provenance/versioning, disclosure aligned to GHG Protocol).
- Validation and benchmark concepts (“golden feeder” sets) and acceptance thresholds for hosting capacity and forecasts.

## 0. Scope, decisions, and fitness-for-purpose

### 0.1 Decisions the energy twin must support

1. **Hosting capacity / DER interconnection guidance** (where is DER likely feasible; what upgrades might be needed)
2. **DER siting and value stacking** (storage, PV + storage, resilience hubs)
3. **EV charging load impacts** (transformer/feeder stress; managed charging potential)
4. **Resilience / backup power planning** (critical facilities, outage scenarios, microgrids)
5. **DR / managed charging program design** (targets, guardrails, measurement)
6. **Emissions and co-benefits reporting** (GHG impacts by program/scenario)

### 0.2 Fitness-for-purpose labels (publishability and misuse prevention)

All outputs must carry a label. Labels are controls, not marketing.

| Label | Allowed uses | Forbidden uses | Typical data/model burden | Publishability |
|---|---|---|---|---|
| **Advisory / Screening** | early planning, candidate siting, program sizing ranges | engineering interconnection decisions, compliance claims | aggregated data, proxy models | can be public if aggregated + caveated |
| **Planning-grade** | capital planning, non-wires alternatives screening, resilience portfolio | device-level operational control | partial topology + validated models | public with approvals + redaction |
| **Engineering-grade (Utility)** | interconnection studies, operational constraints | public release of sensitive topology | full topology + utility validation | generally restricted |

Rules:

- Default label is **Advisory / Screening** when `grid_visibility_level = limited`.
- Any public “hosting capacity map” must include:
  - label badge
  - last-updated date
  - uncertainty/range and what it excludes
  - prohibition statement: “Not an interconnection determination.”

## 1. Utility–city collaboration operating model (make it concrete)

### 1.1 Governance structure

Establish a **Utility–City Energy Data & Modeling Board** with a clear charter.

Minimum bodies:

1. **Joint Steering Committee** (monthly/quarterly)
   - city energy office lead + utility planning lead + CIO/CISO + legal/privacy
   - owns scope, priorities, and publication posture

2. **Data Custodianship Working Group** (biweekly)
   - data stewards from utility + city platform
   - owns data contracts, refresh cadence, quality SLAs, access grants

3. **Model Review Panel** (monthly; ad hoc for releases)
   - utility engineers + city analysts
   - approves model versions, benchmarks, and acceptance thresholds

### 1.2 Data custodianship and dispute resolution

- Each dataset has:
  - `system_of_record`
  - `custodian` (accountable)
  - `allowed_uses` + `prohibited_uses`
  - retention and sharing rules

Dispute resolution:

- step 1: custodianship group attempts resolution within 5 business days
- step 2: model review panel adjudicates technical disputes
- step 3: steering committee resolves policy/publication disputes

### 1.3 Publish approvals for aggregated outputs

Any public release (e.g., hosting capacity zones) follows:

1. draft artifact + label + uncertainty disclosure
2. redaction review (sensitive infrastructure inference risks)
3. utility approval (engineering + security)
4. city approval (comms + legal/privacy)
5. publish with immutable release metadata and rollback plan

### 1.4 Procurement and vendor boundary management

- City-owned platform vendors must not receive raw restricted topology unless explicitly contracted and isolated.
- Prefer a separation model:
  - utility runs sensitive studies in its enclave
  - city consumes aggregated outputs with provenance
- Contract clauses:
  - data handling, audit logs, breach notification
  - portability and exit plan
  - model transparency (inputs/assumptions disclosed at the allowed level)

## 2. Model fidelity vs data access trade-offs (limited visibility)

### 2.1 Decision tree: proxy vs detailed

Given `grid_visibility_level = limited`, choose the simplest model that is safe for the decision.

| Decision | Minimum safe model class | If visibility improves | Notes |
|---|---|---|---|
| Public screening map | zonal/proxy constraints + conservative margins | feeder-level power flow | must be labeled advisory |
| City siting for public chargers | transformer screening + historical peak proxies | feeder/transformer models | include equity metrics |
| DR/managed charging program sizing | load shape + diversity factors + weather | feeder constraints where available | comfort constraints required |
| Resilience hubs planning | critical facility load + outage scenarios | utility outage model integration | avoid operational dependencies |

### 2.2 Uncertainty representation

Under incomplete topology, uncertainty is not optional.

- represent outputs as ranges (P10/P50/P90) or conservative bounds
- separate uncertainty sources:
  - topology unknowns
  - load forecast error
  - DER adoption uncertainty

Example disclosure snippet:

- “Hosting capacity shown is a **screening estimate** derived from aggregated constraints and historical peaks; local equipment conditions may materially change results.”

### 2.3 Misuse-prevention patterns

- UI label badges + hover “allowed/forbidden uses.”
- Default to **zonal** aggregation for public maps.
- Watermark exports.
- “What would change this result?” fields on reports (data needed for upgrade).

## 3. Equity and affordability guardrails (high focus)

### 3.1 Required equity/affordability metrics

Report distributions, not just averages:

- **bill impact distribution** (P10/P50/P90) by customer segment
- **energy burden** proxies where feasible (share of income spent on energy)
- participation representativeness (enrollment vs eligible population)
- comfort constraints compliance for DR (e.g., temperature bounds, medical equipment exemptions)
- outage resilience by neighborhood (hours without power at critical services)

### 3.2 Program safeguards

For DR/managed charging and electrification incentives:

- opt-in by default; strong protections for vulnerable households
- “no harm” constraints:
  - max bill increase limits
  - opt-out and override mechanisms
  - equipment safety constraints
- avoid siting reinforcement and public charging in ways that systematically exclude underserved areas

### 3.3 Auditable monitoring and reporting

- publish an internal quarterly equity report with:
  - metrics above
  - sampling/audit methodology
  - corrective actions
- require third-party or cross-department review for high-impact programs

## 4. Cyber-physical boundary and actuation

### 4.1 Default: advisory-by-default

The city twin does not directly write to utility OT systems.

- any “control” is via utility programs or aggregators with explicit contracts

### 4.2 If limited partner-mediated control is used

Examples:

- managed charging via aggregator API
- DR event calls via OpenADR

Safety/security constraints:

- approvals for events (two-person rule for high-impact dispatch)
- explicit rollback (event cancellation)
- segmentation between analytics and control planes
- audit logs for every event (who/what/when/targeting)
- emergency exception handling + post-event review

## 5. Emissions accounting governance

### 5.1 Average vs marginal emissions (how to choose)

- **Average (location-based)**: appropriate for inventory-style reporting and annual summaries.
- **Marginal**: appropriate for operational decisions and DR/managed charging timing value (when shifting load changes which generators run).

Rules:

- every report must declare:
  - factor type (average vs marginal)
  - geography (balancing area)
  - time granularity (hourly/daily/annual)
  - version + provenance

### 5.2 Provenance, versioning, and disclosure

- maintain an `EmissionsFactorRegistry` with:
  - `factor_id`, `type`, `region`, `time_granularity`
  - `source_url`, `method_summary`
  - `valid_from/to`, `version`

Public disclosure rules:

- publish both the factor and caveats at the chosen transparency level
- align public claims to the city’s GHG inventory practices

## 6. Validation and benchmarks

### 6.1 “Golden feeder” benchmark sets

Work with the utility to define a small set of representative feeders:

- urban dense
- suburban
- high DER penetration
- constrained transformer-heavy areas

Use these to:

- validate hosting capacity methodology
- validate forecasts under normal and extreme regimes

### 6.2 Acceptance thresholds by decision type

Examples (calibrate locally):

- screening maps: conservative bounds; prioritize low false positives
- planning-grade: backtested against feeder benchmarks with documented error
- any upgrade toward engineering-grade requires utility acceptance

### 6.3 Backtesting plan

- forecast backtesting by season and extreme events
- hosting capacity validation against benchmark data and interconnection outcomes (where possible)
- publish a quarterly “model performance” report internally

## 7. Operational runbooks

### 7.1 Annual emissions factor refresh and publication

- pull latest factor sources
- update registry version
- rerun baseline emissions reports
- publish release notes and caveats

### 7.2 Hosting capacity map release workflow

- generate artifact with label + uncertainty
- run redaction review
- obtain utility + city approvals
- publish with release metadata; define rollback

### 7.3 Data-sharing incident / utility data embargo

- stop publishes dependent on embargoed data
- switch to coarse proxy outputs with explicit labels
- notify steering committee and consumers

### 7.4 Model degradation/drift detected

- freeze current public artifacts
- revert to last-known-good model
- run diagnostics; update benchmarks
- publish internal incident report

## 8. Key metrics

- data access coverage (% feeders/assets represented; fidelity level)
- forecast accuracy and calibration by regime
- hosting capacity validation hit rate / false positives
- equity/affordability metrics (bill impacts, participation)
- publishability compliance (% outputs correctly labeled + approved)

## 9. Implementation roadmap

### 0–3 months

- stand up steering + custodianship + model review bodies
- complete data inventory and access tiering
- implement labeling scheme and default disclosures
- build initial proxy models for screening

### 3–12 months

- define benchmark/golden feeder sets and acceptance thresholds
- improve fidelity where visibility allows
- implement equity guardrails and reporting cadence
- build publish workflow with approvals and redaction

### 12–24 months

- integrate interdependency/resilience coupling (critical facilities, outage scenarios)
- mature partner-mediated actuation where justified and governed
- continuous validation and transparency maturity

## 10. References

### 10.1 Workspace source

- Item 5 in [`kali-task-research.md`](../kali-task-research.md:1)

### 10.2 External references (retrieved via Firecrawl MCP)

- NREL. *Data Validation for Hosting Capacity Analyses* (report). https://docs.nrel.gov/docs/fy22osti/81811.pdf — Guidance on validating inputs and methods for hosting capacity analyses to improve reliability of published results.
- NREL. *Advanced Hosting Capacity Analysis* (overview). https://www.nrel.gov/solar/market-research-analysis/advanced-hosting-capacity-analysis — Defines hosting capacity analysis as quantifying DER additions without violating operational criteria.
- GHG Protocol. *Scope 2 Guidance* (PDF). https://ghgprotocol.org/sites/default/files/2023-03/Scope%202%20Guidance.pdf — Defines location-based (grid-average) and market-based methods and disclosure expectations for Scope 2 electricity emissions.
- NIST. *NISTIR 7628 — Guidelines for Smart Grid Cybersecurity* (PDF). https://www.nist.gov/system/files/documents/smartgrid/nistir-7628_total-2.pdf — Smart grid cybersecurity guidance relevant to segmentation and boundary controls between analytics and control systems.
- NREL. *Chapter 5. Low-Income Energy Bill Equity and Affordability* (LA100 Equity Strategies) (PDF). https://docs.nrel.gov/docs/fy24osti/85952.pdf — Discusses equity/affordability considerations for energy system transitions; useful for defining guardrails and reporting.
