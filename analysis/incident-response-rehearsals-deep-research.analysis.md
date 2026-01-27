# Analysis — Incident response rehearsals

Source: [`incident-response-rehearsals-deep-research.md`](../themes/incident-response-rehearsals-deep-research.md:1)

## Relevance
Incident response rehearsals are directly relevant to operating a city digital twin as a mission-critical platform. A twin touches many incident types (cyber, infrastructure failures, disasters, public safety, public health), and rehearsals build the organizational capability to respond under stress. For a twin program, rehearsal practice is also a forcing function to validate data availability, dashboards, access controls, and cross-agency coordination.

## Depth / maturity
The document is a clear and practical program blueprint:
- Identifies realistic incident categories and common failure modes.
- Covers stakeholder roles that matter (incident commanders, on-call teams, comms, legal, external partners).
- Provides a sensible “system” for rehearsals: scenario library, simulation engine, management, capture, after-action review (AAR), tracking/reporting.
- Includes best practices: varied rehearsal types, scenario design, neutral facilitation, non-punitive learning.
- Adds metrics and SLO examples (completion rate, action closure).

It is mature as an operational playbook template but not tightly tailored to the unique risks of a city digital twin (privacy breaches, cross-agency entitlements, model failures, policy-as-code regressions).

## Gaps / blind spots
1) **Twin-specific incident types and exercises**
The list is broad, but missing twin-specific failure scenarios:
- incorrect model output driving harmful recommendations
- data pipeline corruption or drift producing misleading dashboards
- policy-as-code misconfiguration blocking legitimate access during emergencies
- re-identification incident from dataset joins or exports

2) **Integration with governance/privacy and access controls**
Rehearsals should test:
- time-bound entitlements for incident roles
- audit logging integrity and investigation workflows
- data export controls under pressure

These are not spelled out.

3) **Measurement of effectiveness**
KPIs exist, but depth is limited:
- focus is on “action items closed,” not on reduced real-world MTTR or improved decision quality.
- no guidance on “leading indicators” (e.g., alert quality, runbook coverage, access review readiness).

4) **Simulation realism vs safety**
“Simulation engine” is mentioned but not defined:
- how to run realistic drills without impacting production systems
- use of sandboxes, synthetic data, and replay

5) **Cross-agency and external partner coordination**
Utilities and vendors are listed as stakeholders, but missing:
- joint exercise agreements
- information sharing constraints
- escalation protocols across org boundaries

## Latent research needs (only as implied by the gaps)
- **Twin-specific rehearsal scenario catalog**: data integrity incidents, model governance failures, privacy events, and access-control failures.
- **Safe rehearsal infrastructure**: replay and simulation patterns that preserve realism without destabilizing production.
- **Effectiveness measurement**: tying rehearsal metrics to real incident outcomes and platform reliability.

## Contribution to collective sensemaking
This topic strengthens the overall set by emphasizing that “trustworthy digital twins” are as much operational discipline as architecture. It connects naturally to observability (what to monitor), governance/privacy (who can see what during incidents), and HITL safeguards (decision workflows under stress).

As a shared program model, it can help align agencies on a cadence of learning and accountability; adding twin-specific scenarios would make it an even more distinctive contribution.