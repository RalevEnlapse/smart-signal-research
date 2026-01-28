# Incident response rehearsals — Deep research

## Executive summary
Incident response rehearsals are **repeatable, measurable exercises** (tabletops, functional drills, and full-scale simulations) that validate whether a City Digital Twin program can respond to incidents without amplifying harm.

For a digital twin, rehearsals must go beyond generic “IT outage” scenarios. They must test twin-specific failure modes:
- **model failures** that drive harmful recommendations
- **data integrity failures** that corrupt dashboards and decision support
- **policy-as-code / IAM failures** that block legitimate emergency access
- **privacy incidents** (re-identification from joins/exports)
- **federation failures** across agencies/partners
- **vendor remote-access compromise** affecting OT-adjacent domains

“Good” is not “we held a drill.” Good is:
- reduced time-to-detect/triage/contain/restore
- improved decision quality under stress
- verified ability to grant least-privilege emergency access quickly
- audit evidence sufficient to reconstruct what happened
- fewer repeat findings across exercises

This document deepens item 14 in [`kali-task-research.md`](kali-task-research.md:1): *“Incident response rehearsals: Run tabletop exercises and on-call drills to reduce downtime during real incidents.”*

---

## 1) Purpose, scope, and what good looks like

### 1.1 Purpose
Rehearsals are a capability to validate end-to-end:
- **detect → triage → contain → recover**
- governance under stress (privacy, access, evidence, comms)
- cross-agency coordination and partner integration

### 1.2 Scope (default)
Scope includes **cyber + data + model + governance** incidents that touch:
- ingestion pipelines and data products
- analytics/model services
- policy enforcement (OPA/ABAC), IAM, federation
- export/egress controls
- audit logs and evidence handling
- dashboards and downstream operational integrations

### 1.3 Non-goals
- Not a blame exercise.
- Not a compliance theater exercise.
- Not a full penetration test replacement.

### 1.4 “What good looks like” (acceptance criteria)
An exercise is “effective” only if it produces evidence that:
- responders can access the right systems within the time budget
- containment decisions are timely and proportional
- privacy controls hold (or fail safely) under pressure
- comms decisions are controlled (no accidental public alerts)
- corrective actions are implemented and re-tested

---

## 2) Twin-specific scenario library (concrete)

Use a scenario catalog with measurable objectives. Each scenario has a “golden path” plus injects.

| Scenario | Trigger symptoms | Primary risks | Systems involved | Expected decisions + required artifacts |
|---|---|---|---|---|
| Harmful recommendation due to incorrect model output | sudden spike in Tier 2/3 recommendations; operator reports “nonsense” | safety, rights, reputational | model service, feature store, HITL workflow, decision logs | decision to disable model (feature flag), downgrade tiers; capture model version/run IDs, validation status, override rationale; open incident ticket and comms plan |
| Data pipeline corruption/drift misleading dashboards | KPIs shift abruptly; schema changes; data quality alerts | operational harm, misallocation, loss of trust | ingestion pipelines, transformations, SLA monitors, dashboards | decision to freeze dashboards / label degraded; rollback pipeline version; produce lineage evidence, diff report, and affected products list |
| Policy-as-code/IAM misconfig blocks emergency access | incident roles can’t access dashboards/exports; “permission denied” errors | delayed response, unsafe workarounds | IAM/SSO, ABAC/OPA, break-glass workflow | activate time-bound entitlements; approve break-glass with scope; capture approval chain, policy diff, audit log extracts |
| Re-identification / privacy incident from joins/exports | unusual export event; small-cell outputs detected; complaint/partner notice | privacy/legal, public trust | catalog, query layer, export service, audit logs | stop dissemination; revoke access; incident classification; notify legal/privacy/partners; produce chain-of-custody and suppression/DP config evidence |
| Cross-agency federation failure during incident | partner cannot authenticate; token failures; role mapping breaks | coordination collapse | identity federation, partner portals, incident comms | switch to fallback identity path; scope access; document continuity plan and post-incident remediation |
| Vendor remote access compromise (OT-adjacent impact) | unusual vendor session; anomalous commands; alerts from EDR | safety/infra risk, lateral movement | vendor PAM, jump hosts, OT-adjacent connectors, logs | disable vendor access; rotate creds/keys; isolate connectors; preserve evidence; coordinate vendor incident manager |

Add additional scenarios on a rolling basis based on near-misses and changes in architecture.

---

## 3) Exercise types + proportionality

### 3.1 Exercise types
- **Tabletop (TTX):** decision-making and coordination; validates runbooks and decision rights.
- **Functional exercise:** uses real tools (IAM, logging, dashboards) in a safe environment.
- **Full-scale:** multi-agency + partner; tests comms, paging, and operational continuity end-to-end.
- **On-call drills:** focused muscle memory (e.g., revoke export token, disable model, rotate keys).

### 3.2 Proportionality rules
- Tier 0–1 decision products: quarterly tabletop + targeted drill.
- Tier 2–3 decision products: quarterly functional exercises + at least annual full-scale.
- When a major system changes (IAM, export controls, model rollout), run a **change-triggered exercise** within 30–60 days.

---

## 4) Safe rehearsal infrastructure (replay/sandbox)

Goal: realism with **no production impact**.

### 4.1 Replay environment patterns
- Maintain a **replayable event/log store** (sanitized) to simulate incidents with time-warp.
- Support “inject controller” to introduce drift, schema breaks, and delayed data.

### 4.2 Synthetic data for privacy-safe realism
- Use synthetic datasets that preserve:
  - distributions (counts, seasonality)
  - correlations (exposure↔outcome)
  - edge cases (rare conditions represented safely)
- Explicitly prohibit using real identifiable health/sensitive records in exercises.

### 4.3 Sandboxed integrations
- Paging, email/SMS, dashboards, and ticketing must have **exercise mode** endpoints.
- Hard safety interlocks:
  - cannot publish public alerts from exercise environments
  - cannot execute actuation commands (or require explicit dual-control test harness)

### 4.4 Safety officer and guardrails
- Name a Safety Officer with authority to pause/stop.
- Pre-run checklist:
  - environment isolation verified
  - comms channels clearly labeled
  - rollback steps tested

---

## 5) Governance / privacy / access control test plan

Rehearsals must include injects that test governance controls under stress.

### 5.1 Time-bound entitlements
Inject: responder needs elevated access to a restricted dataset/product.
- Measure **time-to-grant** least-privilege emergency role.
- Verify entitlements expire automatically.

### 5.2 Audit log integrity and investigative workflows
Inject: conflicting timelines between systems.
- Retrieve audit logs; verify immutability.
- Establish chain-of-custody for evidence exports.

### 5.3 Export controls and emergency overrides
Inject: urgent request to export data for partner coordination.
- Verify approvals, redaction, and logging.
- Test emergency exception path: time-boxed, compensating controls, post-hoc review.

### 5.4 FOI / public-records vs tactical secrecy
Inject: media/public request during active incident.
- Decision point: what can be disclosed now vs later.
- Required artifact: comms decision record + legal basis.

---

## 6) Cross-agency and external partner coordination

### 6.1 Participation agreements
Before joint exercises:
- define roles, contact points, and minimum commitments
- define what logs/evidence partners must provide
- agree information sharing constraints and confidentiality

### 6.2 Escalation paths and constraints
- create a cross-agency escalation matrix (who calls whom)
- define “need-to-know” evidence sharing rules
- define vendor escalation and executive notification thresholds

### 6.3 Vendor requirements
Vendors participating must bring:
- named incident manager + on-call escalation
- log access and export procedures
- evidence preservation process
- commitment to corrective actions and re-test participation

---

## 7) After-action review (AAR) and corrective action system

### 7.1 AAR template (exercise output)
Use an AAR/IP structure:
- objectives and scenario summary
- timeline with decision points
- what we expected vs what happened
- strengths
- gaps (root causes + contributing factors)
- improvement plan (actions, owners, due dates)
- verification evidence required
- re-test requirement and date

### 7.2 Corrective action tracking
Rules:
- every finding has an owner and due date
- closure requires evidence (runbook update, test result, config diff)
- repeat findings are escalated to leadership

---

## 8) Metrics (effectiveness, not theater)

### 8.1 Core time metrics
Track by scenario class:
- MTTD (mean time to detect)
- MTTA (mean time to acknowledge/triage)
- time-to-contain
- time-to-restore

### 8.2 Decision quality metrics
- false escalations avoided (signal verified before public action)
- correct escalation timing (stage changes aligned to evidence)
- % decisions with complete evidence packet

### 8.3 Access readiness
- time-to-grant emergency role (least privilege)
- % entitlements expiring on time
- # of “unsafe workaround” attempts (shared accounts, unmanaged exports)

### 8.4 Audit evidence quality
- can we reconstruct: who did what, when, with what data/model version?
- % incidents where chain-of-custody is complete

### 8.5 Program effectiveness trending
- repeat-finding rate
- closure quality (findings closed with verification)
- real incident outcome correlation (MTTR trend over quarters)

---

## 9) Operational runbooks (minimum)

### 9.1 Plan an exercise
- define scope, objectives, scenario class
- identify participants (city + partners)
- safety plan + comms isolation verification
- pre-brief materials (rules, no-fault, artifact expectations)

### 9.2 Run the exercise
- facilitator manages pace and injects
- safety officer enforces guardrails
- observers capture timeline and decision artifacts

### 9.3 Post-exercise
- conduct hotwash
- produce AAR/IP within 3–4 weeks
- track actions to closure; schedule re-test

---

## 10) Implementation roadmap

### 0–3 months
- scenario library MVP (include at least one data and one model incident)
- adopt AAR/IP template and tracking
- run first tabletop with governance/privacy injects

### 3–12 months
- build safe replay environment + synthetic data approach
- run cross-agency functional exercise incl. emergency access grant drill
- vendor participation agreements and joint drill

### 12–24 months
- annual full-scale exercise across agencies and key partners
- mature metrics program and quarterly effectiveness trend reports
- re-test discipline institutionalized (repeat findings near-zero)

---

## References (retrieved via Firecrawl MCP)

### Exercise program design + AAR practices
- CISA Tabletop Exercise Packages — https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages
  - Takeaway: Provides structured exercise resources aligned to public-sector needs, including planning guidance and templates.
- After-Action Report / Improvement Plan (AAR/IP) Template (CISA) — https://www.cisa.gov/sites/default/files/publications/8%20-%20CTEP%20AAR-IP%20Template%20%282020%29%20FINAL_508.pdf
  - Takeaway: Concrete AAR/IP structure to capture objectives, strengths, gaps, and an improvement plan with owners and timelines.
- Improvement Planning (HSEEP resources) — https://preptoolkit.fema.gov/web/hseep-resources/improvement-planning
  - Takeaway: Guidance and templates for improvement planning, emphasizing corrective action tracking and re-testing.

### Cyber incident response readiness measurement
- Computer Security Incident Handling Guide (NIST SP 800-61r2) — https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf
  - Takeaway: Defines incident handling lifecycle phases that map to measurable times (detect/analyze/contain/eradicate/recover) and evidence handling.

### Role-based access and incident readiness context
- Zero Trust Architecture (NIST SP 800-207) — https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-207.pdf
  - Takeaway: Zero trust framing emphasizes least privilege and continuous evaluation; relevant for rehearsing emergency access without over-broad permissions.
