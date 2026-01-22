# Public transport — Deep research

## Executive summary
A public transport “twin” is a decision-support system that models service supply (routes, stops, vehicles, schedules, headways), passenger demand (boarding/alighting, crowding), and disruptions (incidents, weather, operator constraints) to improve reliability, capacity utilization, and passenger experience.

The critical technical outputs are: (1) real-time service state (vehicle locations, delays, headway adherence), (2) predictive ETAs and crowding forecasts, (3) disruption impact simulation, and (4) optimization recommendations (dispatching, holding, short-turns, priority measures). Achieving these requires high-quality operational data, robust inference under missing/late telemetry, and strong governance to ensure recommendations are safe, explainable, and aligned with labor and safety constraints.

This document deepens item 4 in [`kali-task-research.md`](../kali-task-research.md:1): *“Public transport: Simulate routes, headways, capacity, and disruptions to improve reliability and passenger experience.”*

---

## 1. Background and context
Public transport performance hinges on variability:
- Demand variability (peaks, events)
- Traffic variability (mixed-traffic buses)
- Operations variability (dwell time, dispatching, operator availability)

Common issues:
- Bus bunching and headway instability
- Overcrowding, missed connections
- Unreliable ETAs due to traffic and dwell uncertainty
- Disruptions (road closures, vehicle breakdowns)

A twin approach supports:
- Real-time control strategies (holding, stop-skipping, short-turning)
- Tactical planning (schedule adjustments, priority measures)
- Strategic planning (route redesign, fleet sizing)

---

## 2. Stakeholders
- **Transit agency operations control center (OCC)**: real-time management
- **Schedulers/planners**: timetable and resource planning
- **Drivers/operators and unions**: work rules, safety constraints
- **Maintenance**: vehicle availability and breakdown response
- **Passengers/public**: reliability, crowding, accessibility
- **City DOT/TMC**: signal priority, lane management, incident coordination
- **Accessibility advocates**: equitable service and station/stop accessibility
- **IT/data**: telemetry, integration, security

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity of real-time vehicle location and schedule adherence data
- Integrity of control commands (dispatching instructions)
- Availability of passenger information systems (PIS) and OCC tools
- Privacy of passenger data (if APC, fare, or app data is used)

### 3.2 Abuse/failure cases
- **Telemetry spoofing** causing false ETAs or control actions
- **Denial of service** against real-time feeds leading to blind operations
- **Unauthorized service changes** (route/stop updates) causing widespread confusion
- **Re-identification** from fine-grained passenger traces

### 3.3 Controls
- Signed/secured telemetry channels where feasible
- Role-based control and approvals for operational interventions
- Graceful degradation: fall back to scheduled service when telemetry fails
- Privacy-preserving aggregation for passenger analytics

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Service configuration repository**
   - Routes, trips, stops, calendars, operator constraints

2. **Real-time vehicle telemetry ingestion**
   - AVL/GPS pings, door open/close events, odometer, vehicle health

3. **Passenger demand and load estimation**
   - Automated passenger counts (APC), fare validation aggregates, platform counts

4. **Inference layer**
   - Map matching to route shapes
   - ETA prediction, dwell time modeling
   - Headway adherence and bunching detection

5. **Simulation layer**
   - Stochastic simulation of trips under traffic and demand variability
   - Disruption simulation (detours, station closures)

6. **Control/optimization layer**
   - Holding strategies, dispatch control, short-turn recommendations
   - Priority requests coordination (signal priority, bus lanes)

7. **Serving layer**
   - OCC dashboards, passenger info APIs, alerts

8. **Governance/observability**
   - Data quality, model drift, audit logs

### 4.2 Data flows
- Static schedule/config → baseline model
- AVL + traffic → real-time state and ETA
- APC/fare aggregates → load estimation → crowding forecasts
- Events (incidents, closures) → simulation constraints → service impact
- Optimizer → recommended interventions → OCC approval → execution

---

## 5. Methods / algorithms / standards

### 5.1 Real-time state estimation
- Map matching of vehicle locations to route geometry
- Trip inference (which trip/block the vehicle is serving)
- Headway computation and stability metrics
- Time-series autoencoders and LSTM/GRU forecasters for deviation detection

### 5.2 ETA prediction
- Segment travel time estimation with time-of-day and incident conditioning
- Dwell time prediction based on boarding/alighting and crowding
- Uncertainty estimates (prediction intervals)
- Machine learning models for travel time prediction with incident conditioning

### 5.3 Passenger load and crowding
- Load estimation from APC when available
- Inference from fare aggregates and historical patterns when APC is sparse
- Capacity constraints: seated/standing, accessibility spaces
- Federated learning with differential privacy for privacy-preserving load estimation

### 5.4 Disruption modeling
- Detour travel time impacts, stop removals
- Short-turn feasibility with turnaround constraints
- Fleet and operator constraints (duty time, layover)
- Stochastic simulation for corridors/routes under traffic and demand variability

### 5.5 Control strategies (operational)
- Holding to maintain headways
- Limited stop-skipping with passenger impact constraints
- Short-turning to recover service frequency
- Priority measures: transit signal priority requests, lane enforcement

### 5.6 Standards and protocols
- GTFS (General Transit Feed Specification) for static schedule data
- GTFS Realtime for vehicle positions, trip updates, and alerts
- MQTT/AMQP for real-time telemetry with QoS levels
- RESTful APIs for platform integration
- NGSI-LD context models for interoperability

---

## 6. Data requirements

### 6.1 Minimum datasets
- Route/stop geometry, schedules, blocks (service plan)
- Real-time vehicle positions with timestamps
- Operational events: trip start/end, doors, layover
- Disruption feed: incidents, roadworks, detours

### 6.2 High-value datasets
- APC load by stop/trip
- Real-time traffic speeds on bus corridors
- Station crowding sensors for rail/metro
- Accessibility constraints and elevator/escalator status

### 6.3 Data quality requirements
- Time sync and late telemetry handling
- Stop-level accuracy for ETAs
- Consistent vehicle and trip identifiers

---

## 7. Implementation plan (phases)

### Phase 0 — Baseline and governance
- Define reliability metrics (on-time, headway adherence) and thresholds
- Define privacy approach for passenger data

### Phase 1 — Real-time state and ETA foundation
- Ingest AVL, map match, compute real-time delay/headway
- Provide stable passenger-facing ETA API

### Phase 2 — Crowding and disruption support
- Implement load estimation and crowding forecasts
- Integrate disruption feed and detour logic

### Phase 3 — Simulation and decision support
- Stochastic simulation for corridors/routes
- Post-incident what-if analysis and schedule adjustments

### Phase 4 — Operational optimization
- Provide intervention recommendations with explainability
- Implement approval workflows, rollback, and post-action evaluation

---

## 8. Testing and validation
- Data pipeline tests: telemetry parsing, late/out-of-order handling
- ETA backtesting: compare predictions to actual arrival times by corridor
- Load estimation accuracy vs APC ground truth
- Simulation calibration: reproduce headway distributions and bunching frequency
- Human factors validation: ensure recommendations are usable and safe

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- ETA error (MAE) and calibration (coverage of prediction intervals)
- Headway adherence (% time within target band)
- Telemetry freshness/coverage by route
- Disruption handling latency
- End-to-end latency (ms)
- Synchronicity error between physical and virtual states
- Update rate (Hz)
- Service availability (%)
- Mean time to detect/recover (MTTD/MTTR)

### 9.2 Example SLOs
- 99% of vehicle pings processed within 10 seconds
- Passenger ETA API p95 latency < 300 ms
- ETA MAE < threshold for top corridors before enabling controls
- System uptime ≥ 99.9%
- Update rate ≥ 1 Hz for critical corridors
- End-to-end latency < 5 seconds for real-time state updates

---

## 10. Governance, compliance, and labor constraints
- Auditability of interventions (who recommended/approved/executed)
- Ensure strategies comply with safety rules and operator agreements
- Public transparency where changes affect service patterns
- Data retention policies for passenger-related datasets

---

## 11. Risks and mitigations
- **Bunching control harms passengers** → constrain stop-skipping; optimize for total passenger delay
- **Telemetry gaps** → fallback to schedule-based estimates; detect sensor failures
- **Model drift during major changes** → recalibrate after network/service changes
- **Equity impacts** → measure crowding and reliability by neighborhood

---

## 12. Costs and FinOps
- Compute for real-time inference and simulation
- Data costs (traffic feeds, passenger counting infrastructure)
- Operational training and change management

Unit costs to track:
- Cost per route supported with real-time + simulation
- Cost per million telemetry messages processed

---

## 13. KPIs
- On-time performance / headway adherence improvement
- Reduction in bunching frequency and severity
- Passenger wait time reduction and reliability increase
- Crowding reduction (percent of trips over capacity)
- Disruption recovery time (service restored faster)

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Real-time transit state service (vehicle positions, delays)
 triggering, MTTR
- ETA API with uncertainty
- Crowding/load estimation service
- Disruption simulation and decision-support dashboard
- Governance/runbooks for interventions

### 14.2 Readiness checklist
- [ ] Route/trip identity mapping is stable
- [ ] ETA backtests meet performance thresholds
- [ ] Disruption feed integrated and tested
- [ ] Approval workflow for interventions is in place

---

## 15. References

### 15.1 Workspace source
- Item 4 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- GTFS Overview (GTFS Schedule + GTFS Realtime): https://gtfs.org/documentation/overview/
- GTFS Realtime Reference: https://gtfs.org/documentation/realtime/reference/
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Flexible hyper-distributed IoT–edge–cloud platform for real-time digital twin applications." Future Internet, 16(11), 431.

### 15.3 Suggested further reading (not fetched)
- Headway-based control and bus bunching literature
- Real-time ETA prediction and uncertainty estimation
- Transit operations research for holding/short-turn strategies
- Digital Twin Implementation Readiness Level (DT-IRL) framework
- Zero-trust architecture for transit systems
- Federated learning for privacy-preserving transit analytics