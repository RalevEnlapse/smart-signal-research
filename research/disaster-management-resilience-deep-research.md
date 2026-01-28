# Disaster management & resilience — Deep research

## Executive summary

In a City Digital Twin portfolio, disaster management is where “decision support” becomes **real operations**. The platform’s job is not to draw nicer maps; it is to reduce time-to-verify, time-to-decide, and time-to-coordinate across agencies—while preventing the most dangerous failure modes: **false alarms**, **misinformation harm**, **unsafe automation**, and **leakage of sensitive locations**.

This document operationalizes disaster management around how incidents are actually run under the Incident Command System (ICS): incident objectives, operational periods, situation status, resource requests/assignments, and public communications through a Joint Information System/PIO function. It provides:

- An **ICS-aligned operating model** and minimal canonical incident data model.
- **Inter-agency interoperability** patterns and shared operating picture governance (“who owns truth”).
- **Alert governance under uncertainty** (staged alerts, verification workflows, confidence scoring, two-person rule).
- **Rumor control / misinformation operations** integrated with auditability and cross-channel messaging.
- **Privacy and sensitive location protections** (precision tiering, retention/legal hold, vulnerable populations).
- **Resilience engineering** for the platform itself (offline-first field ops, degraded mode, dependency mapping).
- **Recovery as a longitudinal program** tied to asset management, procurement/permitting, finance/grants, and equity outcomes.

## 0. Scope and positioning

### 0.1 What this capability is

A city-governed incident management capability that:

- creates and manages incident records (multi-jurisdiction where required)
- maintains a shared operating picture (situation + resources + impacts)
- orchestrates alerting and public information with safety governance
- supports field operations (offline-first data capture and assignments)
- tracks recovery work as a program integrated with city operations

### 0.2 What it is not (safety boundaries)

- Not an autonomous “response bot.” No automated dispatch or evacuation orders without explicit human authorization.
- Not a social media sentiment engine. Social content can be a signal but not a “truth source” without verification.
- Not a public map of sensitive facilities or vulnerable populations.

## 1. Operational model anchored in ICS

### 1.1 ICS workflow mapping (platform capabilities → incident practice)

| ICS/EOC practice | Platform capability |
|---|---|
| Incident creation + size-up | incident record + initial situation report (SitRep) + affected area |
| Incident objectives | objectives list with owners and success criteria |
| Operational periods (“Planning P”) | operational period objects with plans, assignments, and briefings |
| Resource requests/assignments | resource ordering, status tracking, mutual aid, demob |
| Situation status | SitRep cadence, validated observations, confidence scoring |
| Public information (PIO/JIC) | message drafting, approvals, rumor control, channel publishing log |
| Damage assessment | field forms, photos, geotagging (precision-tiered), validation |
| Recovery | work packages linked to assets, contracts, finance, equity tracking |

### 1.2 Minimal canonical incident data model (entities + required fields)

Keep the canonical model small; map agency-specific systems into it.

**Entity: `Incident`**

Required fields:

- `incident_id` (UUID)
- `name`, `incident_type` (taxonomy)
- `status` (active/contained/closed)
- `jurisdictions[]` (city, county, mutual aid)
- `start_time_utc`, `last_updated_utc`
- `severity_level` (policy-defined)
- `ic_uc` (Incident Command / Unified Command)
- `eoc_activation_level` (if applicable)
- `affected_area` (geometry + precision tier)
- `current_objectives[]` (see below)
- `operational_period_current` (reference)
- `sitreps[]` (time series)
- `public_message_state` (active alerts, last public statement)

**Entity: `Objective`**

- `objective_id`, `incident_id`
- `statement` (plain language)
- `owner_role` (ICS role)
- `success_criteria`
- `status` (planned/in-progress/done)

**Entity: `OperationalPeriod`**

- `op_period_id`, `incident_id`
- `start_time`, `end_time`
- `iap_ref` (Incident Action Plan link)
- `assignments[]`

**Entity: `Resource`**

- `resource_id`
- `kind`, `type` (per local catalog)
- `owner_agency`
- `status` (available/assigned/enroute/onscene/outofservice)
- `last_known_location` (precision-tiered)

**Entity: `ResourceRequest`**

- `request_id`, `incident_id`
- `requested_by_role`
- `resource_kind/type`, `quantity`
- `priority`
- `needed_by_time`
- `status` (open/filled/partial/cancelled)

**Entity: `Observation`** (validated signal)

- `observation_id`, `incident_id`
- `source` (sensor, CAD, field report, partner agency)
- `time_observed`, `time_received`
- `location` (precision-tiered)
- `claim` (what is asserted)
- `confidence` (0–1) + `confidence_rationale`
- `verification_state` (unverified/triangulated/confirmed/false)

**Entity: `PublicMessage`**

- `message_id`, `incident_id`
- `stage` (watch/advisory/warning/evacuation)
- `geo_scope`, `audience`
- `issued_by`, `approved_by`
- `time_issued`, `expires_at`
- `channels[]` + per-channel delivery status
- `corrections[]` (retractions, edits)

## 2. Interoperability & shared operating picture governance

### 2.1 Integration patterns (multi-agency, multi-jurisdiction)

Common sources:

- CAD/911 and dispatch logs
- CAP alerting systems
- public works/road closures and work orders
- utility outage systems
- GIS layers (critical infrastructure, shelters)
- hospitals/shelter occupancy (often constrained)

Patterns:

1. **System-of-record + federation**
   - Each agency keeps its legal/operational system-of-record.
   - The twin platform ingests into the canonical incident model and serves a governed shared view.

2. **Event-driven integration**
   - normalize key state transitions (resource status, closure posted, shelter occupancy update) as events.

3. **Role/jurisdiction-tiered access**
   - mutual aid users get time-boxed access tied to incident activation.

### 2.2 Shared operating picture governance (“who owns truth”)

Define precedence and reconciliation rules:

- **Source precedence by field** (example):
  - `road_closure_status`: public works is authoritative
  - `fire_perimeter`: fire agency is authoritative
  - `utility_outage_extent`: utility is authoritative
  - `casualty_counts`: public health is authoritative

Conflict handling workflow:

- detect conflicts (two authoritative sources disagree)
- record both values + provenance
- assign to “truth steward” (role) for reconciliation
- publish resolved value with audit trail

All overrides must be logged with:

- who changed, why, when
- what evidence supported the change

## 3. Alert governance under uncertainty (false-alarm prevention)

### 3.1 Staged alert model

Use explicit stages:

- **Watch**: credible possibility; prepare.
- **Advisory**: conditions present; take cautionary actions.
- **Warning**: imminent/occurring; take protective action.
- **Evacuation**: move now (highest impact).

Each stage must have:

- required verification level
- authorized roles
- default message templates
- minimum/maximum geographic scope rules

### 3.2 Verification workflow and two-person rule

For high-impact alerts (Warning/Evacuation):

- two-person rule: initiator + approver
- evidence checklist (minimum):
  - 2 independent sources OR 1 authoritative sensor/system + human confirmation
  - location scope validated
  - “what would change our assessment” documented

### 3.3 Confidence scoring

Maintain a `confidence` field for key claims.

Rules:

- confidence must be justified with `confidence_rationale`
- confidence affects allowed stage:
  - low confidence → Watch/Advisory only
  - high confidence → Warning/Evacuation eligible

### 3.4 Trust repair after false positives

False alarms will happen; manage them as safety incidents.

- immediate correction message (retraction) with plain language
- after-action review (AAR) that includes:
  - what triggered the alert
  - why verification failed
  - controls added (thresholds, training, gating)
- transparency policy: publish summary at the “balanced” level without exposing sensitive details

## 4. Misinformation and information operations (rumor control)

### 4.1 Trusted source registry

Maintain a registry of:

- official accounts/domains
- authorized spokespeople (PIO/JIC)
- partner agency comms contacts

Where feasible, use message signing / strong provenance (at minimum: internal signing of messages and immutable audit logs).

### 4.2 Rumor intake and triage

Intake channels:

- 311/CRM tagged as “rumor/misinformation”
- social monitoring queue (signals only)
- partner agency reports

Triage workflow:

- classify rumor type (safety, evacuation, shelter, infrastructure)
- assign owner (PIO + incident command)
- verify with authoritative sources
- publish correction across channels with shared `message_id`

### 4.3 Auditability of communications

Every message must be logged:

- who authored and approved
- time issued and channels
- edits/retractions
- evidence references

## 5. Privacy and sensitive location protections

### 5.1 Location precision tiering

Define tiers:

- **Public**: coarse polygons/areas (neighborhood/zone); no individual addresses for sensitive contexts.
- **Responder**: higher precision for assigned personnel.
- **Analyst**: controlled access for damage assessment and recovery planning.

Default to the least precise representation that supports the task.

### 5.2 Sensitive facilities and vulnerable populations

- maintain a protected layer for sensitive sites (shelters, critical facilities)
- strict role-based access + audit
- additional redaction for public records responses

### 5.3 Retention, legal discovery, and redaction

- define retention by record type (messages, location traces, field photos)
- legal hold workflow for incidents
- redaction workflow for public records requests

## 6. Platform resilience engineering (offline & degraded mode)

### 6.1 Offline-first field operations

Field capabilities that must work offline:

- assignment lists
- map tiles for assigned area
- damage assessment forms + photos
- critical contacts and checklists

Sync semantics:

- local-first writes with conflict resolution
- prioritize “safety critical” updates (closures, hazards)
- backlog age metrics

### 6.2 Degraded-mode behaviors

When upstream systems fail:

- ingest: queue and annotate staleness
- serve: show “last updated” + confidence degradation
- alerting: require manual confirmation; disable automation

### 6.3 Dependency mapping and fallbacks

Maintain an explicit dependency map:

- telecom/cellular carriers
- power and backup power sites
- cloud region dependencies
- GIS services and tile servers

Plan with PACE (Primary/Alternate/Contingency/Emergency) communications patterns for core workflows.

## 7. Recovery as a longitudinal program

### 7.1 Integrations

Link recovery tracking to:

- asset management work orders and condition assessments
- permitting and inspections
- procurement and contractor mobilization
- finance/grants (eligibility, documentation)
- benefits realization tracking (what was restored and for whom)

### 7.2 Equity outcomes

Track distribution of:

- shelter access and occupancy vs need
- restoration times by neighborhood
- aid delivered and delays

Publish at “balanced” transparency: enough to show fairness and performance without exposing vulnerable individuals.

## 8. Operational runbooks (minimum set)

### 8.1 Major incident activation

- EOC activation checklist
- access grants for mutual aid (time-boxed)
- comms posture (staged alerts; rumor control channels active)

### 8.2 False alarm and retraction

- stop further escalation
- issue correction
- preserve evidence and logs
- schedule AAR within 72 hours

### 8.3 Misinformation campaign during incident

- activate rumor control workflow
- coordinate with partner agencies
- publish corrections, pin official sources

### 8.4 Platform outage during incident

- failover to alternate region/edge
- switch field ops to offline mode
- activate manual comms fallbacks

### 8.5 Post-incident AAR + corrective action tracking

- capture timeline, decisions, and comms
- assign corrective actions with owners and deadlines
- track completion and retest in next drill

## 9. Key metrics

### 9.1 Alert governance metrics

- time-to-verify (signal → confirmed)
- staged alert progression time
- false positive proxy rate (retractions per incident)
- override/two-person compliance

### 9.2 Communications metrics

- message delivery latency by channel
- cross-channel consistency rate (same message core)
- correction time (rumor detected → correction issued)

### 9.3 Shared operating picture metrics

- freshness/completeness by source
- conflict rate and time-to-reconcile

### 9.4 Offline resilience metrics

- offline sync success rate
- backlog age distribution
- % field operations completed offline without data loss

### 9.5 Recovery program metrics

- restoration throughput (work orders closed/week)
- time-to-restore by neighborhood
- equity distribution metrics for aid and restoration

## 10. Implementation roadmap

### 0–3 months

- define ICS-aligned canonical data model and role mappings
- integrate basic feeds (CAD/911, GIS, CAP)
- implement staged alert governance + two-person rule for high-impact alerts
- run tabletop drill using the new workflows

### 3–12 months

- implement offline-first field ops + degraded mode
- build rumor control workflow + trusted source registry
- harden cross-agency interoperability and conflict resolution workflows

### 12–24 months

- multi-jurisdiction federation (mutual aid access + governance)
- resilience engineering maturity (multi-region/edge fallbacks, dependency testing)
- integrate recovery program end-to-end (asset mgmt, permitting, procurement, finance)

## 11. References

### 11.1 Workspace source

- Item 38 in [`kali-task-research.md`](../kali-task-research.md:1)

### 11.2 External references (retrieved via Firecrawl MCP)

- FEMA. *Incident Action Planning Process* (PDF). https://www.fema.gov/sites/default/files/documents/fema_incident-action-planning-process.pdf — Describes the ICS incident action planning process and operational period planning cycle, anchoring how objectives and plans are iterated.
- FEMA Training. *ICS Organizational Structure and Elements* (PDF). https://training.fema.gov/emiweb/is/icsresource/assets/ics%20organizational%20structure%20and%20elements.pdf — Defines core ICS roles and responsibilities, useful for mapping platform workflows to real incident roles.
- FEMA. *IPAWS Best Practices* (PDF). https://www.fema.gov/sites/default/files/documents/fema_ipaws-best-practices-guide.pdf — Provides recommended policies, procedures, training, and operational practices for alerting authorities; supports staged alert governance and false-alarm reduction.
- FEMA. *NIMS Basic Guidance for Public Information Officers* (PDF). https://www.fema.gov/sites/default/files/documents/fema_nims-basic-guidance-public-information-officers_12-2020.pdf — Operational guidance for PIO/JIC functions including rumor control and public information practices.
- CISA. *Emergency Communications Guidance Documents and Publications.* https://www.cisa.gov/emergency-communications-guidance-documents-and-publications — Collection of guidance supporting operable/interoperable emergency communications and continuity planning.
