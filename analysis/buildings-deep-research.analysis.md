# Analysis — Buildings

Source: [`buildings-deep-research.md`](../themes/buildings-deep-research.md:1)

## Relevance
Buildings are one of the highest-leverage domains for city decarbonization, operational efficiency, and service quality (schools, hospitals, municipal offices). A buildings twin is also a pragmatic “portfolio analytics” use case: the city often directly owns/operates a subset of buildings, making interventions feasible. The document appropriately emphasizes that value comes from semantic/operational understanding (systems, points, performance) rather than photorealism.

## Depth / maturity
The topic shows good maturity and is grounded in real deployment constraints:
- Clear articulation of the main blockers: inconsistent metadata, heterogeneous BMS vendors, point naming chaos, limited comparability.
- Sound reference architecture: canonical building registry + BIM/doc repository + operational ingestion + semantic layer + analytics + action layer + governance.
- Strong threat model for a building/OT-adjacent domain (unauthorized control changes, spoofing, privacy harms).
- Practical phased plan (registry/baselines → semantic onboarding + FDD pilots → scale + retrofit pipeline → continuous optimization).
- Uses relevant standards references (BACnet, Haystack, Brick, IFC).

The depth is “program blueprint”; it does not go into concrete integration patterns or the nitty-gritty of tagging/ontology implementation.

## Gaps / blind spots
1) **Semantic layer implementation details**
Haystack/Brick are named, but missing is a pragmatic approach to:
- point/entity identity and reconciliation across BMS exports, meters, and asset inventories
- how tagging is validated and kept current as vendors change point lists
- coverage targets per building tier (what is the minimum semantic completeness required for FDD vs benchmarking)

2) **OT/IT integration and safety boundaries**
Controls mention segmentation and change workflows, but the architecture doesn’t specify:
- read-only vs write-back capabilities and when write-back is allowed
- safety constraints and rollback for control tuning proposals
- how remote access and vendor integrators are governed/audited

3) **Measurement & verification (M&V) rigor**
Savings persistence is mentioned, but not the methodology:
- which M&V options are standard (normalized baselines, metering plans)
- how uncertainty is reported and used in decision making
- how baseline resets are governed after retrofits or operational changes

4) **Occupancy/IAQ privacy nuance**
The document flags privacy but doesn’t define practical rules:
- aggregation thresholds and access tiers for occupancy proxies
- what “privacy-preserving authentication options” or consent models apply (especially for schools/hospitals)

5) **Portfolio scaling economics**
It notes BMS mapping is labor-heavy but doesn’t quantify onboarding effort drivers:
- what % of cost is semantic mapping vs connectivity vs field verification
- which building archetypes yield the best ROI first

## Latent research needs (only as implied by the gaps)
- **Repeatable onboarding playbooks and semantic QA**: methods to automate point classification/tagging with confidence scoring and human review loops.
- **Safe actuation governance**: operating models for when analytic outputs can propose/execute BMS changes, including change control, audit, and safety constraints.
- **Standardized M&V for city portfolios**: lightweight, defensible approaches to quantify savings and persistence across heterogeneous buildings.

## Contribution to collective sensemaking
This document contributes a clear “no-photorealism” stance and a practical architecture that aligns facilities, sustainability, IT/security, and vendors around the real work: normalization and semantics. It also connects building analytics to action (work orders, control tuning, retrofit prioritization), helping avoid dashboards-without-impact. As part of the broader set, it reinforces cross-cutting themes: canonical registries, governance, observability, and phased delivery.