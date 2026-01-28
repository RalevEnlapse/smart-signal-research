# Buildings — Deep Research (City Portfolio)

## Executive summary
A “buildings twin” for a municipal portfolio is a **portfolio operations system**: a governed, semantically consistent layer that connects **building identity + assets + meters + BMS/OT telemetry + work** to produce **repeatable decisions and safe actions**.

**No-photorealism stance (sharpened):** 3D/BIM visualization is optional. The primary value comes from:
- a canonical identity model (building → system → equipment → point/meter)
- reliable semantic tagging (Haystack/Brick) with validation and drift control
- safe OT/IT integration that defaults to read-only, with tightly governed actuation
- measurement & verification (M&V) methods that make energy/savings claims defensible

This document deepens item 6 in [`kali-task-research.md`](kali-task-research.md:1): “Buildings: Connect BIM and operations data to optimize comfort, efficiency, and retrofit prioritization at scale.”

---

## 1) Scope and non-goals
### In scope
- City-owned/operated buildings: schools, municipal offices, libraries, community centers, depots (and optionally housing where the city is operator)
- Portfolio analytics: benchmarking, baselining, anomaly detection, FDD, commissioning/retro-commissioning support, retrofit prioritization
- Operational workflows: fault-to-work, investigation, evidence packs, change control
- Safety boundaries for analytics that influence controls

### Non-goals (explicit)
- Building-by-building bespoke “digital twin art projects”
- Mandatory BIM/3D: use BIM/as-builts where they exist, but do not block value delivery on 3D completeness
- Closed-loop autonomous control unless a formal safety case exists (see §4)

---

## 2) Stakeholders and decision rights
- Facilities / building operations (site staff)
- Energy & sustainability office
- OT/BMS engineers and integrators
- IT/network/security
- Capital planning / retrofit program
- Privacy / legal (especially for schools/health facilities)
- Procurement/vendor management

**Decision rights (summary):**
- Facilities owns comfort/safety outcomes and approves actuation.
- Security owns remote access controls and network boundaries.
- Energy team owns savings claims, baselines, and M&V governance.
- Platform/data team owns data contracts, lineage, and semantic QA tooling.

---

## 3) Canonical entity model (required)
Implement a canonical model that supports reconciliation across CMMS, BMS exports, meters, and drawings.

### 3.1 Entity hierarchy
**Building → System → Equipment → Point (+ Meter)**

### 3.2 Required identifiers (minimum)
| Entity | Required IDs | Notes |
|---|---|---|
| Building | `building_id` (stable UUID), address, site name | Link to GIS parcel/site if available |
| System | `system_id`, `building_id`, `system_type` | HVAC, lighting, DHW, envelope, etc. |
| Equipment | `equipment_id`, `system_id`, manufacturer/model/serial (if known) | Align with CMMS assets |
| Point | `point_id`, `equipment_id`, `point_type`, units, `source_ref` | BMS point name/path is a `source_ref` |
| Meter | `meter_id`, `building_id` or `system_id`, utility, channel | Utility meters + submeters |

**Rule:** IDs above must be **platform-owned** (not vendor-owned) so they survive vendor swaps.

---

## 4) OT/IT integration architecture and safety boundaries
### 4.1 Reference architecture (trust boundaries)
- **OT/BMS network**: controllers, supervisory server, fieldbus; safety-critical.
- **OT DMZ / Edge gateway**: protocol translation, buffering, allow-listed flows.
- **IT analytics platform**: data lake/time-series store, semantic store, analytics, dashboards.

**Data flow default:** OT → DMZ/edge → IT (one-way where feasible). Any IT → OT path must be explicitly governed.

### 4.2 Actuation postures (policy)
Your override: `actuation_posture = read-only-by-default`.

#### A) Read-only-by-default (default)
Allowed:
- dashboards, diagnostics, FDD, anomaly alerts
- **setpoint proposals** as recommendations only
Not allowed:
- automated write-back to controllers

#### B) Limited-writeback (tightly governed)
Allowed only when:
- specific buildings are approved as “writeback-enabled”
- commands are restricted to a **small allow-list** (e.g., schedule changes, setpoint bounds)
- changes are **time-boxed** to approved windows
Required controls:
- approval workflow (two-person rule for high-risk changes)
- automatic rollback plan (see below)
- session logging and command audit trail

#### C) Closed-loop optimized (exceptional)
Only with:
- formal safety case and hazard analysis
- canary deployment (one building/system first)
- continuous monitoring of guardrails (comfort/IAQ/safety)
- immediate rollback on anomaly

### 4.3 “Propose → approve → execute” workflow
1. Analytics produces a **tuning proposal** (what/why/expected effect/risk).
2. Proposal reviewed by site ops + OT engineer; security checks scope.
3. If approved, proposal becomes a **change request** with:
   - command allow-list validation
   - time window
   - rollback procedure
   - monitoring plan
4. Execution performed by authorized operator (or automated executor in limited-writeback mode).
5. Post-change verification and evidence pack stored.

### 4.4 Rollback and safety constraints (minimum)
- Store pre-change configuration snapshot (or equivalent export) where possible.
- Define **safe defaults** for comms outage (e.g., revert to local schedules; controller autonomy; conservative setpoints).
- Enforce hard bounds:
  - temperature setpoints within policy-defined min/max
  - ventilation minimums (where applicable)
  - rate limits for changes (no rapid oscillations)

### 4.5 Vendor remote access governance
Minimum requirements (audit-ready):
- MFA + device posture checks
- jump host/bastion in OT DMZ (no direct inbound to OT)
- time-bound access approvals with ticket numbers
- session recording/logging and periodic review
- least-privilege accounts; no shared vendor accounts

Implementation anchor sources include NIST ICS security guidance and CISA remote access practice guides (see Sources).

---

## 5) Semantic layer implementation playbook (Haystack/Brick)
### 5.1 Pragmatic adoption patterns (options-not-prescriptions)
- **Project Haystack**: strong tagging conventions for points/equipment; widely used in operational tagging and integration.
- **Brick**: ontology with explicit classes/relationships; supports richer reasoning and formal validation.

Practical city approach:
- Start with **Haystack-like tags** for fast onboarding and operator usability.
- Introduce **Brick relationships** for the subset of buildings where system topology is needed (AHU→VAV→zone, heat plant→loops).

### 5.2 Identity reconciliation rules
Reconciliation is a deterministic pipeline with human review:
1. **Building match**: address/site ID → `building_id`.
2. **Equipment match**:
   - primary: CMMS asset IDs / serials
   - secondary: naming patterns + location + network IDs
3. **Point match**:
   - map BMS point path/name to `point_id`
   - store vendor source string in `source_ref`
4. **Meter match**:
   - utility account/meter ID → `meter_id`
   - align to building/system using known metering topology

**Rule:** keep a crosswalk table `{source_system, source_identifier} → {canonical_id}` and never overwrite history; add new mappings as systems evolve.

### 5.3 Minimum semantic coverage matrix (targets)
Define coverage by building tier/use case. Example minimums (tune to your portfolio):

| Tier | Intended use cases | Minimum data | Minimum semantic coverage target |
|---|---|---|---|
| **Tier A (FDD + optimization)** | FDD, diagnostics, tuning proposals | key HVAC equipment points + schedules + key meters | ≥ **85%** of required points tagged; topology for major airside/waterside |
| **Tier B (benchmarking + anomaly)** | portfolio benchmarking, baselines, major anomalies | whole-building meters + limited key equipment run/status | ≥ **60%** tagging for key equipment + meters |
| **Tier C (basic visibility)** | utility tracking, basic dashboards | whole-building meters + operating hours/schedules | ≥ **30%** (mostly registry + meters + schedules) |

### 5.4 Tagging QA, validation, and drift control
Minimum controls:
- **Automated validators** run on every onboarding and nightly:
  - required tags present for Tier targets
  - units compatibility (°C vs °F, kW vs W)
  - point-type sanity checks (a valve position is 0–100%)
- **Drift detection**:
  - detect new/removed/renamed points from BMS exports
  - compare semantic coverage delta (% change)
- **Human review workflow**:
  - changes above threshold (e.g., >5% point churn) require review
  - maintain a review log (who approved semantic updates)

---

## 6) Measurement & Verification (M&V) and savings governance
### 6.1 Recognized M&V approaches (when to use which)
Use M&V options aligned to intervention type.

| Intervention type | Typical M&V approach | Notes |
|---|---|---|
| Controls tuning / scheduling | Option C (whole-building) or Option B (system-level) | Often feasible with metered data + regression |
| Equipment retrofit (single system) | Option A or B | Isolation with measured key parameters vs full measurement |
| Multi-measure retrofit bundle | Option C | Needs baseline adjustments for weather/operations |
| Major remodel / new equipment + interactions | Option D | Calibrated simulation where necessary |

The U.S. DOE FEMP summary of the four options provides a practical public-sector anchor (see Sources).

### 6.2 Baseline normalization
- Weather normalization: degree days or change-point models for heating/cooling sensitivity.
- Operational normalization: schedules, known occupancy proxies (aggregated), exceptional events.
- Document adjustments and keep them reproducible.

### 6.3 Uncertainty reporting (minimum)
For each savings claim:
- report savings as a range (e.g., ± uncertainty band) or confidence interval where feasible
- document meter accuracy assumptions and model error metrics
- state what was **not measured** and where engineering estimates were used

### 6.4 Baseline reset governance
Baseline reset is an explicit governance action.
Reset triggers (examples):
- major retrofit affecting load shape
- significant schedule/operational policy change (e.g., extended hours)
- meter replacement or major sensor topology change

Policy:
- preserve prior baseline for historical comparability
- start new baseline period with documented rationale and sign-off (energy lead + facilities)

### 6.5 Evidence artifacts and audit trail
Required artifacts for Tier A/B/C buildings when claiming savings:
- intervention record (what/where/when)
- pre/post data snapshots (meters + relevant points)
- analysis method description + parameters
- QA checks and reviewer sign-off

---

## 7) Occupancy / IAQ privacy model (portfolio-ready)
### 7.1 Data minimization rules
- Prefer **aggregated counts/indices** over raw identifiers.
- Avoid persistent tracking of individuals (no device-level surveillance).
- Store only what is necessary for comfort/IAQ and energy outcomes.

### 7.2 Access tiers
| Tier | Audience | Example access |
|---|---|---|
| A | Building operators | building-level and zone-level aggregates needed for operations |
| B | Analysts | anonymized/aggregated datasets; no raw identifiers |
| C | Public | building-level summaries only; suppression rules applied |

### 7.3 Aggregation thresholds (minimum policy)
- Spatial: no reporting below a minimum zone size / minimum group size threshold.
- Temporal: publish at coarse granularity (e.g., hourly/daily) unless operational need.

### 7.4 Sensitive facilities (schools, hospitals)
- stricter default: only building-level or coarse zone-level aggregates
- tighter access controls and auditing
- explicit notice/consent patterns aligned to local law and facility policy

### 7.5 Prohibited uses and enforcement
Prohibited:
- employee/student surveillance
- disciplinary decisions based on occupancy analytics

Enforcement:
- mandatory logging and periodic access reviews
- policy checks in data access layer (role-based + purpose limitation where feasible)

NIST privacy risk management framing can be used as a governance anchor (see Sources).

---

## 8) Portfolio scaling economics and onboarding strategy
### 8.1 Onboarding effort drivers
Break down work per building:
1. **Connectivity** (network paths, protocol gateways, security uplift)
2. **Semantic mapping** (tagging, equipment templates, topology)
3. **Field verification** (spot checks, sensor/unit validation)
4. **Cybersecurity uplift** (segmentation, remote access hardening)
5. **Operations integration** (work order linkage, training)

Typical cost drivers (qualitative, tune with your data):
- Tier C is dominated by connectivity + metering alignment.
- Tier B adds moderate semantic mapping.
- Tier A is dominated by semantic mapping + field verification + FDD tuning.

### 8.2 Prioritization rubric
Score buildings on:
- ROI potential (EUI high, known issues, large floor area)
- data readiness (meter availability, BMS access, point list quality)
- criticality (sensitive facilities, resilience hubs)
- archetype repeatability (similar HVAC/metering patterns)
- security feasibility (can we meet OT controls quickly?)

### 8.3 Recommended “start set” archetypes
Start with 8–15 buildings that maximize repeatability:
- one common HVAC pattern per cohort (e.g., AHU+VAV schools)
- consistent metering availability
- stable operations team willing to participate

### 8.4 Staffing model assumptions
- City product owner (facilities/energy)
- semantic modeler (Haystack/Brick)
- OT integration engineer
- security engineer (shared)
- analyst for baselines/M&V
- vendor/integrator capacity for BMS exports and on-site work

---

## 9) Operational runbooks (minimum)
### 9.1 Onboarding a new building
1. Register building identity + metadata.
2. Confirm meters and utility accounts; ingest history.
3. Obtain BMS export/point list; establish OT→DMZ→IT data path.
4. Map equipment + points; apply templates; compute coverage.
5. Run validators; fix units/tags.
6. Establish baseline + initial dashboards.
7. If Tier A: enable FDD rules/models; define fault-to-work workflow.

### 9.2 Semantic drift detected (point rename/removal)
1. Alert triggers on point churn/coverage drop.
2. Freeze downstream model updates.
3. Reconcile point changes; update crosswalk mappings.
4. Re-run validators; require human sign-off.
5. Backfill or annotate missing data intervals.

### 9.3 Suspected OT security incident / unauthorized control change
1. Switch to read-only posture; disable writeback executors.
2. Review remote access sessions and controller change logs.
3. Isolate affected segments; rotate credentials.
4. Restore known-good configs; verify safe operation.
5. Post-incident review and control improvements.

### 9.4 M&V discrepancy investigation (savings claim challenged)
1. Confirm intervention dates and scope.
2. Recompute baseline with documented parameters.
3. Check meter integrity and data gaps.
4. Run sensitivity checks (weather/occupancy adjustments).
5. Publish revised savings range and document changes.

---

## 10) Key metrics (portfolio SLIs/SLOs)
### Data and semantics
- telemetry freshness (% points on-time per day)
- completeness (% expected points present)
- semantic coverage by tier (% tagged vs required)
- semantic drift rate (point churn per month)

### FDD quality
- validated fault confirmation rate (proxy for precision)
- time-to-triage, time-to-resolution
- recurrence rate after close

### Savings and performance
- weather-normalized EUI trend
- savings persistence (post-intervention decay)
- uncertainty bands for savings claims

### OT security posture
- patch latency for key OT assets
- remote access sessions: % time-bound + recorded
- number of unauthorized change attempts detected

---

## 11) Implementation roadmap
### 0–3 months
- connectivity inventory + OT boundary documentation
- canonical registry stand-up
- meter ingestion + baseline dashboards
- pilot semantic mapping for 3–5 buildings (read-only)

### 3–12 months
- scale onboarding with templates and validators
- introduce FDD for Tier A cohort
- establish M&V process + baseline reset governance
- implement limited-writeback governance for 1–2 pilot buildings (optional)

### 12–24 months
- portfolio optimization and retrofit pipeline integration
- expand topology-rich modeling where needed
- evaluate closed-loop only with safety case; otherwise stay in limited-writeback/read-only

---

## 12) Risks and mitigations
| Risk | Example | Mitigation |
|---|---|---|
| Automation bias / unsafe setpoint changes | staff trusts model blindly | default read-only; propose→approve workflow; bounds + rollback |
| Vendor remote access compromise | credential theft, lateral movement | jump host, MFA, time-bound access, session recording |
| Privacy harms | occupancy used for surveillance | minimization, access tiers, aggregation thresholds, audits |
| Savings inflation / gaming | baseline manipulation | governed baseline resets, uncertainty reporting, peer review |

---

## Sources (high-signal anchors)
- Brick Schema — Introduction. https://brickschema.org/ — Brick is an open-source ontology for consistent semantic descriptions of building assets and their relationships.
- NIST SP 800-82 Rev. 2 — Guide to Industrial Control Systems (ICS) Security (PDF). https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-82r2.pdf — Guidance on securing ICS environments including segmentation concepts and risk management relevant to BMS/OT.
- CISA — Configuring and Managing Remote Access for Industrial Control Systems (PDF). https://www.cisa.gov/sites/default/files/2023-01/RP_Managing_Remote_Access_S508NC.pdf — Practical recommended practices for governing and hardening remote access into control environments.
- U.S. DOE FEMP — Measurement and Verification Options for Federal Energy- and Water-Saving Projects. https://www.energy.gov/femp/measurement-and-verification-options-federal-energy-and-water-saving-projects — Summarizes four widely used M&V options (A–D) and their applicability.
- NIST — Privacy Framework. https://www.nist.gov/privacy-framework — A risk management framework for identifying and managing privacy risks; supports minimization and governance patterns.
