# Mobility — Deep research

## Executive summary
Mobility modeling for a city digital twin is the capability to represent and forecast multimodal movement (private vehicles, freight, transit, micromobility, walking) and to optimize outcomes such as travel time reliability, safety, emissions, curb utilization, and equity under changing demand and constraints.

A “mobility twin” is not a single model; it is a layered system: data pipelines (counts, speeds, OD, incidents), network representation, demand modeling, traffic assignment/simulation, policy levers (pricing, signal timing, curb rules), and evaluation metrics. The hardest engineering problems are (1) trustworthy inputs (sensor fusion, map matching, incident labeling), (2) calibration and validation across regimes (peak/off-peak, disruptions), and (3) operationalizing decisions with human-in-the-loop controls.

This document deepens item 3 in [`kali-task-research.md`](../kali-task-research.md:1): *“Mobility: Model multimodal traffic dynamics to optimize flows, pricing, and curb access under changing demand.”*

---

## 1. Background and context
Cities experience continuous shifts in mobility demand driven by:
- Weather, events, school/work schedules
- Construction, incidents, and disruptions
- Policy interventions (pricing, lanes, curb rules)
- Long-term land use and economic change

Mobility systems are complex adaptive systems with feedback loops:
- Congestion affects route choice
- Pricing affects demand and mode shift
- Curb rules affect freight dwell time and double-parking, impacting flow

A twin-based approach enables:
- Scenario exploration before deploying policy
- Real-time operations (incident response, signal retiming)
- Post-deployment measurement (did the policy achieve goals?)

---

## 2. Stakeholders
- **Transportation agency / DOT**: network operations, policy design
- **Traffic management center (TMC)**: incident response, real-time control
- **Public transit operator**: coordination with signal priority, curb access
- **Freight/logistics and business community**: deliveries, curb availability
- **Micromobility providers** (public/private): fleet distribution, compliance
- **Residents/road users**: travel time, safety, equity outcomes
- **Law enforcement/parking**: curb enforcement, compliance
- **Urban planning**: long-term network and land-use coordination
- **IT/data**: platform, integrations, privacy

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity of pricing, signal timing, and curb rule configurations
- Integrity of data feeds used for control decisions (speeds, incidents)
- Availability of mobility operations dashboards/APIs
- Privacy of location data (probe traces, app telemetry)

### 3.2 Abuse/failure cases
- **Data spoofing** (fake congestion signals) to influence routing or pricing
- **Probe data re-identification** if privacy controls are weak
- **Model gaming**: actors changing behavior to avoid pricing/enforcement
- **Misconfiguration** of pricing zones or signal plans causing widespread disruption
- **Feedback instability** from overly aggressive control policies

### 3.3 Controls
- Configuration management with approvals and rollback
- Input data validation, anomaly detection, cross-source corroboration
- Differential privacy/aggregation for probe datasets
- Human-in-the-loop for high-impact changes and incident mode

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Data ingestion and fusion**
   - Fixed sensors (loops, radar), cameras/CV events, Bluetooth/Wi-Fi travel time
   - Probe data (aggregated), incident feeds, roadworks, weather, events

2. **Network digital representation**
   - Road graph with lanes, capacities, control devices
   - Curb graph (segments, rules, time-of-day constraints)
   - Multimodal network (bike lanes, sidewalks, transit lines)

3. **Demand and OD estimation**
   - OD matrices by time slice, purpose, vehicle class
   - Mode choice/shift modeling inputs

4. **Simulation and assignment layer**
   - Static assignment (planning) and dynamic traffic assignment (operations)
   - Mesoscopic/microscopic simulation for corridor-level interventions

5. **Optimization/control layer**
   - Signal timing optimization, ramp metering (where applicable)
   - Pricing optimization (congestion/cordon/parking)
   - Curb allocation and reservation (freight windows, loading zones)

6. **Serving and decision workflows**
   - Dashboards (TMC), APIs, alerting
   - Scenario runner and experiment tracking

7. **Governance and observability**
   - Data quality SLAs, model drift monitoring
   - Audit logs of policy changes and decisions

### 4.2 Data flows
- Sensors/probes → ingestion → map matching/aggregation → network metrics
- Incidents/roadworks → event stream → simulation constraints
- OD estimation → assignment/simulation → predicted speeds/queues
- Optimizer → recommended actions → operator approval → deployment
- Post-deployment telemetry → evaluation and model recalibration

---

## 5. Methods / algorithms / standards

### 5.1 Map matching and network metrics
- Matching noisy GPS/probe points to road segments/lane groups
- Aggregation to segment speeds, travel times, reliability percentiles
- AI/ML models for improved map matching accuracy
- Time-series autoencoders for anomaly detection in traffic patterns

### 5.2 Demand modeling
- OD estimation from counts + probes (matrix estimation)
- Time-of-day profiles; segmentation by vehicle class (freight, passenger)
- Elastic demand response to pricing (short- and long-run)
- Federated learning for privacy-preserving demand estimation
- Differential privacy for protecting individual mobility patterns

### 5.3 Traffic assignment and simulation
- Static user equilibrium (planning baseline)
- Dynamic assignment with time-dependent flows and queue spillback
- Microsimulation for intersections and curb interactions
- Digital twin-based real-time traffic optimization
- Connected corridor real-time data-driven traffic digital twin simulation

### 5.4 Curb management modeling
- Stochastic dwell times (delivery, ridehail) and compliance behavior
- Interaction with double-parking and lane blockages
- Reservation/permit mechanisms and enforcement intensity
- Smart parking systems with dynamic pricing and reservation

### 5.5 Pricing optimization (conceptual)
- Objective: reduce congestion/externalities while meeting equity and revenue constraints
- Constraints: max tolls, exemptions, geographic caps, political feasibility
- Control: time-varying pricing by zone/corridor
- AI-powered dynamic pricing algorithms

### 5.6 Safety and equity evaluation
- Safety surrogate metrics (conflict indicators) where crash data is sparse
- Distributional analysis across neighborhoods and demographics
- Graph neural networks for analyzing device–device communication patterns

### 5.7 Standards and protocols
- DATEX II for traffic data exchange
- TPEG for traffic and travel information
- MQTT/AMQP for real-time mobility data
- NGSI-LD context models for interoperability
- OpenLR for location referencing

---

## 6. Data requirements

### 6.1 Core datasets
- Road network graph + control devices (signals, restrictions)
- Counts/occupancy/speeds (fixed sensors) by interval
- Aggregated probe travel times by segment/time
- Incident and roadworks logs with time/location/severity
- Curb rules inventory (signage, allowed uses, time windows)

### 6.2 Calibration datasets
- Ground truth travel times (floating car runs, validation probes)
- Turn movement counts at key intersections
- Transit performance and headways (for multimodal interactions)

### 6.3 Privacy constraints
- Use aggregation thresholds, spatial/temporal binning
- Avoid storing raw individual trajectories unless strictly governed

---

## 7. Implementation plan (phases)

### Phase 0 — Define objectives and policy levers
- Select priority corridors/zones and target metrics (reliability, safety, emissions)
- Decide decision cadence (real-time ops vs weekly policy tuning)

### Phase 1 — Build network + metrics foundation
- Canonical network model, map matching, segment metrics
- Incident ingestion + dashboards
- Data quality SLAs for key feeds

### Phase 2 — Baseline modeling and validation
- Calibrate assignment/simulation to reproduce baseline speeds/queues
- Establish evaluation harness (before/after, counterfactuals)

### Phase 3 — Decision support and controlled interventions
- Scenario runner for signal timing, curb rule changes, pricing proposals
- Implement change approvals and rollback mechanisms

### Phase 4 — Optimization at scale
- Automate routine recommendations (e.g., signal retiming candidates)
- Integrate enforcement and curb reservation systems
- Continuous drift monitoring and periodic recalibration

---

## 8. Testing and validation
- Unit tests for map matching, aggregation, and incident parsing
- Backtesting interventions using historical data and simulated deployment
- Sensitivity analysis: elasticity assumptions, incident durations
- Fairness checks: distribution of burdens/benefits
- Chaos testing for data feed outages and late data

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- Data freshness/coverage for segment speeds and incidents
- Model fit: RMSE/MAPE between predicted and observed speeds/queues
- Recommendation latency (time from detection to suggested action)
- Deployment error rate and rollback frequency
- End-to-end latency (ms)
- Synchronicity error between physical and virtual states
- Update rate (Hz)
- Service availability (%)
- Mean time to detect/recover (MTTD/MTTR)

### 9.2 Example SLOs
- 99% of segment speed metrics updated within 3 minutes
- Incident feed processing p95 < 60 seconds
- Model fit thresholds per corridor before recommendations are enabled
- System uptime ≥ 99.9%
- Update rate ≥ 1 Hz for critical corridor monitoring
- End-to-end latency < 5 seconds for real-time state updates

---

## 10. Governance and compliance
- Clear ownership of policy levers and approval authority
- Public transparency for pricing/curb policies and evaluation results
- Privacy-by-design for probe data and ridehail data
- Audit trails for decisions and deployments

---

## 11. Risks and mitigations
- **Model mismatch during unusual events** → incident mode, fallback heuristics
- **Equity backlash** → co-design exemptions/discounts, publish distributional impacts
- **Curb rule noncompliance** → enforcement integration and behavioral modeling
- **Data vendor lock-in** → vendor-neutral schemas and portability requirements

---

## 12. Costs and FinOps
- Major costs: probe data licensing, compute for simulation, CV processing (if used)
- Control costs via:
  - Right-sizing simulation fidelity (macro/meso/micro)
  - Batch scenario runs and autoscaling
  - Unit cost: cost per scenario run, per corridor per day

---

## 13. KPIs
- Travel time reliability (p95/p50) on priority corridors
- Mean delay and queue spillback frequency
- Curb occupancy compliance and double-parking incidents
- Mode share changes (where measurable)
- Emissions proxy reductions (VKT and speed profiles)
- Safety outcomes (crashes, near-miss proxies)

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Multimodal network model and curb inventory
- Mobility metrics pipelines with SLAs
- Calibrated baseline simulation/assignment model
- Scenario runner + experiment tracking
- Governance playbooks and change management process

### 14.2 Readiness checklist
- [ ] Network representation validated against GIS truth
- [ ] Probe data privacy controls documented and enforced
- [ ] Model calibration metrics meet thresholds
- [ ] Deployment approvals and rollback tested

---

## 15. References

### 15.1 Workspace source
- Item 3 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- Victoria Transport Policy Institute — Transportation Elasticities (PDF): https://www.vtpi.org/elasticities.pdf
- Pricing Urban Congestion (Resources for the Future, PDF): https://www.rff.org/documents/1313/RFF-DP-08-35.pdf
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Saroj et al. (2021). "Development of a connected corridor real-time data-driven traffic digital twin simulation model." Journal of Transportation Engineering, 147(4), 04021096.
- Crespo-Aguado et al. (2024). "Flexible hyper-distributed IoT–edge–cloud platform for real-time digital twin applications." Future Internet, 16(11), 431.

### 15.3 Suggested further reading (not fetched)
- Dynamic traffic assignment and traffic flow theory
- Curb management and freight curb allocation research
- Transport economics for congestion pricing and elasticity
- Digital Twin Implementation Readiness Level (DT-IRL) framework
- Zero-trust architecture for mobility systems
- Federated learning for privacy-preserving mobility analytics
- Graph neural networks for traffic pattern analysis