# Water & stormwater — Deep research

## Executive summary
A water and stormwater "twin" models hydraulic behavior, flooding risk, and asset performance to improve resilience and service continuity. It combines network topology (pipes, manholes, pumps, valves), hydrologic inputs (rainfall, soil moisture), operational states (pump status, tank levels), and observations (flow/level sensors, flooding reports) to forecast outcomes and support interventions (gate operations, pumping strategies, maintenance, and capital upgrades).

The engineering difficulty is data and model coupling: stormwater systems are partially observed, exhibit nonlinear dynamics, and are sensitive to localized rainfall and blockages. A pragmatic approach uses a layered architecture with (1) high-quality asset registry and telemetry ingestion, (2) event-based nowcasting and alerting, and (3) progressively more detailed hydraulic/hydrologic simulation, validated via historical events.

This document deepens item 7 in [`kali-task-research.md`](../kali-task-research.md:1): *"Water & stormwater: Forecast hydraulic behavior, flooding, and asset failures to improve resilience and service continuity."*

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

## 5. Tooling and model stack

### 5.1 Hydrology and hydraulic tools

| Tool | Type | Use Case | Notes |
|------|------|----------|-------|
| **SWMM** (EPA) | Open-source | 1D hydraulic, rainfall-runoff | Widely used, good for combined sewers |
| **HEC-HMS** | Free (USACE) | Hydrologic modeling | Watershed-scale rainfall-runoff |
| **HEC-RAS 1D/2D** | Free (USACE) | 1D/2D hydraulic modeling | Riverine and urban flood modeling |
| **InfoWorks ICM** | Commercial | Integrated 1D/2D | Full-featured, expensive, good for complex systems |
| **MIKE URBAN / MIKE+** | Commercial | 1D/2D, water distribution | DHI suite, strong in water quality |
| **XPSWMM** | Commercial | 1D/2D, real-time | Good for operational applications |
| **TUFLOW** | Commercial | 2D flood modeling | High-resolution surface flooding |
| **OpenFOAM** | Open-source | CFD | Complex flow scenarios, computationally intensive |

### 5.2 Fast vs detailed operational model strategy

#### Fast models (nowcasting, minute-to-minute operations)
- **Purpose**: Real-time decision support during active events
- **Characteristics**:
  - Run time: seconds to minutes
  - Update frequency: 1-15 minutes
  - Simplified physics, reduced-order models
  - Surrogate models (ML-based emulators of detailed models)
- **Typical tools**:
  - SWMM with simplified catchments
  - Pre-computed lookup tables / response surfaces
  - ML models (LSTM, gradient boosting) trained on detailed simulations
  - Statistical nowcasting (ARIMA, Kalman filters)
- **Use cases**:
  - Pump scheduling optimization
  - Gate operation timing
  - Real-time flood alerting
  - CSO prediction and mitigation

#### Detailed models (planning, scenario testing, forensic replay)
- **Purpose**: Design, capital planning, post-event analysis
- **Characteristics**:
  - Run time: minutes to hours
  - High-fidelity physics
  - Full 1D/2D coupling
  - Detailed terrain and infrastructure
- **Typical tools**:
  - InfoWorks ICM, MIKE+, XPSWMM
  - HEC-RAS 2D for surface flooding
  - TUFLOW for high-resolution 2D
- **Use cases**:
  - Capital project design and sizing
  - Green infrastructure evaluation
  - Historical event reconstruction
  - Regulatory compliance modeling
  - Training and scenario planning

### 5.3 Operational model pipeline architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    STREAMING LAYER (Real-time)                   │
├─────────────────────────────────────────────────────────────────┤
│  Rainfall (radar/gauges)  │  SCADA/Telemetry  │  Observations   │
│         ↓                 │         ↓         │        ↓        │
│  Message Queue (Kafka/Pulsar) ←→ Time-series DB (Influx/Timescale) │
└─────────────────────────────────────────────────────────────────┘
                               ↓
┌─────────────────────────────────────────────────────────────────┐
│                  FAST MODEL EXECUTION (Nowcasting)               │
├─────────────────────────────────────────────────────────────────┤
│  Scheduler (Airflow/K8s Cron)  →  Model Runner (Containerized)   │
│         ↓                                                        │
│  Reduced-order models / ML emulators (GPU/CPU cluster)           │
│         ↓                                                        │
│  Results → Alert Engine → Dashboard / API                        │
└─────────────────────────────────────────────────────────────────┘
                               ↓
┌─────────────────────────────────────────────────────────────────┐
│                  BATCH LAYER (Detailed Analysis)                 │
├─────────────────────────────────────────────────────────────────┤
│  Event trigger → Detailed model job queue (SLURM/AWS Batch)      │
│         ↓                                                        │
│  Full 1D/2D models (HPC cluster or cloud)                        │
│         ↓                                                        │
│  Results → Post-processing → Archive / Reporting                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Compute and scheduling considerations
- **Fast models**: Deploy on Kubernetes with auto-scaling; target <2 min end-to-end latency
- **Detailed models**: Use HPC cluster or cloud batch; queue management for concurrent runs
- **Data locality**: Pre-stage model inputs near compute; use shared storage (NFS/S3)
- **Caching**: Cache intermediate results for common scenarios
- **Priority queues**: Operational runs > calibration runs > scenario analysis

---

## 6. Surface flooding coupling + ground truth

### 6.1 Ground-truth data sources

| Source | Type | Frequency | Coverage | Quality Notes |
|--------|------|-----------|----------|---------------|
| **311/citizen reports** | Crowdsourced | Event-based | City-wide | Variable quality, bias toward reported areas |
| **Insurance claims** | Administrative | Post-event | Affected properties | High accuracy, delayed, privacy constraints |
| **Road closure logs** | Operational | Event-based | Public roads | Reliable, limited to major roads |
| **Field crew observations** | Professional | Event-based | Targeted areas | High quality, limited coverage |
| **IoT water level sensors** | Automated | 1-15 min | Instrumented locations | High accuracy, sparse coverage |
| **Traffic cameras** | Visual | Continuous | Major intersections | CV-based detection, weather-dependent |
| **Social media** | Crowdsourced | Continuous | City-wide | High noise, requires filtering |
| **Remote sensing (SAR)** | Satellite | Per-pass | Regional | Weather-independent, coarse resolution |
| **Remote sensing (optical)** | Satellite/Drone | Per-pass | Targeted | Weather-dependent, high resolution |

### 6.2 Practical ingestion patterns

#### 311/citizen reports
- **Integration**: API integration with 311 system or daily CSV export
- **Processing**: Geocode addresses, deduplicate, classify severity
- **Validation**: Cross-reference with sensor data and model predictions

#### Insurance claims
- **Integration**: Secure data sharing agreements with insurers
- **Processing**: Aggregate to block/neighborhood level for privacy
- **Use**: Post-event model validation, risk mapping

#### IoT water level sensors
- **Types**: Ultrasonic, pressure transducers, radar
- **Placement**: Low points, flood-prone intersections, critical infrastructure
- **Telemetry**: Store-and-forward with local buffering (see Section 9)

#### Traffic cameras
- **Approach**: Computer vision for water detection (avoid full AI hype)
- **Limitations**: Night visibility, heavy rain obscuration, camera angle
- **Use**: Qualitative confirmation, not quantitative measurement

#### Social media
- **Platforms**: Twitter/X, local community groups
- **Filtering**: Keyword-based + geolocation + confidence scoring
- **Caution**: High false positive rate; use as corroborating signal only

### 6.3 Bias and coverage issues

#### Under-reporting by neighborhood
- **Causes**: Digital divide, language barriers, trust issues
- **Mitigation**:
  - Targeted sensor placement in under-reported areas
  - Community outreach and alternative reporting channels
  - Statistical normalization using population/density metrics

#### Equity impacts
- **Risk**: Model calibration biased toward well-instrumented areas
- **Mitigation**:
  - Equity audit of sensor placement and incident coverage
  - Explicit coverage targets for vulnerable communities
  - Confidence-weighted predictions with uncertainty bands

#### Confidence scoring
- **Multi-source fusion**: Combine observations with reliability weights
- **Temporal decay**: Recent observations weighted higher
- **Spatial interpolation**: Kriging with uncertainty for unobserved areas

---

## 7. Data governance + liability

### 7.1 Forecast publishing governance

#### Uncertainty communication
- **Uncertainty bands**: Always provide confidence intervals (e.g., 50%, 90%)
- **Ensemble forecasts**: Show range of model outcomes
- **Probability thresholds**: Express as probabilities (e.g., "70% chance of exceedance")
- **Avoid overconfidence**: Never present deterministic forecasts as certain

#### Disclaimers and limitations
- **Standard disclaimer**: *"This system provides decision support, not decision replacement. Operators must use professional judgment and consider local conditions."*
- **Model limitations**: Document known weaknesses and assumptions
- **Data quality flags**: Highlight when input data is degraded or missing

### 7.2 Retention and evidentiary logging

#### Retention policies
| Data Type | Retention Period | Rationale |
|-----------|------------------|-----------|
| Telemetry (raw) | 2-5 years | Incident investigation, model calibration |
| Model outputs | 5-10 years | Claims defense, regulatory compliance |
| Control actions | Permanent | Audit trail, liability protection |
| Incident reports | Permanent | Historical record, trend analysis |
| Calibration datasets | 5+ years | Model reproducibility |

#### Evidentiary logging requirements
- **Immutable logs**: Write-once storage for critical events
- **Chain of custody**: Track data provenance from source to model
- **Timestamp synchronization**: NTP-synchronized clocks across all systems
- **Tamper detection**: Cryptographic hashing of log entries

### 7.3 Model versioning and reproducibility

#### Model versioning
- **Semantic versioning**: Major.minor.patch for model releases
- **Configuration as code**: Store all model parameters in version control
- **Data versioning**: Track input datasets with checksums
- **Containerization**: Docker images for reproducible execution environments

#### Dataset lineage
- **Data catalog**: Track source systems, transformations, and quality metrics
- **Pipeline provenance**: Record all processing steps from raw data to model input
- **Change tracking**: Alert on schema changes or data quality degradation

#### Audit trails
- **Who**: User/service identity for all actions
- **What**: Action taken (model run, parameter change, control action)
- **When**: Precise timestamp
- **Why**: Context and justification (especially for control actions)

---

## 8. Operational playbooks + decision authority

### 8.1 Pre-storm readiness checklist

#### 24-48 hours before forecasted event
- [ ] Verify all critical telemetry is online and reporting
- [ ] Check pump station fuel levels and backup power
- [ ] Confirm SCADA communication paths are functional
- [ ] Review and update model parameters (seasonal adjustments)
- [ ] Pre-position field crews and equipment
- [ ] Test communication channels (radio, cellular backup)
- [ ] Review and distribute updated contact lists
- [ ] Clear catch basins in high-risk areas (if time permits)

#### 4-12 hours before event
- [ ] Run pre-event model scenarios with forecast rainfall
- [ ] Identify at-risk assets and potential flood zones
- [ ] Pre-stage pumps at optimal levels
- [ ] Confirm gate/valve positions are correct
- [ ] Activate emergency operations center if warranted
- [ ] Issue public communications if threshold exceeded

### 8.2 During-storm monitoring and triggers

#### Monitoring dashboard requirements
- Real-time rainfall (gauge + radar)
- Water levels at critical nodes
- Pump station status and flow rates
- Gate/valve positions
- Active flood reports (map view)
- Model predictions vs observations

#### Trigger thresholds (example)

| Metric | Warning | Action | Critical | Action |
|--------|---------|--------|----------|--------|
| Rainfall rate | 25 mm/hr | Increase monitoring frequency | 50 mm/hr | Activate EOC |
| Water level at node X | 80% capacity | Prepare pumps | 95% capacity | Maximize pumping |
| Pump station flow | Below expected | Investigate | Failure | Deploy backup |
| Flood reports | 1-2 in area | Dispatch crew | 5+ in area | Road closure |

#### Pump/gate operation guidelines
- **Pumps**: Operate to maintain target levels; avoid rapid cycling
- **Gates**: Balance upstream vs downstream impacts; document rationale
- **CSO structures**: Minimize overflows; document for compliance

### 8.3 Post-storm workflow

#### Immediate (0-24 hours)
- [ ] Document all control actions and decisions
- [ ] Collect field observations and damage reports
- [ ] Verify all telemetry returned to normal
- [ ] Begin data collection for model calibration

#### Short-term (1-7 days)
- [ ] Run forensic model replay with actual rainfall
- [ ] Compare predictions vs observations
- [ ] Identify model calibration needs
- [ ] Generate after-action report
- [ ] Update asset condition database

#### Long-term (1-3 months)
- [ ] Calibrate models with event data
- [ ] Review and update operational procedures
- [ ] Plan infrastructure improvements based on findings
- [ ] Conduct lessons learned session

### 8.4 Escalation paths and decision authority

#### Decision authority matrix

| Action | Authority | Approval Required |
|--------|-----------|-------------------|
| Routine pump scheduling | Shift supervisor | None |
| Gate position changes | Operations manager | None (documented) |
| Road closures | Public works director | Emergency manager notification |
| Public flood warnings | Emergency manager | Mayor/CAO (for major events) |
| CSO reporting | Environmental compliance | Legal review (if litigation risk) |
| Model parameter changes | Model owner | Technical lead review |

#### Escalation triggers
- **Level 1**: Single asset anomaly → Shift supervisor
- **Level 2**: Multiple assets or minor flooding → Operations manager
- **Level 3**: Widespread flooding or major failure → Emergency operations center
- **Level 4**: Life-threatening situation → Mayor/CAO + state emergency management

---

## 9. Resilience of telemetry during storms

### 9.1 Communications outage assumptions

#### Design assumptions
- **Cellular**: May be degraded during severe storms (tower damage, congestion)
- **Internet**: May experience outages; assume 4-8 hour maximum
- **Radio**: Primary backup for critical SCADA; assume 99%+ availability
- **Satellite**: Tertiary backup for remote sites

#### Redundancy patterns
- **Primary**: Fiber/cable internet
- **Secondary**: Cellular (multiple carriers)
- **Tertiary**: Radio mesh or satellite
- **Local**: Store-and-forward with local buffering

### 9.2 Power outage assumptions

#### Design assumptions
- **Grid**: May experience extended outages during major storms
- **Backup**: Assume 4-24 hours generator fuel depending on site
- **Solar**: Limited value during storms; assume minimal contribution

#### Power redundancy
- **Critical sites**: Dual feeds + generator + UPS
- **Important sites**: Grid + generator
- **Standard sites**: Grid only with battery backup for graceful shutdown

### 9.3 Redundant rainfall sources

| Source | Primary/Backup | Latency | Reliability | Use Case |
|--------|----------------|---------|-------------|----------|
| Local rain gauges | Primary | <1 min | High (if maintained) | Ground truth, calibration |
| Weather radar | Primary | 5-15 min | High | Spatial coverage, nowcasting |
| Third-party feeds | Backup | 15-30 min | Medium | Cross-validation |
| Satellite estimates | Tertiary | 30-60 min | Medium | Regional context |

#### Fallback strategy
- If primary gauge fails: Use nearest gauge + radar bias correction
- If radar unavailable: Use gauge interpolation + satellite estimates
- If all remote sources fail: Use persistence forecast with high uncertainty

### 9.4 Store-and-forward telemetry

#### Local buffering requirements
- **Buffer size**: Minimum 7 days of data for critical sensors
- **Storage**: Industrial-grade SD card or local SSD
- **Power**: Battery-backed to ensure graceful shutdown

#### Time synchronization
- **Primary**: NTP over internet
- **Backup**: GPS time at critical sites
- **Fallback**: Local clock with drift tracking (document uncertainty)

### 9.5 Degraded-mode operations

#### Degradation levels

| Level | Conditions | Operations |
|-------|------------|------------|
| Normal | All systems operational | Full automation, real-time modeling |
| Degraded | Partial telemetry loss | Manual monitoring, simplified models |
| Critical | Major telemetry loss | Field observations, manual procedures |
| Emergency | No telemetry | Crew-based reporting, no modeling |

#### Fallback observation methods
- **Manual readings**: Field crews with portable sensors
- **Visual observations**: Crew reports, citizen reports
- **Historical patterns**: Use similar historical events as reference
- **Conservative assumptions**: Assume worst-case for critical decisions

### 9.6 Redundancy for critical OT/SCADA pathways

#### Network redundancy
- **Dual paths**: Primary and secondary network routes
- **Protocol diversity**: Mix of wired and wireless
- **Device redundancy**: Dual RTUs at critical sites

#### Failover testing
- **Quarterly**: Test failover to backup communications
- **Annually**: Full disaster recovery drill
- **After major changes**: Re-test failover procedures

---

## 10. Water quality + CSO emphasis

### 10.1 CSO compliance reporting workflow

#### Event detection
- **Real-time**: Monitor flow levels at CSO structures
- **Threshold-based**: Trigger when flow exceeds treatment capacity
- **Model-based**: Predict CSO events before they occur

#### Reporting requirements (typical regulatory)
| Report Type | Timing | Content | Recipient |
|-------------|--------|---------|-----------|
| Immediate notification | Within 2-4 hours | Location, estimated volume, duration | Regulator |
| Preliminary report | Within 24-48 hours | Detailed event summary | Regulator |
| Final report | Within 30 days | Full analysis, mitigation measures | Regulator |
| Annual summary | Yearly | All events, trends, improvements | Regulator + public |

#### Automated reporting
- **Data capture**: Automatic logging of all CSO events
- **Template generation**: Pre-populated reports with event data
- **Review workflow**: Compliance officer review before submission
- **Audit trail**: Track all report versions and approvals

### 10.2 Environmental impact metrics

#### Volume-based metrics
- **Total overflow volume**: Per event and annual totals
- **Volume per receiving water**: Segment by outfall location
- **Volume per capita**: Normalize for population served

#### Duration-based metrics
- **Event duration**: Time from start to end of overflow
- **Annual overflow days**: Total days with any overflow
- **Peak duration**: Longest single event

#### Receiving water sensitivity
- **Classification**: High/medium/low sensitivity based on:
  - Designated uses (swimming, fishing, drinking water)
  - Ecological value (endangered species, critical habitat)
  - Proximity to intakes and recreation areas
- **Weighting**: Higher weight for overflows to sensitive waters

#### Combined metrics
- **Load-duration**: Volume × duration × sensitivity weight
- **Frequency-severity**: Event count × average volume

### 10.3 Sampling and sensor strategy

#### Sensor types and placement

| Sensor Type | Parameter | Placement | Frequency | Limitations |
|-------------|-----------|-----------|-----------|-------------|
| Flow meters | Flow rate | CSO structure | Continuous | Accuracy at low flows |
| Level sensors | Water level | CSO structure | Continuous | Indirect flow measurement |
| Turbidity | Suspended solids | Outfall | Continuous | Fouling, calibration |
| Conductivity | Salinity/contamination | Outfall | Continuous | Non-specific |
| pH | Acidity | Outfall | Continuous | Limited information |
| DO | Dissolved oxygen | Receiving water | Continuous | Location-dependent |
| Bacteria (E. coli) | Fecal contamination | Outfall | Event-based | Lab processing time |
| Nutrients (N, P) | Eutrophication potential | Outfall | Event-based | Lab processing time |

#### Sampling strategy
- **Routine**: Weekly/biweekly grab samples at key outfalls
- **Event-based**: Automatic samplers triggered by overflow events
- **Composite**: Flow-proportional sampling for load calculations
- **Quality control**: Field blanks, duplicates, matrix spikes

#### Calibration and maintenance
- **Calibration schedule**: Quarterly for in-situ sensors
- **Field verification**: Monthly comparison with grab samples
- **Fouling mitigation**: Automatic wipers, anti-fouling coatings
- **Data quality flags**: Mark suspect data based on maintenance status

### 10.4 Event-based reporting and evidentiary retention

#### Event data package
For each CSO event, retain:
- **Telemetry**: Flow, level, pump status (full time series)
- **Rainfall**: Gauge and radar data (spatial and temporal)
- **Model outputs**: Predictions vs observations
- **Field observations**: Crew reports, photos
- **Sampling results**: Lab reports, quality control data
- **Control actions**: Pump/gate operations with timestamps
- **Communications**: Public notifications, regulator communications

#### Retention requirements
- **Minimum**: 5 years (typical regulatory requirement)
- **Recommended**: 10+ years for trend analysis and defense against claims
- **Format**: Immutable storage with checksums

#### Chain of custody
- **Data provenance**: Track from sensor to final report
- **Audit trail**: All data access and modifications logged
- **Third-party validation**: Option for independent audit

---

## 11. Methods / algorithms / standards

### 11.1 Rainfall-runoff modeling
- Catchment delineation from terrain and drainage maps
- Loss models (infiltration), imperviousness, storage
- Event-based vs continuous simulation
- Machine learning-based rainfall-runoff prediction
- LSTM/GRU networks for hydrological forecasting
- Digital twin-based real-time rainfall-runoff simulation

### 11.2 Hydraulic modeling
- 1D network flow routing (pipes/channels)
- Storage nodes (tanks, basins) and pump curves
- Backwater effects and surcharge
- Computational fluid dynamics (CFD) for complex flow scenarios
- AI-enhanced hydraulic modeling with real-time calibration
- Digital twin-based hydraulic simulation

### 11.3 Flood impact mapping
- Linking surcharge/outfall conditions to surface flooding proxies
- Combining with terrain models to estimate ponding extents
- GIS-based flood inundation modeling
- Remote sensing for flood extent validation
- Machine learning for flood risk prediction

### 11.4 Asset failure risk
- Condition and failure history models (break rates, blockage probability)
- Prioritization using consequence maps and criticality
- Predictive maintenance for water infrastructure
- Anomaly detection using time-series autoencoders
- Digital twin-based asset health monitoring

### 11.5 Data fusion
- Assimilate sensor observations to update model states
- Uncertainty handling for uninstrumented areas
- Multi-sensor data fusion for improved accuracy
- Kalman filtering for state estimation
- Ensemble modeling for uncertainty quantification

### 11.6 Standards and protocols
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

## 12. Data requirements

### 12.1 Minimum viable
- GIS network with connectivity (even if imperfect)
- Rain gauges (or radar/rainfall product) with timestamps
- Key telemetry: pump station status, major trunk levels
- Flood incident logs (location/time)

### 12.2 High-value
- High-resolution terrain/DEM
- Radar rainfall nowcasts
- Catch basin inventory and cleaning history
- Water quality sensors at outfalls (for overflow impacts)

### 12.3 Data quality
- Connectivity correctness (topology errors break models)
- Sensor calibration and drift monitoring
- Consistent coordinate systems and time sync

---

## 13. Implementation roadmap

### 0-3 months: Foundation
- [ ] Establish governance structure and decision authority
- [ ] Inventory existing assets and telemetry
- [ ] Identify high-risk basins and critical assets
- [ ] Define alert thresholds and operational playbooks
- [ ] Set up basic data ingestion (SCADA, rain gauges)
- [ ] Create initial monitoring dashboard
- [ ] Establish data retention and logging policies

### 3-12 months: Core capabilities
- [ ] Implement fast model for priority catchments (SWMM or similar)
- [ ] Calibrate model against 2-3 historical events
- [ ] Deploy nowcasting and alerting system
- [ ] Integrate ground-truth data sources (311, field reports)
- [ ] Implement store-and-forward for critical sensors
- [ ] Conduct first operational drill
- [ ] Establish CSO monitoring and reporting workflow

### 12-24 months: Advanced capabilities
- [ ] Deploy detailed 1D/2D models for full system
- [ ] Implement surrogate models for real-time operations
- [ ] Expand sensor network to under-served areas
- [ ] Implement automated CSO compliance reporting
- [ ] Add water quality monitoring at key outfalls
- [ ] Develop scenario analysis for capital planning
- [ ] Full integration with emergency management systems

---

## 14. Key metrics

### 14.1 Operational metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Telemetry freshness (critical assets) | <30 seconds | SCADA latency monitoring |
| Sensor uptime during storms | >95% | Availability monitoring |
| Alert latency (threshold to notification) | <3 minutes | End-to-end timing |
| False alarm rate | <10% | Alert validation |
| Missed event rate | <5% | Post-event analysis |
| System availability | >99.9% | Uptime monitoring |

### 14.2 Model performance metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Water level MAE (calibration events) | <10% of range | Backtesting |
| Flood prediction accuracy | >85% | Event validation |
| CSO prediction accuracy | >80% | Event validation |
| Nowcast update frequency | 5-10 minutes | During events |
| Detailed model run time | <2 hours | Performance monitoring |

### 14.3 Equity and bias metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Sensor coverage equity index | >0.8 (normalized) | Coverage analysis |
| Incident reporting rate parity | <20% variance | Demographic analysis |
| Model accuracy by neighborhood | <15% variance | Spatial validation |
| Vulnerable community alert reach | >90% | Outreach tracking |

---

## 15. Testing and validation
- Topology tests: connectivity and flow direction sanity checks
- Backtesting: replay historical storms and compare predicted vs observed levels
- Alert validation: precision/recall for flood incidents
- Operational drills: ensure playbooks work under data outages
- Model calibration: quantitative metrics for fit quality
- Equity audit: validate coverage and accuracy across demographics

---

## 16. Observability (SLIs/SLOs)

### 16.1 SLIs
- Telemetry freshness and sensor uptime during storms
- Forecast error for water levels (MAE) and exceedance detection
- Alert latency from threshold exceedance to notification
- Digital twin synchronization metrics (DT-IRL framework)
- Model prediction accuracy for flood forecasting
- Cross-system data exchange success rate
- Sensor health and calibration status

### 16.2 Example SLOs
- 99% of critical pump station tags updated within 30 seconds
- Flood nowcast updated every 5–10 minutes during events
- System uptime ≥ 99.9%
- Model accuracy ≥ 90% for flood predictions
- End-to-end latency < 3 seconds for critical alerts

---

## 17. Governance, compliance
- Regulatory compliance for overflows and reporting
- Data retention for incident investigations and claims
- Access control for OT data
- Transparent communication of forecast uncertainty
- Model versioning and reproducibility
- Audit trails for control actions
- Equity and bias monitoring

---

## 18. Risks and mitigations
- **Incomplete topology** → phased modeling, field verification, continuous correction
- **Sparse sensors** → prioritize instrumentation and use data assimilation
- **False alarms** → calibrate thresholds; incorporate corroborating signals
- **Operational overload during storms** → automation + clear prioritization rules
- **Telemetry outages during storms** → redundant communications, store-and-forward
- **Model overconfidence** → uncertainty bands, ensemble forecasts, disclaimers
- **Equity gaps** → targeted sensor placement, coverage monitoring

---

## 19. Costs and FinOps
- Instrumentation and maintenance of sensors
- Compute for frequent nowcast simulation
- Storage for high-frequency telemetry
- Software licenses (if using commercial tools)
- Personnel for operations and maintenance

Track unit costs:
- Cost per modeled catchment
- Cost per sensor per year (including maintenance)
- Cost per forecast run
- Cost per CSO report generated

---

## 20. KPIs
- Reduction in flood incidents (or severity) in targeted areas
- Reduced response time and improved situational awareness
- % critical assets monitored with high uptime
- Improved maintenance targeting (blockage incidents reduced)
- CSO volume reduction (percent and absolute)
- Compliance reporting timeliness and accuracy
- Equity improvement in coverage and service

---

## 21. Deliverables and checklists

### 21.1 Deliverables
- Asset topology and critical basin registry
- Real-time monitoring dashboards and alerting
- Calibrated hydraulic/hydrologic models for priority areas
- Operational playbooks and after-action reporting templates
- CSO compliance reporting system
- Data governance and retention policies
- Model documentation and version control

### 21.2 Readiness checklist
- [ ] Topology QA completed for priority basins
- [ ] Sensor uptime and calibration process in place
- [ ] Alert thresholds validated via backtests
- [ ] Storm operations runbooks exercised
- [ ] Redundant communications and power verified
- [ ] Data retention and logging policies implemented
- [ ] Decision authority matrix documented
- [ ] Equity audit completed for coverage

---

## 22. References
### 22.1 Workspace source
- Item 7 in [`kali-task-research.md`](../kali-task-research.md:1)

### 22.2 External references (retrieved via Firecrawl MCP)
- FEMA Flood Risk Communication Toolkit for Community Officials: https://www.fema.gov/floodplain-management/manage-risk/communication-toolkit-community-officials
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.

### 22.3 Suggested further reading (not fetched)
- Urban hydrology and drainage modeling
- Nowcasting rainfall and flood forecasting approaches
- Asset management for water networks and failure modeling
- Digital twins for water infrastructure management
- Machine learning for flood prediction and early warning
- IoT-based water quality monitoring systems
- Federated learning for distributed water analytics
- EPA SWMM user manual and applications
- HEC-RAS 2D modeling guide
- CSO regulatory compliance frameworks