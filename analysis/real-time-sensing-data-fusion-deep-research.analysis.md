# Analysis — Real-time sensing & data fusion

Source: [`real-time-sensing-data-fusion-deep-research.md`](../themes/real-time-sensing-data-fusion-deep-research.md:1)

## Relevance
Real-time sensing and data fusion is a foundational capability for operational city digital twins. Many downstream themes (mobility operations, public safety, disaster management, air quality, noise, infrastructure monitoring) depend on having a coherent, time-aligned “city state” with provenance, confidence, and data quality indicators. Without this, the twin becomes a dashboard of disconnected feeds rather than a decision-support system.

## Depth / maturity
This is one of the more technically substantial documents:
- Clearly defines the real problem: time/space alignment, semantic consistency, and trust under noise/missingness/delay/conflict.
- Provides practical conceptual primitives (stream vs event vs state; entity resolution; fusion spectrum).
- Includes a credible threat model and concrete abuse cases (spoofing, replay, poisoning, DoS).
- Goes beyond naming components: it outlines step-by-step build sequence (schema, timestamp strategy, validation/quarantine, enrichment, entity resolution, quality scoring, fusion, serving with provenance).
- Covers key implementation concerns (event-time semantics, late data, deduplication, replay/backfill, spatial indexing) and common pitfalls.

It is still “architecture blueprint” rather than an implementation specification; it doesn’t commit to specific designs for identity across domains or to a standard provenance model schema.

## Gaps / blind spots
1) **Canonical ontology and entity registry governance**
It stresses canonical IDs but doesn’t specify:
- governance process for identity resolution (who decides mappings?)
- how to handle merges/splits over time and downstream impact
- standard entity types and event taxonomies across domains

2) **Quality scoring calibration and validation**
Quality scores are proposed, but missing:
- how to calibrate quality metrics against ground truth
- how quality scores propagate into fused confidence

3) **Serving semantics and consistency guarantees**
It lists state store and event store, but missing:
- how consumers reason about “current state” freshness and staleness
- consistency model (eventual vs strong) and implications for operations

4) **Privacy boundaries and minimization specifics**
It mentions privacy-preserving fusion, but lacks concrete rules:
- what CCTV-derived outputs are allowed (metadata vs detections vs tracks)
- minimization and retention for citizen reports
- safe publishing practices for fused outputs (avoid inference)

5) **Operational ownership and on-call boundaries**
It mentions stewardship, but not:
- who is on-call for which pipeline segment
- how to route incidents (data source vs platform vs fusion logic)

6) **Testing strategy**
It includes metrics but not detailed test harness approaches:
- replay-based regression tests with known incidents
- synthetic data for adversarial tests and late-data scenarios

## Latent research needs (only as implied by the gaps)
- **Ontology + identity governance for city twins**: canonical entity/event schemas, controlled evolution, and mapping stewardship.
- **Confidence/quality calibration**: methods to validate fused confidence and avoid overconfidence.
- **Privacy-by-design fusion contracts**: enforceable constraints on sensitive source usage and publication.

## Contribution to collective sensemaking
This topic is a key integrator: it explains why a twin is not “many data feeds” but a trustable state system with provenance and confidence. It also provides the conceptual bridge between infrastructure (brokers, stream processing) and governance (auditability, access control), making it valuable for aligning engineering and policy stakeholders.

Its biggest contribution is giving the portfolio a shared technical language for “state,” “events,” and “trust,” which reduces confusion across domain-specific twins.