# Human-in-the-loop safeguards — Deep research

## Executive summary
Human-in-the-loop (HITL) safeguards are governance and system design patterns that ensure humans retain meaningful control over high-impact decisions made with the assistance of automated models (analytics, optimization, ML/AI). In a city digital twin, HITL is essential because decisions can affect safety, civil liberties, service access, and large budgets.

HITL is not simply “put a button for approval.” Effective safeguards require:
- Clear decision boundaries (what can be automated vs must be approved)
- Role-based authority and accountability
- Explainability and evidence presentation
- Monitoring for model drift and harmful outcomes
- Fail-safe modes and rollback

This document deepens item 23 in [`kali-task-research.md`](../kali-task-research.md:1): *“Human-in-the-loop safeguards: Keep humans in approval loops for high-impact decisions to reduce automation risk.”*

---

## 1. Background and context
Automation in city systems appears in:
- Incident detection and dispatch recommendations
- Mobility pricing and signal optimization
- Utility demand response activation
- Risk scoring for inspections and enforcement

Risks of “automation-only” approaches:
- Automation bias and over-trust
- Hidden value tradeoffs (equity vs efficiency)
- Feedback loops that amplify harms
- Lack of recourse and accountability

HITL aims to ensure decisions remain:
- Legitimate (appropriate authority)
- Safe (bounded and tested)
- Auditable (traceable)
- Contestable (recourse and review)

---

## 2. Stakeholders
- **Decision owners** (agency heads, incident commanders): accountable outcomes
- **Operators** (dispatch, TMC staff): daily decision execution
- **Model owners** (data science/product): model performance and updates
- **Risk/compliance** (privacy, legal, ethics): constraints and oversight
- **Affected communities**: fairness, transparency, recourse
- **IT/security**: access control and auditability

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity of decision workflows and approvals
- Confidentiality of sensitive evidence shown to approvers
- Availability of override mechanisms

### 3.2 Abuse/failure cases
- **Rubber-stamping** approvals under time pressure
- **Privilege escalation**: unauthorized users approving actions
- **Gaming**: operators adjust inputs to force a desired recommendation
- **Model drift** produces harmful recommendations that slip through

### 3.3 Controls
- Separation of duties for high-impact actions
- Time-bound, incident-scoped privileges
- Mandatory “reason codes” and evidence capture on approvals
- Monitoring of override and approval patterns

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Decision policy registry**
   - Decision types, risk tiers, required approver roles, constraints

2. **Recommendation service**
   - Produces suggested actions with confidence and supporting evidence

3. **Approval workflow engine**
   - Routes approvals, enforces quorum for high-risk actions
   - Captures decisions, reasons, and timestamps

4. **Explainability/evidence UI**
   - Shows key features, counterfactuals, uncertainty, and constraints

5. **Audit log and case record**
   - Immutable record of recommendations, approvals, overrides, outcomes

6. **Monitoring and drift detection**
   - Tracks model performance, outcome impacts, bias indicators

7. **Fallback and safety controls**
   - Manual mode, safe defaults, rollback mechanisms

### 4.2 Data flows
- Data → model → recommendation + confidence + evidence
- Recommendation → workflow → approval/deny with justification
- Approved actions → execution systems (dispatch, control)
- Outcomes → monitoring → model review / policy updates

---

## 5. Methods / patterns

### 5.1 Decision tiering
Define tiers by potential harm:
- Tier 0: informative only (no approvals needed)
- Tier 1: low-impact operational tuning (single approval)
- Tier 2: high-impact (multi-party approval, tighter constraints)
- Tier 3: exceptional (requires incident commander / executive approval)

### 5.2 "Meaningful human control" criteria
- Human understands the action and its likely impacts
- Human has authority and time to decide
- Human can reject/override without penalty
- System supports recourse and review

### 5.3 Evidence and explainability
- Provide concise rationale and key drivers
- Show uncertainty intervals, not single-point outputs
- Provide "what would change the recommendation?"
- AI/ML explainability methods: SHAP, LIME, counterfactual explanations
- Attention visualization for deep learning models

### 5.4 Guardrails and constraints
- Hard constraints (cannot violate safety/legal limits)
- Soft constraints with explicit tradeoffs
- Rate limits and cooldown periods
- Digital twin-based constraint validation
- Real-time constraint monitoring and enforcement

### 5.5 Post-decision evaluation
- Track approval outcomes vs counterfactuals
- Monitor disparate impacts and unintended consequences
- Continuous learning from human feedback
- Model performance monitoring and drift detection

### 5.6 Standards and protocols
- NIST AI Risk Management Framework (AI RMF 1.0)
- ISO/IEC 23894 for AI risk management
- IEEE 7000 series for AI ethics and governance
- GDPR requirements for automated decision-making
- OPA (Open Policy Agent) for policy enforcement
- SPIFFE for service identity and authentication
- NGSI-LD for context-aware decision data

External reference: NIST AI Risk Management Framework (AI RMF 1.0) provides cross-sector guidance for AI risk management and governance ([NIST AI RMF PDF](https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf)).

---

## 6. Data requirements
- Decision type taxonomy and risk tier labels
- User roles and authority mappings
- Recommendation inputs and feature provenance
- Approval/override logs and outcome metrics
- Feedback from operators (reason codes, free-text notes)

---

## 7. Implementation plan (phases)

### Phase 0 — Define decision boundaries
- Inventory automated decisions across the twin
- Define risk tiers and required approvals

### Phase 1 — Workflow instrumentation
- Add audit logging and approval capture for key actions
- Implement time-bound entitlements for incidents

### Phase 2 — Explainability and evidence design
- Build operator UI that presents evidence succinctly
- Add uncertainty and “why/why not” explanations

### Phase 3 — Monitoring and controls
- Drift monitoring and outcome tracking
- Override analytics and “rubber-stamp” detection

### Phase 4 — Mature governance
- Periodic reviews, red teaming, and public transparency (where appropriate)
- Expand to additional decisions and agencies

---

## 8. Testing and validation
- Access control tests for approval workflows
- Simulation tests: high-volume incident scenarios to prevent rubber-stamping
- Usability tests with operators under time pressure
- Post-deployment audits: sample decisions reviewed for policy compliance

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- % high-risk actions with correct approval chain
- Override rate (by model version and incident type)
- Approval latency (time to decision)
- Outcome quality metrics (e.g., response time, safety incidents)
- Model drift detection metrics
- Explainability quality scores
- Digital twin synchronization metrics

### 9.2 Example SLOs
- 100% of Tier 2+ actions have immutable audit records
- <1% of Tier 2+ actions executed without required approvals
- System uptime ≥ 99.9%
- Approval latency < 5 minutes for Tier 2 actions
- Model accuracy ≥ 95% for recommendations

---

## 10. Governance and compliance
- Align HITL policies with legal authority and accountability
- Ensure audit logs meet retention and evidentiary requirements
- Define recourse processes for affected parties where applicable
- Establish model change management and “who can update what” policies

---

## 11. Risks and mitigations
- **Approval fatigue** → tiering and automation for low-risk actions
- **Rubber-stamping** → friction for high-risk actions, sampling audits
- **Delays during incidents** → pre-approved playbooks with bounded automation
- **Operator distrust** → explainability and post-action learning loops

---

## 12. Costs and FinOps
- Workflow engine and audit log storage
- UI/UX and training for operators
- Ongoing monitoring and evaluation

Track:
- Cost per 1,000 decisions logged
- Cost per audit review cycle

---

## 13. KPIs
- Reduction in harmful/incorrect automated actions
- Operator trust and adoption (survey + usage)
- Decision latency improvements without safety regressions
- Audit compliance rate

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Decision tiering policy and registry
- Approval workflow engine and audit logs
- Explainability UI and evidence templates
- Monitoring dashboards and review playbooks

### 14.2 Readiness checklist
- [ ] Decision tiers defined and approved
- [ ] Approval roles mapped and enforced
- [ ] Audit logging immutable and queryable
- [ ] Monitoring and drift alerts configured

---

## 15. References
### 15.1 Workspace source
- Item 23 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- NIST — Artificial Intelligence Risk Management Framework (AI RMF 1.0), PDF: https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf
- Yessef et al. (2025). "Digital twin technology in smart cities: A step toward intelligent urban management." Energy Reports, 14, 5539-5557. DOI: 10.1016/j.egyr.2025.11.097
- Crespo-Aguado et al. (2024). "Digital twins for smart cities: A systematic review." IEEE Access.
- Jerkovic et al. (2025). "Smart grid IoT framework for predicting energy consumption using federated learning homomorphic encryption." Sensors.

### 15.3 Suggested further reading (not fetched)
- Human factors engineering for decision support under time pressure
- Auditing and accountability for algorithmic decision-making
- Explainable AI (XAI) methods and best practices
- Digital twins for human-in-the-loop decision support
- Federated learning for privacy-preserving decision support
- Zero-trust architecture for automated decision systems
