# Health — Deep research

## Executive summary
A municipal “health twin” connects **city exposures and interventions** (heat, air quality, mobility, housing conditions, service access) to **population health outcomes** (syndromic indicators, ED visit aggregates, EMS demand, mortality aggregates) to support:

- **Population-level planning and evaluation (core):** quantify co-benefits/harms of policy, prioritize investments, and monitor equity.
- **Operational surveillance and alerts (optional):** detect unusual patterns that may merit public health investigation and coordinated response.
- **Clinical decision-making (out of scope):** no patient-level decision support or clinical recommendations unless separately governed, clinically owned, and legally authorized.

This document assumes **realistic municipal constraints**: cities often receive **delayed, aggregated** health indicators (weekly ED syndrome counts by area; monthly mortality by cause; EMS demand summaries), delivered under data use agreements, with strict disclosure control. The design is therefore **privacy-safe by default**, focusing on:

- explicit **minimum cell sizes**, suppression and precision rules
- **join governance** (when exposure↔health linkage is allowed; safe joins; prohibited joins)
- **tiered causal evaluation playbooks** with guardrails for confounding, selection bias, measurement error, spillovers, and multiple concurrent interventions
- **model governance** (versioning, approval, monitoring, rollback)
- integrated **public health operations workflows** (staged alerts, verification, thresholds, RACI, false-alarm handling)

This document deepens item 12 in [`kali-task-research.md`](kali-task-research.md:1): *“Health: Link environmental, mobility, and service data to analyze population outcomes and intervention effectiveness.”*

---

## 1) Scope: “safe municipal scope”

### 1.1 In-scope (default / core)
1) **Population health situational awareness** at safe geographic and temporal scales (e.g., weekly syndromic rates by district).
2) **Intervention evaluation** for city policies and operations (cooling centers, low-emission zones, street redesign, housing remediation programs).
3) **Equity monitoring**: distribution of exposures, access, and outcomes across populations and places, using governance-approved strata.
4) **Service planning support**: EMS surge planning, cooling center resourcing, outreach targeting (without targeting individuals).

### 1.2 Optional (higher governance burden)
1) **Syndromic/outbreak signal alerting**: automated detection of unusual patterns that trigger investigation workflows.
2) **Cross-organization coordinated response**: routing verified signals into emergency management / incident processes.

These require additional controls: verification steps, escalation authority, audit trails, and clear communication practices (see §7).

### 1.3 Explicitly out-of-scope (unless separately governed)
- **Clinical decision support** (diagnosis, triage advice, treatment recommendations)
- **Individual-level risk scoring** or case finding
- **Enforcement uses** (e.g., housing code enforcement targeted from health outcomes) without an explicit legal basis, DPIA, and community oversight

---

## 2) Data access realism and partnership model

### 2.1 What cities can typically obtain (and what they usually cannot)
Municipal public health teams typically obtain **some combination** of:

**A) Syndromic / near-real-time (often the fastest available):**
- ED syndromic counts/rates by syndrome group and geography (daily–weekly; often 1–7 day delay)
- EMS/911 demand aggregates by call type and area (same-day to weekly; definitions vary)

**B) Utilization / outcomes (often delayed):**
- ED visits / hospital admissions aggregates by diagnosis group (weekly–monthly; 2–8 week delay)
- Mortality aggregates by cause and geography (monthly–quarterly; longer for final counts)

**C) Program / service access (often municipal-owned):**
- Cooling center attendance aggregates; outreach contacts aggregated
- Social services caseload aggregates (high governance sensitivity)

**D) Denominators and vulnerability context (generally accessible):**
- population counts, age bands, deprivation indices, vulnerability indices (census + admin)

What cities often **cannot** obtain without special legal basis:
- patient-level clinical records, full encounter notes, identifiable lab results
- fine-grained timestamps + fine-grained locations for rare conditions

### 2.2 Hospital heterogeneity, incentives, and data product normalization
Reality constraints you must design for:
- **Different EHRs and coding practices** (ICD/LOINC/SNOMED usage varies; syndrome mappings differ)
- **Different geocoding pipelines** (home address vs postal code vs facility catchment)
- **Different reporting cadences** (batch weekly exports vs automated feeds)
- **Incentive misalignment** (provider burden; reputational concerns; cost of data extraction)

Design response:
- Treat hospital feeds as **products** with explicit contracts: schema, allowed strata, cell-size rules, latency SLO, and revision rules.
- Use **standardized syndrome/value sets** and mappings where possible; otherwise capture mapping tables with provenance and versioning.
- Maintain a **Data Product Conformance Report** per partner (completeness, coding drift, geography coverage, late revisions).

### 2.3 Regional boundary misalignment (patients don’t match city boundaries)
Health systems serve regions; residents cross borders.

Practices:
- Choose a **primary population definition** (e.g., residents vs events-in-city). Use both where feasible.
- Maintain **cross-border denominators**: day/night population proxies, commuter-adjusted denominators for exposure analyses.
- Use **regional compacts** or data trusts where possible; otherwise implement “edge-of-city uncertainty” flags.

### 2.4 Minimum viable data set (MVDS) for a municipal health twin
Start with an MVDS that is both useful and feasible under constrained access:

1) ED syndromic counts by syndrome group, week, and approved geography
2) EMS demand counts by call type, week, and approved geography
3) Mortality aggregates (monthly/quarterly) by broad cause group and approved geography
4) Exposure surfaces (heat index, PM2.5, NO2) at approved resolution + uncertainty metadata
5) Population denominators and vulnerability indices by same geography
6) Intervention registry (what changed, when, where, why)

### 2.5 Data sharing agreements (DSAs): minimum clauses
A DSA (or DUA) should explicitly cover:
- purpose limitation (planning/evaluation; alerting only if opted-in)
- allowed outputs, allowed strata, **minimum cell size rules**, and redaction obligations
- latency expectations, revision/backfill policy, and outage notification
- join permissions (what may be linked; where; by whom)
- audit logging, incident response, and breach notification
- retention periods and deletion requirements

---

## 3) Privacy & disclosure control (operational, enforceable)

### 3.1 Data classification for health analytics (municipal view)
Use the governance/privacy program as the umbrella (see [`research/governance-privacy-deep-research.md`](research/governance-privacy-deep-research.md:1)). For health specifically, assume:

- **Default: Aggregated, de-identified indicators only**.
- Any move toward individual-level linkage requires a separate legal basis + DPIA + clinical ownership.

### 3.2 Minimum cell sizes, suppression, and safe publishing rules
Adopt explicit rules; tune them with legal/privacy and partner DSAs.

**Baseline (internal analytic products):**
- Minimum cell size: **k ≥ 11** (counts < 11 suppressed)
- Complementary suppression: prevent differencing attacks in published tables
- Rate limits: do not publish rates where numerator < 11 or denominator < 1000 (example; calibrate)

**Public products (default):**
- Minimum cell size: **k ≥ 25**
- Publish only at coarse geography and coarse time (see §3.3)
- No publication of rare condition strata or intersectional slicing that can isolate small groups

**Small-n handling:**
- Suppress (preferred) → aggregate further (space/time) → combine categories → withhold

### 3.3 Spatial/temporal aggregation and precision limits
Define a small set of approved aggregation “lenses” and forbid ad-hoc slicing.

**Geographies (examples):**
- G0: Citywide
- G1: District / ward
- G2: Neighborhood clusters (privacy-reviewed)
- G3: Grid cells (internal-only, only when densities support k rules)

**Time windows:**
- T0: Monthly (public default)
- T1: Weekly (internal default)
- T2: Daily (internal only; only for high-count syndromic indicators)

**Precision rules:**
- Never publish point locations of health events.
- Public maps: use choropleths at G1 or coarser; smooth/blur only as a supplement to proper aggregation.

### 3.4 Join governance (exposure ↔ health linkage)
The key risk in a health twin is not a single dataset—it is **linkage**.

#### 3.4.1 When linkage is allowed
Allowed (typical):
- Link aggregated health indicators (by Gx, Tx) to aggregated exposure surfaces summarized to the same Gx, Tx.
- Link to vulnerability indices at matching geography.

Allowed only with elevated review:
- Linkage that increases granularity (e.g., daily + small geography)
- Linking across multiple sensitive strata that could isolate protected groups

Not allowed (default):
- Any attempt to reconstruct trajectories, households, or identifiable individuals
- Joining health indicators to enforcement datasets for punitive actions

#### 3.4.2 Approval workflow and audit logging
Required steps for any new join:
1) **Join request**: purpose, datasets, keys, proposed strata, expected outputs
2) **Risk review**: privacy officer + data steward + partner rep (if partner data)
3) **Safe-join design**: approved geometry/time lens, suppression strategy
4) **Execution controls**: run in controlled environment; no raw exports
5) **Audit evidence**: immutable log of inputs, query template hash, reviewer approvals, output checks

#### 3.4.3 Safe-join patterns
- Join at **approved aggregation lens** (G1/T1 etc.)
- Use **one-way enrichment**: exposures summarized to match health aggregates
- Prefer **privacy-preserving record linkage (PPRL)** only for limited, pre-approved studies (see §3.6)

#### 3.4.4 Prohibited joins (examples)
- Joining small-area health counts with granular mobility traces
- Joining to school-level attendance + small-area ED visits
- Any join enabling “who got sick where/when” inference

### 3.5 Differential privacy (DP): when to use it (and when not to)
DP can help when you must publish more granular statistics, but it adds complexity.

Use DP when:
- releasing repeated statistics over time where differencing risk is material
- enabling broader internal self-serve while controlling leakage

Avoid DP when:
- the main risk is already solved by coarse aggregation and suppression
- partners will not accept DP assumptions or you cannot govern ε properly

### 3.6 DP privacy budget governance (ε governance)
If DP is used, governance must be explicit:

- **ε owner:** a named Privacy Budget Owner (PBO) within the city privacy office, co-signed by the health department data steward.
- **Budget allocation:** allocate ε by *product family* (e.g., weekly syndromic dashboard, public monthly report) with a fixed cadence.
- **Approval gates:** any new DP release or increased granularity requires PBO approval.
- **Reporting:** quarterly report of ε consumption and products released.
- **Revocation/rollback:** ability to stop releases if risk posture changes.

---

## 4) Causal evaluation playbook (standard template + pitfalls)

### 4.1 Tiered rigor model (match method to decision risk)
- **Tier A (screening / prioritization):** descriptive + associations with strong caveats (used to decide what to evaluate)
- **Tier B (decision support):** quasi-experimental designs (DiD, synthetic control) with pre-registered analysis plan
- **Tier C (high-stakes):** multiple robustness checks, sensitivity analysis, external peer review, and (where possible) randomized or stepped-wedge designs

### 4.2 Reusable evaluation template (fill-in for each intervention)
**1) Intervention definition**
- What changed? (policy/operations)
- When did it start? (implementation date(s), ramp-up)
- Where? (geography, eligibility)
- Who is targeted/affected? (population)

**2) Theory of change (ToC)**
- Causal pathway from intervention → exposure/behavior → intermediate outcomes → health outcomes
- Expected timing (lags), direction, and plausible null

**3) Outcome specification**
- Primary outcome(s): e.g., weekly heat-related ED syndrome rate
- Secondary outcomes: EMS calls, all-cause ED, displacement indicators
- Negative controls: outcomes not plausibly affected

**4) Confounder strategy**
- Seasonality + weather controls; long-term trends
- Concurrent policies and events (school closures, strikes, epidemics)
- Economic shocks and mobility shifts

**5) Selection bias / policy endogeneity checks**
- Why was the intervention deployed where/when it was?
- Pre-trends assessment; placebo tests
- Use of synthetic controls / matched controls to address targeted rollout

**6) Measurement error handling**
- Exposure measurement uncertainty (sensor coverage, model error)
- Geocoding error (postal code vs neighborhood)
- Use errors-in-variables sensitivity; report impact of misclassification

**7) Multi-intervention environments**
- Build an “intervention registry” and encode overlaps
- Use stacked DiD / event studies; hierarchical models where appropriate

**8) Spillovers / displacement**
- Define spillover zones (adjacent neighborhoods, commuting corridors)
- Measure potential displacement outcomes (neighboring ED demand)

**9) Uncertainty and limitations**
- Report effect sizes with intervals; include robustness results
- State where identification assumptions may fail

**10) Decision interpretation**
- What decision does this support (scale up, modify, stop)?
- Equity implications: who benefits/harms

### 4.3 Common failure modes and mitigations
- **Ecological fallacy:** do not infer individual risk from area-level aggregates; enforce narrative rules (§6).
- **Multiple comparisons:** pre-register primary outcomes and time windows.
- **Data revisions:** lock analysis datasets with versioned snapshots; document revisions.

---

## 5) Ground truth, validation, and model governance

### 5.1 Backtesting against historical events (structured)
Create a “historical events library”:
- heatwaves, wildfire smoke events, floods
- major mobility disruptions
- known outbreaks (where public information exists)

For each event, define:
- expected exposure pattern
- expected syndromic signal pattern (direction + lag)
- acceptable detection delay (if alerting)

Run backtests:
- exposure model: compare against monitoring stations; track bias by area
- syndromic models: retrospective detection performance
- evaluation models: replicate known intervention studies (if available)

### 5.2 Acceptance criteria (by use case)
- Planning dashboards: stability + disclosure compliance + representativeness
- Evaluation outputs: minimum robustness checks, sensitivity bounds, peer review
- Alerts: minimum precision proxy + time-to-verify; false alarm rate targets

### 5.3 Model lifecycle governance
Define and enforce:
- **Model registry**: version, owner, training data snapshot, intended use, limitations
- **Approval gates:**
  - Tier A: internal analytics lead approval
  - Tier B: cross-functional review (public health + privacy + methods reviewer)
  - Tier C: external/independent review + executive sign-off
- **Monitoring:** drift in coding, geography coverage, exposure model errors, calibration
- **Rollback:** revert to last approved version if drift thresholds exceeded or partner feed changes

---

## 6) Ethics, communications, and community recourse

### 6.1 Safe publishing scales and narrative framing
Rules:
- Publish at the **coarsest scale that supports action**.
- Avoid “ranking neighborhoods by sickness.” Prefer framing around **exposure reduction opportunities** and **service access improvements**.
- Always pair maps with:
  - uncertainty notes
  - disclosure rules statement
  - “what this does NOT mean” (no individual inference)

### 6.2 Stigma avoidance checklist
- Do not publish intersectional slices that isolate small communities.
- Use strengths-based language; avoid blame narratives.
- Ensure community review for any public product about a specific area.

### 6.3 Community feedback / contestation workflow
1) Publish a clear channel for feedback and contestation.
2) Triage within 5 business days: data error, framing concern, methodological challenge.
3) Methods review panel responds with:
   - what can change (data correction, aggregation level, narrative)
   - what cannot (privacy constraints)
4) Log outcomes and publish a quarterly “changes made” summary.

### 6.4 Transparency reporting template (health analytics)
Include quarterly:
- datasets used + partner list (high level)
- products released (internal/public)
- suppression rates and any violations
- DP ε consumption (if used)
- evaluation studies conducted and their decision use
- alerts triggered and verification outcomes

---

## 7) Public health operations integration (alerts)

### 7.1 Staged alerting posture (default)
- **Stage 0: Monitor** (no alert): routine monitoring; anomalies logged.
- **Stage 1: Signal** (internal): anomaly detected; requires verification.
- **Stage 2: Advisory** (cross-agency): verified elevated risk; prepare resources.
- **Stage 3: Warning / Response** (public-facing): after approval; coordinated comms.

### 7.2 Thresholds (examples; calibrate locally)
Use syndrome-specific, seasonally adjusted thresholds:
- statistical control charts / baseline deviations
- persistence (e.g., 2 of 3 days above threshold) to reduce false alarms
- geographic coherence checks (not one small cell)

### 7.3 Verification workflow
Before escalation:
- confirm data pipeline health and coding changes
- compare with exposure context (heat/air quality)
- consult partners (hospital/EMS) for plausibility
- document “what would change our assessment”

### 7.4 RACI (alerts)
- **Responsible:** Public health duty officer / surveillance lead
- **Accountable:** Public health director (or delegated incident lead)
- **Consulted:** Hospitals/EMS liaison, privacy officer, comms/PIO, emergency management
- **Informed:** Mayor/city leadership per policy, relevant department ops leads

### 7.5 False alarm handling and trust repair
- After-action review within 10 business days
- Publish internal lessons learned; adjust thresholds
- If public messaging occurred, issue clarification with transparency about uncertainty

---

## 8) Operational runbooks (minimum set)

### 8.1 Privacy incident / unintended disclosure response
- stop dissemination; revoke access
- incident triage: what was released, to whom, and why
- notify privacy office + legal + partners per DSA
- remediation: delete/replace products; update suppression rules; retrain staff

### 8.2 Contested analysis claim review process
- log the claim; freeze the disputed output
- methods review panel reviews within 30 days
- publish correction/clarification internally (and publicly if needed)

### 8.3 Data feed disruption from a hospital/partner
- detect via freshness SLIs; notify partner
- switch to degraded mode (coarser aggregates; confidence flags)
- record outage and backfill policy; update conformance report

### 8.4 Outbreak / health-alert escalation workflow
- Stage 1 signal created → verification checklist
- if verified: Stage 2 advisory routed to emergency management
- if escalation needed: approvals + comms plan + monitoring plan

---

## 9) Key metrics

### 9.1 Data freshness / coverage / representativeness
- % of expected partner feeds received on time
- coverage of population by geography (known gaps)
- representativeness indicators (e.g., facility mix changes, coding drift)

### 9.2 Disclosure-control compliance
- suppression rate by product
- count of policy violations (target: 0)
- DP ε consumption (if applicable)

### 9.3 Model performance and calibration
- exposure model RMSE/bias vs monitors
- calibration curves for surge forecasts (if used)
- drift indicators (coding changes, geography reassignments)

### 9.4 Alerts quality
- time-to-verify
- precision proxy (verified / triggered)
- false positive rate by syndrome family

### 9.5 Equity distribution metrics
- exposure burden by vulnerability deciles
- access-to-service time distributions
- intervention benefit incidence by group (where allowed)

---

## 10) Implementation roadmap

### 0–3 months (foundation)
- finalize MVDS and DSAs
- define aggregation lenses (Gx/Tx) and suppression rules; implement automated checks
- build internal dashboards (weekly) + data conformance reporting

### 3–12 months (evaluation capability)
- implement intervention registry + tiered evaluation playbook
- create model registry + approval gates + backtesting library
- pilot 2–3 causal studies with external peer review

### 12–24 months (mature operations; optional alerting)
- if chosen: implement staged alerting workflow, thresholds, and after-action loop
- automate disclosure checks and join governance controls
- scale to cross-agency federation and regional compacts

---

## References (retrieved via Firecrawl MCP)

### Public health data standards / interoperability
- Public Health FHIR Playbook (CDC), July 2023 — https://www.cdc.gov/data-interoperability/media/pdfs/PHFIC_Public-Health-FHIR-Playbook.pdf
  - Takeaway: Practical guidance for public health agencies to exchange data using FHIR with realistic patterns (public health as client/server, common exchange scenarios).
- About FHIR (ONC eCQI resource) — https://ecqi.healthit.gov/fhir/about
  - Takeaway: High-level description of HL7 FHIR as a standard for electronic health information exchange and how it is used across the ecosystem.

### Privacy-preserving record linkage / disclosure control
- Privacy Preserving Record Linkage (PPRL) Strategy and … (NIH/NIA), preliminary report — https://www.nia.nih.gov/sites/default/files/2023-08/pprl-linkage-strategies-preliminary-report.pdf
  - Takeaway: Describes practical components and architectures for PPRL, including Bloom-filter-based encoding and roles for data owners vs linkage units.
- Privacy Protecting Record Linkage (PPRL) (CDC Foundation) — https://www.cdcfoundation.org/CDCFoundationPPRLSummary.pdf?inline
  - Takeaway: Overview of how PPRL enables linkage for public health analytics without exposing direct identifiers, including governance considerations.
- Privacy-preserving record linkage in the context of a National Statistics Institute (UK Government) — https://www.gov.uk/government/publications/joined-up-data-in-government-the-future-of-data-linking-methods/privacy-preserving-record-linkage-in-the-context-of-a-national-statistics-institute
  - Takeaway: Government-oriented discussion of PPRL concepts and matching encoded identifiers under privacy constraints.

### Causal inference / evaluation playbooks
- Synthetic control methodology as a tool for evaluating population-level health interventions (BMJ J Epidemiol Community Health) — https://jech.bmj.com/content/72/8/673
  - Takeaway: Explains why synthetic control is useful when randomized designs are infeasible and provides a population health evaluation framing.
- A comparison of methods for health policy evaluation with controlled interrupted time series (NIH PMC) — https://pmc.ncbi.nlm.nih.gov/articles/PMC7080394/
  - Takeaway: Compares DiD, synthetic control, and related approaches in health policy evaluation, highlighting practical differences and pitfalls.

### Syndromic surveillance / escalation norms
- Overview of Syndromic Surveillance (CDC MMWR) — https://www.cdc.gov/mmwr/preview/mmwrhtml/su5301a3.htm
  - Takeaway: Defines syndromic surveillance, typical uses (early outbreak detection, trend monitoring), and positioning as a complement to traditional surveillance.
- Implementing Syndromic Surveillance: A Practical Guide Informed by Experience (NIH PMC) — https://pmc.ncbi.nlm.nih.gov/articles/PMC353021/
  - Takeaway: Practical implementation considerations for syndromic systems, including alert investigation and operational realities.
