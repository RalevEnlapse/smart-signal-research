# Digital signage — Deep research

## Executive summary

City digital signage is not a “screen rollout.” It is a **city-wide communications and actuation surface**: a managed output channel for the City Digital Twin that publishes situational awareness, service status, and emergency guidance to people where they are—especially when apps are unavailable, networks are congested, or crowds are present.

Done well, signage becomes:

- A resilient, multi-channel emergency outlet with **priority overrides** and end-to-end testing.
- A secure, governable edge fleet (players + screens) with strong device identity, patch discipline, segmentation, and incident response.
- A public information service with **truthfulness rules**, caching, and degraded modes.
- An accessibility-first public interface (language, placement, legibility, non-touch alternatives), not just WCAG checklists.

This rewrite focuses on the governance choices and operational mechanics that make a signage fleet safe and trustworthy at city scale.

## 0. Scope and positioning (signage as a twin output channel)

### 0.1 What this is

A city-operated (or city-governed) service that:

- publishes **standard messages** (service updates, advisories, closures, detours)
- publishes **emergency overrides** (life safety instructions)
- publishes **location-aware public info** (wayfinding, accessibility routes, next arrivals)
- provides **fleet health telemetry** (player status, connectivity, content version)

Signage is treated as a **data consumer** of digital twin products and also as an **operational actuator** (influence behavior through messaging).

### 0.2 Explicit non-goals / safety boundaries

Non-goals are controls:

- No targeted political persuasion; no partisan messaging.
- No covert surveillance.
- No biometric identification; no demographic inference.
- No individualized targeting based on inferred identity.
- No “all-clear” messaging without high-confidence upstream confirmation.

### 0.3 Default parameters assumed (unless overridden)

- Fleet scale: citywide
- Network context: mixed (municipal fiber + cellular)
- Emergency posture: CAP-integrated with **local override governance**
- Viewer analytics: **no-by-default**
- Content governance: central policy + local ops execution
- Integration targets: city alerts + mobility + events + weather + hazard dashboards

## 1. Stakeholders, ownership, and decision rights

### 1.1 Primary stakeholders

- **Public information / communications**: message policy, tone, approvals
- **Emergency management**: alerting authority, drills, after-action
- **IT/security (CISO/CIO)**: fleet security baseline, incident response
- **Facilities / street furniture ops**: installation, power, physical access
- **Transport / public works / utilities**: service status inputs
- **Vendors**: hardware, CMS, device management, field maintenance
- **Legal/privacy office**: acceptable analytics posture, records/retention

### 1.2 Decision rights (must be explicit)

- Who can publish routine content by category?
- Who can trigger emergency overrides and terminate them?
- Who can change templates for emergency messages?
- Who can enable/disable viewer analytics features?
- Who can push firmware/player updates and how are rollbacks handled?

## 2. Reference architecture (citywide, resilient, twin-integrated)

### 2.1 Components

1. **Content control plane (authoring + approval)**
   - templates, translation workflow, accessibility checks
   - approval gates and audit trails

2. **Distribution plane (publishing)**
   - content packaging, CDN/cache layers, TTLs
   - channel consistency checks (signage vs web vs SMS/app)

3. **Emergency override plane (high priority)**
   - CAP ingestion + local emergency console
   - priority rules + kill switch + confirmation

4. **Device management plane (fleet ops)**
   - device identity and provisioning
   - OTA updates with rollback
   - attestation/health reporting where feasible

5. **Edge player (unattended)**
   - local storage for offline operation
   - deterministic rendering
   - local policies: what can be shown offline

6. **Telemetry and audit**
   - device online/offline, content version, last update
   - security logs (auth events, config changes)
   - message publication logs (who/what/when/where)

### 2.2 Data contracts (twin → signage)

Treat feeds as contracted products:

- `alerts.public.v1` (CAP + local)
- `mobility.disruptions.v1` (closures, detours, arrivals)
- `weather.advisories.v1`
- `events.city_calendar.v1`

Minimum contract fields per message:

- `message_id`, `category`, `priority`, `audience`, `language`, `geo_scope`
- `valid_from`, `valid_to`, `ttl`
- `source_system`, `confidence`, `last_updated_utc`
- `render_template_id` + template version

## 3. Emergency broadcasting governance & safety

CAP integration is necessary but not sufficient. The safety question is: **who is allowed to override everything, how do we test it, and how do we prevent incorrect “official” messages?**

### 3.1 Authority model (roles)

- **Alerting Authority (AA)**: can originate CAP messages (emergency management).
- **Local Override Authority (LOA)**: can issue local overrides (e.g., incident commander) when CAP isn’t appropriate (local hazards, evacuations within a small zone, shelter-in-place at a facility).
- **Publisher Operators**: execute routine publication.

Rules:

- **Two-person rule** for highest-severity overrides (life safety): one initiator + one approver.
- All emergency messages must be **templated** (limit free text) and must include:
  - what happened, where, what to do now, where to get more info, time issued.

### 3.2 Priority rules

- Emergency overrides supersede all content.
- CAP messages supersede local overrides **unless** local override is explicitly configured for “facility local hazard” classes (documented).
- Conflicts must resolve deterministically and be auditable:
  - `priority` ordering
  - most specific `geo_scope` wins within same priority
  - newest message wins if same scope + priority

### 3.3 Testing, drills, and end-to-end validation

- **Drill cadence**:
  - monthly: silent tests (content path, audit logs)
  - quarterly: live drill on a test group of screens
  - annually: citywide drill coordinated with other channels
- Validate:
  - CAP-to-screen latency
  - override success rate (% targeted screens actually showing message)
  - multi-channel consistency (same message core across web/SMS/app/signage)

### 3.4 Degraded mode (offline and partial failure)

When offline, screens must show a **last-known safe** content set:

- static emergency instructions (e.g., evacuation basics, emergency numbers)
- facility-specific instructions (e.g., nearest shelter, assembly points)
- last updated timestamp + “connectivity degraded” indicator

Rule: offline mode must **not** display dynamic “all clear” or time-sensitive service status without a fresh update.

## 4. Security-by-design for an unattended edge fleet

Signage fleets are attractive targets: compromise is visible, reputationally damaging, and potentially safety-impacting.

### 4.1 Device identity lifecycle

- Provision unique device identity at manufacturing or secure onboarding.
- Maintain inventory of:
  - device model/serial
  - firmware/player version
  - certificates/keys (rotation schedules)
  - location + owner
- Decommissioning:
  - revoke credentials
  - wipe storage
  - update inventory and physical chain-of-custody

### 4.2 Secure boot, signed firmware, OTA updates

- Secure boot and signed firmware/player images.
- OTA pipeline requirements:
  - staged rollout (rings)
  - health checks before advancing rings
  - rollback to last-known-good
  - signed content packages

### 4.3 Attestation (where feasible)

- Periodic evidence of device integrity (TPM/TEE-backed where possible).
- Server-side policy:
  - if attestation fails → restrict to static safe content + raise incident

### 4.4 Segmentation: separate planes

- **Management plane** (device admin): strongest controls, limited access, MFA, jump host.
- **Content plane** (distribution): signed packages, CDN, read-only on devices.
- **Data plane** (telemetry): least privilege, rate limits.

No inbound access to players from the public internet; prefer outbound-only check-in.

### 4.5 Patch cadence and SBOM expectations

- Define patch SLAs by severity:
  - critical: patch within days (or compensating controls + isolation)
  - high: within weeks
  - medium/low: regular cycles
- Require SBOM for player software stack and key dependencies.

### 4.6 Incident response runbooks (security)

**A) Suspected content compromise**

- Trigger: unexpected content, defacement, channel drift, signature failures.
- Actions:
  - activate kill switch: revert to safe static playlist
  - isolate affected device groups
  - rotate credentials
  - preserve logs and content hashes
  - public comms coordination (transparent statement if public impact)

**B) Device theft/tamper**

- Actions:
  - revoke device credentials
  - mark as stolen in inventory
  - forensic review if recovered

## 5. Privacy and legality of viewer analytics (policy: no-by-default)

### 5.1 Default posture

- No cameras by default.
- No biometric processing.
- No inference of age, gender, ethnicity, identity.

If any analytics are enabled, they must be explicitly approved and constrained.

### 5.2 If aggregate-only analytics are approved (optional)

- On-device aggregation only.
- Store only counts (e.g., approximate footfall, interaction counts) with short retention.
- No raw video retention.
- No linkage to device identifiers.

### 5.3 Notice, approval gates, retention

- Clear public notice on/near screens describing what is collected.
- Privacy review and legal basis documented.
- Retention rules:
  - minimize; default short windows; publish retention schedule

### 5.4 Prohibited uses

- Law enforcement targeting.
- Profiling neighborhoods.
- Combining viewer analytics with other datasets to identify individuals.

## 6. Accessibility beyond WCAG (public-space realities)

### 6.1 Multilingual requirements

- define supported languages per location category
- translation workflow with emergency templates pre-translated
- fallbacks if translation unavailable (iconography + short text)

### 6.2 Physical placement and legibility

- placement/height for wheelchair users and standing viewers
- font sizes and contrast designed for viewing distance and lighting
- avoid glare; ensure night-time readability

### 6.3 Low vision/hearing support and alternatives

- high-contrast modes; large-text emergency layouts
- audio support where appropriate (and safe), plus captioning
- QR + short URLs as optional, not sole access for critical info

### 6.4 Interaction alternatives

- do not require touch-only interaction for critical services
- support “browse-only” and printed route options (where relevant)

## 7. Content governance & misinformation controls

### 7.1 Editorial policy and approvals

Create a written editorial policy:

- allowed content categories (service status, safety guidance, civic info)
- prohibited content (political advocacy, discriminatory messaging)
- tone guidance and crisis comms style

Approval workflow (central-policy-local-ops):

- central templates and policy
- local operators publish within policy
- audit trail for every publication (who/when/where/template/version)

### 7.2 Misinformation/rumor response

- maintain “rumor control” message templates
- rapid correction workflow:
  - detect rumor
  - approve corrective message
  - publish across channels
  - post-event review

### 7.3 Rollback/kill switch

- one-click rollback to last-known-good content set
- emergency kill switch to safe static content

## 8. Digital-twin integration and degraded modes

### 8.1 Caching/CDN strategy

- signed content bundles with TTL
- local cache on players (hours–days depending on content class)
- bandwidth shaping for cellular segments

### 8.2 Consistency across channels

Define a canonical “message core” and enforce:

- same `message_id` across channels
- consistency checks for headline + severity + instructions
- drift alerts when channels diverge

### 8.3 Truthfulness rules

- if data is stale or uncertain, display “last updated” and uncertainty cues
- never display confident “all clear” unless upstream authoritative confirmation exists

### 8.4 Outage behavior

- if central systems down: play safe cached playlist and emergency basics
- if feed down: show last-known data with clear timestamp or switch to static

## 9. Operational runbooks (minimum set)

### 9.1 Emergency override activation + termination

- verify authority + two-person rule
- confirm geo-scope
- publish override
- verify screen acknowledgements
- terminate override with explicit “end of alert” event; revert to normal scheduling

### 9.2 Network outage / player offline surge

- identify impacted segment (carrier, fiber ring, site)
- trigger degraded mode policy
- dispatch field checks where needed

### 9.3 Suspected content compromise

- activate kill switch
- isolate cohort
- rotate creds
- forensics and public comms

### 9.4 Broken screen / field maintenance dispatch

- detect via heartbeat
- create work order
- update status to local operators

### 9.5 Analytics privacy incident (if analytics enabled)

- disable analytics feature flag
- assess scope
- legal/privacy notification procedures

## 10. Key metrics (SLIs/SLOs)

### 10.1 Reliability and delivery

- fleet uptime (% screens healthy)
- MTTR for player offline
- content delivery latency (publish → display)
- CAP-to-screen latency (p50/p95)
- emergency override success rate (% screens showing within target)

### 10.2 Security metrics

- patch compliance (% within SLA)
- unauthorized access attempts
- attestation pass rate (if used)
- number of devices in “restricted safe mode”

### 10.3 Content governance metrics

- % content published via approved templates
- time-to-publish for emergency messages
- rollback frequency and reasons

### 10.4 Privacy and audit metrics

- analytics disabled-by-policy adherence
- retention compliance
- audit log completeness

### 10.5 Accessibility metrics

- language coverage by location class
- periodic accessibility audits (field checks)
- complaint rate by location/issue

## 11. Implementation roadmap

### 0–3 months (governance + pilot + security baseline)

- define authority model, two-person rule, priority policy
- create emergency templates (pre-translated)
- implement CAP ingestion and test screen cohort
- establish device inventory + provisioning
- baseline segmentation and access controls
- define privacy posture and prohibit viewer analytics by default

### 3–12 months (scale-out + OTA maturity + observability)

- scale fleet; implement rollout rings and rollback
- implement monitoring dashboards (health, content versions)
- implement channel consistency checks
- expand accessibility improvements (placement, large-text modes)
- run quarterly drills and after-action loop

### 12–24 months (redundancy + mature drills + transparency)

- add redundancy for critical sites
- mature attestation and SBOM processes
- publish transparency reporting (governance, incidents, audits)
- mature rumor-control workflows and exercises

## 12. References

### 12.1 Workspace source

- Item 28 in [`kali-task-research.md`](../kali-task-research.md:1)

### 12.2 External references (retrieved via Firecrawl MCP)

- FEMA. *Integrated Public Alert & Warning System (IPAWS).* https://www.fema.gov/emergency-managers/practitioners/integrated-public-alert-warning-system — Overview of IPAWS as the US public alerting system; supports dissemination of CAP-formatted alerts across multiple channels.
- FEMA. *Common Alerting Protocol.* https://www.fema.gov/emergency-managers/practitioners/integrated-public-alert-warning-system/technology-developers/common-alerting-protocol — Describes CAP as a digital format enabling consistent alert messages to be disseminated simultaneously over multiple warning systems.
- NIST. *NISTIR 8259A — IoT Device Cybersecurity Capability Core Baseline.* https://csrc.nist.gov/pubs/ir/8259/a/final — Defines baseline cybersecurity capabilities for IoT devices (e.g., secure update, identity, configuration, vulnerability management), useful for unattended signage players.
- NIST. *SP 800-213 — IoT Device Cybersecurity Guidance for the Federal Government.* https://csrc.nist.gov/pubs/sp/800/213/final — Guidance for establishing IoT device cybersecurity requirements and integrating IoT devices into secure systems.
- ICO (UK). *Facial Recognition Technology (FRT) and surveillance.* https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/cctv-and-video-surveillance/guidance-on-video-surveillance-including-cctv/additional-considerations-for-technologies-other-than-cctv/facial-recognition-technology-frt-and-surveillance/ — Notes that biometric data processing for unique identification is prohibited unless strict lawful basis/conditions apply, informing a “no biometrics” viewer analytics policy.
- U.S. Access Board. *ADA Standards — Chapter 7 (Communication Elements and Features) / Signs.* https://www.access-board.gov/ada/chapter/ch07/ — Federal accessibility requirements and guidance relevant to public information displays and signage.
