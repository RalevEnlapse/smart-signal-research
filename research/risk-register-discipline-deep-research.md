# Risk register discipline — Deep research

## Executive summary
A risk register is a living log of identified risks, their likelihood and impact scores, mitigations, owners, and escalation paths. In a city digital twin context, the risk register spans technical risks (data quality, model failures, outages), operational risks (process gaps, human error), and strategic risks (misalignment, funding cuts). The discipline is not maintaining a spreadsheet; it is ensuring the register is:
- Continuously updated with new risks and changes to existing risks
- Reviewed regularly with clear ownership and accountability
- Tied to mitigation actions with due dates and verification
- Integrated into decision-making and planning

Without this discipline, risks accumulate invisibly until they materialize as incidents. A well-maintained risk register enables proactive risk management, informed tradeoffs, and defensible decisions about where to invest in resilience.

This document deepens item 15 in [`kali-task-research.md`](../kali-task-research.md:1): *“Risk register discipline: Maintain a living risk log with likelihood/impact scoring, mitigations, and clear escalation paths.”*

---

## 1. Background and context
Risks in a city twin program include:
- **Technical risks**: data quality issues, model failures, outages, security breaches
- **Operational risks**: process gaps, human error, vendor dependencies
- **Strategic risks**: misalignment with city priorities, funding cuts, political changes
- **External risks**: regulatory changes, extreme events, supply chain disruptions

Common failure patterns:
- Risk register created but never updated
- Risks identified but no mitigations assigned
- Mitigations assigned but never implemented
- No escalation path for high-severity risks

A disciplined risk register ensures:
- Risks are visible and tracked
- Mitigations are prioritized and implemented
- Escalation happens when risks exceed thresholds
- Risk posture is communicated to leadership

---

## 2. Stakeholders
- **Risk owners**: accountable for specific risks and their mitigations
- **Risk manager**: maintains the register, facilitates reviews
- **Technical teams**: identify and assess technical risks
- **Program leadership**: approve mitigations, allocate resources
- **Governance bodies**: review high-severity risks, provide oversight
- **Auditors**: verify risk management practices

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Integrity of risk assessments and mitigation status
- Confidentiality of sensitive risk information
- Availability of the risk register during incidents

### 3.2 Abuse/failure cases
- **Risk underreporting** to avoid scrutiny
- **Risk overreporting** to justify unnecessary spending
- **Stale risk data** leading to blind spots
- **Mitigations marked complete without verification

### 3.3 Controls
- Regular risk reviews with documented attendance
- Independent verification of mitigation completion
- Escalation triggers for high-severity risks
- Audit trail of risk changes and decisions

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Risk register database**
   - Risk ID, description, category, likelihood, impact, risk score
   - Mitigation actions, owners, due dates, status
   - Escalation paths and triggers

2. **Risk assessment workflow**
   - Risk identification and intake
   - Likelihood/impact scoring
   - Mitigation planning and approval

3. **Review and escalation**
   - Regular review cadence (weekly, monthly, quarterly)
   - Escalation to leadership for high-severity risks
   - Risk dashboard and reporting

4. **Mitigation tracking**
   - Action item tracking and verification
   - Risk re-assessment after mitigations

5. **Integration with other systems**
   - Link to incident management (risks that materialize)
   - Link to project management (mitigation projects)
   - Link to audit findings

### 4.2 Data flows
- Risk identification → assessment → register entry
- Regular review → risk updates → mitigation actions
- Mitigation completion → re-assessment → risk closure
- Escalation triggers → leadership review → resource allocation

---

## 5. Methods / best practices

### 5.1 Risk identification
- Brainstorming sessions with cross-functional teams
- Post-incident reviews to identify new risks
- External threat intelligence and industry trends
- Audit findings and compliance gaps

### 5.2 Likelihood and impact scoring
- Use consistent scales (e.g., 1–5 for likelihood and impact)
- Risk score = likelihood × impact
- Define criteria for each level (e.g., impact 5 = critical service outage)

### 5.3 Mitigation planning
- For each risk, define:
  - Mitigation actions (preventive, detective, corrective)
  - Owners and due dates
  - Cost and resource requirements
  - Success criteria

### 5.4 Escalation paths
- Define thresholds for escalation (e.g., risk score > 15)
- Identify escalation chain (owner → manager → leadership)
- Define response times for escalated risks

### 5.5 Regular reviews
- Weekly: high-severity risks and new risks
- Monthly: all risks, mitigation status
- Quarterly: risk posture review with leadership

External reference: NIST Special Publication 800-30 Rev. 1 provides guidance for conducting risk assessments ([NIST SP 800-30r1 PDF](https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-30r1.pdf)).

---

## 6. Data requirements
- Risk register with required fields (ID, description, category, likelihood, impact, score, owner, status)
- Mitigation action items with due dates and status
- Review logs with attendance and decisions
- Escalation records and responses

---

## 7. Implementation plan (phases)

### Phase 0 — Define risk framework
- Define risk categories and scoring scales
- Establish escalation paths and triggers
- Assign risk manager and governance structure

### Phase 1 — Initial risk register
- Conduct risk identification workshops
- Populate initial risk register
- Define mitigation actions for high-severity risks

### Phase 2 — Regular reviews and tracking
- Establish review cadence
- Implement mitigation tracking
- Create risk dashboard

### Phase 3 — Mature program
- Integrate with incident management and audits
- Automate risk reporting and escalation
- Continuously improve risk identification and assessment

---

## 8. Testing and validation
- Validate risk scoring consistency across teams
- Verify mitigation completion through independent review
- Test escalation paths with simulated high-severity risks
- Audit risk register for completeness and accuracy

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- % of risks with assigned owners
- % of mitigations completed on time
- Time from risk identification to mitigation start
- Number of risks escalated per quarter

### 9.2 Example SLOs
- 100% of high-severity risks reviewed weekly
- 90% of mitigations completed within due date

---

## 10. Governance and compliance
- Clear ownership and accountability for risks
- Regular reviews with documented attendance
- Alignment with organizational risk management frameworks
- Audit trail for risk decisions and mitigations

---

## 11. Risks and mitigations
- **Risk register becomes stale** → regular reviews and automated reminders
- **Risk underreporting** → anonymous risk submission, independent reviews
- **Mitigations not implemented** → tie to performance metrics, track completion
- **Escalation ignored** → define clear triggers and response times

---

## 12. Costs and FinOps
- Staff time for risk identification and reviews
- Tooling for risk register and tracking
- Mitigation implementation costs

Track:
- Cost per risk mitigated
- Cost per risk review cycle

---

## 13. KPIs
- Reduction in high-severity risks over time
- Increase in mitigations completed
- Reduction in incidents from known risks
- Risk posture improvement (average risk score)

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Risk register with required fields and data
- Risk assessment framework and scoring guide
- Escalation paths and triggers
- Risk dashboard and reporting
- Mitigation tracking system

### 14.2 Readiness checklist
- [ ] Risk categories and scoring scales defined
- [ ] Escalation paths documented
- [ ] Risk manager assigned
- [ ] Review cadence established

---

## 15. References
### 15.1 Workspace source
- Item 15 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- NIST — Guide for Conducting Risk Assessments (SP 800-30r1), PDF: https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-30r1.pdf

### 15.3 Suggested further reading (not fetched)
- Risk management frameworks (ISO 31000, COSO)
- Risk assessment matrices and scoring methods
- Risk communication and reporting best practices