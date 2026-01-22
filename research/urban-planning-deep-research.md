# Urban planning — Deep research

## Executive summary
Urban planning in a city digital twin context is the capability to explore land-use, density, infrastructure, and policy scenarios and quantify impacts *before* implementation. A planning twin integrates spatial data (parcels, zoning, networks), socioeconomic projections, and domain models (mobility, energy, water, climate risk) into a reproducible scenario framework that supports:
- Alternatives analysis with traceable assumptions
- Impact assessment (transport, emissions, affordability, service capacity)
- Equity and distributional analysis
- Stakeholder engagement with transparent narratives

The core success factor is governance and reproducibility: scenario definitions must be versioned; inputs must be documented; and model outputs must be communicated with uncertainty and limitations. A layered reference architecture—with a scenario registry, interoperable domain models, and an evaluation harness—enables incremental adoption.

This document deepens item 10 in [`kali-task-research.md`](../kali-task-research.md:1): *“Urban planning: Explore land-use, density, and infrastructure scenarios to quantify impacts before implementation.”*

---

## 1. Background and context
Planning decisions have long lifetimes and cross-domain consequences:
- Zoning and density changes alter travel demand and energy use
- Infrastructure investments create induced demand and reshape development
- Climate adaptation requirements influence siting and design standards

Planning “twin” goals:
- Replace ad-hoc spreadsheet analysis with reproducible scenario modeling
- Enable consistent comparisons (“apples to apples”) across alternatives
- Provide a defensible record for public process and audits

Challenges:
- Data fragmentation (parcels, permits, networks, demographics)
- Model coupling across domains
- Political sensitivity and risk of model misuse

---

## 2. Stakeholders
- **Planning department**: zoning, comprehensive plans, permitting policy
- **Transport agency**: mobility impacts and infrastructure needs
- **Utilities**: capacity planning for growth (energy/water)
- **Housing agency**: affordability and displacement risk
- **Elected officials**: tradeoffs and accountability
- **Community groups**: equity, local impacts, transparency
- **Developers**: feasibility and policy compliance
- **Environmental and resilience offices**: emissions and climate risk
- **IT/data**: platform and governance

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity and neutrality of scenario comparisons
- Confidentiality of certain development or critical infrastructure plans
- Public trust in planning analytics

### 3.2 Abuse/failure cases
- **Scenario gaming**: selectively favorable assumptions
- **Black-box modeling**: outputs used without understanding limitations
- **Equity washing**: aggregate improvements hiding localized harms
- **Data leakage** of confidential project details

### 3.3 Controls
- Scenario governance: required assumptions checklist and peer review
- Versioned scenario definitions and reproducible pipelines
- Mandatory distributional metrics and equity review
- Access controls for sensitive datasets

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Scenario registry and configuration**
   - Canonical scenario objects: land use, density, infrastructure, policies
   - Versioning, authorship, approvals

2. **Baseline city data layer**
   - Parcels, zoning, land use, building stock
   - Networks: roads, transit, utilities
   - Demographics and employment distributions

3. **Domain model adapters**
   - Mobility model interface (OD, mode split, network performance)
   - Energy/buildings model interface (load/emissions)
   - Water/stormwater capacity model interface
   - Climate risk interface (hazard/exposure)

4. **Evaluation harness**
   - Standard outputs and KPI calculations
   - Uncertainty tracking and sensitivity runs

5. **Visualization and engagement**
   - Map-based scenario viewer
   - Narrative reports and comparison dashboards

6. **Governance/observability**
   - Lineage, model versioning, audit logs

### 4.2 Data flows
- Baseline datasets → scenario generator (apply zoning/land-use changes)
- Scenario state → domain models → outputs (traffic, emissions, capacity)
- Outputs → evaluation harness → KPIs and equity metrics
- Results → dashboards/reports → feedback to scenario design

---

## 5. Methods / algorithms / standards

### 5.1 Land-use change modeling
- Parcel-level capacity under zoning and FAR constraints
- Development feasibility approximations (cost, rent, absorption)
- Allocation of growth to parcels via rules or optimization
- Machine learning-based development prediction
- Digital twin-based land-use scenario simulation

### 5.2 Integrated transport–land use
- Activity-based models linking location choices to travel behavior
- Accessibility metrics as drivers of development patterns
- Agent-based modeling for urban dynamics
- Digital twin-based transport-land use interaction simulation

### 5.3 Infrastructure capacity and phasing
- Capacity constraints for utilities and transport
- Phasing logic: when growth triggers upgrades
- Multi-objective optimization for infrastructure planning
- Digital twin-based capacity planning

### 5.4 Equity and distributional analysis
- Metrics by neighborhood and vulnerable populations
- Displacement risk proxies and access-to-opportunity metrics
- Fairness-aware machine learning for equitable planning
- Digital twin-based equity impact assessment

### 5.5 Uncertainty and sensitivity
- Sensitivity to growth assumptions, elasticities, and policy compliance
- Use ensembles of plausible futures rather than a single forecast
- Monte Carlo simulation for uncertainty quantification
- Robust decision making for planning under uncertainty

### 5.6 Standards and protocols
- ISO 19152 for Land Administration Domain Model (LADM)
- OGC CityGML for 3D city models
- OGC standards for geospatial planning data
- NGSI-LD for context-aware planning data sharing
- MQTT/AMQP for real-time planning data streaming
- OpenStreetMap for base mapping
- GTFS for transit data integration

---

## 6. Data requirements

### 6.1 Core datasets
- Parcel geometry, zoning, land use, assessed values
- Building footprints, height/area, age, use type
- Permits and development pipeline
- Transport networks and transit service
- Population and employment distributions

### 6.2 High-value datasets
- Real estate market indicators for feasibility
- Accessibility and travel survey data for calibration
- Utility capacity constraints and planned upgrades

### 6.3 Data quality needs
- Consistent geospatial joins across datasets
- Temporal versioning (zoning changes over time)
- Metadata: source, update cadence, limitations

---

## 7. Implementation plan (phases)

### Phase 0 — Define decisions and governance
- Identify planning decisions to support and required outputs
- Create scenario definition templates and review process

### Phase 1 — Baseline data and scenario registry
- Build canonical baseline datasets and scenario registry
- Implement simple growth allocation and KPI reporting

### Phase 2 — Domain model integration
- Integrate mobility model for transport KPIs
- Add buildings/energy and emissions estimation
- Add water/stormwater capacity checks

### Phase 3 — Equity and robustness
- Implement distributional metrics as standard outputs
- Add sensitivity/uncertainty workflows and “robust” scenario comparisons

### Phase 4 — Operational adoption
- Embed in planning processes (staff workflows, public engagement)
- Maintain model and data refresh cycles

---

## 8. Testing and validation
- Baseline validation: reproduce current observed travel and building stock metrics
- Scenario sanity checks: capacity changes consistent with zoning rules
- Domain model validation: corridor-level travel time comparisons
- Backtesting: compare predicted development patterns to historical periods

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- Scenario reproducibility rate
- Data freshness for baseline datasets
- Model run success rate and time-to-results
- Digital twin synchronization metrics (DT-IRL framework)
- Model prediction accuracy for development patterns
- Cross-domain model integration consistency
- Data quality scores for planning datasets

### 9.2 Example SLOs
- Standard scenario comparisons produced within a defined planning cycle time
- Full lineage captured for every published scenario report
- System uptime ≥ 99.9%
- Model accuracy ≥ 80% for development predictions
- End-to-end latency < 30 minutes for scenario runs

---

## 10. Governance and compliance
- Transparent publication of assumptions and limitations
- Public records retention for scenario results
- Access tiering for confidential development pipeline details
- Avoid automation bias: require human review for policy conclusions

---

## 11. Risks and mitigations
- **Model complexity stalls adoption** → start with minimal KPIs and scale
- **Political misuse** → governance, independent review, multiple scenarios
- **Equity blind spots** → enforce neighborhood-level reporting

---

## 12. Costs and FinOps
- Costs: geospatial ETL, domain model compute, staffing and training
- FinOps: cache baseline layers, incremental recomputation, cost per scenario

---

## 13. KPIs
- Time to produce defensible alternatives analysis
- Accuracy of baseline reproduction metrics
- % of planning decisions using scenario evidence
- Equity: distributional impacts included in 100% of reports

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Scenario registry and templates
- Baseline city data layer with versioning
- Integrated KPI dashboard for land use + transport + emissions
- Equity reporting module and sensitivity harness

### 14.2 Readiness checklist
- [ ] Scenario governance process established
- [ ] Baseline datasets validated and versioned
- [ ] Standard KPI definitions approved
- [ ] Distributional reporting required by default

---

## 15. References
### 15.1 Workspace source
- Item 10 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- Navigating urban complexity: The transformative role of digital twins… (ScienceDirect): https://www.sciencedirect.com/science/article/pii/S2210670724004086
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.

### 15.3 Suggested further reading (not fetched)
- Integrated land use–transport modeling (LUTI) methods
- Robust decision making in planning
- Equity metrics and displacement risk modeling
- Digital twins for urban planning
- Agent-based modeling for urban dynamics
- Machine learning for land-use prediction
- Federated learning for distributed planning analytics
