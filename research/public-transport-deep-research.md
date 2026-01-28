# Public transport — Deep research

## Executive summary
A public transport “twin” is a **decision-support system** for transit operations and service planning. It integrates operational supply (routes, trips, blocks, headways, crew/vehicle constraints), demand signals (APC, fare aggregates, station/stop counts), and disruptions (incidents, closures, maintenance, weather) to improve:
- **reliability** (headway regularity and predictable travel time)
- **crowding** (load distribution and capacity constraints)
- **passenger information trust** (stable, truthful ETAs and service alerts)
- **equity and accessibility reliability** (service quality by place and protected-class proxies in a privacy-safe way)

The twin is not a magic optimizer. It must be operationally realistic about:
- labor agreements, break rules, operator discretion and safety
- what dispatch/OCC can actually instruct and enforce
- the difference between **advice** and **actions** (especially stop-skipping and short-turns)
- data gaps and noisy telemetry that can cause “ETA thrashing” and erode trust

This document deepens item 4 in [`kali-task-research.md`](kali-task-research.md:1): *“Public transport: Simulate routes, headways, capacity, and disruptions to improve reliability and passenger experience.”*

---

## 1) Scope, decisions, and non-goals (operationally anchored)

### 1.1 Decisions the twin supports
**Operations Control Center (OCC) / dispatch (real-time):**
- holding / headway management at control points
- dispatch spacing adjustments (where allowed)
- incident response support: detours, turnbacks, short-turn proposals
- passenger information: consistent ETAs and service alerts

**Service planning (tactical/strategic):**
- schedule and runtime adjustments
- stop/station capacity and crowding mitigation
- scenario evaluation for infrastructure measures (bus lanes, signal priority)

### 1.2 Non-goals (explicitly not automated without separate governance)
- automated stop-skipping and short-turn execution without human approval
- automated passenger messaging that contradicts OCC/CAD decisions
- individual-level passenger targeting or per-person trip tracing

---

## 2) Constraint model for labor + safety (explicit “constraints library”)
Transit control is constrained optimization. Encode the constraints explicitly, version them, and subject changes to governance.

### 2.1 Constraints library (parameterizable)
Define a shared library of constraints, keyed by route/mode/garage/crew agreement.

**Holding constraints**
- `max_hold_seconds` (global and by control point)
- `max_hold_as_percent_of_headway`
- “no-hold if behind schedule beyond X” (policy-dependent)

**Stop-skipping constraints**
- `prohibited_stops[]`:
  - ADA/accessible stops
  - major transfer points
  - schools/hospitals (policy)
  - stops with active accessibility incidents
- maximum skipped stops per trip
- “never skip last stop before major transfer” (policy)

**Short-turn constraints**
- allowable turnback locations (infrastructure)
- minimum passenger offload/communication requirements
- crew relief and operator sign-off requirements

**Break / layover / duty constraints**
- minimum layover time
- break timing windows
- maximum duty time
- deadhead constraints

**Operator discretion boundaries**
- what the operator may override
- what must be acknowledged vs confirmed

### 2.2 Governance for changing constraints
A realistic process:
1. **Draft change** (ops engineering + planners) with rationale.
2. **Union consultation** (where applicable) and operations leadership review.
3. **Safety sign-off** (risk assessment; training impact).
4. **Pilot** (shadow → advisory → limited corridor) with measurement plan.
5. **Rollout** with rollback criteria and post-implementation review.

All constraint changes must be:
- versioned
- auditable (who approved)
- testable in simulation and in controlled pilots

---

## 3) Passenger data governance (privacy-by-design)
Passenger telemetry is high re-identification risk when fine-grained. Default to aggregation and purpose limitation.

### 3.1 Data classes and access tiers
**Data classes**
- **Operational telemetry**: vehicle positions, trip updates, dwell events
- **Passenger counts**: APC counts by stop/trip/time bucket
- **Fare validation aggregates**: taps by station/route/time bucket
- **App/RTPI usage**: query volumes, error rates (not user-level)

**Access tiers**
- **Tier A (OCC/ops)**: operational and aggregated passenger counts; no person-level data
- **Tier B (planning/analytics)**: aggregated demand/crowding; limited joins
- **Tier C (privacy/security/legal)**: audit logs; access-by-case
- **Vendors**: least privilege, scoped to contracted services, expiring access

### 3.2 Aggregation thresholds and publishability rules
- No fine-grained trip traces in public outputs.
- Apply minimum cell sizes (example policy):
  - internal analytics: `k >= 11`
  - public reporting: `k >= 25`
- Coarsen space/time for public equity reporting (e.g., 30–60 min buckets; neighborhood level; no stop-level release where sparse).

### 3.3 Minimization and purpose limitation
- Collect only what is needed for reliability/crowding/equity metrics.
- Prohibit secondary use (e.g., law enforcement targeting) without separate legal basis and oversight.

### 3.4 Retention windows (starter)
- Raw high-granularity operational telemetry: short hot retention (e.g., 7–30 days)
- Aggregated passenger metrics: longer (e.g., 6–24 months) for planning and equity trend analysis
- Audit logs: longer (e.g., 1–2 years) with integrity controls

### 3.5 Re-identification risk mitigation
- Avoid releasing linked origin-destination traces.
- Limit joins (e.g., do not join fare + app + precise AVL at individual level).
- Conduct periodic re-identification risk reviews for new datasets.

---

## 4) Decision boundaries + HITL tiering (advisory-by-default)
Define what is advice vs what triggers actions, and who can approve.

### 4.1 Evidence packet requirement (for any intervention suggestion)
Every recommendation must carry an evidence packet:
- `route_id`, `trip_id/block_id`, `vehicle_id`
- `timestamp`, `last_updated`, telemetry freshness
- headway state (ahead/behind, variance)
- predicted impact (passenger delay, crowding)
- confidence/uncertainty and failure flags (missing AVL, detour active)
- constraints applied (which rules bound the suggestion)

### 4.2 Risk-tiering table
| Intervention | Risk level | Default mode | Required approver | Rollout pattern | Notes |
|---|---:|---|---|---|---|
| Holding at control point | 1 | advisory | OCC supervisor | shadow → advisory → pilot | bounded by max hold |
| Headway management prompts | 1 | advisory | OCC supervisor | advisory → pilot | must respect operator discretion |
| Signal priority request | 2 | advisory | OCC + TMC approver | pilot by corridor | constrained by street ops approvals |
| Short-turn proposal | 3 | advisory only | operations manager / incident commander | shadow → limited pilot | high passenger impact |
| Stop-skipping | 3 | advisory only | operations manager | pilot only | prohibited stops and accessibility constraints |
| Automated passenger messaging | 2 | limited automation with HITL | comms + OCC | pilot | must avoid contradictions across channels |

### 4.3 Rollback criteria and incident mode
Rollback triggers:
- increased customer complaints
- increased missed connections / crowding
- safety incidents
- evidence packet quality failures (stale data, missing confidence)

Incident mode policy:
- during major disruptions, broaden human authority but tighten logging and audit.

---

## 5) PIS integration: truthfulness and stability
Passenger Information Systems (PIS) are trust systems. “ETA thrashing” destroys credibility.

### 5.1 Consistency guarantees across channels
Define a “single source of truth” service for passenger-facing messaging:
- apps/web, station/stop signs, onboard announcements consume the same message object
- versioned messages with `last_updated` and confidence
- no contradictory messages across channels

### 5.2 Data contracts for ETA and alerts
For ETA APIs:
- include `generated_at`, `valid_until`, `confidence_band`
- include `data_freshness` and degraded-mode flags
- define behavior when telemetry is missing (fallback to schedule)

For service alerts:
- stable identifiers, start/end validity windows
- clear severity levels

### 5.3 ETA stability (anti-thrashing)
- smoothing rules (e.g., bounded rate of change)
- hysteresis for switching between model vs schedule fallback
- degrade gracefully: show wider uncertainty rather than rapidly changing numbers

### 5.4 Caching and degraded-mode behavior
- edge caching for station/stop signs
- offline signage behavior: show last-known + “may be stale”

---

## 6) Equity & accessibility measurement
Equity must be measured and must constrain optimization objectives.

### 6.1 Privacy-safe equity metrics
Report service quality by neighborhood and demographic proxies (policy-approved):
- headway regularity by neighborhood
- crowding exceedance rates
- travel time reliability
- missed connection proxies

### 6.2 Accessibility reliability as equity
Track and report:
- elevator/escalator outages (where applicable)
- stop accessibility disruptions
- “accessible trip success” proxy (percentage of trips with functioning accessible path)

### 6.3 Equity constraints in optimization
- add constraints/penalties to avoid shifting unreliability/crowding to disadvantaged areas
- require equity impact review for control strategies

---

## 7) Multi-modal interaction (system coupling)
Transit does not operate in isolation.

### 7.1 Coupling to street operations
- signal priority constraints and approvals (TMC coordination)
- bus lanes and curb enforcement interactions
- incident detours and temporary curb rules

### 7.2 Interaction at stops and curb
- micromobility and ridehail conflicts at stops
- curb availability and dwell time impacts

---

## 8) Operational runbooks (minimum set)
- ETA thrashing event (PIS stabilization + fallback)
- Telemetry outage (AVL missing → degrade to schedule)
- Major disruption (turnbacks/short-turn governance)
- Equity regression detected (pause rollout, review constraints)
- Data privacy incident (suspected re-identification/leak)

---

## 9) Key metrics
- headway adherence and variability by route and neighborhood
- ETA error and calibration; thrashing rate
- telemetry freshness/coverage
- crowding exceedance and distribution
- accessibility reliability (outage minutes)
- intervention outcomes (before/after) with auditability

---

## 10) Implementation roadmap

### 0–3 months
- integration inventory + data contracts
- stable ETA service with degraded mode
- privacy governance for passenger data
- initial equity reporting (aggregated)

### 3–12 months
- advisory headway management with constraints library
- PIS consistency layer and anti-thrashing
- pilot governance process (union + safety sign-off)

### 12–24 months
- expanded multimodal coordination (signal priority, curb)
- mature equity-constrained optimization and continuous audit readiness

---

## References (high-signal sources; starting points)

### GTFS / GTFS-RT and passenger information reliability
- GTFS Realtime Best Practices: [`gtfs.org/documentation/realtime/realtime-best-practices/`](https://gtfs.org/documentation/realtime/realtime-best-practices/) — Practical expectations for freshness and handling of Trip Updates/Vehicle Positions/Service Alerts to keep passenger information reliable.

### Transit operations interventions and headway management
- Daganzo (2009) — A headway-based approach to eliminate bus bunching: [`www.sciencedirect.com/science/article/abs/pii/S0191261509000484`](https://www.sciencedirect.com/science/article/abs/pii/S0191261509000484) — Foundational headway-based holding strategy framing (useful for policy design even if implementation details vary).

### Passenger data privacy and re-identification risk
- TransitCenter — “Do Not Track” fare/payment privacy guide: [`transitcenter.org/wp-content/uploads/2021/03/DoNotTrack_RGB_interactive.pdf`](https://transitcenter.org/wp-content/uploads/2021/03/DoNotTrack_RGB_interactive.pdf) — Transit-focused privacy guidance for fare systems and minimizing passenger tracking.
- Ontario IPC — De-identification Guidelines for Structured Data: [`www.ipc.on.ca/en/media/5946/download`](https://www.ipc.on.ca/en/media/5946/download) — Emphasizes that aggregation is not automatically de-identification and describes risk-aware de-identification practices.

### Equity metrics in transit service
- CNT AllTransit metrics: [`alltransit.cnt.org/metrics/`](https://alltransit.cnt.org/metrics/) — Catalog of transit access and quality metrics, including equity-relevant measures.
