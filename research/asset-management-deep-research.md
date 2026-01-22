# Asset management — Deep research

## Executive summary
Asset management in a city-scale “digital twin” context is the discipline of maintaining a continuously accurate, decision-ready representation of infrastructure assets (what they are, where they are, their condition, risk, criticality, and lifecycle status) and using that representation to optimize inspection, maintenance, renewal, and capital planning.

A twin-enabled asset management system goes beyond CMMS/EAM record-keeping: it links *physical reality* (inspections, sensor telemetry, imagery, failures) to *engineering models* (deterioration/remaining useful life), and to *work execution* (work orders, crews, budgets, permits). The payoff is improved service reliability, safer operations, reduced total cost of ownership, and defensible prioritization of spend.

This document deepens item 2 in [`kali-task-research.md`](../kali-task-research.md:1): *“Asset management: Track lifecycle, condition, and maintenance of infrastructure assets using twin-based inspection and work planning.”*

---

## 1. Background and context
City infrastructure spans heterogeneous asset classes (roads, bridges, streetlights, signals, water mains, pumps, substations, buildings, parks equipment). Each class has distinct:
- Failure modes and criticality
- Inspection regimes (visual, NDT, telemetry-based)
- Maintenance actions and safety constraints
- Data sources and standards

Traditional asset management challenges:
- Fragmented systems (GIS, CMMS/EAM, SCADA/OT, spreadsheets)
- Incomplete/incorrect asset registry (unknown assets, wrong locations)
- Condition data that is sparse, inconsistent, or non-comparable
- Reactive maintenance driven by failures rather than risk

Twin-based asset management addresses these by:
- Establishing a canonical asset identity and geospatial representation
- Integrating inspection evidence (photos, LiDAR, CV features, notes)
- Maintaining versioned asset state (as-installed vs as-maintained)
- Enabling scenario planning (budget vs risk vs service impacts)

---

## 2. Stakeholders and operating model
Primary stakeholders:
- **Asset owners** (public works, utilities, transport agencies): accountable for service and safety
- **Operations & maintenance (O&M)**: executes inspections and work orders
- **Capital planning / engineering**: renewal programs, design standards, long-term planning
- **Finance/procurement**: budgets, contracts, vendor management
- **Field contractors**: construction, specialized inspection
- **Public safety/regulators**: compliance for critical assets (bridges, pressure vessels)
- **IT/data/platform teams**: integration, security, data governance

RACI patterns (example):
- Asset registry stewardship: Asset owner (A), Data team (R), GIS (C), CMMS admins (C)
- Condition model governance: Engineering (A/R), Data science (C), O&M (C)
- Work planning rules: O&M (A/R), Engineering (C), Finance (C)

---

## 3. Threat model / abuse cases
Even “non-security” domains have meaningful abuse/failure modes.

### 3.1 Assets to protect
- Integrity of asset registry (location, attributes, criticality)
- Integrity of condition/inspection results
- Availability of work planning and dispatch
- Confidentiality of sensitive asset locations (critical infrastructure)

### 3.2 Abuse/failure cases
- **Fraudulent inspection records** (fabricated condition scores to meet KPIs)
- **Tampering with criticality** to influence prioritization and budgets
- **Work order manipulation** (unauthorized creation/closure)
- **Data poisoning** of predictive models via biased labeling
- **Denial of service** on dispatch tools affecting response times

### 3.3 Controls
- Strong identity, least-privilege roles, and separation of duties
- Immutable audit logs for inspection edits and work order lifecycle
- Data validation rules and outlier detection for condition scoring
- Approval workflows for critical fields (criticality class, replacement cost)
- Secure handling of critical infrastructure data (need-to-know)

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Canonical asset registry**
   - Unique asset IDs, geometry, hierarchy (system → subsystem → component)
   - Linkages to GIS feature IDs and CMMS asset IDs

2. **Condition evidence store**
   - Inspections (structured forms), photos/video, LiDAR/point clouds, NDT results
   - Versioned, time-stamped, with provenance and inspector identity

3. **Telemetry/monitoring ingestion** (where applicable)
   - IoT sensors (vibration, pressure, current draw), SCADA tags

4. **Asset state model**
   - Current condition, health index, risk score, RUL estimate, confidence
   - History for trend analysis

5. **Work management integration**
   - CMMS/EAM connectors for work orders, crews, costs, inventory

6. **Planning and optimization services**
   - Inspection planning, preventive maintenance scheduling
   - Renewal prioritization under budgets and constraints

7. **Serving layer**
   - APIs, dashboards, mobile field apps, reporting exports

8. **Governance/observability**
   - Data quality checks, access logs, lineage, SLA monitoring

### 4.2 Core data flows
- **Register/ingest**: GIS + as-built + CMMS seed → canonical asset registry
- **Inspect**: Field app captures structured inspection + media → evidence store
- **Enrich**: CV/analytics extract features (cracks, corrosion) → condition metrics
- **Fuse/update**: Condition/telemetry → asset state model (health, risk, RUL)
- **Plan**: Optimizer proposes work packages → CMMS work orders
- **Execute/close**: Work completion updates parts/costs → feeds back to state

---

## 5. Methods / algorithms / standards

### 5.1 Asset identity and hierarchy
- Deterministic mapping: GIS feature ID ↔ CMMS asset ID ↔ canonical ID
- Hierarchical modeling: location-based (network) + functional decomposition
- Digital twin-based asset representation with real-time state synchronization

### 5.2 Condition assessment and scoring
- Structured condition indices per asset class (e.g., 1–5, or 0–100)
- Evidence-based scoring with required fields and photo requirements
- Inter-rater reliability processes (training sets, QA reviews)
- AI/ML-based automated condition assessment from imagery and sensor data
- Computer vision for defect detection (cracks, corrosion, wear)
- Time-series analysis for predictive condition monitoring

### 5.3 Deterioration and risk modeling
Typical approaches:
- **Markov deterioration models** (condition states with transition probabilities)
- **Survival/hazard models** (time-to-failure as function of covariates)
- **Bayesian updating** to incorporate new inspections and uncertainty
- **Health index** aggregation (weighted subcomponent scores)
- **Machine learning models** for remaining useful life (RUL) prediction
- **LSTM/GRU networks** for time-series-based deterioration forecasting
- **Random forests** and **gradient boosting** for failure prediction

Risk framing:
- Risk = Probability of failure × Consequence of failure
- Consequence dimensions: safety, service, environment, cost, reputational
- Digital twin-based risk simulation and scenario analysis

### 5.4 Work planning and optimization
- Preventive maintenance scheduling with constraints (crew, access windows)
- Renewal prioritization as constrained optimization:
  - Maximize risk reduction / service improvement under budget
  - Consider bundling/coordination (dig-once, corridor works)
- Reinforcement learning for dynamic maintenance scheduling
- Genetic algorithms for multi-objective optimization
- Digital twin-based work planning simulation

### 5.5 Predictive maintenance
- Condition-based maintenance (CBM) using real-time sensor data
- Predictive maintenance (PdM) with ML-based failure prediction
- Anomaly detection using time-series autoencoders
- Digital twin-based predictive maintenance workflows

### 5.6 Data/semantic standards (guidance)
Keep standards selection pragmatic:
- GIS feature schemas and linear referencing for network assets
- BIM for building assets (where present)
- Consistent unit and timestamp conventions
- Controlled vocabularies for defect types, severity, and actions
- ISO 55000 series for asset management
- ISO 19650 for BIM information management
- IEC 62264 for enterprise-control system integration
- MQTT/AMQP for real-time asset telemetry
- NGSI-LD for context-aware asset data sharing
- OGC SensorThings API for IoT asset monitoring
- OPC UA for industrial asset integration

---

## 6. Data requirements

### 6.1 Minimum viable data
- Asset registry: ID, type, geometry, install date (or estimate), ownership, criticality, replacement cost
- Work history: work orders, failure events, costs, downtime
- Condition snapshots: last inspection date, score, method, inspector

### 6.2 High-value enhancements
- Environmental covariates (corrosion zones, soil type, traffic loads)
- Telemetry for condition proxies (pressure transients, vibration signatures)
- Imagery analytics outputs with confidence scores
- Inventory/parts lead times

### 6.3 Data quality dimensions
- Completeness: % assets with location, type, install year
- Accuracy: field-verified location/attributes
- Timeliness: stale condition > threshold by criticality
- Consistency: taxonomy and scoring rubric adherence

---

## 7. Implementation plan (phases)

### Phase 0 — Foundation
- Establish asset taxonomy and canonical ID strategy
- Define data contracts for GIS and CMMS integration
- Identify critical asset classes and service objectives

### Phase 1 — Build the canonical registry + integrations
- Ingest GIS/CMMS, reconcile duplicates, create mapping tables
- Implement audit logging and role-based access
- Deliver basic dashboards (inventory, completeness, map views)

### Phase 2 — Condition evidence and inspection workflows
- Deploy field inspection forms and media capture
- Add QA workflows and scoring governance
- Build condition history views and basic prioritization lists

### Phase 3 — Risk and deterioration modeling
- Implement risk scoring per asset class
- Calibrate deterioration models using history + expert review
- Add scenario planning (budget vs risk) and defensible reporting

### Phase 4 — Optimization + continuous improvement
- Work bundling/route optimization, preventive scheduling
- Feedback loops: outcomes → model recalibration
- Expand to additional asset classes and agencies

---

## 8. Testing and validation

### 8.1 Data validation
- Schema validation at ingest (types, required fields)
- Spatial plausibility checks (assets in wrong admin area, invalid geometry)
- Duplicate detection (near-identical geometry + attributes)

### 8.2 Model validation
- Backtesting: historical inspections vs predicted deterioration
- Ground truth sampling: field audits for a subset of assets
- Uncertainty calibration: confidence bands that match observed error

### 8.3 Process validation
- Work order reconciliation: planned vs executed vs closed outcomes
- Inspector QA: inter-rater consistency metrics

---

## 9. Observability (SLIs/SLOs) and operations

### 9.1 Suggested SLIs
- Asset registry completeness (% required fields filled)
- Condition freshness (% critical assets inspected within policy window)
- Work order latency (time from identification → creation → start → close)
- Data pipeline health (ingest success rate, lag, error rate)
- Digital twin synchronization metrics (DT-IRL framework)
- Model prediction accuracy for condition and RUL estimates
- Anomaly detection precision/recall for predictive maintenance
- Cross-system data exchange success rate

### 9.2 Example SLOs
- 99% of inspection submissions processed within 5 minutes
- <0.5% daily ingest failures requiring manual intervention
- 95% of tier-1 critical assets have condition age < 180 days
- System uptime ≥ 99.9%
- Model accuracy ≥ 90% for failure predictions
- End-to-end latency < 5 seconds for critical alerts

### 9.3 Runbooks
- Duplicate asset resolution procedure
- Backfill/replay after integration outage
- Emergency work order creation during CMMS downtime
- Model retraining and deployment procedures
- Data quality incident response

---

## 10. Governance, privacy, compliance
- Stewardship: named owners for asset classes and datasets
- Change control: schema evolution, scoring rubric updates
- Retention: keep work history and condition snapshots for lifecycle analysis; define retention for raw media
- Sensitive infrastructure: apply access tiering and redaction as needed

---

## 11. Risks and mitigations
- **Incomplete registry** → prioritize reconciliation and field verification sprints
- **Inconsistent scoring** → training, QA sampling, and rubric enforcement
- **Integration brittleness** → contracts, versioning, and monitoring
- **Overfitting models** → simple baselines first, documented assumptions
- **Operational resistance** → co-design with field teams; reduce clicks; show value

---

## 12. Costs and FinOps considerations
Cost drivers:
- Storage for inspection media (photos/video/point clouds)
- Stream/ETL compute for enrichment and CV inference (if used)
- GIS and routing services
- Mobile devices/data plans for field teams
- Integration maintenance with CMMS/EAM vendors

FinOps controls:
- Tiered storage (hot vs cold), lifecycle policies
- Batch CV processing where real-time is not required
- Unit-cost metrics: cost per inspected asset, per work order, per GB of media

---

## 13. KPIs (outcome-oriented)
- Reduction in unplanned failures (count and severity)
- MTBF increase for targeted assets
- Maintenance cost per asset (normalized) and trend
- % work that is planned vs reactive
- Risk reduction per dollar spent (renewal effectiveness)
- Service impact reductions (downtime, outages)

---

## 14. Deliverables and checklists

### 14.1 Key deliverables
- Canonical asset registry + mapping to source systems
- Inspection forms, mobile workflow, and evidence store
- Condition scoring rubric and QA process
- Risk model and renewal prioritization dashboards
- Work planning integration and runbooks

### 14.2 Readiness checklist
- [ ] Asset taxonomy agreed and documented
- [ ] Canonical ID rules implemented and auditable
- [ ] Data quality metrics and ownership in place
- [ ] Condition scoring trained and calibrated
- [ ] Work order lifecycle metrics tracked end-to-end

---

## 15. References
### 15.1 Workspace source
- Item 2 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- ISO 55000:2024 — Asset management — Vocabulary, overview and principles (ISO): https://www.iso.org/standard/83053.html
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.
- Arun et al. (2025). "AI-driven smart building systems: A comprehensive review of deep learning model analysis." Scientific Reports, 15, 29230.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.

### 15.3 Suggested further reading (not fetched)
- Reliability engineering (hazard models, survival analysis)
- Risk-based maintenance and renewal prioritization methods
- Digital twins for predictive maintenance
- Computer vision for infrastructure inspection
- Time-series forecasting for asset condition monitoring
- Federated learning for distributed asset analytics
