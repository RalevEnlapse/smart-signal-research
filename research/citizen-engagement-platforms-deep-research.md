# Citizen engagement platforms — Deep research (operations-integrated)

## Executive summary
Citizen engagement platforms are not “an app”. They are a **service-delivery feedback loop** that turns resident input into **dispatchable work** and turns city action into **truthful, auditable status**.

A City Digital Twin benefits because this channel provides:
- a structured “human sensor” stream (reports, photos, perceptions, repeated nuisances)
- a measurable operational loop (intake → triage → dispatch → resolution → learning)
- legitimacy and adoption mechanisms (two-way updates, transparency)

This document strengthens item 27 in [`kali-task-research.md`](kali-task-research.md:1) by specifying **integration patterns**, **governed triage**, **identity/eligibility models**, **equity operations**, **moderation**, and **privacy/retention** so the platform remains defensible under public scrutiny.

---

## 1) Scope, success criteria, and non-goals
### Scope
Covers:
- **311-style issue reporting + case tracking** (multi-channel intake)
- **Participatory processes** (participatory budgeting, surveys, deliberation)
- Integration with operations systems (311/CRM, CMMS/EAM, departmental ticketing, GIS/asset registry)
- Trust & safety: moderation, abuse resistance, transparency reporting

### Success criteria (operational)
- Reports become **actionable work** with clear ownership and traceable timestamps.
- Resident-facing status is **truthful**, not “closed-to-hit-SLA”.
- Equity is operationalized: participation bias is measured and corrected.
- Safety and privacy: geotagged reports and media are handled with minimization and redaction.

### Non-goals (explicit)
- Not a policing tip line unless separately governed (different risk model).
- Not an unmoderated social network.
- Not a replacement for emergency services.

---

## 2) End-to-end integration patterns (explicit)
### 2.1 Reference flow (text diagram)
```
Intake (web/app/phone/SMS/in-person)
  → Normalize + validate (category, location, media safety checks)
  → De-dupe + merge (with audit trail)
  → Triage (priority score + human review where required)
  → Route (department/service area)
  → Create/Update downstream record (CRM/CMMS/Dept ticket)
  → Status updates (from system-of-record)
  → Closure (with evidence/closure code)
  → Reopen / appeal (audit trail)
  → Analytics + learning (hotspots, recurrence, SLA integrity)
```

### 2.2 Integration landscape
Minimum integrations for a municipal “truth loop”:
- **311/CRM** (case master, resident communications)
- **CMMS/EAM** (asset-linked work orders, crews, materials)
- **Departmental ticketing** (transport, sanitation, parks, code enforcement)
- **GIS/asset registry** (service areas, ownership, asset IDs)

### 2.3 Patterns (practical)
- **System-of-record (SoR) rules**
  - Case master: typically **311/CRM**.
  - Work execution: typically **CMMS/EAM** or departmental system.
  - The engagement platform must not invent status; it **mirrors** status from SoR.

- **Idempotency**
  - Every intake creates an immutable `intake_event_id` and a stable `case_id`.
  - Downstream creates/updates must be idempotent on `{case_id, downstream_system}`.

- **Timestamps and closure codes**
  - `acknowledged_at`: when city accepts the report.
  - `dispatched_at`: when work assigned.
  - `resolved_at`: when fix applied (field-confirmed or sensor-confirmed).
  - `closed_at`: administrative closure (may follow resolution).
  - Closure codes are standardized and mapped across systems.

### 2.4 “Truthful status” policy (anti-premature closure)
Rules:
- A case can be **closed** only if:
  1) it has a valid closure code, and
  2) it has a resolution evidence reference (work order completion, inspection note, or verified no-action reason).
- All status transitions are append-only events (no silent edits).
- **Reopen** is always allowed within a defined window and requires a reason; reopen counts are tracked.

---

## 3) Governed triage: dedupe, priority, fairness
### 3.1 Auditable triage model (inputs)
Priority scoring must be explainable and logged using auditable inputs:
- severity (safety hazard, accessibility impact)
- vulnerability context (sensitive facilities, schools, senior centers)
- asset criticality (critical infrastructure vs cosmetic)
- recurrence (repeat complaints, repeat work)
- service impact (road closure, flooding)
- confidence (location accuracy, media evidence, reporter credibility signals)

### 3.2 Duplicate detection governance
- Use automated similarity (location proximity + category + text/image similarity) with:
  - **confidence thresholds** (auto-merge only above high threshold)
  - **human review queue** for mid-confidence matches
- Maintain merge history:
  - `merged_case_ids[]`, `merge_reason`, `merge_confidence`, `reviewer_id`, `merged_at`
- Never delete: merged cases remain retrievable.

### 3.3 Priority scoring governance
- Require:
  - explanation string (“why this is high priority”) assembled from inputs
  - periodic bias audits (who gets routed faster by neighborhood/channel)
  - appeal/review channel for residents (case review request)

### 3.4 Anti-gaming controls for SLA metrics
Detect and deter “close-to-hit-SLA” behavior:
- anomaly signals:
  - high closure rate near SLA boundary
  - high reopen rate by team/category
  - reclassification spikes to categories with easier SLAs
  - high “no issue found” codes without evidence
- governance:
  - quarterly sampling audits
  - penalties/incentives tied to **integrity metrics**, not just speed

---

## 4) Identity, anonymity, and eligibility
Override defaults: `identity_posture = privacy-preserving-by-default`, `anonymity_allowed = yes`, `participation_eligibility = residents + local workers`.

### 4.1 Issue reporting identity model (anonymity-friendly)
Support three modes:
1. **Anonymous** (allowed):
   - no account required
   - rate-limited; limited follow-up
   - higher fraud/abuse controls
2. **Pseudonymous account**:
   - stable handle; privacy-preserving
   - supports follow-ups and reputation signals
3. **Verified account**:
   - optional for reporting; required for some high-risk categories

Accountability controls:
- device/browser risk signals and rate limiting
- abuse escalation (repeat malicious submissions → blocks)
- protected reporting paths for sensitive whistleblowing categories

### 4.2 Participatory budgeting / voting eligibility
Eligibility target: **residents + local workers**.

Patterns (options-not-prescriptions):
- **In-person + assisted verification** (most inclusive): community centers verify eligibility; platform records a verification token.
- **Document check with minimization**: verify once, then store a minimal “eligibility proof” token, not raw documents.
- **Third-party verification**: trusted civic partner verifies and issues token.

Fraud controls:
- one-person-one-vote enforcement (unique eligibility token)
- anomaly detection (burst voting, correlated patterns)
- rate limits and cooling-off for suspicious sessions

Anti-coercion / anti vote-buying
- Avoid providing “vote receipts” that enable proof to a coercer.
- Limit public disclosure of individual participation.
- Provide safe reporting channel for coercion attempts.

Account recovery
- recovery mechanisms must not enable identity takeover; use step-up verification for recovery.

---

## 5) Moderation & harm prevention operating model
Assumption: `moderation_capacity = limited`.

### 5.1 Policy categories
Minimum categories:
- harassment/threats
- hate content
- doxxing and personal data exposure
- explicit content
- spam/scams
- misinformation affecting safety (e.g., false outage instructions)

### 5.2 Workflow and SLAs
- intake filters:
  - malware scanning for uploads
  - PII detection in free text (flag for review)
- queues:
  - standard queue
  - urgent safety/legal queue
- SLAs:
  - urgent: hours
  - standard: 1–2 business days

### 5.3 Appeals and transparency
- appeal workflow with independent reviewer
- publish a quarterly transparency report (template below)

### 5.4 Coordinated abuse response
Controls:
- bot and brigading detection (burst + similarity)
- temporary friction (rate limits, proof-of-work, step-up challenges that are accessibility-safe)
- incident runbook (see §9)

### 5.5 Moderator safety
- rotate staff, provide mental health support
- define escalation paths for credible threats

#### Transparency report template
| Metric | Value |
|---|---|
| Reports submitted |  |
| Content removals |  |
| Accounts restricted |  |
| Appeals filed / upheld |  |
| Median time-to-review |  |
| Coordinated abuse incidents |  |

---

## 6) Equity & representativeness (operationalize)
### 6.1 Measurement plan
Measure representativeness using privacy-safe aggregation:
- channel mix (app/web/phone/in-person)
- participation rates by neighborhood (and demographic proxies only where lawful)
- accessibility and language usage

### 6.2 Correction mechanisms
- assisted digital intake (phone/in-person) with equal status visibility
- outreach partnerships (libraries, schools, community orgs)
- multilingual support (beyond UI: translated notifications and call scripts)
- accessibility testing aligned to WCAG 2.2 AA

### 6.3 “Do no harm” rules
- avoid public maps that stigmatize neighborhoods (use aggregation and context)
- avoid feeding biased enforcement (ensure triage is not a proxy for policing)

---

## 7) Data governance, retention, and privacy
### 7.1 Data classification (minimum)
| Data | Classification | Notes |
|---|---|---|
| Account identifiers | PII | Minimize, encrypt |
| Free-text descriptions | Potentially sensitive | PII/doxxing detection |
| Photos/videos | Potentially sensitive | faces, addresses, license plates |
| Precise location | Sensitive | minimize for public display |
| Allegations about individuals | Highly sensitive | restricted access, legal review |

### 7.2 Precision minimization for public views
- public maps use:
  - rounding/jitter
  - aggregation to blocks/areas
  - suppression for low counts

Anonymisation guidance should consider singling-out and linkability risks (see ICO source).

### 7.3 Redaction workflow
- automated detection (faces/plates/addresses) → flag
- human review for publishable media
- store redacted and original versions with access controls

### 7.4 Retention and legal hold
- define retention by data type (cases, media, identity proofs)
- legal hold process for litigation/FOI/public records obligations
- immutable audit logs for edits/merges/status changes

---

## 8) Security & abuse resistance
- rate limiting and spam detection
- bot mitigation with accessibility-friendly approaches
- DoS resilience (CDN/WAF)
- admin protection: MFA, least privilege, session monitoring
- tamper-evident event log for status changes (append-only)

---

## 9) Operational runbooks (minimum)
### 9.1 Surge event intake (storm/outage)
- activate surge mode categories
- broaden dedupe radius and increase human triage staffing
- communicate service expectations publicly

### 9.2 Coordinated abuse / misinformation campaign
- enable friction controls
- isolate affected categories
- publish rapid transparency note

### 9.3 “Ticket closed but unresolved” escalation
- resident escalation path
- audit closure evidence
- reopen with supervisor review

### 9.4 Privacy incident (exposed sensitive report/photo)
- remove from public view immediately
- incident response and notification per policy
- postmortem and control fixes

---

## 10) Key metrics (include integrity/anti-gaming)
Operational:
- intake volume, dedupe rate
- time-to-ack, time-to-dispatch, time-to-resolve
- reopen rate

Equity:
- channel mix by neighborhood
- assisted vs digital share
- accessibility/language usage

Moderation:
- time-to-review, appeals rate, appeal uphold rate

Integrity:
- closure anomaly rate near SLA boundary
- reclassification spikes
- % closures with evidence references

---

## 11) Implementation roadmap
### 0–3 months
- integration mapping and SoR decisions
- shared taxonomy + closure codes
- minimal truthful status loop (append-only event log)
- pilot department integration

### 3–12 months
- expand integrations (CMMS + more departments)
- governed triage (dedupe + priority governance + audits)
- moderation operating model + transparency report v1
- equity measurement baseline and assisted-channel improvements

### 12–24 months
- participatory modules at scale with eligibility tokens
- advanced analytics (hotspots, recurrence, service learning)
- mature transparency reporting and independent audits

---

## 12) Risks & mitigations
| Risk | Mitigation |
|---|---|
| Harassment / doxxing | moderation policies, urgent queue, redaction |
| Fake reports / bots | rate limits, anomaly detection, friction controls |
| Biased triage | explainability, bias audits, appeal channel |
| SLA gaming | closure anomaly detection, evidence-required closure |
| Privacy harms from geodata/photos | precision minimization, redaction workflows |
| Legitimacy failure | truthful status policy, transparency reporting |

---

## Sources (high-signal anchors)
- Service Manual (UK Government Digital Service). https://www.gov.uk/service-manual — Practical guidance for building and running public digital services, including user needs, accessibility, and service operations.
- Web Content Accessibility Guidelines (WCAG) 2.2. https://www.w3.org/TR/WCAG22/ — The core standard for accessibility requirements; use AA as baseline.
- ICO — “How do we ensure anonymisation is effective?”. https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-sharing/anonymisation/how-do-we-ensure-anonymisation-is-effective/ — Explains identifiability risk, singling out, and linkability; useful for designing privacy-safe public maps and releases.
- HUD Exchange — Participatory Budgeting overview. https://www.hudexchange.info/programs/participatory-budgeting/ — Public-sector framing of PB; supports defining scope and governance for budgeting participation.
