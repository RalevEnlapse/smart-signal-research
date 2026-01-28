# Public safety — Deep research

## Executive summary
A public safety “twin” supports **shared situational awareness** and reduces **coordination friction** across police/fire/EMS, emergency management/EOC, and critical infrastructure partners by integrating operational data (CAD incidents, unit status/AVL, closures, hazards, facility status) into a governed common picture with ICS-aligned workflows.

The twin’s value is not “predict everything.” It is:
- fewer missed handoffs across agencies
- faster triage with minimum viable context
- safer field operations through hazards/preplans
- better surge management and mutual aid coordination
- defensible, auditable decisions under stress

Because this domain is high-sensitivity and adversarial, the system must be built with non-negotiables:
- **decision-support-first** (no unsafe automation by default)
- **human-in-the-loop** for high-impact recommendations
- **strong access controls** (RBAC/ABAC, need-to-know segmentation)
- **immutable auditability** (including alert silencing / break-glass)
- **resilience engineering** (offline/degraded operation during disasters)

This document deepens item 11 in [`kali-task-research.md`](kali-task-research.md:1): *“Public safety: Use twins for situational awareness, incident simulation, and resource allocation during emergencies.”*

---

## 1) Scope, goals, and non-goals (operational realism)

### 1.1 Goals
1. **Common operational picture (COP)** that is current, trusted, and role-appropriate.
2. **ICS/EOC support**: facilitate command, control, and coordination (not replace it).
3. **Context enrichment**: surface hazards, preplans, access constraints, and verified updates.
4. **Surge operations**: manage major incidents and multi-incident periods with clear degraded modes.
5. **Post-incident learning**: after-action reviews, improvement tracking, and data quality remediation.

### 1.2 Non-goals (explicitly out of scope unless separately governed)
- **Predictive policing / individual risk scoring**.
- **Automated enforcement actions** based solely on model outputs.
- **Full replacement of CAD/RMS/radio** (the twin integrates; it does not “rip and replace”).
- **Single-source-of-truth without provenance** (the COP must show source, confidence, and last-updated).

---

## 2) Integration playbook: CAD / radio / AVL and minimum data contracts
Public safety integrations are heterogeneous: vendor-specific CAD feeds, varying event lifecycles, local codes, patchy AVL coverage, and late/manual updates. Design the twin as an **adapter + normalization** layer with explicit contracts.

### 2.1 Integration realities (plan for variability)
- **CAD vendors differ** in:
  - incident identifiers (global vs per-agency)
  - status lifecycles (e.g., created/queued/dispatched/enroute/onscene/closed)
  - update patterns (append-only vs overwrite)
  - how locations are represented (address vs GIS point vs intersection)
- **Jurisdiction differences**:
  - code tables and incident type semantics
  - mutual aid conventions
  - what fields are legally shareable cross-agency
- **AVL/radio variability**:
  - some agencies have per-second GPS; others have sparse or manual unit status
  - radio metadata may be inaccessible; voice is often out of scope

### 2.2 Minimum viable integration contract (MVIC)
Define the minimum fields required for a safe COP. If the feed cannot supply these reliably, the twin must show degraded-mode.

**Canonical incident schema (minimum):**
- `incident_id` (stable; with `source_system_id`)
- `source_agency_id`
- `created_at`, `updated_at` (source timestamps)
- `status` + `status_changed_at`
- `incident_type` (mapped to canonical taxonomy + original code)
- `location` (geometry) + `location_confidence` + `location_method` (geocoded, caller GPS, dispatcher pin)
- `priority` (mapped + original)
- `call_origin` (911, officer-initiated, citizen app, alarm company)
- `units_assigned[]` with unit IDs and timestamps
- `closure_code` / disposition (if available)

**Unit/AVL schema (minimum):**
- `unit_id`, `unit_type/capability`
- `unit_status` (standardized where possible)
- `last_gps` + `gps_confidence` + `last_gps_at`

**Idempotency + dedup rules:**
- Treat all events as upserts keyed by `(source_system_id, incident_id)`.
- Maintain an **event log** (append-only) and a **current incident state** (materialized view).
- Deduplicate “near-identical” incidents using a governed heuristic (time window + spatial proximity + incident type) but:
  - never auto-merge without audit trail
  - allow manual merge/unmerge by authorized supervisors

**Late updates and reconciliation:**
- Expect late location corrections and unit status backfills.
- Preserve prior values with timestamps and provenance.
- Always show `last_updated_at` and “data age” in the UI.

### 2.3 Deep integration path (optional)
- Bi-directional sync for:
  - attaching COP notes back to CAD (with strict controls)
  - unit recommendations as “draft” suggestions (not dispatching)
- Integration with RMS for historical patterns (strict retention/access)
- CAP/EDXL integrations for outbound alerts (with comms approval workflows)

### 2.4 Operational change management (do not underestimate)
- CAD workflows are safety-critical; any UI change needs:
  - role-based training
  - progressive rollout
  - simulation/tabletop practice
- Measure “time-to-proficiency” and incorporate into roadmap.

---

## 3) Responder-centered UX and cognitive load requirements

### 3.1 Design principle: minimum viable context (MVC)
For every incident view, provide a compact MVC panel first:
- **What / where / when** (type, location, last updated)
- **Confidence** (location confidence, source reliability)
- **Immediate hazards** (hazmat flags, violence risk flags where legally allowed, weather hazard)
- **Access constraints** (closures, gates, hydrants access)
- **Known occupants / vulnerable populations** only if lawful and operationally justified
- **Verified updates** vs unverified reports

### 3.2 Prioritization under stress (what to show first)
1. Life safety hazards and responder safety constraints
2. Location and confidence
3. Assignments + unit positions (with freshness)
4. Verified situation updates
5. Optional enrichments (preplans, historical notes) behind a deliberate action

### 3.3 Failure-safe UX for partial/stale data
- **Stale data banner**: “Last updated X minutes ago (source: CAD feed)”
- **Degraded-mode indicator**: “AVL unavailable — unit positions may be stale”
- **Confidence flags** for:
  - geocode quality
  - duplicate incident suspicion
  - citizen report unverified
- **Offline indicator** + “sync pending” counts
- Make “unknown” explicit; do not fill gaps with guessed values.

---

## 4) Governance, legality, and retention (privacy and public records)
Public safety records are simultaneously:
- high-sensitivity (tactical info, victim data, responder movement)
- frequently subject to public records / FOI regimes

Design governance explicitly.

### 4.1 Data classification (example)
- **Public**: aggregated stats, non-sensitive advisories
- **Internal**: operational status without PII
- **Sensitive**: incident narratives, victim info, detailed unit traces
- **Tactical/Restricted**: live tactical operations, critical infrastructure vulnerabilities

### 4.2 Retention tiers (starter policy; align to local law)
- **Incident records (structured fields)**: retain per records schedule; separate “operational copy” vs “official record of record.”
- **Incident narratives / attachments**: shortest lawful retention; encrypt at rest; strict access.
- **Responder/unit location traces (AVL)**:
  - hot retention short (e.g., 30–90 days) for operations and investigations
  - longer retention only with explicit lawful basis and oversight
- **Audit logs (access, exports, break-glass, silences)**:
  - retain longer (e.g., 1–2 years or per security policy)
  - immutable storage and integrity checks
- **Citizen submissions**:
  - retain long enough to support validation and abuse investigations
  - minimize PII fields; redact where possible

### 4.3 FOI/public-records handling vs tactical secrecy
Implement a workflow that can:
- mark records as subject to exemption (ongoing investigation, tactical details)
- produce redacted exports with audit trails
- preserve chain-of-custody for disclosed materials

### 4.4 Cross-agency lawful basis and contract patterns
For multi-agency sharing:
- define purpose limitation (“incident coordination”, “mutual aid”, “EOC situational awareness”)
- define data minimization (what fields share at each classification tier)
- define access controls, logging, and retention alignment
- use formal data sharing agreements + vendor DPAs where applicable

---

## 5) Citizen-input validation + misinformation handling
Citizen input is valuable but adversarial and error-prone.

### 5.1 Intake validation workflow (human review integrated)
1. **Triage** (automated): rate-limit, classify, deduplicate, geocode with confidence.
2. **Corroboration**: check against:
   - CAD events (nearby/time)
   - sensors (traffic anomalies, camera metadata where lawful)
   - other independent reports
3. **Human review** for:
   - high-impact claims
   - suspicious clusters
   - anything triggering response allocation
4. **Disposition**:
   - promote to CAD lead
   - hold for more info
   - reject as abuse (with logging)

### 5.2 Abuse controls that remain accessible
- progressive rate limits (per device/IP/account) with escalation paths
- CAPTCHA only after abuse signals; avoid blanket barriers
- allow “voice/assisted” channels for accessibility
- maintain an allowlist for trusted sources (utilities, hospitals) with governance

### 5.3 Rumor control and correction workflow
- designated comms owner and approval chain
- publish corrections with timestamps and references
- log who published what and why (auditability)

---

## 6) Model governance + HITL coupling (decision-support-first)
Models can help, but must be governed as advisory unless explicitly approved.

### 6.1 Allowed model classes (examples) and failure modes
- **ETA prediction**: fails under disasters/closures; bias if trained on normal traffic.
- **Resource coverage surfaces**: can mislead if AVL stale or incidents duplicated.
- **Anomaly detection** (false report surge, unusual patterns): can trigger alert fatigue.

### 6.2 Risk tiering for model outputs (example)
| Tier | Example output | Allowed action | Required approval | Notes |
|---|---|---|---|---|
| 0 | Informational risk flags | display only | none | no routing/dispatch changes |
| 1 | Advisory recommendations | suggest options | supervisor review | must show confidence + rationale |
| 2 | Operational decision support | propose resource staging | incident commander approval | audit sampling + rollback plan |
| 3 | High-impact automation | auto-dispatch/deny service | prohibited by default | requires separate program + legal | typically out of scope |

### 6.3 Validation gates and monitoring
Tie to [`research/model-validation-protocols-deep-research.md`](research/model-validation-protocols-deep-research.md:1):
- offline validation (historical + disaster-like scenarios)
- shadow mode before any advisory use
- drift monitoring by regime (normal vs surge/disaster)
- rollback triggers (error budget burn, drift threshold, operator contest rate)

---

## 7) Resilience engineering (offline/degraded operation)

### 7.1 What must work offline (minimum)
When cloud/network is down, the system should still support:
- cached basemaps + local incident list
- basic unit roster and last-known positions
- preplans/hazards cache for critical facilities
- offline forms/notes for situation updates
- queued outbound messages to CAD/EOC when connectivity returns

### 7.2 Sync semantics
- explicit “last synced” per source
- conflict resolution policy for offline edits (supervisor merge)
- idempotent replays on reconnect

### 7.3 Communications redundancy and dependency mapping
Maintain a critical dependency map:
- telecom links
- power/UPS
- cloud regions
- identity provider
- CAD vendor endpoints

Redundancy patterns:
- secondary connectivity (LTE + terrestrial)
- local edge caches
- read-only degraded COP mode

### 7.4 Fail-safe fallback procedures
- define manual dispatch and radio procedures when COP unavailable
- define “minimum safe operation” checklists
- rehearse offline mode in drills

---

## 8) Operational runbooks (minimum set)

### 8.1 Major incident surge / EOC activation and access grants
- activate ICS/EOC roles
- enable break-glass access with incident ID
- increase audit sampling and comms cadence

### 8.2 CAD feed degradation / duplicate incident storm
- detect: duplicate rate, late update spikes, location confidence drop
- mitigate: degrade UI, suppress noisy alerts, coordinate with CAD admin
- reconcile: controlled merge/unmerge workflows

### 8.3 Misinformation campaign / false report surge
- raise triage thresholds
- enable enhanced corroboration
- coordinate with comms rumor-control workflow

### 8.4 Platform outage during incident (offline mode + manual fallback)
- switch to offline COP
- manual dispatch fallback
- post-incident AAR and corrective actions

### 8.5 Contested decision / audit inquiry
- retrieve evidence packet (sources, timestamps, user actions)
- produce redacted export under FOI workflow

---

## 9) Key metrics
- COP freshness and completeness by source (`dataset_id`, `tenant_id`).
- Location confidence coverage (% incidents with high-confidence location).
- Duplicate incident rate and merge/unmerge activity.
- Time-to-verify citizen reports; false escalation proxy metrics.
- Offline sync success rate + conflict rate.
- Model drift indicators and rollback frequency (if models used).

---

## 10) Implementation roadmap

### 0–3 months: inventory + canonical schema + read-only COP + retention draft
- Integration inventory (CAD/radio/AVL) and MVIC contract.
- Canonical incident/unit schema + mapping tables.
- Read-only COP MVP with last-updated/confidence/degraded-mode.
- Draft retention and FOI workflows with legal/privacy.

### 3–12 months: resilience/offline + validation/HITL + federation hardening
- Offline/degraded mode + edge caches.
- Citizen intake validation workflow + abuse controls.
- Model governance gates + drift monitoring.
- Cross-agency federation hardening (RBAC/ABAC, audit, break-glass).

### 12–24 months: interoperability maturity + rehearsals + continuous audit readiness
- Mature interoperability (standardized status codes, deeper CAD workflow integration).
- Regular rehearsals and AAR closure discipline.
- Continuous audit readiness for access, exports, and public disclosures.

---

## References (high-signal sources; starting points)

### NIMS/ICS-aligned incident management
- FEMA — NIMS Incident Complexity Guide: [`www.fema.gov/sites/default/files/documents/nims-incident-complexity-guide.pdf`](https://www.fema.gov/sites/default/files/documents/nims-incident-complexity-guide.pdf) — Overview of incident complexity concepts and ICS alignment that inform role design, span-of-control, and escalation patterns.

### CAD/AVL integration realities
- APCO — Public Safety Communications Common Status Codes for Data Exchange (APCO ANS 1.116.2-2020): [`www.apcointl.org/~documents/standard/11162-2020-common-status-codes-for-data-exchange/?layout=file`](https://www.apcointl.org/~documents/standard/11162-2020-common-status-codes-for-data-exchange/?layout=file) — Standardized unit status codes to support interoperability across disparate CAD systems.

### Public records / records retention guidance
- U.S. National Archives (NARA) — Records management regulations and guidance: [`www.archives.gov/records-mgmt/policy`](https://www.archives.gov/records-mgmt/policy) — High-level federal records management guidance useful for shaping retention policies, disposition, and audit readiness.

### Resilient/offline operations and continuity
- FCC — Emergency Planning: Public Safety Answering Points: [`www.fcc.gov/research-reports/guides/emergency-planning-public-safety-answering-points`](https://www.fcc.gov/research-reports/guides/emergency-planning-public-safety-answering-points) — Guidance for PSAP continuity-of-operations planning and maintaining operability during disruptions.
