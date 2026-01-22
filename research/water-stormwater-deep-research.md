# Water & stormwater — Deep research

## Executive summary
A water and stormwater “twin” models hydraulic behavior, flooding risk, and asset performance to improve resilience and service continuity. It combines network topology (pipes, manholes, pumps, valves), hydrologic inputs (rainfall, soil moisture), operational states (pump status, tank levels), and observations (flow/level sensors, flooding reports) to forecast outcomes and support interventions (gate operations, pumping strategies, maintenance, and capital upgrades).

The engineering difficulty is data and model coupling: stormwater systems are partially observed, exhibit nonlinear dynamics, and are sensitive to localized rainfall and blockages. A pragmatic approach uses a layered architecture with (1) high-quality asset registry and telemetry ingestion, (2) event-based nowcasting and alerting, and (3) progressively more detailed hydraulic/hydrologic simulation, validated via historical events.

This document deepens item 7 in [`kali-task-research.md`](../kali-task-research.md:1): *“Water & stormwater: Forecast hydraulic behavior, flooding, and asset failures to improve resilience and service continuity.”*

---

## 1. Background and context
Urban water systems include:
- Potable water distribution
- Wastewater collection
- Stormwater drainage (often combined or separated)

Stormwater and flooding challenges:
- Intensifying rainfall extremes
- Aging assets and sediment/blockage issues
- Limited as-built accuracy and unknown connections
- High consequence of failure: property damage, health risks, environmental spills

Twin capabilities enable:
- Early warning for flooding and surcharge conditions
- Prioritization of cleaning and maintenance (e.g., catch basins)
- Evaluation of green infrastructure and storage interventions
- Outage and overflow risk management during storms

---

## 2. Stakeholders
- **Water utility / drainage authority**: operations, maintenance, capital works
- **Emergency management**: flood response coordination
- **Public works**: road closures, debris removal
- **Environmental regulators**: overflow compliance
- **Residents/businesses**: flood warnings, claims, service continuity
- **IT/data**: telemetry, integrations, governance

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity of telemetry (levels, flows, pump status)
- Availability of monitoring and alerts during storms
- Integrity of control actions (pumps, gates)

### 3.2 Abuse/failure cases
- **Telemetry spoofing** hiding rising levels
- **Denial of service** during severe weather causing blind operations
- **Unauthorized control changes** (pump scheduling) increasing flooding
- **Model misuse** leading to false confidence in protection

### 3.3 Controls
- Segmented OT networks; strict remote access
- Alerting redundancy and graceful degradation
- Audit logs for control actions; approvals for high-impact operations

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Asset registry + GIS**
   - Pipes, manholes, inlets, outfalls, pumps, basins
   - Connectivity and attributes (diameters, materials, slopes)

2. **Telemetry ingestion**
   - Level and flow sensors, pump SCADA tags, rain gauges

3. **Observation and incident intake**
   - Citizen flood reports, field crew observations, road closure events

4. **Hydrologic/hydraulic modeling**
   - Rainfall-runoff modeling (catchments)
   - Hydraulic routing through the network

5. **Nowcasting and alerting**
   - Short-term forecasts (minutes to hours)
   - Threshold exceedance and impact zones

6. **Decision support**
   - Pump/gate operation recommendations
   - Maintenance prioritization and cleaning schedules

7. **Serving layer**
   - Dashboards, map-based risk layers, APIs for emergency response

8. **Governance/observability**
   - Data quality monitoring, model validation, audit trails

### 4.2 Data flows
- Rainfall forecasts + gauges → hydrologic inputs
- SCADA + sensors → current system state
- Model → predicted levels/flows/flooding zones
- Alerts → operations and emergency response workflows
- Post-event data → calibration and after-action reports

---

## 5. Methods / algorithms / standards

### 5.1 Rainfall-runoff modeling
- Catchment delineation from terrain and drainage maps
- Loss models (infiltration), imperviousness, storage
- Event-based vs continuous simulation
- Machine learning-based rainfall-runoff prediction
- LSTM/GRU networks for hydrological forecasting
- Digital twin-based real-time rainfall-runoff simulation

### 5.2 Hydraulic modeling
- 1D network flow routing (pipes/channels)
- Storage nodes (tanks, basins) and pump curves
- Backwater effects and surcharge
- Computational fluid dynamics (CFD) for complex flow scenarios
- AI-enhanced hydraulic modeling with real-time calibration
- Digital twin-based hydraulic simulation

### 5.3 Flood impact mapping
- Linking surcharge/outfall conditions to surface flooding proxies
- Combining with terrain models to estimate ponding extents
- GIS-based flood inundation modeling
- Remote sensing for flood extent validation
- Machine learning for flood risk prediction

### 5.4 Asset failure risk
- Condition and failure history models (break rates, blockage probability)
- Prioritization using consequence maps and criticality
- Predictive maintenance for water infrastructure
- Anomaly detection using time-series autoencoders
- Digital twin-based asset health monitoring

### 5.5 Data fusion
- Assimilate sensor observations to update model states
- Uncertainty handling for uninstrumented areas
- Multi-sensor data fusion for improved accuracy
- Kalman filtering for state estimation
- Ensemble modeling for uncertainty quantification

### 5.6 Standards and protocols
- IEC 60870-5-104 for SCADA communication
- DNP3 for remote terminal unit communication
- MQTT/AMQP for IoT sensor data streaming
- OGC WaterML for water data exchange
- OGC SensorThings API for water monitoring
- ISO 19136 for geographic information (GML)
- ISO 24519 for water quality data
- NGSI-LD for context-aware water data sharing
- OPC UA for industrial water system integration

---

## 6. Data requirements

### 6.1 Minimum viable
- GIS network with connectivity (even if imperfect)
- Rain gauges (or radar/rainfall product) with timestamps
- Key telemetry: pump station status, major trunk levels
- Flood incident logs (location/time)

### 6.2 High-value
- High-resolution terrain/DEM
- Radar rainfall nowcasts
- Catch basin inventory and cleaning history
- Water quality sensors at outfalls (for overflow impacts)

### 6.3 Data quality
- Connectivity correctness (topology errors break models)
- Sensor calibration and drift monitoring
- Consistent coordinate systems and time sync

---

## 7. Implementation plan (phases)

### Phase 0 — Governance and critical areas
- Identify high-risk basins and critical assets
- Define alert thresholds and operational playbooks

### Phase 1 — Monitoring + event-based alerting
- Ingest gauges/telemetry and build dashboards
- Implement threshold alerts and incident intake

### Phase 2 — Baseline modeling and calibration
- Build hydrologic/hydraulic model for priority catchments
- Calibrate against historical storms and sensor traces

### Phase 3 — Forecasting and decision support
- Integrate rainfall forecasts and run nowcasts
- Provide recommended actions (pumping, closures)

### Phase 4 — Portfolio scaling and capital planning
- Expand models to more catchments
- Use scenario analysis for green infrastructure and storage investments

---

## 8. Testing and validation
- Topology tests: connectivity and flow direction sanity checks
- Backtesting: replay historical storms and compare predicted vs observed levels
- Alert validation: precision/recall for flood incidents
- Operational drills: ensure playbooks work under data outages

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- Telemetry freshness and sensor uptime during storms
- Forecast error for water levels (MAE) and exceedance detection
- Alert latency from threshold exceedance to notification
- Digital twin synchronization metrics (DT-IRL framework)
- Model prediction accuracy for flood forecasting
- Cross-system data exchange success rate
- Sensor health and calibration status

### 9.2 Example SLOs
- 99% of critical pump station tags updated within 30 seconds
- Flood nowcast updated every 5–10 minutes during events
- System uptime ≥ 99.9%
- Model accuracy ≥ 90% for flood predictions
- End-to-end latency < 3 seconds for critical alerts

---

## 10. Governance, compliance
- Regulatory compliance for overflows and reporting
- Data retention for incident investigations and claims
- Access control for OT data
- Transparent communication of forecast uncertainty

---

## 11. Risks and mitigations
- **Incomplete topology** → phased modeling, field verification, continuous correction
- **Sparse sensors** → prioritize instrumentation and use data assimilation
- **False alarms** → calibrate thresholds; incorporate corroborating signals
- **Operational overload during storms** → automation + clear prioritization rules

---

## 12. Costs and FinOps
- Instrumentation and maintenance of sensors
- Compute for frequent nowcast simulation
- Storage for high-frequency telemetry

Track unit costs:
- Cost per modeled catchment
- Cost per sensor per year (including maintenance)

---

## 13. KPIs
- Reduction in flood incidents (or severity) in targeted areas
- Reduced response time and improved situational awareness
- % critical assets monitored with high uptime
- Improved maintenance targeting (blockage incidents reduced)

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Asset topology and critical basin registry
- Real-time monitoring dashboards and alerting
- Calibrated hydraulic/hydrologic models for priority areas
- Operational playbooks and after-action reporting templates

### 14.2 Readiness checklist
- [ ] Topology QA completed for priority basins
- [ ] Sensor uptime and calibration process in place
- [ ] Alert thresholds validated via backtests
- [ ] Storm operations runbooks exercised

---

## 15. References
### 15.1 Workspace source
- Item 7 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- FEMA Flood Risk Communication Toolkit for Community Officials: https://www.fema.gov/floodplain-management/manage-risk/communication-toolkit-community-officials
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.

### 15.3 Suggested further reading (not fetched)
- Urban hydrology and drainage modeling
- Nowcasting rainfall and flood forecasting approaches
- Asset management for water networks and failure modeling
- Digital twins for water infrastructure management
- Machine learning for flood prediction and early warning
- IoT-based water quality monitoring systems
- Federated learning for distributed water analytics
