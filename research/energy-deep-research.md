# Energy — Deep research

## Executive summary
An urban energy “twin” represents electricity (and often heat/gas) supply, demand, and network constraints to support decarbonization planning, resilience, and operational decision-making. It connects asset topology (feeders, substations, DER), consumption profiles (buildings, EV charging), and grid physics/constraints (thermal limits, voltage, contingency limits) with policy levers (electrification, storage, demand response).

The practical value is in answering: *What happens to reliability, costs, and emissions if we electrify buildings, add EV charging, deploy storage, and implement demand response—under realistic peak conditions and outage scenarios?* Delivering this requires careful data governance with utilities, uncertainty-aware forecasting, and a reference architecture that separates sensitive operational grid data from city analytics while enabling secure collaboration.

This document deepens item 5 in [`kali-task-research.md`](../kali-task-research.md:1): *“Energy: Represent supply, demand, and grid constraints to plan decarbonization, storage, and demand response.”*

---

## 1. Background and context
Cities influence energy through:
- Building electrification and efficiency
- EV adoption and charging infrastructure
- Distributed energy resources (DER): rooftop PV, batteries
- Demand response programs and flexible loads

Constraints:
- Distribution grid capacity and hosting limits
- Peak demand and coincident peaks (hot/cold events)
- Interdependencies with telecom, water, transportation

Twin-enabled energy planning supports:
- DER hosting capacity and interconnection planning
- Electrification pathway analysis and grid reinforcement needs
- Resilience: critical load continuity during outages
- Demand flexibility and peak shaving strategies

---

## 2. Stakeholders
- **Electric utility / distribution system operator (DSO)**: grid operations and planning
- **City sustainability/energy office**: decarbonization goals, programs
- **Building owners and facility managers**: retrofits, load management
- **DER developers**: PV/storage projects, interconnection
- **EV charging operators**: site planning and load management
- **Regulators**: reliability standards, consumer protection
- **Emergency management**: critical facilities, outage response
- **IT/data platform**: secure data exchange and modeling infrastructure

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Confidentiality of grid topology and operational states
- Integrity of forecasts and planning models used for investment decisions
- Availability of modeling services during emergencies

### 3.2 Abuse/failure cases
- **Sensitive grid data leakage** enabling targeting of critical infrastructure
- **Model manipulation** to favor specific projects or hide constraints
- **Incorrect hosting capacity outputs** leading to unsafe interconnections
- **Outage scenario underestimation** causing resilience gaps

### 3.3 Controls
- Data sharing agreements and tiered access
- Secure enclaves / segmentation for operational grid datasets
- Reproducible modeling with audit logs and versioned assumptions
- Independent review for high-impact planning outputs

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Energy data exchange layer**
   - Secure ingestion from utility and city datasets
   - Data contracts, de-identification/aggregation

2. **Grid representation layer**
   - Network topology (substations, feeders, transformers)
   - Constraints (thermal ratings, voltage limits)

3. **Load and DER modeling**
   - Baseline load profiles by node/area
   - DER generation profiles (PV), storage dispatch models
   - EV charging profiles and managed charging strategies

4. **Scenario and planning engine**
   - Electrification uptake scenarios
   - Reinforcement planning and non-wires alternatives
   - Resilience/outage simulations for critical loads

5. **Optimization layer**
   - Storage siting and dispatch
   - Demand response program design and activation policies
   - Managed EV charging control policies

6. **Serving layer**
   - Dashboards, APIs, scenario reports

7. **Observability/governance**
   - Data freshness/quality SLAs, model drift, audit trails

### 4.2 Data flows
- Utility topology + constraints → grid model store (restricted)
- Meter/load aggregates + building data → load modeling
- DER/EV plans → scenario inputs
- Scenario engine → outputs: constraint violations, reinforcement needs, costs/emissions
- Recommendations → program design and infrastructure investment planning

---

## 5. Methods / algorithms / standards

### 5.1 Forecasting demand and flexibility
- Time-series forecasting by feeder/zone with weather covariates
- Coincident peak analysis and extreme event stress testing
- Flexibility modeling: controllable loads (HVAC, water heating, EV)
- AI/ML models for energy consumption prediction with federated learning
- Differential privacy for privacy-preserving energy analytics

### 5.2 Power flow and constraint analysis
- Distribution power flow (steady-state) for voltage and thermal constraints
- Hosting capacity analysis for DER integration
- Contingency analysis for critical components (where data supports)
- Digital twin-based predictive maintenance for grid infrastructure
- Real-time monitoring and anomaly detection

### 5.3 DER and storage modeling
- PV generation profiles (irradiance-based)
- Storage dispatch (rule-based or optimization) under tariffs and constraints
- Value stacking: peak shaving, resilience, ancillary services (where applicable)
- Solar panel digital twin features for performance optimization
- Wind energy digital twin framework for enhanced monitoring

### 5.4 Demand response and managed charging
- Program targeting and enrollment models
- Dispatch strategies that respect customer comfort and fairness
- Managed charging to avoid transformer overloads and reduce peaks
- Smart grid IoT framework for predicting energy consumption

### 5.5 Emissions accounting
- Marginal vs average emissions factors
- Temporal granularity matters (hourly emissions for DR value)
- Real-time carbon intensity monitoring (e.g., Microsoft Azure for BP)

### 5.6 Standards and protocols
- IEC 61850 for substation automation
- OpenADR for demand response communication
- IEEE 2030.5 for smart energy profile
- MQTT/AMQP for real-time energy data
- NGSI-LD context models for interoperability

---

## 6. Data requirements

### 6.1 Minimum viable data
- Aggregated load by geography (and ideally by feeder/transformer with controls)
- Asset capacity metadata (substation/feeder/transformer ratings)
- Weather and climate normals/extremes
- Building stock and electrification plans (where available)

### 6.2 High-value data
- Advanced metering (AMI) time series (aggregated and privacy-protected)
- Detailed grid topology (restricted)
- DER interconnection queue and installed DER inventory
- EV registrations and charging station utilization

### 6.3 Data governance
- Define which outputs can be public (e.g., zonal hosting capacity) vs restricted
- Ensure privacy constraints for meter data aggregation thresholds

---

## 7. Implementation plan (phases)

### Phase 0 — Partnership and scope
- Establish utility-city data sharing and governance
- Define planning questions and decision cadence

### Phase 1 — Baseline load + coarse constraint model
- Build aggregated load models by zone
- Introduce simplified capacity constraints (proxy model) if topology is limited

### Phase 2 — Detailed feeder-level modeling (where feasible)
- Integrate restricted topology and run distribution power flow
- Produce hosting capacity and reinforcement candidate lists

### Phase 3 — DER/storage/DR scenario engine
- Implement electrification and EV scenarios
- Evaluate non-wires alternatives and program designs

### Phase 4 — Operational integration and resilience
- Tie to outage management inputs for resilience planning
- Deploy dashboards for critical facilities and continuity plans

---

## 8. Testing and validation
- Data QA: load totals reconcile across sources and time
- Forecast validation by season and extreme events
- Power flow validation against utility engineering benchmarks
- Scenario sanity checks and sensitivity analysis (tariffs, adoption rates)

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- Data freshness for load and DER inventory updates
- Forecast error metrics by zone/feeder
- Scenario run success rate and reproducibility (same inputs → same outputs)
- End-to-end latency (ms)
- Synchronicity error between physical and virtual states
- Update rate (Hz)
- Service availability (%)
- Mean time to detect/recover (MTTD/MTTR)

### 9.2 Example SLOs
- 99% of daily load aggregates available by 06:00 local time
- Scenario runs complete within defined time budget for decision cycles
- System uptime ≥ 99.9%
- Update rate ≥ 1 Hz for critical grid monitoring
- End-to-end latency < 5 seconds for real-time state updates

---

## 10. Governance and compliance
- Formal agreements for critical infrastructure data
- Access tiering and secure enclaves
- Model transparency: publish assumptions and uncertainty ranges for policy use
- Procurement governance to avoid vendor lock-in

---

## 11. Risks and mitigations
- **Limited topology access** → use proxy constraint models and refine with utility
- **Forecast failure during extremes** → stress testing and conservative margins
- **Equity impacts of DR** → opt-in design, safeguards, distributional analysis
- **Cybersecurity concerns** → strict segmentation and least privilege

---

## 12. Costs and FinOps
- Compute for scenario simulation and power flow
- Data storage and secure environment costs
- Staff time for utility coordination and model maintenance

FinOps metrics:
- Cost per scenario run
- Cost per feeder modeled per year

---

## 13. KPIs
- Reduction in projected overload violations under target adoption scenarios
- DER interconnection cycle time improvements (if supported)
- Peak demand reduction from DR/managed charging
- Emissions reductions attributable to interventions
- Resilience: % critical load served under outage scenarios

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Secure energy data exchange and governance model
- Baseline load and constraint dashboards
- Hosting capacity outputs (appropriately aggregated)
- Electrification/EV/DER scenario engine with reports
- Resilience analysis for critical facilities

### 14.2 Readiness checklist
- [ ] Data sharing and access controls approved
- [ ] Load reconciliation validated
- [ ] Scenario assumptions documented and versioned
- [ ] Sensitivity and uncertainty communicated

---

## 15. References

### 15.1 Workspace source
- Item 5 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- Advanced Hosting Capacity Analysis (NREL): https://www.nrel.gov/solar/market-research-analysis/advanced-hosting-capacity-analysis
- Hosting Capacity Analysis — overview (IREC): https://irecusa.org/our-work/hosting-capacity-analysis/
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.
- Chen & Fang (2024). "Harnessing digital twin and IoT for real-time monitoring, diagnostics, and error correction in domestic solar energy storage." Energy Reports, 11, 3614-3623.

### 15.3 Suggested further reading (not fetched)
- Distribution system planning and hosting capacity methods
- Demand response program design and measurement
- Power systems modeling and resilience planning
- Digital Twin Implementation Readiness Level (DT-IRL) framework
- Zero-trust architecture for energy systems
- Federated learning for privacy-preserving energy analytics
- Blockchain-based digital twins for energy management