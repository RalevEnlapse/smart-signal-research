# Data quality SLAs — Deep research

## Executive summary

A City Digital Twin is a **decision-support system**. If the underlying data products degrade silently (missingness, latency, schema drift, sensor bias, geometry invalidity, clock drift), the twin becomes operationally risky: alerts misfire, models drift, dashboards mislead, and confidence collapses.

**Data quality SLAs** (and their operational counterpart, SLOs/SLIs) convert “trust” into enforceable, measurable commitments for each **data product** (streams, registries, event logs, external feeds). This document turns generic SLA concepts into a city-twin-ready playbook with:

- Dataset **tiering (T0–T3)** with enforcement posture and default targets by data type.
- **Accuracy under weak/partial ground truth** (audits, cross-source triangulation, uncertainty).
- **Geospatial and temporal semantics** as first-class SLA dimensions (CRS consistency, geometry validity/topology, event-time vs processing-time, clock sync/drift).
- **Exception lifecycle governance** (time-boxed waivers, compensating controls, “quality debt”).
- **Remediation playbooks + cross-vendor/cross-agency RACI**, plus downstream coupling (error budgets, gating, safe degradation).

Standards anchoring the dimension taxonomy include ISO/IEC 25012 (data quality model) and ISO 19157 (geographic data quality). Time synchronization considerations reference NTP (RFC 5905) and IEEE 1588 PTP; geospatial validity references OGC Simple Features / ISO 19125 and practical validity checks such as PostGIS `ST_IsValid`.

## Why this theme matters for a City Digital Twin (and how it helps you run it)

### Why you need it

A city twin integrates heterogeneous producers (IoT vendors, utilities, enterprise systems, third-party APIs) into shared products used for:

- operational response (minutes)
- field work planning (hours–days)
- budgeting and capital planning (months–years)
- public transparency dashboards

Without explicit quality contracts, failures default to **blame games** (“the data is wrong”) and expensive rework.

### How it helps you run the twin (practical operational impact)

- **Fitness-for-purpose by tier:** different use cases get different promises (freshness vs completeness vs accuracy).
- **Incident-ready quality operations:** violations become incidents with owners, escalation, and runbooks.
- **Model stability:** schema/behavior drift is detected early and coupled to retraining/recalibration gates.
- **Vendor accountability:** contracts tie payment, credits, and acceptance testing to measurable outcomes.
- **Safe degradation:** systems fall back to safe defaults when quality drops (rather than quietly producing wrong answers).

## 1. Definitions and scope boundaries

### 1.1 Data product (unit of accountability)

A **data product** is a named, versioned interface with explicit contracts:

- identifier: `domain.product.vX` (e.g., `transport.signals.phase_state.v1`)
- owner + on-call
- consumers (systems/use cases)
- schemas + semantics
- SLIs/SLOs + reporting window
- exception policy and change policy

SLA scope is per product, not per pipeline component.

### 1.2 SLI / SLO / SLA

- **SLI**: measured indicator (e.g., p95 event-time lateness ≤ 30s).
- **SLO**: target for an SLI over a window (e.g., 99% of 5-minute windows per month meet lateness target).
- **SLA**: contractual agreement including remedies, exclusions, and governance.

### 1.3 Data quality dimensions (anchor taxonomy)

Use ISO/IEC 25012 as a baseline for data quality characteristics, then extend with city-twin-specific space/time semantics.

For geospatial products, align quality reporting to ISO 19157 categories (completeness, logical consistency, positional accuracy, temporal quality, thematic accuracy, usability) where applicable.

## 2. Dataset tiering (T0–T3) and enforcement posture

Tiering prevents a single “gold standard” from blocking useful work while ensuring operational safety.

### 2.1 Tier definitions

| Tier | Name | Typical use | Failure impact | Default enforcement posture |
|---:|---|---|---|---|
| T0 | Safety / control-adjacent | incident response, safety alerts, actuation-adjacent decisioning | can cause harm or major disruption | **Hard gate** at ingress + serve; paging; no long-lived exceptions |
| T1 | Operational | work orders, dispatch, near-real-time ops dashboards | material operational cost | **Gate for critical consumers**, degrade non-critical; paging during hours |
| T2 | Planning / analytics | weekly/monthly planning, performance reporting | bad decisions over time | soft gates; issue tickets; weekly review |
| T3 | Exploratory / research | prototypes, experimentation | limited | best-effort; annotate quality; no paging |

### 2.2 Tier assignment rules

Assign tier using the strictest downstream consumer.

- If a product feeds automated alerting that triggers field dispatch → at least **T1**.
- If a product influences safety-critical decisions (e.g., evacuation routing, structural hazard alerts) → **T0**.

Tier changes require a lightweight ADR and consumer sign-off.

## 3. SLA dimensions and city-twin-specific SLIs

### 3.1 Core dimensions (generic)

| Dimension | What it means operationally | Example SLIs |
|---|---|---|
| Availability | can consumers access the product | API uptime; successful query ratio |
| Freshness / timeliness | data arrives in time for the use case | p95 ingest latency; max staleness |
| Completeness | expected fields/records/coverage exist | % missing required fields; spatial coverage % |
| Validity | conforms to schema and rules | schema-pass rate; range-check pass rate |
| Consistency | no contradictions across systems/time | cross-source mismatch rate; referential integrity |
| Accuracy (when measurable) | closeness to ground truth | audited error rate; calibration drift |
| Lineage/traceability | can we explain provenance | % records with source + processing version |

### 3.2 Geospatial semantics (mandatory for spatial products)

Add explicit SLIs for:

- **CRS correctness and consistency** (e.g., EPSG codes recorded; no mixed CRS in a layer).
- **Geometry validity** (e.g., OGC validity rules; no self-intersections).
- **Topology constraints** (domain-dependent):
  - parcels must not overlap
  - road centerlines must not self-cross incorrectly
  - address points must fall within jurisdiction boundary
- **Positional accuracy** (tier-dependent): audit against higher-trust sources.

Practical checks:

- validate geometries using database-native checks (e.g., PostGIS `ST_IsValid`) before publishing.
- measure drift: % features moving > X meters without a corresponding work order / authoritative update.

### 3.3 Temporal semantics (mandatory for event/stream products)

Specify:

- **event_time** vs **processing_time** and which is authoritative for consumers.
- **lateness budget** (event-time lateness distribution, not just ingestion latency).
- **clock synchronization posture** for producers:
  - systems that can: NTP aligned (RFC 5905)
  - high-precision networks (OT/industrial): PTP (IEEE 1588) where justified

Temporal SLIs:

- p95 event-time lateness ≤ X
- % events with event_time in the future (beyond drift tolerance) ≤ Y
- monotonicity violations per device/day ≤ Z

## 4. SLA templates by common city-twin data types

These templates are starting points; calibrate to local reality.

### 4.1 Streaming sensor time series (IoT)

| Dimension | T0 target (example) | T1 target | T2/T3 |
|---|---:|---:|---:|
| Freshness | p95 ≤ 15s | p95 ≤ 60s | best-effort |
| Completeness | ≥ 99.5% points/interval | ≥ 99% | ≥ 95% |
| Validity | ≥ 99.9% schema+range | ≥ 99.5% | ≥ 98% |
| Temporal | p95 lateness ≤ 10s; clock drift ≤ 1s | p95 lateness ≤ 30s; drift ≤ 5s | define but don’t page |
| Accuracy | audited drift ≤ threshold | audited drift ≤ threshold | periodic sampling |

Mandatory product metadata:

- sampling interval, jitter tolerance
- units and calibration date
- device health signals (battery, RSSI, firmware)

### 4.2 Geospatial registries (assets, networks, parcels)

| Dimension | T0/T1 target (example) | T2 target | Notes |
|---|---:|---:|---|
| Geometry validity | ≥ 99.99% valid | ≥ 99.9% | invalid geometries must be quarantined |
| CRS | 100% labeled, consistent | 100% | reject unknown CRS |
| Completeness | ≥ 99% required attrs | ≥ 98% | include attribute completeness by class |
| Topology rules | ≥ 99.9% pass | ≥ 99% | define per layer |
| Freshness | updates published ≤ 24h | ≤ 7d | depends on workflow |

### 4.3 Operational event logs (311, CAD, work orders)

| Dimension | T1 target (example) | T2 target | Notes |
|---|---:|---:|---|
| Freshness | p95 ≤ 5m | p95 ≤ 60m | include end-to-end not just ETL |
| Completeness | ≥ 99% required fields | ≥ 98% | require reason codes for nulls |
| Validity | ≥ 99.5% | ≥ 99% | controlled vocabularies |
| Consistency | mismatch ≤ 0.5% | mismatch ≤ 1% | e.g., status transitions |
| Privacy constraints | 0 critical violations | 0 critical violations | treat privacy as hard gate |

### 4.4 Third-party/external feeds (weather, mobility, imagery)

Treat as **best-effort** unless contractually strengthened.

- publish explicit provider exclusions
- add drift monitors (schema, volume, distribution)
- maintain fallback sources and cache policies

## 5. Measuring accuracy under weak ground truth

Many city contexts lack a single trusted truth (crowdsourcing, inferred occupancy, ML outputs). The SLA must shift from “absolute accuracy” to a **measurable, auditable accuracy program**.

### 5.1 Accepted accuracy measurement strategies

1. **Cross-source triangulation**
   - compare independent sources (e.g., loop detectors vs probe data vs signal controller logs)
   - report disagreement distributions, not just averages

2. **Periodic audits / field sampling**
   - stratified sampling by neighborhood, asset class, time-of-day
   - publish audit protocol, sample size, confidence intervals

3. **Gold subset + holdout**
   - maintain a small curated “high-trust” subset for calibration

4. **Instrumented change events**
   - require work orders / authoritative events to justify large changes

5. **Uncertainty as a first-class output**
   - provide per-record confidence (where feasible)
   - set SLIs for calibration (e.g., confidence reliability curves)

### 5.2 Accuracy SLIs when ground truth is limited

- disagreement rate between independent sources (within tolerance)
- audit pass rate (field checks)
- stability: unexpected step changes per period
- bias signals: systematic errors by geography/demographics (equity lens)

## 6. Exception lifecycle governance (“quality debt” discipline)

Exceptions are sometimes necessary (e.g., sensor outage, vendor API changes), but unmanaged exceptions destroy trust.

### 6.1 Exception types

- **Temporary waiver**: allow serving data that violates a threshold.
- **Compensating-control exception**: allow violation only if a fallback control is enabled.
- **Measurement exception**: SLI is temporarily invalid due to monitoring failure.

### 6.2 Mandatory exception fields

- product, tier, violated SLI/SLO
- start time, expiry time (hard)
- impact assessment (which consumers, what risk)
- compensating controls (required for T0/T1)
- owner and approver
- remediation plan and ETA

### 6.3 Expiry and escalation rules

- T0: max 24–72h waivers, daily executive visibility
- T1: max 7–14 days, weekly governance review
- T2/T3: time-boxed but can be longer with annotated outputs

Track:

- **quality debt balance**: active exceptions weighted by tier and consumer criticality
- **repeat offender** sources/vendors

## 7. Remediation playbooks (standard incidents)

### 7.1 Sensor stream playbooks

**A) Missingness spike**

- Triage: device health (power/comms), gateway, ingestion service, downstream storage
- Mitigations: switch to cached/last-known-good for non-safety consumers; suppress alerts; open maintenance ticket
- Verification: completeness back to baseline; no “backfilled fiction” unless explicitly marked

**B) Sensor drift / calibration issue**

- Detect: shift vs peer sensors; distribution drift; audit failures
- Mitigations: quarantine device; apply correction only with documented calibration; retrain models if needed
- Verification: post-fix audit sample

**C) Pipeline lag**

- Detect: lateness budget breach
- Mitigations: scale consumers; reduce noncritical transforms; activate degraded mode
- Verification: end-to-end lag p95 back within target

### 7.2 Geospatial registry playbooks

**A) Geometry validity failure**

- Detect: invalid geometries (self-intersections, rings)
- Mitigations: quarantine features; attempt automated fix only if deterministic and logged; request authoritative correction
- Verification: 100% invalid quarantined; fix rate tracked

**B) CRS mismatch / transform errors**

- Detect: coordinates out-of-bounds; mixed EPSG
- Mitigations: block publish; reproject with controlled pipeline; add contract tests

**C) Topology rule violations**

- Detect: overlaps, gaps, dangling nodes
- Mitigations: rule-specific fixes; coordinate with authoritative editor; publish “known issues” layer if needed

### 7.3 Event log playbooks

**A) Schema change from source system**

- Detect: schema contract test fail; new enum values
- Mitigations: stop-the-line for T0/T1 consumers; map new values; backfill with explicit versioning

**B) Status transition inconsistencies**

- Detect: invalid transitions, duplicates
- Mitigations: dedupe rules; enforce append-only; reconcile with system-of-record

## 8. Ownership, RACI, and cross-boundary operations

### 8.1 Minimum roles

- **Data Product Owner (DPO)**: accountable for SLAs and prioritization.
- **Platform/Data Engineering**: pipelines, checks, monitors.
- **Domain Steward**: semantics, business rules, topology rules.
- **Vendor Operator** (if applicable): device/network/source system support.
- **Incident Commander**: coordinates major incidents.

### 8.2 RACI (template)

| Activity | DPO | Data Eng | Domain Steward | Vendor | Gov Council |
|---|---|---|---|---|---|
| Define SLIs/SLOs | A | R | R | C | C |
| Approve Tier | A | C | C | C | I |
| Monitor + alert | C | A/R | C | C | I |
| T0/T1 exception approval | A | R | C | C | C |
| Remediation execution | C | R | R | R | I |
| Post-incident review | A | R | R | C | I |

## 9. Downstream coupling: error budgets, gating, and safe degradation

### 9.1 Error budgets for data products

Translate SLIs into “allowed badness” over a window, per tier.

Examples:

- T0 sensor stream: ≤ 0.5% missing intervals per month; if exceeded → stop enabling new consumers; trigger hardening work.
- T1 geospatial registry: ≤ 0.01% invalid geometries; if exceeded → block publish and require fix.

### 9.2 Gating rules

- **Ingress gate**: quarantine records that violate hard constraints (schema, privacy, CRS unknown).
- **Serve gate**: consumers can request `quality>=T1` views that exclude quarantined/low-quality partitions.
- **Model gate**: block retraining or release if training data quality falls below thresholds.

### 9.3 Safe degradation patterns

- degrade dashboards: show “data stale/partial” banners with last-good timestamp
- degrade analytics: widen uncertainty bands; suppress fine-grained claims
- degrade automation: require human confirmation; disable auto-dispatch

## 10. Implementation guidance (operating model)

### 10.1 Quality as code

- contract tests in CI/CD for schemas and critical semantics
- versioned rulesets and thresholds
- reproducible metric definitions (no “dashboard math”)

### 10.2 Monitoring

- publish SLI dashboards per product and tier
- alerting based on **user-impacting windows** (avoid paging on benign noise)
- track false positives/negatives explicitly

### 10.3 Continuous improvement loop

- monthly SLA review: retire useless metrics, tighten targets where stable
- quarterly audits for T0/T1
- annual re-tiering and consumer review

## 11. Costs and FinOps considerations

Quality work has compute/storage cost (profiling, anomaly detection, retention) and human cost (on-call, audits). Manage it as a product investment:

- tier-based monitoring intensity (T0 highest)
- sampling where full checks are expensive
- cost attribution: charge quality compute to products/owners

## 12. KPIs

- % T0/T1 products with complete SLI coverage
- mean time to detect (MTTD) and mean time to remediate (MTTR) for quality incidents
- active exception count and weighted quality debt
- repeat incidents by source/vendor
- consumer trust: adoption + complaint rate

## 13. Deliverables and checklists

### 13.1 One-time deliverables

- data product inventory + tier assignments
- SLI/SLO catalog (by product)
- exception policy + workflow + tooling
- runbook library (playbooks above)
- contract test suite integrated into CI/CD

### 13.2 Recurring deliverables

- weekly quality ops review (T0/T1)
- monthly SLA performance report + exception debt ledger
- quarterly audit report (sampling protocol + results)

## 14. References

### 14.1 Workspace source

- [`kali-task-research.md`](../kali-task-research.md:1) — Item 21: Data quality SLAs

### 14.2 External references (retrieved via Firecrawl MCP)

- ISO/IEC 25012 (overview). Retrieved from https://iso25000.com/index.php/en/iso-25000-standards/iso-25012 — Data quality model characteristics for structured data products.
- ISO. *ISO/IEC 25012:2008 — Data quality model* (catalog entry). Retrieved from https://www.iso.org/standard/35736.html
- ISO. *ISO 19157:2013 — Geographic information — Data quality* (Online Browsing Platform). Retrieved from https://www.iso.org/obp/ui/#iso:std:iso:19157:ed-1:v1:en
- IETF. *RFC 5905 — Network Time Protocol Version 4* (NTPv4). Retrieved from https://datatracker.ietf.org/doc/html/rfc5905
- OGC. *Simple Feature Access — Part 1: Common Architecture* (ISO 19125). Retrieved from https://www.ogc.org/standards/sfa/
- PostGIS. *ST_IsValid*. Retrieved from https://postgis.net/docs/ST_IsValid.html — Validity checks for geometries according to OGC rules.

### 14.3 Suggested further reading (not fetched)

- DAMA-DMBOK — data management practices and governance
- Great Expectations / Soda / Deequ — “quality as code” tooling patterns
