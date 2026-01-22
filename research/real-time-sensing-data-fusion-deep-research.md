# Real-time sensing & data fusion — Deep research

## Executive summary (concise)
Real-time sensing & data fusion is the foundation of an operational “city state” view: multiple high-velocity streams (IoT telemetry, computer-vision events, GIS layers, citizen reports, and potentially OT/SCADA signals) are aligned in time and space, reconciled for identity and quality, and fused into coherent entities/events usable for detection and decision-making.

The core technical challenge is not “ingesting data” but **producing trustworthy, time-aligned, semantically consistent state** under noise, missingness, delays, and conflicting observations—while meeting security, privacy, and reliability constraints.

This document deepens the first research item in [`kali-task-research.md`](../kali-task-research.md:1): *“Real-time sensing & data fusion: Combine live IoT, camera, GIS, and citizen-reported streams into a coherent, time-aligned city state.”*

---

## 1. Background and context
The “digital twin” concept (as implied by the source file’s broader list of city twin capabilities) requires a continuously updated representation of real-world assets, locations, conditions, and events.

Real-time fusion is the enabling layer for downstream use cases:
- Operational dashboards and situational awareness
- Event detection (incidents, anomalies, threshold exceedances)
- Simulation inputs (traffic, flooding, energy, etc.)
- Automated or semi-automated decisions (dispatch, rerouting, mitigation)

The source’s first research item already outlines:
- **Keywords:** real-time telemetry, data fusion, sensor streams, event detection, spatiotemporal alignment, edge-to-cloud, situational awareness
- **Typical data sources:** IoT sensors (traffic, weather, noise), CCTV/computer vision feeds, GIS basemaps, 311/citizen reports, SCADA/OT signals
- **Methods:** streaming ETL, spatiotemporal joins, entity resolution, sensor calibration, anomaly detection, probabilistic fusion

This document turns those bullets into an actionable blueprint.

---

## 2. Problem statement
Build a system that can:
1) Ingest multiple heterogeneous, high-volume streams and batch feeds.
2) Normalize them into a shared schema/ontology and consistent identifiers.
3) Align observations in **time** (latency, clock drift, out-of-order arrival) and **space** (coordinates, map matching, geofences).
4) Resolve “who/what is this?” across systems (entity resolution).
5) Fuse competing or complementary observations into:
   - A **current state** for relevant entities (e.g., intersection congestion, asset health, rainfall intensity)
   - An **event stream** (e.g., “crash detected”, “flood risk rising”, “illegal dumping report”) with confidence and provenance.
6) Expose state/events to consumers with strong guarantees:
   - Traceability/provenance
   - Data quality indicators
   - Security and privacy controls
   - Reliability (SLOs)

### What “good” looks like
- A consistent “city state” where each state fact answers: **what, where, when, who observed, confidence, and lineage**.
- Minimal false positives/negatives for operational alerts.
- Observability of the fusion pipeline: drift, freshness, completeness, and latency.

---

## 3. Key concepts (practical definitions)

### 3.1 Stream vs state vs event
- **Stream:** raw or enriched observations arriving over time (sensor readings, detections, reports).
- **Event:** an inferred or reported occurrence (often discrete) with severity, location, and time bounds.
- **State:** the system’s current best estimate of a variable/entity (e.g., “road segment speed = 18 km/h”, “pump status = ON”).

### 3.2 Spatiotemporal alignment
- **Time alignment:** assign observations to a time window considering drift/latency.
- **Spatial alignment:** consistent coordinate reference system, map matching to road segments or assets, and applying geofences.

### 3.3 Entity resolution
Match disparate identifiers to a canonical entity:
- “Camera #A12”, “Intersection 5th & Pine”, and “GIS node 18392” may refer to the same location.

### 3.4 Fusion strategy spectrum
- **Rule-based fusion:** thresholds, heuristics, deterministic precedence.
- **Probabilistic fusion:** treat observations as evidence; compute posterior confidence.
- **Learning-based fusion:** models that learn how to reconcile sensors given context.

---

## 4. Threat model and security/resilience considerations
Even though the workspace file is not explicitly “Kali” security research, it includes security-oriented themes elsewhere (e.g., cybersecurity & resilience). Real-time sensing and fusion sits at a high-value trust boundary.

### 4.1 Assets to protect
- Integrity of sensor readings/events (prevent false state)
- Availability of ingest/fusion pipeline (avoid blind spots)
- Confidentiality of sensitive streams (CCTV metadata, citizen reports, OT/SCADA)
- Provenance/audit logs (tamper-resistance)

### 4.2 Adversaries and failure sources
- External attackers: network intrusion, malware/ransomware, API abuse
- Insider misuse: over-permissioned access, data exfiltration
- Supply chain: compromised devices/firmware, vulnerable dependencies
- Accidental faults: misconfigured sensors, clock drift, schema changes

### 4.3 Representative abuse cases
- **Spoofing sensor data:** inject false congestion readings to reroute traffic.
- **Camera detection poisoning:** trigger fake incident alerts.
- **Replay attacks:** resend stale “normal” readings to hide anomalies.
- **Denial of service:** overload message brokers/ingest endpoints.
- **Data poisoning for ML-based fusion:** bias model decisions over time.

### 4.4 Controls (architecture-level)
- Strong device identity and authentication for telemetry sources
- Message integrity (signing) for high-trust feeds; transport encryption
- Network segmentation (especially if OT/SCADA signals are included)
- Strict IAM/RBAC for consuming fused state
- Audit logging and immutable append-only event stores for provenance
- Anomaly detection not just on “city events” but also on pipeline behavior

---

## 5. Use cases and operational scenarios
Using the “typical data sources” from the original item, realistic fused outputs include:

1) **Incident detection**
   - Evidence: CV detections + citizen report + sudden probe-speed drop.
   - Output: a single incident event with confidence and affected road segments.

2) **Hazard nowcasting**
   - Evidence: rainfall radar/gauges + flood sensors + 311 reports.
   - Output: evolving flood-risk state per neighborhood.

3) **Infrastructure monitoring**
   - Evidence: vibration/noise sensors + maintenance notes + SCADA tags.
   - Output: asset health state and anomaly alerts with provenance.

4) **Situational awareness dashboard**
   - Evidence: GIS basemap + fused event overlays + state tiles.
   - Output: a coherent operational view with drill-down lineage.

---

## 6. Methods and techniques (from concept to implementation)
This section expands the “Methods” bullets from the source file into implementable approaches.

### 6.1 Streaming ETL (streaming ELT) pipeline design
Core tasks:
- Decode/validate incoming messages
- Normalize timestamps, units, coordinate systems
- Apply enrichment (geofences, asset metadata lookup)
- Emit standardized observation records

Important design choices:
- Exactly-once vs at-least-once processing
- Event-time vs processing-time semantics
- Backpressure handling and replay

### 6.2 Spatiotemporal joins
Join patterns:
- Observation ↔ GIS asset/segment (spatial containment or nearest-neighbor)
- Observation ↔ observation (co-occurrence in time window + spatial radius)

Practical considerations:
- Use hierarchical spatial indexes (tiles, geohashes) to bound joins.
- Use event-time windows with allowed lateness.

### 6.3 Entity resolution
Tactics:
- Canonical identifier registry (mapping table) for known systems.
- Deterministic match first (shared IDs), then probabilistic match (name + location + attributes).

Outputs should include:
- Canonical entity ID
- Match confidence
- Evidence/provenance (which attributes matched)

### 6.4 Sensor calibration and quality scoring
Calibration is not a one-time step:
- Track drift over time.
- Maintain per-sensor quality score (freshness, variance, bias vs peers).

Quality signals to attach to each observation:
- Completeness
- Accuracy estimate (if available)
- Timestamp certainty
- Sensor health

### 6.5 Anomaly detection
Two complementary layers:
1) **Domain anomalies:** sudden congestion, unusual noise spikes, abnormal OT states.
2) **Data anomalies:** sensor stuck-at, impossible values, missing bursts.

AI/ML methods for anomaly detection:
- Time-series autoencoders for detecting unusual patterns in sensor data
- LSTM/GRU networks for sequential anomaly detection
- Isolation forests for unsupervised anomaly detection
- One-class SVM for novelty detection
- Ensemble methods combining multiple anomaly detectors

### 6.6 Probabilistic fusion (when appropriate)
When multiple imperfect sources report the same latent variable:
- Treat each as noisy evidence.
- Maintain a state estimate with uncertainty.

Implementation patterns:
- Weighted averaging with dynamic weights from quality scores
- Bayesian updates for discrete hypotheses (event occurred / did not occur)
- Kalman filters for state estimation with noisy measurements
- Particle filters for non-linear, non-Gaussian fusion problems
- Deep learning-based sensor fusion using attention mechanisms

### 6.7 Privacy-preserving data fusion
For sensitive data streams (CCTV, citizen reports, location data):
- Differential privacy for aggregated statistics and public dashboards
- Federated learning for distributed model training without raw data sharing
- Secure multi-party computation for joint analytics across agencies
- Homomorphic encryption for encrypted data processing
- Zero-trust architecture with mTLS and policy enforcement
- Edge computing for local data processing before transmission

### 6.8 Standards and protocols
- MQTT/AMQP for real-time telemetry streaming
- Apache Kafka for event streaming and message brokering
- NGSI-LD for context-aware data sharing and semantic interoperability
- OGC SensorThings API for IoT sensor data
- OPC UA for industrial/OT sensor integration
- CoAP for constrained IoT devices
- DTLS/TLS for secure transport
- JSON Schema and Avro for data contracts
- OpenTelemetry for observability and tracing
- W3C Web of Things (WoT) for IoT device integration

---

## 7. Reference architecture (conceptual)
This is an implementation-agnostic architecture aligned to the source’s “edge-to-cloud” and “situational awareness” phrasing.

### 7.1 Ingestion layer
- Device/stream gateways
- Message broker / event bus
- Schema registry and contracts

### 7.2 Stream processing and enrichment
- Streaming compute jobs for parsing, validation, unit conversion
- Geospatial enrichment (GIS lookups, map matching)
- Privacy filters (redaction/minimization where needed)

### 7.3 Fusion and state management
- Entity registry (canonical IDs)
- Event store (append-only)
- State store (latest state per entity/feature)
- Confidence/provenance model

### 7.4 Serving and consumption
- Query APIs (state, history, events)
- Subscriptions (push alerts)
- Operational UI dashboards
- Downstream analytics/simulations

### 7.5 Observability and governance
- Pipeline health metrics (lag, throughput, error rates)
- Data quality metrics (freshness, completeness, validity)
- Audit logs and access reports
- Digital twin synchronization metrics (DT-IRL framework)
- Model performance metrics for ML-based fusion
- Privacy budget tracking (ϵ consumption)
- Cross-domain data lineage and provenance

---

## 8. Tools and implementation approach (vendor-neutral)
The source file doesn’t name specific products. To avoid inventing citations or claiming specific tools were used, this section stays generic and focuses on capability requirements.

### 8.1 Tool categories you will likely need
- **Message broker / event streaming platform** (topics, partitions, replay)
- **Stream processing engine** (event-time windows, stateful processing)
- **Geospatial services** (spatial indexing, joins, map matching)
- **Schema governance** (contracts, evolution, validation)
- **State store** (low-latency reads + TTL where needed)
- **Event store/data lake** (historical replay, audits)
- **IAM and secrets management**
- **Observability stack** (logs, metrics, traces)

### 8.2 Required capabilities checklist
- Event-time processing and late data handling
- Idempotent processing and deduplication
- Backfill/replay workflows
- Data lineage capture
- Multi-tenant access control (agency/vendor separation)

---

## 9. Step-by-step: building a “coherent, time-aligned city state”
This is a practical sequence that maps to the source item’s methods.

1) **Define canonical entities and events**
   - Decide what constitutes an “entity” (asset, road segment, zone, sensor).
   - Define an event taxonomy and required fields.

2) **Define an observation schema**
   - Minimum fields: source, observed_at (event time), received_at, location, metric/value, units, quality, provenance.

3) **Establish timestamp strategy**
   - Require source timestamps where possible.
   - Track clock drift and estimate correction.
   - Define allowed lateness windows per source type.

4) **Build ingestion with validation and quarantine**
   - Reject/park messages that fail schema validation.
   - Keep a “dead letter” path for investigation.

5) **Normalize and enrich spatial context**
   - Standardize CRS.
   - Attach GIS features: zone IDs, road segments, asset IDs.

6) **Implement entity resolution**
   - Create a canonical registry and mapping rules.
   - Include match confidence and evidence.

7) **Compute per-source quality scores**
   - Freshness, noise, drift, missingness.
   - Feed these scores into fusion weights.

8) **Fuse into state + events**
   - Choose fusion rules/models per domain variable.
   - Emit fused state updates and inferred events.

9) **Serve with provenance and security**
   - Every state/event includes lineage (sources, transforms).
   - Enforce least privilege on sensitive layers.

10) **Operationalize (SLOs + runbooks)**
   - Lag SLO, completeness SLO, error budget.
   - Drill incident response for pipeline failures.

---

## 10. Common pitfalls and failure modes
These are predictable failure points in real-time fusion systems.

### 10.1 Latency and out-of-order data
- Pitfall: using processing time only → incorrect temporal correlations.
- Mitigation: event-time windows, late-arrival handling, watermarking.

### 10.2 Brittle spatiotemporal joins
- Pitfall: naive radius joins explode compute costs or miss matches.
- Mitigation: tile-based indexes, staged joins, domain-specific constraints.

### 10.3 Identity mismatch and duplicate entities
- Pitfall: multiple IDs for the same asset lead to inconsistent state.
- Mitigation: canonical entity registry + ongoing reconciliation.

### 10.4 Overtrusting a single source
- Pitfall: CCTV detection errors dominate decisions.
- Mitigation: confidence modeling, multi-source corroboration.

### 10.5 Poor data quality feedback loops
- Pitfall: no visibility into sensor drift → fusion silently degrades.
- Mitigation: data-quality SLAs, sensor health dashboards, calibration workflows.

### 10.6 Governance gaps
- Pitfall: unclear data ownership → slow incident resolution.
- Mitigation: stewardship, on-call ownership, documented runbooks.

---

## 11. Ethics, privacy, and legal notes
This system can combine data sources that raise significant privacy and civil liberties concerns (notably CCTV-derived signals and citizen-reported data).

Principles to apply:
- **Purpose limitation:** fuse only what is needed for defined operational outcomes.
- **Data minimization:** avoid retaining raw personal data when derived features suffice.
- **Transparency:** document what is collected, why, and who can access it.
- **Access controls:** restrict CCTV-derived outputs and sensitive incident details.
- **Retention:** define TTLs; keep aggregates longer than raw data where possible.

If OT/SCADA signals are included:
- Treat as critical infrastructure: segmentation, strict change control, and incident response readiness.

---

## 12. Deliverables and success metrics
### 12.1 Suggested deliverables
- Observation schema + data contract documentation
- Canonical entity registry (initial mappings + reconciliation process)
- Fusion rules/model specs per domain variable
- Data quality dashboard (freshness/completeness/validity)
- Provenance model (lineage fields, audit trail)
- Operational runbooks (pipeline outage, sensor drift, replay/backfill)

### 12.2 Success metrics (examples)
- End-to-end lag p95/p99 per source
- % observations passing validation
- Coverage (% of critical sensors healthy)
- Alert precision/recall (where ground truth exists)
- Mean time to detect pipeline failures (MTTD) and recover (MTTR)

---

## 13. References
### 13.1 Workspace source
- First research item in [`kali-task-research.md`](../kali-task-research.md:1)

### 13.2 External references (retrieved via Firecrawl MCP)
- Comprehensive systematic review of information fusion methods in smart cities (ScienceDirect): https://www.sciencedirect.com/science/article/pii/S1566253524000952
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.
- Murala et al. (2025). "A service-oriented microservice framework for differential privacy-based protection in industrial IoT smart applications." Scientific Reports, 15, 29230.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.

### 13.3 Suggested further reading (not fetched)
- Stream processing "event time" and watermarking concepts
- Geospatial indexing and efficient spatial joins
- Entity resolution / record linkage methods
- Probabilistic sensor fusion (Bayesian filtering) and uncertainty modeling
- Data governance and privacy-by-design for multi-source city data
- Deep learning approaches for multi-modal sensor fusion
- Edge computing architectures for real-time data processing
- Zero-trust architecture for IoT and sensor networks
- Federated learning for distributed sensor analytics
