# Cost and FinOps governance — Deep research (twin-specific, enforceable)

## Executive summary
A City Digital Twin program has cost drivers that don’t look like a typical SaaS app: **continuous sensor ingestion**, **geospatial ETL/tiling**, **bursty scenario runs**, **high-egress public dashboards**, and **shared multi-agency platforms**.

This document defines an **actionable FinOps operating system** for a municipal digital twin with:
- **Twin-specific unit economics** (sensor-day, tile-km² pipeline run, scenario run, API call, dashboard active user-day, GB-month by storage/compute tier).
- **Governance tied to benefits evidence** so scaling decisions are gated by demonstrated value (not just spend tolerance).
- **Practical tagging/ownership enforcement** (admission controls, exception workflows, shared-cost allocation, vendor/contract allocation).
- **Egress and cross-agency sharing controls** (CDN/caching, rate limiting, budgeting by consumer type).
- **Public-sector budgeting/procurement integration** (appropriation cycles, restricted funds, lead times, contract transparency).
- A **FinOps + Security joint operating model** for spend spikes (attack vs legitimate surge) with runbooks and thresholds.

This document deepens item 24 in [`kali-task-research.md`](kali-task-research.md:1): “Cost and FinOps governance: Manage cloud and platform costs with transparency, optimization, and budget controls.”

---

## Defaults (unless overridden)
- `deployment_context`: hybrid (cloud + on-prem) + vendors/partners
- `showback_mode`: showback-first
- `budget_constraints`: public-sector-annual
- `cost_transparency_level`: high
- `risk_posture`: guardrails-first

---

## 1) Twin-specific unit economics catalog (concrete)
### 1.1 Unit-cost catalog (recommended)
These are the “right” cost units for a city twin. Each unit must be computable from tags + run identifiers.

| Unit cost | Definition | Typical levers | Required metadata to compute |
|---|---|---|---|
| **Cost per ingested sensor-day** | Total ingest+store+basic queries for one sensor for one day (by sensor class/frequency) | sampling rate, retention tier, compression, edge filtering | `sensor_id`, `sensor_class`, `freq_hz`, `dataset_id`, `retention_tier`, `pipeline_run_id`, `env`, `funding_source` |
| **Cost per GB-month by tier** | Storage + indexing + typical query overhead per GB-month for hot/warm/cold/archive | lifecycle rules, partitioning, query caching | `dataset_id`, `storage_tier`, `index_type`, `query_class`, `env`, `owner` |
| **Cost per tile-km² pipeline run** | ETL/retiling/indexing cost per km² (or tile count) per run | tiling scheme, parallelism, incremental updates | `pipeline_id`, `pipeline_run_id`, `tile_scheme`, `area_km2`, `source_dataset_id`, `output_dataset_id` |
| **Cost per scenario run** | Compute+data IO per model run (by model class/horizon/resolution) | model complexity, ensemble size, caching, spot/queue | `scenario_id`, `scenario_version`, `model_class`, `horizon`, `resolution`, `run_id`, `owner` |
| **Cost per API call** | Marginal cost per request (compute + cache hits + DB read) | caching, payload size, auth, throttling | `api_name`, `api_version`, `consumer_type`, `client_id`, `request_bytes`, `response_bytes` |
| **Cost per dashboard active user-day** | Serving + queries + egress for one active user-day | caching/CDN, pre-aggregation, render strategy | `dashboard_id`, `tenant/agency`, `user_type`, `public/private`, `egress_bytes` |
| **Cost per GB egress** | Outbound transfer cost by consumer type (internal/partner/public) | CDN, cache TTL, data format | `consumer_type`, `dataset_id`, `api_name`, `egress_bytes`, `funding_source` |

### 1.2 “Unit economics first” rule
- No workload is allowed to scale (retention, frequency, resolution, public availability) without declaring:
  - its unit-cost metric(s)
  - its benefit KPI(s)
  - its guardrails (budget caps, rate limits)

---

## 2) Governance tied to value/benefits (anti “optimize spend, ignore outcomes”)
### 2.1 Benefit-evidence gating model
Adopt a lightweight gate aligned to benefits tracking tiers (conceptual alignment to your benefits realization discipline):

- **Gate A (Pilot / Tier 0–1 evidence)**
  - Allowed spend posture: capped pilot budgets, short retention, low frequency.
  - Requirement: define hypothesis + baseline KPIs + cost units.

- **Gate B (Scale / Tier 2 evidence)**
  - Allowed: expand to more agencies/buildings/sensors; longer retention.
  - Requirement: supported contribution evidence; reproducible metrics; showback of cost+outcomes.

- **Gate C (Institutionalize / Tier 3 evidence)**
  - Allowed: long-term commitments, reserved/committed spend, large public dashboards.
  - Requirement: high-confidence impact OR statutory mandate; independent review.

### 2.2 Portfolio view: cost + outcomes side-by-side
Every monthly portfolio review must show:
- unit costs (from §1) trend lines
- outcome KPIs (service quality, resilience, etc.)
- benefit evidence tier status
- “continue / change / stop” decision

### 2.3 Sunset / downshift rules
Trigger review when:
- <X% active usage for Y days (e.g., dashboard active user-days)
- high unit cost with low benefit evidence tier
- egress spend grows without corresponding public value

Actions:
- reduce retention tier
- downsample
- pause scenario ensembles
- archive datasets

---

## 3) Tagging, ownership, and enforcement (make it real)
### 3.1 Mandatory tagging standard (minimum)
These tags/labels are mandatory across cloud resources, pipelines, datasets, and Kubernetes workloads.

| Tag | Purpose |
|---|---|
| `owner` | accountable person/team |
| `service` | product/service identifier |
| `environment` | prod/stage/dev |
| `data_classification` | public/internal/sensitive |
| `funding_source` | appropriation/grant/cost center |
| `agency` | owning/benefiting agency |
| `vendor` | vendor/partner if applicable |
| `workload_type` | ingest/etl/api/dashboard/scenario |

### 3.2 Enforcement mechanisms (guardrails-first)
- **Admission controls** (block untagged where feasible):
  - Kubernetes: validating admission (require labels) before deploy.
  - IaC pipelines: fail builds if mandatory tags missing.
  - Cloud accounts/projects: SCP/policy equivalents to deny creation without tags (where supported).

- **Exception workflow**
  - time-boxed exceptions (e.g., 7–14 days)
  - compensating controls (manual allocation and monitoring)
  - track **tag debt**: $ spend under exception + days outstanding

### 3.3 Shared resource allocation strategy
Shared costs include data lake, network egress, observability, CI/CD.

Allocation model:
- baseline allocation by usage drivers:
  - storage GB-month by dataset
  - query CPU-seconds by workload
  - egress bytes by consumer type
- disputes resolved by FinOps Council using:
  - run manifests
  - measured usage
  - documented exceptions

### 3.4 Vendor/contract cost allocation
Contract costs are often not taggable at resource level. Require:
- vendor invoices provide line items mapped to `service` / `agency` / `funding_source`
- unit pricing where feasible (per sensor, per GB, per API)
- audit rights and usage reporting requirements

---

## 4) Egress, sharing, and public dashboard cost controls
### 4.1 Egress policy tiers
| Tier | Consumer | Defaults |
|---|---|---|
| Internal | city agencies | high cache, generous limits but monitored |
| Partner | universities/vendors | contract-defined quotas, token auth |
| Public | open dashboards/APIs | strict rate limits, CDN required |

### 4.2 CDN/caching strategy (public by default)
- public dashboards must be served behind CDN
- define cache TTL by data type:
  - static basemaps: long TTL
  - near-real-time: short TTL with pre-aggregation
- prefer precomputed tiles and aggregates over per-request heavy queries

### 4.3 API abuse and cost controls
- rate limiting per API key / IP / consumer type
- request budgeting (max response size, pagination)
- anomaly detection for:
  - egress spikes
  - high-error storms (retries)
  - scraping patterns

### 4.4 Who pays for external sharing?
- default: the benefiting program funds the egress budget line
- partner contracts must specify:
  - quota levels
  - overage handling
  - caching responsibilities

---

## 5) Public-sector budgeting and procurement integration
### 5.1 Annual appropriation mapping
- map services to annual budget lines:
  - ingestion
  - storage tiers
  - scenario compute
  - public serving/egress
- track restricted funds/grants via `funding_source` tag and reporting.

### 5.2 Forecasting under procurement lead times
- include procurement lead times in forecasting:
  - reserved/commitment purchases
  - vendor contract renewals
- maintain a “runway” report:
  - forecasted spend vs remaining appropriation
  - commitments and termination windows

### 5.3 Contract structures for transparency
Require contracts to include:
- usage and cost reporting cadence
- unit price schedules
- change-order mechanics
- audit rights

Public-sector oversight can reference GAO guidance on cost management and cloud requirements (see Sources).

---

## 6) FinOps + Security joint ops (cost-based attack posture)
### 6.1 Joint operating model
- Shared queue for cost anomalies with classification:
  1) misconfiguration
  2) product launch / legitimate surge
  3) compromised credentials / attack

### 6.2 Thresholds and paging rules (example)
- page FinOps on:
  - >20% week-over-week spend spike in prod
  - egress spike beyond public budget threshold
- page Security on:
  - spend spike + auth anomalies (new regions, unusual API keys)
  - repeated high-rate requests from few sources

### 6.3 Two-person rule (high-risk actions)
Require dual approval (FinOps + Security or Service Owner + Security) for:
- raising quotas/limits materially
- disabling rate limits
- bypassing tag enforcement
- enabling expensive regions/services

### 6.4 Evidence and audit logging
- every emergency override must produce:
  - incident ticket
  - decision record (who/why)
  - time-boxed expiry
  - postmortem with cost impact

AWS Cost Anomaly Detection is an example of anomaly tooling patterning (see Sources).

---

## 7) Operational runbooks (minimum set)
### 7.1 Runaway spend incident
1. Confirm scope (service/agency/env) via tags.
2. Identify driver (compute, storage, egress).
3. Triage: misconfig vs surge vs compromise.
4. Apply guardrails: pause jobs, reduce concurrency, tighten rate limits.
5. Document and postmortem; update unit economics.

### 7.2 Public dashboard traffic surge
1. Check CDN cache hit ratio.
2. Increase caching TTL where safe.
3. Enforce rate limits and pagination.
4. Enable “degraded mode” (coarser tiles, slower refresh).

### 7.3 Untagged resource outbreak
1. Block new untagged deploys.
2. Inventory untagged resources.
3. Assign owner and backfill tags.
4. Measure tag debt and close exceptions.

### 7.4 Budget reforecast after scope change
1. Update unit assumptions (sensor-days, scenario runs, user-days).
2. Recompute forecast and runway.
3. Decide scale gate (A/B/C) based on benefit evidence.

### 7.5 Vendor bill dispute / audit
1. Reconcile invoice line items to usage reports.
2. Validate allocation rules.
3. Escalate with audit rights; document outcome.

---

## 8) Key metrics (decision-grade)
- % spend allocated to an `owner` (coverage)
- unit costs and trends (sensor-day, GB-month by tier, scenario run, API call)
- cost anomaly MTTR and false positive rate
- egress spend by consumer type (internal/partner/public)
- tag debt ($ and days) and exception debt
- cost per benefit unit (where feasible): e.g., $ per avoided outage-minute, $ per verified energy-savings unit

---

## 9) Implementation roadmap
### 0–3 months
- publish tagging standard + owners
- showback dashboard MVP (cost + unit costs)
- anomaly alerts MVP (spend + egress)

### 3–12 months
- enforcement gates (IaC + Kubernetes admission)
- unit cost catalog operationalized and measured
- budget integration with appropriation cycles
- egress controls (CDN/rate limiting) for public products

### 12–24 months
- mature chargeback (optional)
- scale gates tied to benefit evidence
- automated optimization playbooks (right-sizing, retention downshift, caching)

---

## Sources (high-signal anchors)
- FinOps Foundation — FinOps Framework Overview. https://www.finops.org/framework/ — Defines FinOps as an operational framework and cultural practice for maximizing business value of cloud/technology with timely decision making.
- FinOps Foundation — FinOps Phases (Inform/Optimize/Operate). https://www.finops.org/framework/phases/ — Describes iterative phases used to execute FinOps capabilities.
- Kubernetes Blog — A Guide to Kubernetes Admission Controllers. https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/ — Explains admission controllers as a mechanism to enforce policy (useful for label/tag enforcement).
- AWS Documentation — Getting started with AWS Cost Anomaly Detection. https://docs.aws.amazon.com/cost-management/latest/userguide/getting-started-ad.html — Shows how anomaly monitors and alerts are configured for spend anomalies.
- U.S. GAO — Cloud Computing: Agencies Need to Address Key OMB Requirements. https://www.gao.gov/products/gao-24-106137 — Highlights government requirements for procuring and managing cloud services, reinforcing public-sector constraints on cost management.
