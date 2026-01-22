# Buildings — Deep research

## Executive summary
A buildings “twin” connects building information models (BIM), asset registers, and operational data (BMS/IoT, work orders, energy use, occupancy proxies) to optimize comfort, efficiency, maintenance, and retrofit prioritization at portfolio scale. The core challenge is not creating a photorealistic model; it is building a reliable semantic representation of systems (HVAC, lighting, envelopes), their control points, and performance metrics that supports repeatable analytics and interventions.

A pragmatic approach starts with a canonical building registry and a small number of high-value use cases (fault detection, energy baseline and anomalies, retrofit scoring), then scales by standardizing schemas and onboarding processes. Governance and privacy are essential because operational building data can reveal occupant behavior and sensitive facility details.

This document deepens item 6 in [`kali-task-research.md`](../kali-task-research.md:1): *“Buildings: Connect BIM and operations data to optimize comfort, efficiency, and retrofit prioritization at scale.”*

---

## 1. Background and context
Buildings are often the largest share of urban energy use and emissions. Cities and large owners manage portfolios across:
- Public buildings (schools, hospitals, offices)
- Social housing
- Commercial buildings (in partnerships)

Key problems:
- Inconsistent building metadata (floor area, system types, retrofit history)
- Limited visibility into operational performance
- Fragmented BMS vendors and point naming conventions
- Difficulty comparing buildings (“apples to oranges”)

A buildings twin provides:
- A normalized building/system data layer
- Benchmarks and baselines
- Fault detection and diagnostics (FDD)
- Retrofit pipeline prioritization with quantified impacts

---

## 2. Stakeholders
- **Facilities management (FM)**: comfort, complaints, maintenance
- **Energy/sustainability teams**: emissions reductions and reporting
- **Building occupants**: comfort and indoor air quality
- **Capital planning**: retrofit programs and investment planning
- **BMS vendors/integrators**: point mapping, control changes
- **IT/security**: OT/IT segmentation, identity management
- **Finance**: budgets, incentives, energy procurement
- **Regulators**: building performance standards, safety compliance

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity and safety of building control systems
- Confidentiality of building layout and critical systems (especially public facilities)
- Privacy of occupancy-related signals

### 3.2 Abuse/failure cases
- **Unauthorized control changes** causing unsafe HVAC operation
- **Sensor spoofing** leading to comfort/air quality harms
- **Over-collection of occupancy data** enabling privacy violations
- **Misleading baselines** causing incorrect savings claims

### 3.3 Controls
- OT network segmentation; limited remote access
- Change management and approval workflows for control updates
- Data minimization and aggregation for occupancy
- Audit logs for control setpoint changes and model outputs

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Canonical building registry**
   - Address, geometry, floor area, usage type, ownership, criticality

2. **BIM/document repository**
   - BIM models where available; as-builts; equipment schedules

3. **Operational data ingestion**
   - BMS points (temperatures, flows, setpoints), meters, alarms
   - Maintenance work orders and asset inventories

4. **Semantic layer / ontology mapping**
   - Standardized naming and tagging of points and equipment
   - Zone/equipment relationships (AHU → VAV → zone)

5. **Analytics and optimization**
   - Baselines and benchmarking
   - FDD and anomaly detection
   - Retrofit prioritization scoring

6. **Action layer**
   - Work order creation, recommendations, control tuning proposals

7. **Serving and visualization**
   - Portfolio dashboards, building drill-down, alerts

8. **Governance/observability**
   - Data quality, access control, audit logs

### 4.2 Data flows
- Building metadata + BIM → registry/semantic store
- BMS/meter streams → normalization → time-series store
- Work orders → join with equipment/points → failure patterns
- Analytics → recommendations → FM review → work orders / control changes

---

## 5. Methods / algorithms / standards

### 5.1 Point normalization and semantic tagging
- Mapping vendor-specific point names to standardized tags
- Use equipment templates and automated tagging where feasible
- AI/ML models for automated point classification and tagging
- Digital twin-based building analytics

### 5.2 Energy baselines and benchmarking
- Weather-normalized baselines
- Change-point models for heating/cooling dependence
- Portfolio benchmarking by building type and size
- AI-driven smart building systems with deep learning model analysis
- Predictive maintenance for building systems

### 5.3 Fault detection and diagnostics (FDD)
- Rule-based diagnostics (simultaneous heating/cooling, stuck dampers)
- Model-based diagnostics (expected vs observed performance)
- Alarm rationalization (reduce noise; prioritize actionable alarms)
- Time-series autoencoders for anomaly detection
- LSTM/GRU forecasters for performance prediction

### 5.4 Comfort and indoor air quality
- Comfort KPIs: temperature/humidity excursions, variability
- IAQ proxies: CO2, ventilation rates where measured
- Real-time monitoring and optimization
- Smart home digital twin for comfort optimization

### 5.5 Retrofit prioritization
- Scoring models combining:
  - Savings potential (baseline vs target)
  - Cost and feasibility
  - Asset condition and lifecycle timing
  - Equity/criticality factors
- Digital twin-based retrofit planning and simulation

### 5.6 Standards and protocols
- BIM standards (IFC, Industry Foundation Classes)
- Building Automation and Control Networks (BACnet)
- Haystack tagging for semantic data
- MQTT/AMQP for real-time building data
- NGSI-LD context models for interoperability
- Brick schema for building metadata

---

## 6. Data requirements

### 6.1 Minimum viable
- Building registry fields (type, area, age, systems)
- Utility meter data (electricity, heat/gas if applicable)
- Basic BMS data (zone temps, setpoints, runtimes) for priority buildings
- Work order history and equipment inventory

### 6.2 High-value
- Submetering for end uses
- Occupancy proxies (aggregated)
- Equipment performance data (flows, valve positions)
- Retrofit project tracking and measured savings

### 6.3 Data quality
- Time synchronization across meters and BMS
- Point completeness and tagging accuracy
- Handling missing data and sensor drift

---

## 7. Implementation plan (phases)

### Phase 0 — Portfolio scope and governance
- Define building tiers (critical, high energy use, public-facing)
- Establish privacy and OT security constraints

### Phase 1 — Registry and energy baselines
- Build canonical building registry
- Ingest meter data, compute weather-normalized baselines
- Identify top outliers and quick wins

### Phase 2 — Semantic layer and FDD pilots
- Onboard a small set of buildings with deep BMS integration
- Create tagging standards and templates
- Deploy FDD and alert workflows to FM

### Phase 3 — Scale onboarding and retrofit pipeline
- Standardize onboarding playbooks for new buildings
- Build retrofit scoring and portfolio planning tools
- Integrate with capital planning and procurement

### Phase 4 — Continuous optimization
- Commissioning/retro-commissioning cycles
- Measure persistence of savings and comfort outcomes
- Iterate models and control strategies

---

## 8. Testing and validation
- Data validation: point mapping correctness, unit consistency
- Baseline validation: backtesting against historical periods
- FDD validation: sample of diagnosed faults verified by technicians
- M&V: verify retrofit savings with appropriate methods and uncertainty

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- Data availability: % time-series points present per day
- Tagging coverage: % points mapped to semantic tags
- Alert quality: actionable rate, false positive rate
- Comfort: % time within comfort band
- End-to-end latency (ms)
- Synchronicity error between physical and virtual states
- Update rate (Hz)
- Service availability (%)
- Mean time to detect/recover (MTTD/MTTR)

### 9.2 Example SLOs
- 99% of meter reads ingested within 2 hours
- 95% of tier-1 buildings have complete baseline KPIs monthly
- System uptime ≥ 99.9%
- Update rate ≥ 1 Hz for critical building monitoring
- End-to-end latency < 5 seconds for real-time state updates

---

## 10. Governance, privacy, compliance
- Privacy: avoid granular occupancy; aggregate and apply access controls
- OT security: segmentation, least privilege, patching policies
- Compliance: building performance standards reporting; safety regulations
- Change control: documented approvals for control tuning

---

## 11. Risks and mitigations
- **Vendor heterogeneity** → semantic tagging standards and onboarding templates
- **Alarm fatigue** → prioritize and tune thresholds; measure alert quality
- **Privacy concerns** → minimization and transparency
- **Savings non-persistence** → ongoing monitoring and commissioning cycles

---

## 12. Costs and FinOps
- Integration cost per building (BMS mapping is labor-heavy)
- Storage for time-series data
- Analytics compute (FDD, baselines)

Unit economics to track:
- Cost per onboarded building
- Savings per dollar invested

---

## 13. KPIs
- Energy use intensity (EUI) reduction (weather-normalized)
- Comfort compliance (% time in band)
- Maintenance efficiency (MTTR, planned vs reactive)
- Retrofit pipeline throughput and realized savings
- Emissions reductions from electrification and efficiency

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Canonical building registry and metadata governance
- Meter ingestion and baseline dashboards
- Semantic tagging standard and point library
- FDD service with alert workflows
- Retrofit prioritization model and portfolio plan

### 14.2 Readiness checklist
- [ ] Building tiers and priorities agreed
- [ ] OT security constraints documented and enforced
- [ ] Tagging standard approved and adopted
- [ ] Baselines validated and used in decisions

---

## 15. References

### 15.1 Workspace source
- Item 6 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- Metrics and Methods to Assess Building Fault Detection and Diagnostics (NREL, PDF): https://docs.nrel.gov/docs/fy19osti/72801.pdf
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Arun et al. (2025). "Investigating the performance of AI-driven smart building systems through advanced deep learning model analysis." Energy Reports, 13, 5885-5899.
- Crespo-Aguado et al. (2024). "Flexible hyper-distributed IoT–edge–cloud platform for real-time digital twin applications." Future Internet, 16(11), 431.

### 15.3 Suggested further reading (not fetched)
- Building analytics and FDD methods
- Measurement and verification (M&V) guidance
- Semantic tagging approaches for building systems
- Digital Twin Implementation Readiness Level (DT-IRL) framework
- Zero-trust architecture for building systems
- Federated learning for privacy-preserving building analytics
- BIM to digital twin transformation methods