# Analysis — Urban planning

Source: [`urban-planning-deep-research.md`](../themes/urban-planning-deep-research.md:1)

## Relevance
Urban planning is one of the highest-leverage digital twin use cases because planning decisions:
- have long lifetimes and lock in future costs
- cut across mobility, housing, energy, water, and climate resilience
- require defensible public narratives and auditability

A planning “twin” is also a natural hub for scenario-based decision making, where the value of a twin is not real-time control but **reproducible alternatives analysis**.

## Depth / maturity
This document is notably more mature than many other domain briefs:
- It explicitly defines the twin as a scenario framework with governance and reproducibility as the core success factor.
- Includes a thoughtful threat model (scenario gaming, black-box misuse, equity washing).
- Proposes a layered architecture (scenario registry, baseline data, domain model adapters, evaluation harness, engagement layer) that is realistic for incremental adoption.
- Names relevant standards (OGC CityGML, ISO 19152 LADM, GTFS, NGSI‑LD) and includes uncertainty/sensitivity methods.

Overall, it reads like a strong conceptual blueprint aligned with real planning practice.

## Gaps / blind spots
1) **Model calibration and institutional capacity**
It mentions validation and backtesting, but missing:
- a practical calibration plan (what data is required, who maintains it)
- capacity needs (specialist modelers vs generalist analysts)

2) **Handling contested values and political negotiation**
Planning decisions are not purely technical. Missing:
- processes for representing conflicting objectives transparently
- governance for who sets weights and thresholds in multi-objective comparisons

3) **Data access constraints and sensitive information**
Confidential development pipelines and critical infrastructure are noted, but missing:
- concrete access tiering designs and publishing strategy
- redaction patterns for public engagement outputs

4) **Coupling strategy and computational practicality**
Cross-domain integration is central, but missing:
- guidance on loose coupling vs tight co-simulation
- computational cost management and “fast approximate” models vs detailed ones

5) **Equity metrics specificity**
The document mandates distributional reporting, but missing:
- recommended baseline equity metrics and segmentation choices
- how to avoid re-identification when slicing neighborhood/population data

6) **Change management into statutory planning processes**
Adoption is mentioned, but missing:
- how outputs map to statutory steps (public hearings, environmental review)
- how to prevent “model outputs as authority” (automation bias) in public decision making

## Latent research needs (only as implied by the gaps)
- **Operational playbooks for planning twins**: staffing, maintenance, and calibration approaches that are feasible for municipal teams.
- **Governance of multi-objective decisions**: transparent methods for trade-off setting and documenting value judgments.
- **Privacy-safe equity slicing**: techniques for distributional analysis with robust disclosure control.

## Contribution to collective sensemaking
This topic provides a crisp articulation of what a city digital twin can be when the goal is better decisions rather than real-time automation. It strengthens collective sensemaking by:
- emphasizing reproducibility and governance as first-class design goals
- naming specific misuse modes (scenario gaming, equity washing)
- offering a modular architecture that can unify multiple other themes (mobility, energy, water, climate risk)

It is a strong anchor document for aligning technical work (data pipelines, model validation, observability) with planning legitimacy and public accountability.