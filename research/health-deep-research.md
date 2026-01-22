# Health — Deep research

## Executive summary
A health “twin” for a city links environmental, mobility, and service data to analyze population health outcomes and evaluate intervention effectiveness. It connects exposures (air quality, heat, noise, greenspace access), behaviors (mobility patterns, physical activity proxies), and service access (healthcare facilities, social services) to health indicators (hospital visits, emergency calls, chronic disease prevalence).

The value is in moving from correlation to causal inference: understanding how specific interventions (e.g., low-emission zones, cooling centers, active transport infrastructure) affect health outcomes, and quantifying health co-benefits of climate and mobility policies. Privacy is paramount because health data is highly sensitive; the system must operate on aggregated, de-identified data with strong governance.

This document deepens item 12 in [`kali-task-research.md`](../kali-task-research.md:1): *“Health: Link environmental, mobility, and service data to analyze population outcomes and intervention effectiveness.”*

---

## 1. Background and context
Urban health is shaped by:
- Environmental exposures: air pollution, heat, noise, greenspace
- Built environment: walkability, bikeability, housing quality
- Social determinants: income, education, access to services
- Mobility patterns: commute times, access to jobs and healthcare

Traditional health data sources:
- Hospital admissions and emergency department visits
- Ambulance dispatch and 911 calls
- Public health surveillance (reportable diseases)
- Survey data (limited frequency and coverage)

A twin approach enables:
- Near-real-time exposure mapping (air quality, heat islands)
- Linking exposures to health events with spatial/temporal resolution
- Evaluating policy impacts (e.g., traffic reduction → asthma visits)
- Identifying vulnerable populations and targeting interventions

---

## 2. Stakeholders
- **Public health department**: surveillance, policy evaluation, outbreak response
- **Hospitals and healthcare providers**: capacity planning, community health
- **Emergency services**: demand forecasting, resource allocation
- **Environmental agencies**: air quality, climate adaptation
- **Transportation/planning**: active transport, accessibility
- **Community organizations**: local health priorities, equity
- **Researchers**: causal inference, health impact assessment
- **IT/data**: secure data exchange, privacy controls

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Confidentiality of health data (diagnoses, visits, sensitive demographics)
- Integrity of exposure estimates used for policy decisions
- Trust in health equity analyses

### 3.2 Abuse/failure cases
- **Re-identification** from joined health + location data
- **Stigmatization** of neighborhoods based on health outcomes
- **Misuse** of health data for non-health purposes (e.g., insurance, enforcement)
- **Incorrect attribution** of health effects to policies

### 3.3 Controls
- Aggregation thresholds and spatial/temporal binning
- Strict access controls and audit logging
- Independent review of health impact assessments
- Transparency about limitations and uncertainty

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Exposure modeling layer**
   - Air quality, heat, noise, greenspace access
   - Mobility-derived exposures (walkability, access to services)

2. **Health data ingestion**
   - Hospital admissions, ED visits, ambulance dispatch
   - Public health surveillance and reportable diseases

3. **Population and demographic layer**
   - Census, age/sex distributions, vulnerability indices
   - Sensitive facility locations (schools, nursing homes)

4. **Linkage and analysis engine**
   - Spatiotemporal linkage of exposures and health events
   - Causal inference methods (difference-in-differences, synthetic controls)
   - Health impact functions (e.g., concentration-response for PM2.5)

5. **Intervention evaluation**
   - Before/after analysis of policy changes
   - Scenario modeling for proposed interventions

6. **Serving layer**
   - Dashboards for public health officials
   - Public-facing health indicators (appropriately aggregated)

7. **Governance/observability**
   - Data access controls, audit logs, privacy checks

### 4.2 Data flows
- Environmental/mobility data → exposure estimates
- Health events → de-identified, aggregated counts
- Exposures + health events → statistical analysis
- Interventions → evaluation → policy recommendations

---

## 5. Methods / algorithms / standards

### 5.1 Exposure assessment
- Spatiotemporal interpolation of air quality and heat
- Walkability and accessibility indices
- Cumulative exposure metrics (e.g., person-hours above thresholds)
- Machine learning-based exposure prediction using satellite and IoT data
- Digital twin-based real-time exposure modeling

### 5.2 Health impact assessment
- Concentration-response functions for air pollution and heat
- Burden of disease calculations (DALYs, premature deaths)
- Health economic valuation where appropriate
- AI-powered health outcome prediction models
- Time-series forecasting for health event surges

### 5.3 Causal inference for policy evaluation
- Difference-in-differences for before/after policy changes
- Synthetic control methods for city-level interventions
- Regression discontinuity for threshold-based policies
- Propensity score matching for intervention evaluation
- Counterfactual analysis using digital twin simulations

### 5.4 Equity analysis
- Distributional analysis by neighborhood and demographic groups
- Environmental justice metrics (exposure disparities)
- Access to care and service availability
- Fairness-aware machine learning for health equity

### 5.5 Outbreak and syndromic surveillance
- Anomaly detection in health event patterns
- Early warning for heat-related illness, respiratory events
- Time-series autoencoders for outbreak detection
- LSTM/GRU models for syndromic surveillance
- Digital twin-based epidemic simulation

### 5.6 Privacy-preserving analytics
- Differential privacy for health data aggregation and reporting
- Federated learning for cross-institutional model training
- Secure multi-party computation for joint health analysis
- Homomorphic encryption for encrypted health analytics
- Zero-trust architecture for secure health data exchange

### 5.7 Standards and protocols
- HL7 FHIR for health data interoperability
- ICD-10/ICD-11 for disease classification
- LOINC for laboratory observations
- SNOMED CT for clinical terminology
- GDPR and HIPAA compliance frameworks
- ISO 27001 for health information security
- NGSI-LD for context-aware health data sharing
- MQTT/AMQP for real-time health sensor data
- OGC standards for geospatial health data

External reference: Environmental determinants of population health in urban settings (PMC open-access review) ([PMC article](https://pmc.ncbi.nlm.nih.gov/articles/PMC7271472/)).

---

## 6. Data requirements

### 6.1 Core datasets
- Health events: hospital admissions, ED visits, ambulance dispatch (de-identified)
- Environmental: air quality, temperature, humidity, noise
- Built environment: greenspace, walkability, housing quality
- Demographics: population by age/sex, vulnerability indices

### 6.2 High-value datasets
- Chronic disease prevalence (where available)
- Social determinants: income, education, employment
- Healthcare facility locations and capacity

### 6.3 Data quality and privacy
- Consistent geocoding and spatial aggregation
- Minimum cell size thresholds for privacy
- Temporal aggregation (e.g., weekly counts) for rare events

---

## 7. Implementation plan (phases)

### Phase 0 — Governance and privacy framework
- Define data sharing agreements with health agencies
- Establish privacy thresholds and access controls

### Phase 1 — Exposure mapping and baseline health indicators
- Build air quality and heat exposure maps
- Create baseline health dashboards (appropriately aggregated)

### Phase 2 — Linkage and analysis
- Link exposures to health events using spatiotemporal methods
- Implement health impact functions for key exposures

### Phase 3 — Intervention evaluation
- Evaluate past interventions (e.g., traffic reduction, cooling centers)
- Build scenario modeling for proposed policies

### Phase 4 — Operational integration
- Integrate with public health surveillance
- Provide early warning for environmental health risks

---

## 8. Testing and validation
- Validate exposure models against monitoring data
- Backtest health impact predictions against historical events
- Sensitivity analysis for concentration-response parameters
- Peer review of causal inference methods

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- Data freshness for health event aggregates
- Exposure model accuracy (RMSE vs monitors)
- Analysis run time for policy evaluations
- Digital twin synchronization metrics (DT-IRL framework)
- Model prediction accuracy for health outcomes
- Privacy budget consumption (ϵ tracking)
- Data quality scores for health datasets
- Cross-institutional data exchange success rate

### 9.2 Example SLOs
- Weekly health aggregates available within 7 days
- Exposure maps updated daily during high-risk periods
- System uptime ≥ 99.9%
- Model accuracy ≥ 90% for health outcome predictions
- End-to-end latency < 5 seconds for real-time alerts

---

## 10. Governance, compliance, and ethics
- Align with health data privacy regulations
- Establish data use agreements and oversight
- Ensure community engagement and transparency
- Avoid stigmatization in public communication

---

## 11. Risks and mitigations
- **Re-identification risk** → strict aggregation and access controls
- **Ecological fallacy** → clearly communicate limitations
- **Confounding factors** → use robust causal inference methods
- **Equity blind spots** → include distributional analysis

---

## 12. Costs and FinOps
- Data integration and secure storage
- Statistical modeling and causal inference compute
- Staff time for health agency coordination

Track:
- Cost per health impact assessment
- Cost per exposure map update

---

## 13. KPIs
- Reduction in exposure-related health events
- Improved targeting of interventions to vulnerable populations
- Faster detection of environmental health risks
- Health co-benefits quantified for climate/mobility policies

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Exposure mapping service (air quality, heat, greenspace)
- Health indicator dashboards (appropriately aggregated)
- Intervention evaluation reports
- Health impact assessment toolkit

### 14.2 Readiness checklist
- [ ] Data sharing agreements in place
- [ ] Privacy thresholds defined and enforced
- [ ] Exposure models validated
- [ ] Causal inference methods peer-reviewed

---

## 15. References
### 15.1 Workspace source
- Item 12 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- Environmental determinants of population health in urban settings (PMC): https://pmc.ncbi.nlm.nih.gov/articles/PMC7271472/
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Murala et al. (2025). "A service-oriented microservice framework for differential privacy-based protection in industrial IoT smart applications." Scientific Reports, 15, 29230.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.

### 15.3 Suggested further reading (not fetched)
- Health impact assessment methods and guidelines
- Causal inference for policy evaluation
- Environmental justice and health equity frameworks
- Digital twins for public health surveillance
- Federated learning for healthcare analytics
- Privacy-preserving health data sharing frameworks