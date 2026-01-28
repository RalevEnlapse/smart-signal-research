# Noise pollution monitoring — Deep research

## Executive summary
Noise is a highly visible “quality of life” domain where cities often have regulatory authority, residents have direct lived experience, and interventions are concrete (routing, construction permits, curfews, barriers, pavement, zoning). A City Digital Twin can add value by turning noisy-signal fragments (complaints, spot checks) into **decision-grade, privacy-safe evidence layers**:

- **Planning-grade noise exposure layers** (block / corridor / neighborhood) to target mitigations and evaluate impact.
- **Operations-grade event detection** (short-lived spikes) to prioritize response and verify resolution.
- **Governance-grade auditability** so noise sensing does not become speech surveillance or selective enforcement.

This domain has one hard constraint that must be made explicit: **microphones are surveillance-adjacent**. A realistic program must start with a privacy-first sensing architecture (no raw audio retention; on-device feature extraction; auditable configurations), and must be explicit about what is and is not appropriate for enforcement.

This document deepens item 32 in [`kali-task-research.md`](kali-task-research.md:1): *"Noise pollution monitoring: Deploy acoustic sensors to map noise levels, identify hotspots, and enforce regulations to improve urban livability."*

---

## 1) Scope, outcomes, and non-goals (set expectations early)

### 1.1 In-scope outcomes
1. **Citywide baseline + trends**
   - A-weighted sound pressure level summaries by time-of-day and day-of-week.
   - Heatmaps at policy-safe resolution (see privacy section).
   - “Before/after” evaluation for interventions.

2. **Hotspot management loop**
   - Identify persistent hotspots.
   - Prioritize interventions and operational responses.
   - Re-measure and verify effect.

3. **Event detection for response (not automatic guilt)**
   - Detect “unusual” spikes by sensor-relative baselines.
   - Create tickets for inspection / outreach / targeted mitigation.

4. **Planning integration**
   - Noise layers in zoning, permitting, environmental impact checks, nighttime freight routing, event permitting.

### 1.2 Explicit non-goals (to protect legitimacy)
- **No speech surveillance**: the system is not a “microphone network for conversations”.
- **No continuous identification of people**: no voiceprints, speaker ID, or face/plate linkage.
- **No automated citation issuance** from unattended sensors without defined evidentiary processes.
- **No neighborhood targeting** via selective placement without equity governance.

### 1.3 Two product tiers (separate planning from enforcement)
Use two labeled product classes to prevent misuse:

- **Planning / Public Health Grade (P-Grade)**
  - Aggregated exposure metrics (e.g., hourly Leq) at coarse spatiotemporal resolution.
  - Suitable for planning, prioritization, and evaluation.

- **Regulatory / Enforcement Support Grade (E-Grade)**
  - Only where local law, calibrated equipment, chain-of-custody, and due process allow.
  - Typically still *supporting evidence* requiring human verification and context.
  - Not default; must be a separately approved program.

---

## 2) Stakeholders and operating model

### 2.1 Stakeholders
- **Environmental health / noise control office**: policy owner, thresholds, enforcement posture.
- **Public health**: exposure/health correlation and equity impacts.
- **Urban planning & permitting**: impact assessment, zoning constraints, mitigation requirements.
- **Transportation**: routing, calming, pavement and barrier programs.
- **Construction permitting**: permitted hours, variance handling.
- **Legal & privacy office**: privacy impact assessment, public transparency, evidentiary review.
- **IT / security**: device hardening, key management, logging.
- **Community groups**: legitimacy, placement review, complaints channel design.

### 2.2 Decision rights (minimum)
- **Policy owner** approves:
  - Metric definitions and thresholds.
  - P-Grade vs E-Grade deployment.
  - Public map resolution and suppression rules.
- **Privacy officer** approves:
  - Sensor configuration profiles.
  - Any exception that increases capture risk.
- **Security owner** approves:
  - Device identity, secure boot, signing, logging.

---

## 3) Measurement fundamentals (what you actually measure)

### 3.1 Core acoustic metrics (planning-grade)
Define a minimum set that is stable, comparable, and explainable:
- **LAeq,T**: A-weighted equivalent continuous sound level over a window T (e.g., 1 min, 15 min, 1 hour).
- **LAmax**: maximum A-weighted level within a window.
- **Percentiles (LA10 / LA50 / LA90)**: characterize variability and background.

Operational defaults (tune to your ordinance):
- Event detection windows: 1–5 minutes.
- Exposure reporting windows: 1 hour and day/night splits.

### 3.2 Time-weighting and frequency content (be conservative)
- Support **Fast/Slow time-weighting** for event detection; keep rules clear and auditable.
- Frequency (octave/third-octave or band summaries) can improve source characterization *without retaining intelligible audio*, but still raises privacy risk. Treat as a governed feature set.

### 3.3 Calibration and accuracy targets
- If you advertise “±2 dB”, you must back it with:
  - Defined sensor class/specification (e.g., IEC 61672-class sound level meter classes),
  - Field calibration procedures,
  - Drift monitoring,
  - Maintenance SLAs.

For most cities, the pragmatic approach is:
- Use higher-grade devices for a subset (“reference nodes”) and lower-cost devices for dense coverage (“coverage nodes”), with cross-calibration.

---

## 4) Privacy-first acoustic sensing architecture (non-negotiable)

### 4.1 Data minimization principle
Store and transmit **noise metrics, not audio**.

Default posture:
- **No raw audio retention** (0 seconds persisted).
- **No raw audio transmission** off device.
- Compute features locally and discard raw samples immediately.

### 4.2 On-device processing pipeline (reference)
1. Microphone samples → short buffer
2. Compute approved features (e.g., LAeq, LAmax, band energy)
3. Run **speech presence detection** (privacy filter)
4. Output:
   - Metrics
   - Quality flags
   - Configuration/version metadata
5. Immediately discard audio buffer

### 4.3 Speech/privacy handling modes
Establish explicit, auditable modes:

- **Mode A (preferred): speech-safe by design**
  - Only compute non-invertible metrics (LAeq/LAmax/percentiles) + coarse band energy.
  - Never store audio.

- **Mode B (conditional): speech-filtered features**
  - If speech presence is detected, either:
    - drop the record entirely, or
    - publish only very coarse aggregated metrics with a privacy flag.
  - This mode is for public legitimacy; it does not create enforcement-grade evidence.

- **Mode C (exception, E-Grade only): short “evidence snippet” retention**
  - Not recommended unless ordinance and courts require it and privacy office approves.
  - Must be time-bounded, encrypted, access-controlled, and logged.
  - Must include explicit rules for when it is captured and when it is deleted.

If you cannot credibly operate Mode C with strict governance, do not claim “automated enforcement”.

### 4.4 Public transparency commitments (publishable)
- Plain-language description of what sensors do and do not collect.
- Sensor placement map (at safe resolution if necessary).
- Data retention schedule.
- Audit summary metrics (how often exceptions were used, how often privacy filter triggered).

---

## 5) Enforcement and evidentiary realism

### 5.1 Enforcement posture options
Choose one and state it publicly:

1. **Planning + response only (default)**
   - Measurements guide mitigation and inspections.
   - Violations are established via traditional mechanisms (officer measurement, site inspection).

2. **Enforcement support (conditional)**
   - Sensor events create “leads” that trigger inspection.
   - Citations still require human verification and ordinance-compliant measurement.

3. **Automated enforcement (rare; high burden)**
   - Only if local law allows unattended measurement for citations.
   - Requires calibrated instruments, documented procedures, and robust chain-of-custody.

### 5.2 Define “violation” precisely
Your ordinance language must map to measurable quantities:
- Threshold (dB(A))
- Time window (e.g., exceedance sustained for N seconds/minutes)
- Context conditions (time-of-day, zoning, permitted activity)
- Exemptions/variances (construction permits, emergencies, permitted events)

### 5.3 Evidence packet (for any enforcement-support workflow)
For each event, store a structured evidence packet (even if no audio exists):
- Sensor ID + firmware/config hash
- Location (governed precision)
- Timestamp with trusted time source
- Metrics (LAeq/LAmax/percentiles) + window definition
- Calibration status (last calibration date, drift flags)
- Environmental context (wind/rain if available)
- Nearby permits/events context (construction variance, special event permit)
- Uncertainty estimate + flags (occlusion, clipping, saturation)
- Chain-of-custody ledger references (see below)

### 5.4 Chain-of-custody for sensor data
Even without audio, integrity matters for disputes.
Minimum controls:
- Device identity + secure boot (prevent rogue firmware)
- Signed measurements (device private key) and server-side verification
- Append-only logs for:
  - ingestion
  - transformation
  - access
  - export
- Hashing of evidence packets and time-stamping at creation

---

## 6) Source attribution: what is realistic and how to communicate uncertainty

### 6.1 Source attribution use-cases
- **Planning insight**: distinguish broad classes (traffic vs construction vs nightlife vs industrial) for mitigation selection.
- **Operational routing**: prioritize inspection type (e.g., construction vs vehicle).

### 6.2 What not to promise
- Do not promise precise single-source identification in dense urban soundscapes.
- Do not imply you can reliably attribute a noise event to a specific person/vehicle without additional evidence.

### 6.3 Uncertainty and confidence reporting
If you use classification/localization:
- Always output:
  - top-k class probabilities
  - “unknown” option
  - confidence calibration checks
- Gate decisions:
  - low-confidence → informational only
  - high-confidence + corroborating context → inspection lead

### 6.4 Contextual joins (governed)
Attribution improves by joining against contextual data:
- traffic volumes/speeds
- construction permits and permitted hours
- event permits
- land-use and zoning

Apply **join governance**:
- Allowed joins for P-Grade planning layers.
- Prohibited joins that enable person-level targeting.
- Approval workflow + audit logs for any new join.

---

## 7) Sensor placement and representativeness (equity by design)

### 7.1 Placement strategies
Use a hybrid strategy; document it explicitly:
- **Coverage nodes**: represent citywide exposure (sampling frame).
- **Hotspot nodes**: concentrated where complaints/interventions are expected.
- **Reference nodes**: high-quality calibration anchors.

### 7.2 Representativeness risks
- Over-indexing sensors in complaint-heavy neighborhoods can create biased “evidence”.
- Maintenance differences can produce apparent inequities.
- Hotspot-only deployment can be mistaken for citywide exposure.

### 7.3 Governance controls
- Pre-register placement rationale and revision rules.
- Equity review board sign-off for changes.
- Publish a placement and maintenance equity report:
  - sensors per capita / per km road
  - downtime by neighborhood
  - calibration compliance by neighborhood

---

## 8) Reference architecture (components + data flows)

### 8.1 Components
1. **Sensor + edge compute**
   - Microphone + on-device DSP
   - Secure element / TPM-like key storage
   - Optional weather sensor

2. **Connectivity**
   - Cellular/Wi-Fi depending on bandwidth needs
   - LoRaWAN only if payload is minimal (metrics only)

3. **Ingestion + time-series platform**
   - Streaming ingestion (MQTT/HTTP)
   - Time-series store + geospatial indexing
   - Data quality checks + drift monitoring

4. **Analytics + mapping**
   - Hotspot detection
   - Trend and intervention evaluation
   - Optional source-class models

5. **Workflow integration**
   - Ticketing/dispatch
   - Permitting systems (construction/event)
   - Planning GIS

### 8.2 Data flows
- Edge metrics → ingest → validation → storage
- Event detection → evidence packet → ticket → response → closure verification
- Aggregated exposure layers → planning GIS → policy decisions

---

## 9) Data requirements and quality SLAs

### 9.1 Minimum datasets
- Noise metrics (LAeq/LAmax/percentiles) with window definitions
- Sensor metadata (location class, install height, orientation)
- Calibration/maintenance records
- Optional environmental context (wind/rain)

### 9.2 Data quality checks (examples)
- Missingness spikes
- Sensor saturation/clipping detection
- Drift vs reference node
- Time sync health

### 9.3 Example SLIs/SLOs (tune to your tier)
- **Coverage-node uptime**: ≥ 98.5% monthly
- **Reference-node uptime**: ≥ 99.5% monthly
- **Freshness**: 95% of records within 5 minutes
- **Calibration compliance**: 100% reference nodes, ≥ 95% coverage nodes
- **Drift alarms**: investigated within 5 business days

---

## 10) Planning integration and intervention evaluation

### 10.1 How noise layers feed planning
- **Permitting**: require mitigation plans for projects in high-exposure zones.
- **Zoning**: inform buffers, mixed-use constraints, or façade standards.
- **Transport**: evaluate routing changes, calming, pavement resurfacing.
- **Nighttime economy**: manage late-night hotspots without punitive overreach.

### 10.2 Evaluation designs (credible “did it work?”)
- Pre/post with matched control corridors
- Difference-in-differences for interventions rolled out in stages
- Seasonality and event controls

Deliver an “intervention report” template with:
- baseline window
- post window
- confounders (weather, road works, events)
- effect size in dB and exposure distribution shift
- equity impact (who benefits)

---

## 11) Threat model and controls (beyond generic)

### 11.1 Key abuse cases
- Speech capture accusations (loss of legitimacy)
- Selective enforcement claims
- Data tampering (device firmware or in-flight)
- Sensor relocation or obstruction

### 11.2 Controls
- Privacy architecture controls in section 4
- Device hardening: secure boot, signed firmware, attestation
- Measurement signing and append-only logging
- Physical inspection schedule + tamper-evident seals

---

## 12) Governance, compliance, and labor constraints

### 12.1 Governance artifacts (must exist)
- Privacy impact assessment + public summary
- Approved sensor configuration profiles (Mode A/B/C)
- Join governance policy
- Evidence handling SOP (if any enforcement support)
- Public communications plan and complaint handling

### 12.2 Labor realism
- If you create alerts, you must staff review.
- Define queue SLOs for inspection leads (e.g., triage within 1 business day) and a safe degradation mode when staffing is constrained (e.g., planning-only rollups).

---

## 13) Implementation plan (phases)

### Phase 0 — Legitimacy and design constraints
- Draft privacy-first architecture and publish commitments.
- Choose metrics, windows, and resolution rules.
- Establish placement governance and equity review.

### Phase 1 — Pilot (learning, not enforcement)
- Deploy coverage + reference nodes.
- Validate drift, calibration workflows, and map usability.
- Run community review sessions.

### Phase 2 — Citywide planning layers
- Expand coverage nodes.
- Ship planning GIS integration and intervention reporting.

### Phase 3 — Operational event detection (response loop)
- Integrate ticketing/dispatch.
- Add context joins (permits/events/traffic) under join governance.

### Phase 4 — Conditional enforcement support (only if approved)
- Legal + privacy review.
- Evidence packet and chain-of-custody operationalization.
- Human verification steps and audit sampling.

---

## 14) KPIs (tie to outcomes, not vanity metrics)
- Exposure reduction in priority corridors (dB and distribution shift)
- Time-to-triage for verified hotspots
- Intervention success rate (measured post)
- Equity: exposure reduction by neighborhood income/race proxies (policy-approved)
- Trust: complaint resolution satisfaction; privacy complaints rate
- Operational integrity: calibration compliance; drift incidents; tamper incidents

---

## 15) References and starting points

### Workspace references
- Item 32 in [`kali-task-research.md`](kali-task-research.md:1)
- Backlog: [`analysis/noise-pollution-monitoring-deep-research.analysis.md`](analysis/noise-pollution-monitoring-deep-research.analysis.md:1)

### External pointers (starting points)
- OGC Urban Digital Twin Interoperability (environmental layers including noise): [`docs.ogc.org/per/24-067r1.pdf`](https://docs.ogc.org/per/24-067r1.pdf)
- Measurement standards context (sound level meters, calibration concepts): IEC 61672 family (obtain via your standards access) and vendor summaries explaining Class 1/2 and time-weighting (treat as secondary).
- Privacy-preserving acoustic sensing research: work on voice activity / speech detection for privacy filtering and privacy-preserving audio classification (treat as research inputs; implement only with formal privacy review).
- Digital evidence integrity and chain-of-custody surveys (for evidence workflows): see chain-of-custody “SoK” style overviews and time-stamping approaches (use to shape SOPs and audit controls).
