# Asset management — Deep research (municipal, audit-ready, twin-enabled)

## Executive summary
Asset management in a city-scale digital twin context is the discipline of maintaining a **decision-ready, auditable representation of infrastructure assets** (what they are, where they are, their configuration, condition, risk, and lifecycle state) and using that representation to **plan and defend** inspection, maintenance, renewal, and capital investment under real public-sector constraints.

Twin-enabled asset management goes beyond CMMS/EAM recordkeeping:
- It reconciles **GIS location truth** with **CMMS work history truth** and with **engineering evidence** (as-builts, inspections, photos, telemetry).
- It maintains a **digital thread** from as-designed → as-built → as-maintained → as-operated.
- It treats inspection evidence as governed records (QA, sampling, chain-of-custody) because many outcomes touch safety, claims, and regulatory contexts.

This document deepens item 2 in [`kali-task-research.md`](kali-task-research.md:1): “Asset management: Track lifecycle, condition, and maintenance of infrastructure assets using twin-based inspection and work planning.”

---

## Evidence pointers (external sources, implementation-grade)
The program should ground policy language and audit posture in recognized standards/guidance:

- **ISO 55000:2024 — Asset management — Vocabulary, overview and principles (ISO)**
  - URL: https://www.iso.org/standard/83053.html
  - Takeaway: ISO positions asset management as a systematic, lifecycle approach to realize value, improve performance, and manage risk; ISO 55000 provides overview/terminology/principles, with ISO 55001 requirements and ISO 55002 guidance.

- **NIST SP 800-82 — Guide to Operational Technology (OT) Security (NIST)**
  - URL: https://csrc.nist.gov/pubs/sp/800/82/r3/ipd
  - Takeaway: OT/ICS environments have unique safety/reliability constraints; security architectures should respect segmentation and controlled interfaces—useful for defining IT analytics vs OT boundary for SCADA-adjacent asset telemetry.

- **NIST SP 1270 (2022) — A Proposal for Identifying and Managing Bias in AI (NIST, PDF)**
  - URL: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.1270.pdf
  - Takeaway: Bias management must address datasets, evaluation/validation/verification, and human/systemic factors; directly applicable to computer vision (CV) inspection pipelines and labeling QA.

- **AASHTO Innovation Initiative — Linear Referencing System (AASHTO)**
  - URL: https://aii.transportation.org/pages/linearreferencingsystem.aspx
  - Takeaway: LRS aligns linear reference points across databases to integrate roadway-related data (e.g., safety, pavement, asset inventories); relevant to municipal road corridor and network asset modeling.

Note: “Chain of custody” guidance is often sourced from digital forensics literature; for municipal claims contexts, adopt the core principles (provenance, immutability, access logs, exportability) and align with your legal counsel’s evidentiary requirements.

---

## 1) Scope and non-goals

### 1.1 Scope: what “twin-enabled asset management” means here
In scope:
- Canonical asset registry and reconciliation across **GIS + CMMS/EAM + document management + as-builts + field observations**, with traceable lineage.
- Digital thread/configuration management (as-designed/as-built/as-maintained/as-operated).
- Evidence governance: inspection QA, sampling, bias controls for CV/ML, retention and chain-of-custody.
- Decision support for O&M + capital planning under real-world constraints.
- Cyber-physical boundary when telemetry/SCADA is involved.

### 1.2 Non-goals (to keep this implementable)
- Replacing the CMMS/EAM. The twin **integrates and governs**; the CMMS remains the system of record for work execution.
- Full “closed-loop” automation (twin directly dispatches crews or actuates controls). Default posture is **decision support**.
- A single mega-schema for every asset type on day one. Start with a minimum canonical model + domain extensions.

### 1.3 Safety boundary (mandatory policy statement)
- Default: **no direct actuation** from the digital twin into OT/SCADA. Twin outputs are advisory unless a formal safety case exists.
- Any workflow that influences dispatch/operations must have:
  - human approval gates,
  - rollback procedures,
  - training and accountability,
  - incident drills.

---

## 2) Canonical asset data model (minimum viable + extensions)
The canonical model must support: identity resolution, cross-system reconciliation, linear + point assets, configuration, and auditability.

### 2.1 Minimum viable canonical asset record (required fields)
Recommended canonical entity: `Asset` (with `AssetInstance` for physical instances when needed).

| Field | Type | Required | Notes |
|---|---:|:---:|---|
| `asset_id` | string | ✅ | Program-wide stable ID (never reused) |
| `source_ids` | object | ✅ | Mapping keys: `gis_feature_id`, `cmms_asset_id`, `doc_mgmt_id` etc. |
| `asset_class` | enum/string | ✅ | Controlled vocabulary (roads, bridge, hydrant, pump, facility HVAC…) |
| `asset_type` | enum/string | ✅ | More specific classification; supports standards/ontology mapping |
| `lifecycle_state` | enum | ✅ | planned / installed / in_service / out_of_service / retired |
| `owner_org` | string | ✅ | Department/utility/agency |
| `operator_org` | string | ✅ | Often differs from owner |
| `criticality` | enum | ✅ | Tiered (e.g., 1–4) with documented rubric |
| `replacement_cost_est` | number | ⛔ | Useful for risk/renewal; may be estimated |
| `geometry` | GeoJSON/WKT | ✅ | Point/line/polygon; geometry validity enforced |
| `crs` | string | ✅ | EPSG code (e.g., EPSG:4326); store authoritative CRS metadata |
| `linear_ref` | object | ⛔ | For linear assets: route ID, from/to measures, measure method, calibration date |
| `admin_area` | string | ⛔ | Ward/district/service zone for equity reporting |
| `parent_asset_id` | string | ⛔ | Hierarchy (system → subsystem → component) |
| `install_date` | date | ⛔ | If unknown, store `install_year_est` + confidence |
| `material` | string | ⛔ | For pipes/roads/structures |
| `capacity_rating` | string/number | ⛔ | As applicable |
| `authoritative_source` | enum | ✅ | `GIS` for location; `CMMS` for work history; as-built for design attributes (configurable) |
| `data_quality_flags` | array | ✅ | e.g., `missing_install_date`, `geometry_suspect`, `id_conflict` |
| `created_at` / `updated_at` | datetime | ✅ | Audit timestamps |
| `provenance` | object | ✅ | Who/what asserted changes; ingestion job ID; source version |

Defaults (overrideable): GIS authoritative for location; CMMS authoritative for work history.

### 2.2 Linear vs point assets (modeling guidance)
- **Point assets** (hydrants, valves, signs, sensors, poles): use point geometry + optional offset.
- **Linear assets** (roads, pipes, cables): store both:
  - geometry (for mapping and spatial joins), and
  - **linear reference** (route + measures) for stable corridor-level reporting and split/merge handling.

Practical rule: GIS geometry changes frequently; LRS/measures provide a stable reference for “work at mile 3.2–4.1” even after resegmentation.

### 2.3 Ontologies and controlled vocabularies (vendor-neutral)
Start with controlled vocabularies that are:
- small enough to govern,
- mapped to source system codes,
- extensible per asset domain.

Minimum:
- `asset_class`, `asset_type`, `failure_mode` (high-level), `condition_grade`, `work_type`, `defect_type`.

### 2.4 Reconciliation rules (GIS vs CMMS vs as-builts vs field observations)
Use explicit “truth rules” with exception handling:

| Attribute group | Default authoritative source | Common exceptions |
|---|---|---|
| Location/geometry | GIS | Field verification overrides GIS pending steward approval |
| Work history/status | CMMS/EAM | Emergency work during CMMS outage captured as provisional then reconciled |
| Design attributes (material, diameter, rating) | As-built / engineering record | If as-built missing, use field observation with confidence + follow-up |
| Current condition score | Inspection evidence system | If multiple methods disagree, retain both + confidence + adjudication |

Identity resolution approach (recommended):
- deterministic joins where possible (`source_ids`),
- probabilistic matching for legacy/unknown assets (geometry proximity + attributes),
- mandatory steward adjudication for conflicts.

### 2.5 Schema evolution and versioning
- Version the canonical schema and publish a **data contract** for consumers.
- Define breaking vs non-breaking change policy.
- Maintain mapping tables and transformation code with automated tests.

---

## 3) Digital thread / configuration & identity management
Asset management fails when “the asset” changes but the record doesn’t. The twin must represent time and configuration.

### 3.1 Definitions and where each “truth” lives
- **As-designed**: engineering design intent (CAD/BIM, specs). System: engineering document management / BIM repository.
- **As-built**: what was constructed/installed (as-built drawings, redlines, commissioning). System: document management + GIS update workflow.
- **As-maintained**: changes from maintenance/repairs (parts replaced, settings changed). System: CMMS/EAM + maintenance notes.
- **As-operated**: operational state (telemetry, setpoints, runtime conditions). System: OT/SCADA historian / IoT platform (read-only into twin by default).

Rule: the twin stores a **normalized view** + links back to authoritative evidence.

### 3.2 Component hierarchy and partial replacement accounting
Model:
- `Asset` (system) → `AssetComponent` (replaceable subassembly) → `AssetPart` (optional)

Guidance:
- Track **component-level lifecycle** (e.g., pump motor replaced but station remains).
- Costs and work history attach at the correct level.

### 3.3 Split/merge identity playbook (linear assets)
Common events:
- Road segment resegmentation (GIS edits)
- Pipe replacement of a subsection
- Re-alignment/realignment of corridor

Rules (recommended):
- `asset_id` is stable for a physical “lifecycle unit.” When split/merge occurs:
  - Create new child assets with new `asset_id`s.
  - Maintain `identity_events` ledger:
    - `split_from`, `merged_from`, `retired_into`.
  - Keep historical measures/geometry snapshots.

Example: road segment split
- Before: `ROAD-001` measures 0.0–2.0
- After resegmentation:
  - `ROAD-101` 0.0–1.2 (split_from `ROAD-001`)
  - `ROAD-102` 1.2–2.0 (split_from `ROAD-001`)
- Preserve work history linkages via route+measure mapping.

---

## 4) Evidence governance and QA (inspection, imagery, telemetry)
Municipal asset decisions can be contested (claims, audits, safety investigations). Evidence must be trustworthy.

### 4.1 Evidence types
- Field inspection forms (structured)
- Photos/video
- LiDAR/point clouds
- NDT results
- Telemetry summaries (read-only extracts)

### 4.2 Sampling and audit plan (operational QA)
Recommended minimum QA controls:
- **Spot checks**: random sampling of inspections (e.g., 5–10% depending on criticality).
- **Blind re-inspections**: second inspector repeats inspection without seeing prior score.
- **Inter-rater reliability**: measure consistency for categorical scores (e.g., Cohen’s kappa) and continuous scores (ICC) per inspector/team.
- **Targeted audits**: oversample high-impact assets (bridges, pressure systems, critical corridors).

Audit artifacts to retain:
- sampling frame + method,
- QA outcomes and corrective actions,
- inspector training and calibration dates.

### 4.3 CV/ML defect detection bias controls + human review gates
If using CV/ML for inspection support:
- Maintain labeled datasets with provenance and representativeness checks (NIST bias guidance).
- Require human review for:
  - low-confidence detections,
  - high-severity defects,
  - model drift alerts.
- Monitor bias proxies:
  - performance by lighting/weather conditions,
  - performance by neighborhood infrastructure types (avoid inequitable under-detection).
- Document model versions, thresholds, and rollback conditions.

### 4.4 Retention + chain-of-custody (evidence that can stand up to scrutiny)
Principles:
- **Immutability**: raw evidence objects are write-once (no overwrite); corrections create new versions.
- **Provenance**: capture who collected it, device/app version, timestamp, location, and method.
- **Access log**: who viewed/exported/annotated evidence.
- **Evidentiary export**: reproducible export package with hashes.

Minimum evidence metadata:
- `evidence_id`, `asset_id`, `captured_at`, `captured_by`, `capture_device_id`, `capture_method`, `location_at_capture`, `hash`, `storage_uri`, `classification`, `related_work_order_id`.

---

## 5) Decision support + optimization under real public works constraints
Optimization is often oversold. Municipal reality has constraints that dominate the math.

### 5.1 Explicit constraint list (use in planning tools)
Operational constraints:
- permits and lane closures (lead times, allowed windows)
- seasonal windows (freeze/thaw, paving season, storm seasons)
- crew skills/certifications and union rules
- depot geography and travel time
- safety constraints (work at heights, confined space)

Program constraints:
- contract boundaries (different vendors for paving vs utilities)
- “dig once” bundling across utilities and departments
- inventory lead times (parts, valves, transformers)
- procurement rules (bid thresholds, sole source constraints)
- multi-year budget cycles, political commitments, grant conditions

### 5.2 Practical approach: heuristics → decision support → constrained optimization
Start here:
- Heuristics: priority = risk score × service criticality × evidence confidence.
- Corridor bundling rules: bundle by geography + time window + permits.
- Present options with tradeoffs, not a single “optimal” answer.

Graduate to optimization only when:
- constraints are captured reliably,
- work execution data quality is high,
- the city can operationalize recommended schedules.

### 5.3 When not to optimize
Do not optimize (yet) when:
- asset registry has high ID conflict rate,
- condition evidence is sparse or unreliable,
- work orders are not closed accurately,
- the city cannot enforce schedules due to emergency-driven work.

---

## 6) Cyber-physical boundary and security (IT analytics vs OT/SCADA)

### 6.1 Trust boundaries
- OT/SCADA is safety- and reliability-critical; prioritize stability and deterministic behavior.
- IT analytics/twin is decision-support; treat OT data feeds as **inputs**, not control channels.

### 6.2 Default posture
- Read-only ingestion from OT historians/DMZ into IT.
- No write-back to OT.
- Any exceptions require security + safety review (align to NIST SP 800-82 principles).

### 6.3 Safety constraints for operational recommendations
If twin outputs influence dispatch:
- require supervisor approval,
- limit to recommendation sets with confidence and constraints visible,
- log decisions (who accepted/overrode) for audit.

---

## 7) Validation and acceptance criteria

### 7.1 Data model QA checks
- Geometry validity (self-intersections, topology errors)
- CRS correctness + transformation tests
- Required field completeness by criticality tier
- Duplicate detection (near-duplicate geometry + attributes)
- Identity conflict rate (unresolved merges/splits)

### 7.2 Model validation (deterioration/RUL)
- Baseline first (simple survival/hazard/Markov models) before complex ML.
- Backtesting on historical failures and inspections.
- Calibration cadence (quarterly for high-change assets; annually otherwise).
- Accuracy bands by asset class (define acceptable error ranges and confidence reporting).

### 7.3 Operational validation
- Field crew feedback loop: reject/accept reason codes for recommendations.
- Drill-based validation: “time to find evidence + governing record” exercises.

---

## 8) Benefits realization and attribution (defensible, audit-friendly)
Avoid claiming benefits that are really caused by budget/staffing/weather shifts.

### 8.1 Benefits (examples)
- fewer unplanned failures/outages
- reduced emergency callouts/overtime
- better renewal targeting (risk reduction per $)
- improved compliance (inspection coverage)

### 8.2 Measurement designs (credible counterfactuals)
Use one or more:
- **Before/after with controls**: compare similar corridors/areas not using the twin.
- **Matched pairs**: match assets on age/material/traffic and compare outcomes.
- **Seasonal adjustment**: normalize for weather and seasonal work windows.
- **Interrupted time series**: evaluate trend break after adoption.

### 8.3 Confounders to explicitly account for
- staffing levels and contractor availability
- budget changes and grant funding
- extreme weather events
- policy changes (e.g., new inspection mandates)

### 8.4 “Evidence required to claim benefit” thresholds
Examples:
- benefit claim requires ≥ 2 quarters of comparable data plus documented controls.
- for failure reduction: show statistically meaningful change or clear operational narrative + evidence pack.

---

## 9) Implementation roadmap (owners + dependencies)

### 0–3 months (foundation)
Owners: asset owners + GIS lead + CMMS admin + data governance.
- agree asset taxonomy and minimum canonical model
- implement source mapping (`source_ids`) and authoritative-source rules
- stand up evidence metadata capture (even before full CV/ML)

### 3–12 months (integrate + govern)
Owners: platform team + O&M leads + security.
- build reconciliation workflows (duplicate/identity conflicts)
- deploy inspection QA sampling + inter-rater reliability measures
- integrate CMMS work orders end-to-end; enforce closure quality
- define OT boundary and implement read-only telemetry ingestion where needed

### 12–24 months (optimize + scale)
Owners: capital planning + O&M + analytics.
- corridor bundling + constrained scheduling decision support
- mature deterioration/RUL models with backtesting and drift monitoring
- operationalize benefit attribution with controls and audit packs

---

## 10) Key metrics (harder to game)

### 10.1 Operational SLIs/SLOs
- asset registry completeness by criticality tier
- condition freshness vs policy
- work order cycle time (identify → create → start → close)
- evidence coverage (% inspections with required media + metadata)

### 10.2 Outcome KPIs
- unplanned failures avoided (with attribution method)
- downtime reduction for critical services
- backlog risk reduction per $ spent

### 10.3 Data governance metrics
- identity conflict rate (unresolved split/merge)
- split/merge rate (change pressure)
- QA audit findings rate and closure time

---

## 11) Risks & mitigations (detect + respond playbooks)

| Risk | Detection signals | Response playbook |
|---|---|---|
| Fraud/manipulation of inspections | abnormal score distributions; repeated patterns; missing metadata | targeted re-inspections; revoke privileges; retrain; disciplinary process |
| Model misuse/automation bias | crews follow low-confidence recs; overrides not logged | enforce approval gates; require confidence display; add training |
| Vendor lock-in | proprietary exports; closed APIs | require export formats + data contracts in SoWs; escrow/transition clauses |
| Privacy/worker surveillance concerns | complaints; excessive tracking | minimize personal data; purpose limitation; access tiering; retention controls |
| OT risk (unsafe coupling) | write-back paths appear; anomalies in OT | enforce segmentation; disable write-back; incident response drill |

---

## 12) References
### 12.1 Workspace source
- Item 2 in [`kali-task-research.md`](kali-task-research.md:1)

### 12.2 External references (retrieved via Firecrawl MCP)
- ISO 55000:2024 — Asset management — Vocabulary, overview and principles (ISO): https://www.iso.org/standard/83053.html
- NIST SP 800-82 Rev. 3 (IPD) — Guide to Operational Technology (OT) Security (NIST): https://csrc.nist.gov/pubs/sp/800/82/r3/ipd
- NIST SP 1270 (2022) — A Proposal for Identifying and Managing Bias in AI (NIST, PDF): https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.1270.pdf
- AASHTO Innovation Initiative — Linear Referencing System (LRS): https://aii.transportation.org/pages/linearreferencingsystem.aspx
