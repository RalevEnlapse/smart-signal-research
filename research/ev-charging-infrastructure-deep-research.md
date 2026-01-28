# EV charging infrastructure — Deep research

## Executive summary

EV charging is a city-scale, cyber-physical service spanning **curb/land use**, **grid constraints**, **payments/identity**, and **equitable mobility outcomes**. A City Digital Twin can make this decision-grade by turning scattered operator and utility signals into governed decisions:

- where to invest/regulate (siting portfolios, curb policy, concession terms)
- how to coordinate with utilities under limited topology visibility
- how to measure equity and reliability transparently
- how to enforce interoperability and safe data sharing

This revision upgrades the document from a generic reference architecture into an operational playbook:

- Siting optimization objectives/constraints and publishable rationales.
- Grid constraint integration patterns by visibility level (limited/partial/high) including hosting capacity linkage and uncertainty labeling.
- Reliability operations: fault taxonomy, uptime/MTTR SLOs by charger class, remote diagnostics and spares logistics.
- Interoperability + roaming governance with contract patterns and dispute resolution.
- Privacy rules for usage/location data (minimize-by-default; aggregated public reporting).
- Pricing/policy levers with equity guardrails and auditing.

## 0. Scope, decisions, and governance boundaries

### 0.1 City roles under a mixed ownership model

Typical city roles (often simultaneously):

- **Planner/regulator**: sets siting rules, curb/parking policy, accessibility requirements, reporting requirements.
- **Procurer/partner**: funds or co-funds sites; sets concession terms.
- **Data governor**: defines what is reported, how it is aggregated, and what is publishable.
- **Operator (sometimes)**: runs city-owned chargers or manages a contracted operator.

### 0.2 What the twin can recommend vs what utilities/operators control

- City/twin can recommend: portfolio siting, curb policy, incentives, reporting, performance enforcement, equity guardrails.
- Utilities control: interconnection approvals, feeder/transformer upgrades, protection settings, OT operations.
- Operators control: uptime operations, field maintenance, customer support, payment processing.

Rule: the twin is **advisory-by-default** and must label grid-derived outputs as screening/planning vs engineering-grade.

## 1. Siting optimization: objectives, constraints, and transparency

### 1.1 Objective function menu (what you optimize)

Use multi-objective optimization; publish weights internally.

Common objectives:

- **Coverage**: maximize population/jobs within X minutes of a charger.
- **Queueing delay**: minimize expected wait time (peak + events).
- **Utilization**: avoid stranded assets while ensuring redundancy.
- **Grid upgrade cost/lead time**: minimize long-lead interconnection blockers.
- **Equity**: reduce access gaps for underserved neighborhoods.
- **Emissions**: encourage off-peak charging or near low-carbon supply windows.
- **Accessibility**: maximize ADA-compliant access and safe pedestrian approach.

### 1.2 Hard constraints (what you cannot violate)

- curb availability and competing uses (loading zones, transit, emergency access)
- ADA accessibility (stall design, access aisle, route, reach ranges)
- permitting and right-of-way constraints
- safety/security (lighting, sightlines)
- zoning/land use restrictions
- minimum power availability and interconnection feasibility window
- procurement constraints (labor, vendor qualification)

### 1.3 Decision templates

**A) Annual deployment plan (portfolio)**

- target coverage metrics by neighborhood
- charger mix (L2 vs DCFC) and redundancy policy
- candidate sites list with grid feasibility class
- equity impact statement
- budget and procurement plan

**B) Corridor retrofit (DCFC)**

- travel demand + heavy vehicle considerations
- queueing model and redundancy
- interconnection pathway and lead time
- operations plan and uptime SLOs

### 1.4 Transparency requirement

For each site considered, maintain an internal “decision card”:

- objectives and weights used
- constraints triggered
- why selected/declined
- grid feasibility class and uncertainty
- equity/accessibility score impacts

Public transparency (dual internal+public): publish aggregated rationale without exposing sensitive grid details.

## 2. Equity metrics and safeguards (high focus)

### 2.1 Recommended equity metric set

Compute by neighborhood (and for key populations when possible):

- **time-to-access**: % households within 5/10/15 minutes walk/drive of public charging
- **charger density**: ports per 10,000 residents and per km²
- **reliability-adjusted access**: access weighted by uptime (a broken charger doesn’t count)
- **affordability**: effective price per kWh vs income proxies; availability of discounts
- **ADA accessibility coverage**: % sites meeting accessibility checklist
- **housing type proxy**: multi-unit dwellings share vs charging availability

### 2.2 Bias pitfalls and mitigations

Pitfall: forecasting demand from current EV ownership reinforces inequity.

Mitigations:

- scenario weighting to reflect policy adoption goals
- incorporate outreach inputs and planned housing/land-use changes
- set minimum service floors (coverage targets) independent of current demand

### 2.3 Equity safeguards in decisions

- minimum allocation of investments to underserved zones
- require ADA compliance as a gate
- require pricing guardrails and discount programs where affordability is a barrier

## 3. Grid constraints integration depth (grid_visibility_level = limited)

### 3.1 Concepts to include even with limited visibility

- distribution constraints are often **transformer/feeder** bound, not just “city peak.”
- hosting capacity is location-specific and time-varying.

### 3.2 Patterns by visibility level

**Limited visibility (default)**

- use proxy constraints (historical peak by zone, substation capacity signals)
- apply conservative buffers
- utility approval gate before commitment
- label outputs as **screening** (not engineering truth)

**Partial visibility**

- incorporate feeder/transformer constraints where available
- integrate hosting capacity zones from utility
- enable managed charging program pilots

**High visibility**

- transformer/feeder hosting capacity and upgrade triggers
- evaluate managed charging and storage as non-wires alternatives

### 3.3 Uncertainty communication

Every grid-feasibility output must include:

- what data was available
- confidence level
- what would change the assessment (e.g., transformer ratings, protection constraints)

## 4. Operational reliability and maintenance

### 4.1 Fault taxonomy (minimum)

Classify failures consistently across vendors:

- power supply / breaker trip
- comms loss (cellular, backhaul)
- payment/authorization failure
- connector failure / latch
- thermal derate
- software/firmware crash
- vandalism/physical damage

### 4.2 SLOs by charger class

Example targets (calibrate to policy and contracts):

- DCFC: uptime ≥ 97% per port (annual average); MTTR targets in hours–days
- L2: uptime ≥ 95–97%; MTTR targets days
- first-time-fix rate target and spare parts availability metrics

### 4.3 Remote diagnostics and escalation flow

- operator NOC detects fault → remote reset/diagnostic
- if unresolved → dispatch field tech with correct parts
- if upstream grid issue → coordinate with utility

### 4.4 Multi-vendor maintenance realities

- require common telemetry schema from all operators
- maintain a shared spare-part catalog and lead times
- standardize ticket severities and escalation

## 5. Interoperability + roaming governance

### 5.1 Interoperability requirements

- OCPP version expectations and upgrade policy (prefer modern versions where feasible)
- ISO 15118 considerations for Plug & Charge where supported
- backward compatibility and migration windows

### 5.2 Roaming policy (multi-network required)

Define minimums:

- roaming support via an open roaming interface (e.g., OCPI)
- settlement and dispute process
- customer support handoff rules
- minimum data sharing for authorization/session records (privacy-safe)

### 5.3 Dispute resolution

- tier 1: operator-to-operator settlement
- tier 2: city arbitration for concession-funded assets
- tier 3: regulator/legal if systemic

## 6. Data governance, privacy, and publication (minimize-by-default)

### 6.1 What the city collects vs what operators retain

City collects (default):

- station metadata (location, connector types, power)
- availability/uptime by port
- aggregated session counts and energy delivered by time bucket
- fault events (taxonomy) without user identifiers

Operators retain:

- customer identity/payment data
- detailed session-level traces unless explicitly contracted

### 6.2 Retention and aggregation rules

- city retains raw operational aggregates for limited windows, then rolls up
- public reporting uses aggregation + anonymization; suppress small counts
- consider geospatial coarsening for low-volume sites

### 6.3 Prohibitions

- no re-identification attempts
- no secondary use of charging traces for enforcement/profiling

## 7. Pricing and policy levers with guardrails

### 7.1 Policy-aligned pricing goals

- encourage off-peak charging
- manage curb congestion and turnover
- reduce queueing at hotspots

### 7.2 Equity guardrails

- price caps or discounts for low-income programs
- prohibit discriminatory outcomes; audit disparate impacts
- transparency: publish pricing rules and discount eligibility

### 7.3 Managed charging linkage

- integrate managed charging via partners (aggregators/utility programs)
- opt-in controls and override for drivers

## 8. Operational runbooks (minimum set)

- charger outage surge and communications
- suspected payment fraud / cyber incident
- roaming dispute escalation
- utility constraint conflict (site desirable but upgrade long-lead)
- public reporting cycle and privacy review

## 9. Key metrics

- access/coverage (time-to-access, reliability-adjusted access)
- utilization and queueing (wait time proxies)
- reliability (uptime, MTTR, derate time, first-time-fix)
- grid impact (peak coincidence, managed charging participation)
- equity and affordability
- privacy/compliance (publishability checks passed)

## 10. Implementation roadmap

### 0–3 months

- inventory chargers and data-sharing agreements
- baseline reliability dashboard
- siting rubric and decision card template

### 3–12 months

- reliability ops model + fault taxonomy enforcement
- interoperability and roaming requirements for procurements
- pilot siting optimization with equity reporting

### 12–24 months

- grid integration maturity and managed charging partnerships
- portfolio optimization (siting + pricing + reliability) with governance

## 11. References

### 11.1 Workspace source

- Item 35 in [`kali-task-research.md`](../kali-task-research.md:1)

### 11.2 External references (retrieved via Firecrawl MCP)

- Open Charge Alliance. *OCPP (Open Charge Point Protocol).* https://openchargealliance.org/protocols/open-charge-point-protocol/ — OCPP aims to standardize communication between charge points and central systems to improve interoperability.
- Open Charge Alliance. *OCPP 2.0.1 (overview).* https://openchargealliance.org/protocols/ocpp-protocols/ocpp-2-0-1/ — OCPP 2.0.1 is positioned as the successor to OCPP 1.6 and supports modern CSMS–charger capabilities.
- FHWA. *23 CFR Part 680 — National Electric Vehicle Infrastructure (NEVI) Formula Program.* https://www.ecfr.gov/current/title-23/chapter-I/subchapter-G/part-680 — Establishes minimum standards including a >97% average annual uptime requirement per charging port and public data aggregation/anonymization expectations.
- Federal Register. *National Electric Vehicle Infrastructure Standards and Requirements (Final Rule).* https://www.federalregister.gov/documents/2023/02/28/2023-03500/national-electric-vehicle-infrastructure-standards-and-requirements — Final rule describing minimum standards for NEVI-funded infrastructure including uptime requirements.
- NREL. *Embedding Equity into Electric Vehicle Charging Station Design* (PDF). https://docs.nrel.gov/docs/fy24osti/90147.pdf — Discusses design and program choices that affect equitable access and affordability of charging.
- EV Roaming Foundation. *OCPI.* https://evroaming.org/ocpi/ — Describes OCPI as an interface supporting scalable roaming connections between eMobility service providers and charge point operators.
