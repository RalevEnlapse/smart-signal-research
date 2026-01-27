# Analysis — Stage-gate governance

Source: [`stage-gate-governance-deep-research.md`](../themes/stage-gate-governance-deep-research.md:1)

## Relevance
Stage-gate governance is highly relevant to city digital twin programs because these programs are typically:
- cross-departmental (shared data/platform)
- long-lived (multi-year)
- politically visible
- susceptible to scope creep and “pilot forever” dynamics

A stage-gate approach can improve selection discipline, kill failing initiatives early, and connect delivery to benefits realization and risk management.

## Depth / maturity
This is one of the more mature documents in the set:
- Strong articulation of why stage-gate is valuable in municipal contexts.
- Concrete threat model for governance manipulation (gate shopping, criteria relaxation, sunk-cost).
- Provides detailed stages, gate criteria categories, decision outcomes, artifacts, SLIs/SLOs.
- Includes cost ranges and operationalization (PMO, committees, training).

It reads like an implementable governance playbook, not just a thematic overview.

## Gaps / blind spots
1) **Over-bureaucratization risk and agility mechanics**
The document mentions “agile stage-gate,” but lacks:
- explicit proportionality rules (how rigor scales by risk, budget, or data sensitivity)
- patterns for iterative discovery (spikes, experiments) without heavy gate overhead

2) **Integration with data/AI-specific governance**
For digital twins, many risks are data/model-related. Missing:
- explicit gates for data readiness (quality, lineage, privacy) and model validation
- criteria for operational safety, human-in-the-loop, and rollback

3) **Public accountability and transparency specifics**
It references public sector accountability but doesn’t specify:
- what gets published vs kept internal
- how to handle FOI/public-records requests for gate materials

4) **Decision rights and political constraints**
Decision rights are described, but missing:
- handling of elected officials’ influence and urgency overrides
- conflict-of-interest rules for vendors/partners in gate reviews

5) **Outcome measurement loop**
KPIs include “decision accuracy” and portfolio ROI, but missing:
- how accuracy is adjudicated (time horizon, counterfactuals)
- how to prevent metrics gaming (e.g., redefining success at Gate 6)

6) **Tooling vs process**
There’s a “platform” diagram, but missing:
- minimal viable process without new tooling
- integration patterns with existing systems (Jira/MS Project/SharePoint) in practical terms

## Latent research needs (only as implied by the gaps)
- **Risk-tiered gate design**: lightweight governance for experiments vs heavy governance for high-risk production systems.
- **Data/AI-aware gate criteria**: standardized checklists for data readiness, privacy, model validation, and operational safeguards.
- **Transparency patterns**: practical guidance on publishing decisions and rationales without exposing sensitive details.

## Contribution to collective sensemaking
This document provides a governance backbone that complements domain-specific “twin” themes. It helps the collective move from “ideas” to an operating model that can survive political cycles.

Its strongest contribution is naming and mitigating governance failure modes (sunk cost, gaming, pressure) and providing measurable process SLIs/SLOs.

To improve collective sensemaking further, it should explicitly integrate with the technical governance topics already in the corpus—e.g., data quality SLAs, risk registers, model validation, observability—by turning them into gate artifacts and decision criteria so governance is not abstract but directly coupled to technical assurance.