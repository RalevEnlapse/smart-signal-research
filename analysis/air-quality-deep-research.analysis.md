# Analysis — Air quality

Source: [`air-quality-deep-research.md`](../themes/air-quality-deep-research.md:1)

## Relevance
This topic is highly relevant to city digital twins because air quality is a cross-domain outcome that depends on transportation, land use, energy/heating, industry, meteorology, and public health. A twin that can translate multi-source measurements and models into intervention guidance (e.g., low-emission zones, fleet electrification, heating upgrades) is a strong “value demonstration” capability and a compelling public-facing use case. The document also correctly frames “fitness-for-purpose” (advisory/planning vs regulatory-grade) as central, which is critical for public-sector adoption.

## Depth / maturity
Overall maturity is strong for a “deep research” note:
- Provides a clear layered architecture (ingest → QA/QC → emissions → dispersion/chemistry → assimilation → exposure/health impacts → scenario engine → serving + governance).
- Includes a threat/abuse section (sensor tampering, cherry-picked baselines, misuse as regulatory evidence) and mitigations.
- Includes operational considerations (phased rollout, validation methods, SLOs/SLIs, costs/FinOps, KPIs).
- References canonical external anchors (EPA dispersion model guidance; a review of exposure modeling).

Where maturity is moderate rather than advanced:
- “Dispersion/chemistry” is described at a high level without selecting concrete model classes for common city constraints (street canyon vs neighborhood vs regional background).
- “Assimilation/fusion” is named but not decomposed into pragmatic options by data regime (few reference monitors vs dense low-cost sensors; high-latency satellite vs real-time ground sensors).
- Exposure and health impact modeling is included but not tied to a decision process (e.g., which policy questions require DALYs vs simpler exposure metrics).

## Gaps / blind spots
Key gaps that would matter in implementation and credibility:

1) **Governance boundary between advisory and regulatory**
The document warns about misuse but does not specify governance mechanics for preventing it:
- explicit labeling taxonomy (e.g., “informational”, “planning-grade”, “policy evaluation-grade”) and required disclaimers
- approval gates for publishing scenario results
- model version pinning for public releases

2) **Model choice and resolution strategy**
“Multi-scale” is stated, but the design choice is not resolved:
- When do you need a street-canyon model vs a land-use regression vs a hybrid ML approach?
- How do you handle boundary conditions (regional background) and transboundary pollution?
- What is the acceptable error by use case (alerts vs annual planning)?

3) **Low-cost sensor network realities**
Calibration and drift are mentioned, but missing practicalities:
- co-location strategy (how many reference co-locations; rotating calibration)
- metadata requirements (sensor model, inlet height, micro-siting)
- governance of community-deployed sensors and bias/representativeness

4) **Causal evaluation of interventions**
The plan mentions diff-in-diff/synthetic controls but does not define:
- minimum data requirements to credibly estimate causal impacts
- confounder handling (weather, seasonality, background trends)
- pre-registration or “analysis plan” discipline to reduce cherry-picking

5) **Uncertainty communication UX**
Uncertainty is emphasized, but the “how” is absent:
- how uncertainty is computed/propagated across emissions → dispersion → assimilation → exposure
- how it is presented to operators vs the public
- how decision thresholds are set given uncertainty (e.g., trigger advisories)

6) **Data architecture specifics**
There is no concrete data model/contract for observations and model fields:
- spatial indexing strategy (tiles/hexes) and temporal resolution standards
- provenance/lineage fields needed for auditability
- interoperability patterns (e.g., how OGC SensorThings/NGSI-LD map to internal canonical schema)

## Latent research needs (only as implied by the gaps)
- **Fitness-for-purpose governance patterns**: lightweight but enforceable mechanisms to prevent “planning-grade” outputs being used as compliance evidence; operational playbooks for escalation and independent review.
- **Fusion strategy under heterogeneous sensor quality**: comparative evaluation (not just listing) of Bayesian hierarchical fusion vs EnKF vs ML bias-correction across realistic city deployments.
- **Uncertainty-aware decision support**: mapping from probabilistic concentration/exposure estimates to policy actions (risk thresholds, equity constraints) and communicating these transparently.

## Contribution to collective sensemaking
This document contributes a well-structured “system view” of an air-quality twin: it ties technical components (models, sensors, assimilation) to operational needs (phased rollout, validation, SLOs, risk/abuse). It also introduces a governance framing (“fitness-for-purpose”) that generalizes to other sensitive domains in the portfolio (public safety, health, compliance). As a shared reference, it is useful for aligning environmental, transport, and platform teams around a coherent pipeline and for clarifying where credibility is won or lost (QA/QC, validation, uncertainty, and causal evaluation).