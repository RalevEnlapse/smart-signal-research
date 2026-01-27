# Analysis — Data quality SLAs

Source: [`data-quality-slas-deep-research.md`](../themes/data-quality-slas-deep-research.md:1)

## Relevance
Data quality SLAs are foundational for a city digital twin because trust in decisions depends on measurable, monitored data reliability across heterogeneous sources (sensors, enterprise systems, external feeds). Without explicit quality expectations and accountability, downstream analytics and models become brittle, and cross-department disputes arise (“the data is wrong”). Data quality SLAs also support auditability and are a prerequisite for mature governance, especially where public-facing dashboards and operational decisions are involved.

## Depth / maturity
This document is mature as a generic SLA governance framework:
- Clear motivation and stakeholder segmentation (producers, consumers, stewards, governance).
- Realistic abuse cases (SLA gaming, selective reporting, threshold manipulation, exception abuse, backfilling fabricated values).
- Comprehensive platform architecture with flows from SLA definition to monitoring, alerting, remediation, and reporting.
- Good coverage of quality dimensions and metric types, plus phased rollout and SLIs/SLOs.

It is less mature in domain specificity: it doesn’t translate quality dimensions into concrete, domain-relevant checks for city twin data types (time series sensors, geospatial assets, event logs).

## Gaps / blind spots
1) **SLA scope boundaries and “fitness-for-purpose”**
Not all data must meet the same bar. Missing:
- tiering of datasets by criticality/use (operational control vs planning vs research)
- different SLA targets by tier (latency, completeness, accuracy)

2) **Ground truth and accuracy measurement**
Accuracy is defined, but in many city contexts there is no trusted ground truth:
- how to measure accuracy for crowdsourced reports, inferred occupancy, model outputs
- strategies like cross-source consistency, periodic audits, and sampling plans

3) **Geospatial and temporal semantics**
Digital twins rely heavily on space/time; missing:
- checks for coordinate reference system consistency, geometry validity, topology constraints
- time synchronization and clock drift handling as first-class SLA dimensions

4) **Exception lifecycle governance**
Exception abuse is acknowledged but not detailed:
- expiry rules, compensating controls, and escalation
- tracking “quality debt” and preventing permanent exceptions

5) **Remediation playbooks and ownership**
The architecture mentions remediation workflows but doesn’t define:
- standard playbooks by data source type (sensor drift, pipeline lag, schema change)
- clear RACI for resolving violations that span vendors and city departments

6) **Coupling to downstream impacts**
Quality SLAs are most powerful when linked to business impact:
- “error budgets” for data that translate into acceptable model degradation
- automatic gating (block releases/model retrains) when quality falls below thresholds

## Latent research needs (only as implied by the gaps)
- **Dataset tiering and SLA templates for city twins**: standard SLA profiles for sensor streams, geospatial registries, and operational events.
- **Accuracy measurement under weak ground truth**: robust approaches combining audits, cross-validation across sources, and uncertainty reporting.
- **Operationalizing exceptions**: disciplined exception lifecycle management and “quality debt” tracking.

## Contribution to collective sensemaking
This document provides a shared governance vocabulary for what “good data” means and how to make it measurable, monitorable, and enforceable. It also highlights the common failure mode of metric gaming, which is important for maintaining trust. As part of collective sensemaking, it acts as an enabling layer for nearly every other theme: air quality assimilation, asset condition models, and citizen engagement analytics all depend on explicit quality contracts and remediation loops.