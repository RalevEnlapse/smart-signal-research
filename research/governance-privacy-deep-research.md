# Governance & privacy — Deep research

## Executive summary

Governance and privacy are the **operating system** for a City Digital Twin: the enforceable rules, controls, and decision rights that determine who can access which data, for what purposes, under what safeguards, and with what accountability.

A city twin is uniquely risky because it **joins** datasets across domains (mobility + incidents + assets + citizen reports). Even when individual datasets are “non-identifying,” joins create **linkage/inference risk**. Therefore, this playbook is built around:

- explicit **classification tiers** and a **control baseline matrix**
- operational **inference/join risk scoring** and a **join review workflow** (with safe-join libraries)
- practical integration of **records law / FOI** with privacy controls (“publishable vs releasable vs restricted”)
- enforceable **vendor/subprocessor governance** (contract clauses + technical enforcement)
- **federated, multi-agency governance** with dispute resolution and shared taxonomy change control
- community engagement mechanisms with **real decision rights**, transparency reporting, and recourse

The goal is to be auditable: every access, export, join approval, FOI release, and vendor exception should be provable from logs and records.

## 0. Scope assumptions

Defaults assumed (overrideable):

- federation mode: multi-agency
- records law context: applies
- publication mode: balanced
- vendor access posture: least-privilege
- policy-as-code preference: yes

## 1. Classification tiers and control baseline matrix (explicit)

### 1.1 Classification tiers (recommended)

Use 4 tiers that match city-twin reality:

1. **Public (P0)**: safe for proactive publication.
2. **Internal (P1)**: routine operational data; not public; low linkage risk.
3. **Restricted (P2)**: could harm individuals/operations if misused; includes quasi-identifiers and sensitive operational detail.
4. **Sensitive/Critical (P3)**: high-risk personal/safety or critical infrastructure; high linkage risk; special handling required.

### 1.2 Control baseline matrix

Minimum controls by tier. If you cannot meet the baseline, the dataset cannot be onboarded (or must be downgraded/transformed).

| Control area | P0 Public | P1 Internal | P2 Restricted | P3 Sensitive/Critical |
|---|---|---|---|---|
| Encryption in transit | TLS required | TLS required | TLS + mTLS for service-to-service | TLS + mTLS; strong cipher policy |
| Encryption at rest | required for platform stores | required | required + customer-managed keys (preferred) | required + customer-managed keys + key rotation |
| IAM | SSO recommended | SSO + MFA for admins | SSO + MFA for all; ABAC/RBAC | SSO + MFA + ABAC + JIT access |
| Least privilege | basic roles | role-based | role + attribute + row/column security | strict ABAC + row/column + purpose scoping |
| Export controls | open download ok | controlled export roles | export allowlist + justification | exports rare; two-person approval; watermarking |
| Egress monitoring | basic | required for bulk | required + anomaly alerts | required + hard egress limits + DLP signals |
| Logging | access logs | access + admin logs | access + query + export logs | immutable access/query/export logs + alerting |
| Log retention | 90d+ | 180d+ | 1y+ | 2y+ (or per legal/audit) |
| Aggregation / min cell size | required for sensitive metrics | recommended | mandatory (k-min) + suppression | mandatory + stronger disclosure controls |
| Geospatial precision | coarse by default | coarse/medium | precision tiering by role | strict precision tiering; “need-to-know” |
| Review cadence | annual | semi-annual | quarterly access reviews | monthly access reviews + join review audits |
| Retention | publishable schedule | defined + enforced | short raw / longer aggregates | shortest feasible; legal hold workflow |
| DPIA triggers | if new risky publication | if new join/surveillance | mandatory for onboarding + joins | mandatory + exec sign-off for residual risk |
| Vendor access | none or public CDN | tightly scoped | scoped + monitored + time-bound | clean room only; no raw export; audit rights |

Notes:

- “Aggregation/min cell size” is necessary but not sufficient; apply disclosure controls in Section 2.
- “Geospatial precision tiering” is a primary safety control for city twins.

## 2. Inference/linkage risk operationalization

### 2.1 Dataset privacy risk scoring (semi-quantitative)

Score each dataset and each proposed join on dimensions; produce a risk class: Low/Med/High/Critical.

**Dataset risk factors (example 0–3 each):**

- identifiers present (direct identifiers)
- quasi-identifiers present (time + location + device IDs)
- sensitive attributes (health, safety incidents, protected sites)
- granularity (fine spatiotemporal resolution)
- population size (small-n risk)
- external linkability (data can be linked to public datasets)

Compute:

- `risk_score = sum(factors)`
- `risk_class` thresholds are local policy.

### 2.2 Join risk scoring (the core city-twin problem)

A join can escalate risk even if both inputs are “safe.” Score:

- join keys (stable identifiers, location/time proximity)
- output granularity
- whether join enables trajectory reconstruction
- whether join increases uniqueness

Hard rule examples:

- any join that produces **fine time + fine location + stable identifier** is **High/Critical**.
- any join involving P3 inputs requires review.

### 2.3 Join review workflow (enforceable)

When review is required:

- any join involving P2/P3 datasets
- any join that lowers aggregation thresholds or increases geospatial precision
- any join that introduces new linkage keys (device ids, household proxies)

Approval roles:

- **Data Steward** (Responsible)
- **Privacy Officer** (Accountable for P2/P3)
- **Security** (Consulted)
- **Product Owner** (Consulted)

Required evidence:

- purpose and allowed use mapping
- risk score and risk class
- mitigations chosen
- disclosure controls and output classification
- retention and export posture

Mitigation options:

- aggregation (time buckets, spatial bins)
- suppression/top-coding
- sampling
- tokenization/pseudonymization
- differential privacy for published aggregates (where appropriate)
- access tiering (restrict outputs)

### 2.4 Safe-join library (approved patterns) + prohibited list

**Safe-join library**: versioned, approved join “recipes” that teams can reuse without re-litigating each time.

Examples of safe joins:

- join by coarse geography (neighborhood) + weekly bins
- join by asset id when the asset is non-sensitive and does not imply individuals

Prohibited joins (examples):

- joining citizen reports with mobility trajectories at fine resolution
- joining enforcement/incident locations with identifiable household-level data

## 3. Records law / FOI integration (publishable vs releasable vs restricted)

### 3.1 Three-way classification for disclosure

Separate classification for **data security** vs **disclosure posture**.

- **Publishable**: proactively published (after disclosure controls).
- **Releasable**: can be released on request after review/redaction.
- **Restricted**: not releasable except under defined exemptions; requires legal rationale.

Every dataset must have:

- `disclosure_posture` (publishable/releasable/restricted)
- `foi_exemptions_basis` (if restricted)
- `default_redaction_profile`

### 3.2 FOI processing playbook (integrated with catalog and logs)

Workflow:

1. intake FOI request → create case id
2. identify candidate datasets via catalog search
3. run “sensitive join detection” checks:
   - does the requested extract combine datasets?
   - does it include fine time/location?
4. apply redaction profile and disclosure controls
5. legal/privacy review and sign-off
6. release artifact with immutable release log:
   - what was released
   - what was withheld and why
   - transformation steps

### 3.3 Redaction and review steps

- automated redaction for direct identifiers
- aggregation/suppression thresholds
- manual spot checks on high-risk extracts
- require two-person review for P3-derived releases

## 4. Vendor and ecosystem governance (technical + contractual)

### 4.1 Standard contract clauses checklist

Minimum clauses for any vendor/subprocessor touching P2/P3 data:

- audit rights (including subprocessor audits)
- subprocessor disclosure and approval
- breach notification timelines
- data return/destruction on termination
- egress/export limits and logging requirements
- restrictions on model training and secondary use
- security baseline (MFA, encryption, patching)
- incident cooperation and forensics support

### 4.2 Technical enforcement patterns

- scoped credentials; no shared accounts
- just-in-time, time-boxed vendor access
- clean-room patterns for vendor analytics:
  - vendor brings code to data; outputs reviewed before export
- egress monitoring and rate limits
- watermarking for sensitive exports

### 4.3 Vendor exception lifecycle

- exceptions must be time-boxed
- compensating controls required
- P3 exceptions require executive sign-off
- track **exception debt** and repeat offenders

## 5. Federated governance across agencies

### 5.1 Governance bodies and composition

- **Cross-Agency Data Governance Council**: agency data owners + city platform lead + legal/privacy + security + procurement.
- **Privacy Review Board**: privacy officer, legal, community representative (where applicable), security.
- **Security Board**: CISO team + agency security leads.

### 5.2 Dispute resolution and escalation

- disputes resolved in 10 business days at working-group level
- escalate to Governance Council if:
  - cross-agency conflict, or
  - P3 risk, or
  - public publication requested
- escalate to executive sponsors for:
  - repeated policy violations
  - high residual risk decisions

### 5.3 Shared taxonomy alignment and change control

Shared taxonomies are “infrastructure.” Assign owners:

- geographies (boundaries, neighborhood definitions)
- incident types and severity
- asset identifiers and registries

Changes:

- require an ADR-like change record
- migration plan and backward compatibility

## 6. Community engagement mechanisms (with real influence)

### 6.1 Community oversight structure

Establish a **Community Data Oversight Panel** with:

- membership representing impacted communities
- defined decision rights:
  - review high-risk P3 uses
  - recommend prohibitions or additional controls
  - require transparency disclosures

### 6.2 Transparency report cadence + template

Publish quarterly (balanced mode) with:

- datasets onboarded by tier
- access request stats and join reviews
- privacy incidents summary and remediation
- FOI releases summary (counts and categories)
- vendor exceptions and audits summary

### 6.3 Complaint/recourse channels

- intake: web form + hotline + in-person option
- response SLA: acknowledge within days; resolve within weeks
- outcomes tracked and reported

### 6.4 How feedback changes policy

- every report cycle includes a “policy changes since last report” section
- maintained backlog of community-requested changes with status

## 7. Operational runbooks (minimum set)

### 7.1 Suspected privacy incident

- triage: scope, dataset tier, affected individuals
- contain: revoke access, stop exports, rotate keys
- investigate: review logs, identify joins/exports
- notify: legal/privacy process, affected parties as required
- remediate: controls added, training, postmortem

### 7.2 FOI request for sensitive datasets/joins

- identify datasets and requested joins
- run sensitive-join detection
- apply redaction profiles
- two-person review for P3 risk
- release with audit record

### 7.3 Vendor/subprocessor change and review

- require updated subprocessor list
- security and privacy review
- update contracts and access scopes

### 7.4 Emergency access grant during incident response

- time-bound access
- least privilege
- mandatory post-incident review and revocation

## 8. Key metrics

- tier coverage (% datasets classified; % meeting baseline controls)
- join-review throughput and rejected join rate
- privacy incident MTTR and recurrence
- FOI timeliness and redaction error rate
- vendor exceptions and exception debt
- transparency report delivery and community recourse outcomes

## 9. Implementation roadmap

### 0–3 months

- define tiers and baseline matrix; add required catalog fields
- implement minimal join review workflow
- draft FOI playbook and redaction profiles

### 3–12 months

- policy-as-code enforcement at key control points (query/export)
- build safe-join library and prohibited list
- implement vendor clean-room patterns and egress monitoring
- start transparency reporting cadence

### 12–24 months

- cross-agency federation maturity (taxonomy governance, dispute operations)
- automated inference-risk checks in pipelines
- continuous audits and external review

## 10. References

### 10.1 Workspace source

- Item 13 in [`kali-task-research.md`](../kali-task-research.md:1)

### 10.2 External references (retrieved via Firecrawl MCP)

- NIST. *Privacy Framework.* https://www.nist.gov/privacy-framework — NIST’s privacy risk management framework intended to help organizations identify and manage privacy risk.
- NIST. *NIST Privacy Framework CORE* (PDF). https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf — Defines the core functions (Identify-P, Govern-P, Control-P, Communicate-P, Protect-P) used to structure privacy risk management.
- NIST. *SP 800-188 — De-Identifying Government Datasets: Techniques and Governance.* https://csrc.nist.gov/pubs/sp/800/188/final — Government-oriented guidance for de-identification, disclosure risk, and governance beyond simple k-anonymity.
- DOJ OIP. *FOIA Guide: Exemption 6.* https://www.justice.gov/archives/oip/foia-guide-2004-edition-exemption-6 — Explains FOIA Exemption 6 personal privacy protections and balancing tests, relevant to release/redaction decisions.
- NIST. *SP 800-161 — Supply Chain Risk Management Practices.* https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-161.pdf — Guidance on supply chain risk management practices relevant to vendor/subprocessor governance and auditability.
