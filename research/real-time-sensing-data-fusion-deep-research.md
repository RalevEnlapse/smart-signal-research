# Real-time sensing & data fusion — Deep research

## Executive summary (medium-length)
Real-time sensing & data fusion is how a “city twin” becomes operational: heterogeneous, high-velocity signals (IoT/telemetry, CCTV-derived metadata, GIS layers, 311/citizen reports, and vendor feeds) are aligned in time and space, reconciled for identity, and fused into **trustworthy state + event products** with explicit freshness, confidence, provenance, and privacy controls.

The hard parts are governance and operability as much as streaming tech:
- **Canonical ontology + entity registry governance** must support merges/splits, controlled evolution, and downstream compatibility.
- **Quality scoring + confidence propagation** must be calibrated and validated to prevent overconfident alerts.
- **Serving semantics** must define how consumers interpret “current state” under eventual consistency, with a **freshness contract** and fallback behavior.
- **Privacy boundaries and minimization** must be enforceable for sensitive sources (notably CCTV-derived metadata, 311, and fine-grained IoT telemetry).
- **Operational ownership + federated on-call** must route incidents cleanly across city agencies and vendors.
- **Testing strategy** must include replay regression, late/out-of-order harnesses, adversarial cases, and synthetic data.

This document is an implementable blueprint for data/platform engineering, domain owners, SRE/ops, governance/privacy stakeholders, and vendors.

---

## 1. Scope and outcomes

### 1.1 Scope
In scope:
- Ingesting and standardizing multi-source sensing streams and periodic feeds.
- Entity resolution into canonical IDs (assets, locations, sensors, incidents).
- Fusion into:
  - **Events** (discrete occurrences with confidence and provenance).
  - **State** (best current estimate, including uncertainty and freshness).
- Serving to operational consumers (dashboards, alerting, automation) with defined semantics.
- Governance: ontology/identity change control, data contracts, privacy boundaries, ownership.
- Operability: SLOs, incident routing, runbooks.
- Testing: replay regression, adversarial/late data, synthetic generation.

Out of scope (for this document):
- Selecting a specific vendor stack.
- External reference discovery (handled in a separate task).

### 1.2 Outcomes (“what good looks like”)
- Consumers can answer for any state/event: **what, where, when (event time), how fresh, confidence, provenance, and privacy classification**.
- Operational alerts are **calibrated** (confidence correlates to reality) with measurable precision/recall.
- Incidents are routed to the right owner (source vs platform vs fusion logic) with predictable MTTR.

---

## 2. Core product model: observation → event + state

### 2.1 Canonical record types
- **Observation (raw or enriched)**: a single report from a source.
- **Derived evidence**: normalized features used by fusion (e.g., “speed drop”, “vehicle stopped”).
- **Event**: discrete occurrence with time bounds, location, severity, and confidence.
- **State**: continuously updated estimate for an entity/feature with uncertainty and freshness.

### 2.2 Minimum fields (must-have)
Every **event** and **state** record MUST carry:
- `canonical_entity_id` (or `canonical_location_id`)
- `event_time` (and `event_time_end` for intervals)
- `processing_time`
- `freshness` (see §5)
- `confidence` (0–1) and/or `uncertainty` (distribution or bounds)
- `provenance` (source(s), transformations, model/rule versions)
- `quality_signals` (validity, completeness, timeliness, consistency)
- `privacy_classification` (policy tier + allowed uses)

---

## 3. Canonical ontology + entity registry governance (incl merges/splits)
This section operationalizes the “canonical IDs” idea into a governed system that can evolve safely.

### 3.1 Canonical ontology: controlled vocabulary and schemas
Deliverables:
- A **canonical ontology** of entity types (e.g., `Sensor`, `Asset`, `RoadSegment`, `Zone`, `Incident`, `WorkOrder`) and event types (e.g., `Collision`, `FloodRiskIncrease`, `IllegalDumpingReport`).
- For each type: required fields, allowed values, semantics, and privacy tier defaults.

Rules:
- Canonical types are versioned (e.g., `ontology_version`).
- Any consumer-facing change requires compatibility analysis (see §7 change control).

### 3.2 Entity registry: responsibilities and data model
The **Entity Registry** is the system of record for canonical IDs and cross-system links.

Minimum capabilities:
- Map `source_entity_id` → `canonical_entity_id` with evidence and match confidence.
- Track **temporal validity** of mappings (`valid_from`, `valid_to`).
- Support **merges** (many → one canonical) and **splits** (one → many canonical).
- Emit an **Identity Change Event** stream.

Recommended registry tables (conceptual):
- `canonical_entities` (id, type, attributes, status)
- `aliases` (source_system, source_id, canonical_id, valid_from/to)
- `identity_events` (MERGE, SPLIT, DEPRECATE, REKEY)
- `match_evidence` (features used, score, reviewer, decision)

### 3.3 Merges, splits, and downstream impact
Identity changes are inevitable (assets re-labeled, road segments re-modeled, vendor IDs shift).

Policy:
- **Merges**: choose a surviving `canonical_entity_id` and mark others as `deprecated` with redirect.
- **Splits**: create new canonical IDs and define the split rule and time boundary.
- **No silent rewrites**: all changes produce `IdentityChangeEvent` with effective timestamps.

Consumer contract:
- Consumers MUST treat canonical IDs as stable identifiers but subscribe to identity change events if they persist or cache.
- Serving APIs MUST provide “resolve” endpoints (old → current) and history.

### 3.4 Governance workflow (human-in-the-loop)
Identity and ontology governance must be executable, not aspirational.

Workflow:
1) Proposal (issue/ticket): new type, field, mapping, merge, split.
2) Automated checks: schema compatibility, impact report, privacy classification.
3) Review:
   - Domain steward validates semantics.
   - Platform steward validates technical fit.
   - Privacy officer validates minimization/access.
4) Approval and scheduling.
5) Release with version + migration plan.

---

## 4. Quality scoring calibration + confidence propagation
This section turns “quality scoring” into a calibrated system that operators can trust.

### 4.1 Quality signals (per observation)
Track at least:
- **Timeliness**: lag vs source timestamp; late-arrival rate.
- **Completeness**: expected vs received count; missingness bursts.
- **Validity**: schema checks, range checks, unit sanity.
- **Consistency**: cross-source agreement (peer sensors) and temporal smoothness.
- **Integrity indicators**: device auth failures, signature failures, unusual source IPs.

### 4.2 Calibration strategy (make scores meaningful)
Quality scores must be tied to outcomes.

Implementable approach:
- Maintain a labeled set of “known-good/known-bad” periods (from incidents, maintenance logs, manual audits).
- Periodically fit a calibration model:
  - Example: map raw metrics → `p(observation_reliable)` via isotonic regression or logistic calibration.
- Validate calibration with reliability plots (predicted vs observed error rates).

Operational cadence:
- Calibrate monthly (or after major sensor firmware/vendor changes).
- Recalibrate immediately when drift alarms fire (see §8 observability).

### 4.3 Confidence propagation into fused events/state
Define a consistent confidence model so different teams don’t invent ad-hoc meanings.

Recommended semantics:
- `quality_score` answers: “How reliable is this observation channel right now?”
- `confidence` answers: “How likely is the inferred event/state to be true given evidence?”

Propagation rules (implementable):
- Each fusion rule/model must declare:
  - Inputs required and their minimum quality thresholds.
  - How confidence is computed (e.g., Bayesian update, weighted evidence, ensemble).
  - How uncertainty is represented for state (intervals, distributions).
- Confidence must never increase without additional evidence or improved quality.

Guardrails:
- Cap confidence if evidence diversity is low (e.g., only one source type).
- Penalize stale evidence using freshness decay.

---

## 5. Serving semantics: event + state under eventual consistency with a freshness contract
Consumers need a precise model of what “current” means.

### 5.1 Dual-plane serving: events and state
- **Event plane (append-only)**: immutable stream of detected/reported events with possible corrections.
- **State plane (mutable)**: latest estimate per entity/feature, derived from events and observations.

Rule:
- Events are the audit trail; state is the convenience view.

### 5.2 Event semantics (corrections and idempotency)
- Events must be idempotent via stable `event_id`.
- Corrections use explicit event types, e.g. `EventCorrected` / `EventRetracted`, never silent mutation.

### 5.3 State semantics (eventual, but bounded by freshness)
Assume **eventual consistency** across regions/services, but publish a **freshness contract**.

State record MUST include:
- `as_of_event_time`: the latest event time incorporated.
- `computed_at`: processing timestamp.
- `freshness`: one of:
  - `FRESH` (within target)
  - `STALE` (beyond target but within max tolerated)
  - `EXPIRED` (beyond max tolerated; treat as unavailable)
- `freshness_target_ms` and `freshness_max_ms` per state product.

Consumer guidance:
- Dashboards show stale banners and last update time.
- Automation (dispatch/routing) must refuse to act on `EXPIRED`.

### 5.4 Eventual-with-freshness-contract: SLO-friendly definition
Define per product:
- **Freshness SLO**: % of reads returning `FRESH` (e.g., 99% in 60s).
- **Completeness SLO**: % of expected entities updated in window.
- **Correction SLO**: max time to emit correction after late data arrives.

---

## 6. Privacy boundaries and minimization (CCTV-derived metadata, 311, IoT)
This section makes privacy enforceable with technical and contractual controls.

### 6.1 Data classification and boundaries
Define tiers (example):
- **Tier 0 (Public)**: safe aggregates and non-sensitive events.
- **Tier 1 (Operational)**: internal operational metadata.
- **Tier 2 (Sensitive)**: potentially identifying or security-relevant.
- **Tier 3 (Restricted)**: highly sensitive (raw video, precise trajectories, PII).

Boundary rule:
- Fusion pipelines must implement **policy enforcement points** that prevent Tier 3 data from flowing into Tier 0/1 products.

### 6.2 CCTV-derived data: minimization rules
Allowed by default (subject to local policy):
- Derived **counts** (e.g., vehicle count per minute per zone).
- Derived **events** with coarse location/time (e.g., “congestion high”, “object on roadway”) without persistent tracking IDs.

Disallowed by default:
- Raw video export into the fusion platform.
- Persistent person- or vehicle-level tracks in general-purpose state stores.

Controls:
- Edge processing for CV where feasible; transmit only necessary metadata.
- Strip or rotate identifiers; avoid cross-camera re-identification.
- Retention: short TTL for CCTV-derived operational metadata unless explicitly approved.

### 6.3 311/citizen reports: minimization and safe fusion
Rules:
- Separate PII from operational attributes at ingestion.
- Coarsen location for public outputs (e.g., block-level) and limit free-text exposure.
- Retention: keep PII only as long as required for case management; fused events should reference case IDs, not personal details.

### 6.4 IoT sensors: sensitive inference controls
Risks include inference from fine-grained location/time patterns.

Controls:
- Default aggregation windows for public/open data.
- Access controls by purpose; prohibit secondary use without review.
- Auditable query logging for sensitive tiers.

### 6.5 Privacy governance artifacts
- Data Processing Agreements (DPAs) with vendors.
- Data Protection Impact Assessment (DPIA) for CCTV/311 fusion products.
- Documented “allowed uses” per product with enforcement in IAM and APIs.

---

## 7. Implementable governance: RACI, SLAs/SLOs, change control, and contracts

### 7.1 RACI (minimum viable)
Roles:
- **Platform Owner** (data platform engineering)
- **Domain Data Owner** (agency owning the operational outcome)
- **Source Owner** (agency/vendor owning devices/systems)
- **Privacy/Security** (DPO/CISO delegate)
- **SRE/Operations** (shared reliability function)
- **Vendor Operator** (if vendor-managed components exist)

RACI matrix (core activities):
- Ontology change approval: A=Domain Data Owner, R=Platform Owner, C=Privacy/Security, I=SRE, I=Vendors
- Entity merge/split approval: A=Domain Data Owner, R=Domain Steward, C=Platform Owner, C=Privacy/Security
- Source onboarding contract (schema + SLAs): A=Platform Owner, R=Source Owner, C=Domain Owner, C=Privacy/Security
- Pipeline availability SLO: A=SRE, R=Platform Owner, C=Domain Owner
- Data quality SLO (freshness/completeness): A=Domain Owner, R=Platform Owner, C=Source Owner
- Incident response/runbooks: A=SRE, R=Platform Owner, C=Source Owner, I=Domain Owner

### 7.2 SLAs/SLOs (examples you can implement)
Per source feed (supplier-facing SLA):
- Uptime % for delivery endpoint
- Max event-time lag (p95/p99)
- Schema stability and deprecation notice period
- Incident notification window (e.g., notify within 15 minutes of outage)

Per fused product (consumer-facing SLO):
- Read latency p95/p99
- Freshness SLO (% `FRESH`)
- Completeness SLO (entity coverage)
- Correctness proxies (agreement with ground truth where available)

### 7.3 Change control (contracts and versioning)
Required controls:
- **Data contracts** for each topic/API: schema, semantics, allowed lateness, retention, privacy tier.
- Versioning policy:
  - Backward-compatible changes allowed without consumer migration.
  - Breaking changes require parallel run, migration window, and deprecation notice.
- Release checklist:
  - Impact analysis, replay test pass, privacy review pass, rollback plan, comms plan.

### 7.4 Federated operating model (multi-agency + vendors)
- Define “platform boundary” vs “source boundary” vs “domain logic boundary”.
- Use explicit ownership tags in metadata: `owner_team`, `oncall_rotation`, `pager_service`.

---

## 8. Operational ownership, federated on-call, and incident routing

### 8.1 Ownership boundaries (what breaks where)
Segment the system into operational components:
- **Source systems/devices**: sensors, CV edge boxes, vendor APIs.
- **Ingestion**: gateways, brokers, schema validation, quarantine.
- **Processing**: enrichment, geospatial joins, entity resolution.
- **Fusion logic**: rules/models producing events/state.
- **Serving**: APIs, caches, dashboards.

### 8.2 Incident routing rules (implementable)
Routing should be deterministic based on symptoms:
- Schema validation failures spike → Source Owner + Platform Owner.
- Broker lag/partition under-replication → Platform Owner + SRE.
- Freshness SLO breach for a single source → Source Owner.
- Confidence calibration drift (over-alerting) → Fusion Owner + Domain Owner.
- Privacy boundary violation (Tier leakage) → Privacy/Security + Platform Owner (severity high).

### 8.3 Runbooks and escalation
Minimum runbooks:
- Pipeline backpressure/lag
- Late-data surge (watermark adjustments, replay)
- Sensor drift / stuck-at detection
- Identity merge/split rollback
- Privacy incident response (containment, audit, notification)

---

## 9. Testing strategy (replay regression, late-data harness, adversarial + synthetic)

### 9.1 Replay-based regression testing
Goal: ensure changes don’t degrade detection, freshness, or confidence calibration.

Approach:
- Maintain curated “golden windows” of historical data (including known incidents and quiet periods).
- For each pipeline/fusion release:
  - Replay windows through a test environment.
  - Compare outputs to baselines: event counts, confidence distributions, state freshness, correction rates.

### 9.2 Late/out-of-order data harness
Goal: validate event-time semantics and correction behavior.

Harness requirements:
- Inject controlled reordering, duplicates, and delays per source.
- Assert:
  - State transitions are correct given allowed lateness.
  - Corrections are emitted within the Correction SLO.
  - No silent state rewrites without provenance updates.

### 9.3 Adversarial and abuse-case testing
Cover threat-model-driven cases (from spoofing/replay/poisoning):
- Replay stale “normal” observations.
- Inject extreme-but-valid values.
- Simulate compromised device identity.
- Attempt cross-tenant data access (vendor/agency separation).

Acceptance criteria:
- Detection of anomalies in pipeline behavior.
- Access denied and audit logs produced.

### 9.4 Synthetic data generation
Use synthetic data to fill gaps in real labels and to test rare scenarios.

Recommended synthetic sets:
- Spatially correlated events (e.g., multi-intersection congestion wave).
- Sensor drift curves and intermittent outages.
- Privacy-safe synthetic 311 text templates (no real PII).

---

## 10. Reference architecture (capability view)

### 10.1 Ingestion and contracts
- Source adapters/gateways with device identity.
- Schema validation and quarantine.
- Contract registry (schemas + semantic metadata + privacy tier).

### 10.2 Processing and enrichment
- Event-time processing with watermarking and allowed lateness.
- Geospatial enrichment (map matching, geofencing).
- Entity resolution calls to registry.
- Policy enforcement points for privacy boundaries.

### 10.3 Fusion and stores
- Event store (append-only + correction stream).
- State store (latest + freshness metadata).
- Feature store (optional) for model-based fusion.

### 10.4 Serving
- Read APIs for state and event queries.
- Subscription/alert APIs.
- Consumer SDK guidance for freshness handling.

---

## 11. Deliverables and success metrics

### 11.1 Deliverables (minimum viable)
- Canonical ontology (versioned) + glossary.
- Entity registry service + merge/split workflow + identity change event stream.
- Per-product freshness contract definitions and SLO dashboards.
- Confidence calibration playbook + monthly calibration report.
- Privacy boundary policy spec + enforcement implementation + DPIA artifacts.
- On-call ownership map + incident routing rules + runbooks.
- Test harness: replay regression + late-data + adversarial + synthetic generators.

### 11.2 Metrics (examples)
- Freshness: % `FRESH` by product, p95 end-to-end lag.
- Data quality: validation pass rate, missingness rate, drift alarms.
- Fusion correctness: precision/recall where labeled; confidence calibration error.
- Ops: MTTD/MTTR, incident volume by ownership boundary.
- Privacy: access denials, policy violations (should be 0), audit log coverage.

---

## 12. References (placeholders; to be populated in a separate task)

### 12.1 Workspace sources
- [`kali-task-research.md`](../kali-task-research.md:1)
- [`analysis/real-time-sensing-data-fusion-deep-research.analysis.md`](../analysis/real-time-sensing-data-fusion-deep-research.analysis.md:1)

### 12.2 Standards and specifications
- **Akidau, Tyler; Bradshaw, Robert; Chambers, Craig; Chernyak, Slava; Fernández-Moctezuma, Rafael J.; Lax, Reuven; McVeety, Sam; Mills, Daniel; Perry, Frances; Schmidt, Eric; Whittle, Sam. _The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing_.** (Google Research, 2015) https://research.google/pubs/pub43864/ — Canonical description of **event time vs processing time**, **watermarks** (and why they are imperfect), **windowing**, **triggers/panes**, and **refinement** via accumulating and **retracting** updates for late/out-of-order data.
- **Apache Spark. _Structured Streaming Programming Guide_ (Spark 3.5.x).** https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html — Covers **event-time windows**, **watermarking**, and the **lateness/state-retention trade-off** (e.g., dropping or diverting late data once a watermark passes); also documents fault-tolerance guarantees including conditions for **end-to-end exactly-once** vs **at-least-once**.
- **Apache Flink. _Timely Stream Processing_ / event time & watermarks (Flink docs).** https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/time/ — Defines watermarks (e.g., `Watermark(t)` meaning no future elements earlier than `t` are expected) and explains how Flink uses them to drive **event-time progress** and reason about **late events** in windowed computations.
- **Apache Beam. _Beam Programming Guide: Windowing_ and _Triggers_ (Beam docs).** https://beam.apache.org/documentation/programming-guide/#windowing — Explains event-time **windowing**, **triggers** (early/on-time/late firings), and how pipelines can emit multiple results for the same window as late data arrives, making correction semantics explicit.

### 12.3 Privacy and governance references
- **European Union. _General Data Protection Regulation (GDPR) — Recitals and Articles_.** https://eur-lex.europa.eu/eli/reg/2016/679/oj — Primary legal source for data protection principles used in governance design, including **data minimization**, **purpose limitation**, and storage limitation; relevant for defining privacy tiers and retention/allowed-use controls for sensor and CCTV-derived metadata.

### 12.4 Security and resilience references
- **NIST. _Zero Trust Architecture (SP 800-207)_.** https://csrc.nist.gov/publications/detail/sp/800-207/final — Authoritative reference for implementing zero-trust controls around ingestion and serving (identity, policy enforcement points, continuous verification), applicable to multi-agency/vendor sensing pipelines.
- **NIST. _Security and Privacy Controls for Information Systems and Organizations (SP 800-53 Rev. 5)_.** https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final — Baseline control catalog for audit logging, integrity, provenance, and resilience controls (e.g., AU, SI, SC families) to make stream-processing and fusion systems operationally and forensically robust.
