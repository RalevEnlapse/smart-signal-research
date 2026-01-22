# Climate risk — Deep research

## Executive summary
Climate risk stress-testing for a city digital twin is the capability to quantify how hazards (heat, storms, sea level rise, wildfire smoke, compound events) propagate through urban systems and what adaptations reduce impacts. It links hazard models (frequency, severity), exposure (assets, people), vulnerability (fragility functions, adaptive capacity), and interdependencies (power-water-transport) to compute outcomes such as service downtime, health impacts, economic loss, and inequities.

A high-value city climate risk program is operationally useful: it informs capital planning, maintenance priorities, emergency preparedness, and resilience investments with clear uncertainty bounds and decision triggers. The technical foundation is a reference architecture for scenario management, geospatial analytics, and reproducible risk calculations.

This document deepens item 8 in [`kali-task-research.md`](../kali-task-research.md:1): *“Climate risk: Stress-test city systems against heat, storms, sea level rise, and compound hazards for adaptation planning.”*

---

## 1. Background and context
Climate risk differs from historical risk because the hazard distribution is changing:
- More frequent and intense heatwaves
- Heavier rainfall and pluvial flooding
- Sea level rise increasing coastal flooding and storm surge reach
- Compound hazards (heat + outage; storm + high tide)

Cities need to answer:
- Which neighborhoods and assets are most exposed and vulnerable?
- What interventions yield the largest risk reduction per dollar?
- When should specific adaptations be triggered (thresholds)?

A twin-based stress test integrates:
- Asset registries and critical infrastructure maps
- Social vulnerability and health indicators
- Scenario-based hazards and uncertainty

---

## 2. Stakeholders
- **Mayor/city leadership**: resilience strategy and accountability
- **Resilience/climate office**: program design and coordination
- **Infrastructure agencies/utilities**: adaptation of assets and services
- **Emergency management**: preparedness and response planning
- **Public health**: heat illness prevention, air quality impacts
- **Urban planning**: zoning, building codes, green infrastructure
- **Finance**: capital budgeting, insurance and risk transfer
- **Community organizations**: equity and local knowledge
- **IT/data**: geospatial platform and governance

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity of risk results used for investment decisions
- Confidentiality of critical infrastructure vulnerabilities
- Trust and transparency in risk communication

### 3.2 Abuse/failure cases
- **Cherry-picked scenarios** to justify preferred projects
- **Model opacity** leading to distrust and political pushback
- **Data leakage** of sensitive vulnerabilities (e.g., weak points in levees)
- **Overconfidence** from ignoring uncertainty and compounding

### 3.3 Controls
- Versioned scenarios and assumptions with audit trails
- Independent review/red teaming of risk methods
- Access tiering for sensitive layers
- Communication standards that include uncertainty ranges

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Scenario catalog and manager**
   - Hazard scenarios (time horizons, emissions pathways), ensembles

2. **Hazard layers**
   - Heat indices, flood depths/extent, wind fields, wildfire smoke, drought indices

3. **Exposure database**
   - Assets (roads, substations, hospitals), population distributions

4. **Vulnerability/fragility models**
   - Functions mapping hazard intensity to damage/outage/health outcomes

5. **Interdependency model**
   - Cascades: power outage → water pumps offline → service loss

6. **Risk computation engine**
   - Event-based and probabilistic aggregation
   - Loss estimation and downtime estimation

7. **Adaptation portfolio model**
   - Intervention costs, effectiveness curves, constraints

8. **Serving layer**
   - Dashboards, maps, reports, decision brief templates

9. **Governance/observability**
   - Data provenance, QA checks, reproducibility, access logging

### 4.2 Data flows
- Scenario selection → hazard layers pulled/derived
- Hazard + exposure → impact calculations via vulnerability models
- Impacts + interdependencies → service downtime and cascade losses
- Adaptation options → recompute impacts under interventions
- Results → prioritization and capital planning outputs

---

## 5. Methods / algorithms / standards

### 5.1 Risk framing
- Risk = probability × consequence
- Use multiple time horizons: near-term (5–10y), mid (2030–2050), long-term (2100)
- Digital twin-based scenario simulation and visualization

### 5.2 Hazard modeling and ensembles
- Use ensembles to represent uncertainty
- Downscaling considerations: spatial resolution and bias correction
- Machine learning-based climate downscaling
- AI-enhanced hazard prediction models
- Digital twin-based real-time hazard monitoring

### 5.3 Vulnerability and fragility
- Asset fragility curves (hazard intensity → damage state)
- Social vulnerability indices for differential impacts
- Health impact functions for heat and air quality
- Machine learning for vulnerability assessment
- Deep learning for damage prediction from satellite imagery

### 5.4 Compound and cascading risk
- Joint probability modeling for compound hazards
- Dependency graphs for infrastructure interdependencies
- Stress tests with "worst plausible" combined conditions
- Network analysis for cascade failure modeling
- Digital twin-based cascade simulation

### 5.5 Decision-making under uncertainty
- Robust decision making (RDM)
- Real options and adaptive pathways with trigger thresholds
- Multi-objective optimization for adaptation planning
- Reinforcement learning for adaptive management strategies

### 5.6 Standards and protocols
- ISO 14090 for climate change adaptation
- ISO 22301 for business continuity management
- IPCC AR6 climate scenarios and SSP/RCP pathways
- OGC standards for geospatial climate data
- NGSI-LD for context-aware climate risk data
- MQTT/AMQP for real-time hazard monitoring
- WMO standards for meteorological data exchange
- FEMA HAZUS for hazard loss estimation

---

## 6. Data requirements

### 6.1 Core datasets
- Asset registries with geospatial locations and criticality
- Population and sensitive facilities (schools, clinics)
- Historical events for calibration and validation
- Elevation/terrain for flood modeling where relevant

### 6.2 Climate and hazard datasets
- Temperature and heat indices
- Precipitation extremes and IDF curves updates
- Sea level rise projections and coastal flood maps
- Smoke/air quality projections (where relevant)

### 6.3 Data governance
- Sensitive infrastructure layers access-controlled
- Community data: consent and appropriate aggregation

---

## 7. Implementation plan (phases)

### Phase 0 — Define decisions and metrics
- Identify decisions to support (capital plan, emergency thresholds)
- Define outcome metrics: downtime, affected population, cost, equity

### Phase 1 — Baseline risk mapping
- Compile exposure + hazard layers for priority hazards
- Create initial impact estimates with conservative assumptions

### Phase 2 — System stress tests and interdependencies
- Model critical services and cascade pathways
- Stress test under compound events

### Phase 3 — Adaptation portfolio optimization
- Evaluate interventions with costs and effectiveness
- Prioritize based on risk reduction and equity

### Phase 4 — Operational integration
- Tie results to maintenance, asset renewal, and emergency playbooks
- Update scenarios periodically and after major events

---

## 8. Testing and validation
- Historical backtests: reproduce impacts of known events
- Sensitivity analysis: fragility curve parameters, hazard biases
- Peer review: document and validate assumptions
- Ground truth sampling: asset vulnerabilities verified by engineers

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- Scenario reproducibility (same inputs → same outputs)
- Data freshness for exposure layers
- Digital twin synchronization metrics (DT-IRL framework)
- Model prediction accuracy for hazard impacts
- Cross-scenario comparison consistency
- Data quality scores for climate datasets

### 9.2 Example SLOs
- Annual refresh of hazard scenarios and exposure registries
- Published uncertainty bounds for all major risk metrics
- System uptime ≥ 99.9%
- Model accuracy ≥ 85% for impact predictions
- End-to-end latency < 10 seconds for scenario runs

---

## 10. Governance and compliance
- Decision transparency: publish methods and limitations
- Equity governance: community review of prioritization
- Alignment with regulatory reporting and funding requirements
- Secure handling of critical vulnerability information

---

## 11. Risks and mitigations
- **Overly complex models** → start with transparent baselines; add complexity incrementally
- **Scenario paralysis** → define a standard set of scenarios for decisions
- **Equity blind spots** → include distributional metrics and community input

---

## 12. Costs and FinOps
- Costs: geospatial storage, compute for scenario runs, specialized expertise
- FinOps: cost per scenario, caching hazard layers, right-sizing compute

---

## 13. KPIs
- % of capital plan aligned to quantified risk reduction
- Reduction in expected annual downtime for critical services
- Heat vulnerability reduction (population-hours of dangerous heat exposure)
- Funding leveraged for resilience investments

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Climate risk scenario catalog and assumptions register
- Baseline hazard-exposure-impact maps
- Interdependency stress test reports
- Adaptation portfolio prioritization tool
- Communication templates with uncertainty

### 14.2 Readiness checklist
- [ ] Decision questions and metrics agreed
- [ ] Scenarios versioned and approved
- [ ] Sensitive layers access controlled
- [ ] Uncertainty communicated consistently

---

## 15. References
### 15.1 Workspace source
- Item 8 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- Robust Decision Making (RDM) — RAND: https://www.rand.org/pubs/tools/TL320/tool/robust-decision-making.html
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.

### 15.3 Suggested further reading (not fetched)
- Climate risk frameworks (hazard–exposure–vulnerability)
- Robust decision making and adaptive pathways
- Infrastructure interdependency modeling
- Digital twins for climate resilience
- Machine learning for climate risk assessment
- Compound hazard modeling approaches
- Federated learning for distributed climate analytics
