# Incident response rehearsals — Deep research

## Executive summary
Incident response rehearsals (tabletop exercises, on-call drills, and simulations) are structured practice sessions that prepare teams to respond effectively to real incidents. In a city digital twin context, rehearsals use the twin’s data and simulation capabilities to create realistic scenarios, test decision-making, and identify gaps in processes, tools, and coordination.

The goal is to reduce downtime and improve outcomes during real incidents by:
- Building muscle memory and shared mental models
- Validating that tools and data are available when needed
- Testing communication and coordination across agencies
- Identifying and fixing process gaps before they matter

Rehearsals should be regular, varied, and tied to specific learning objectives. They require a governance framework to track findings, assign owners, and ensure improvements are implemented.

This document deepens item 14 in [`kali-task-research.md`](../kali-task-research.md:1): *“Incident response rehearsals: Run tabletop exercises and on-call drills to reduce downtime during real incidents.”*

---

## 1. Background and context
City-scale incidents include:
- Cybersecurity incidents (ransomware, data breaches)
- Infrastructure failures (power outages, water main breaks)
- Natural disasters (floods, heatwaves, storms)
- Public safety emergencies (large fires, mass casualty events)
- Public health emergencies (disease outbreaks)

Common failure modes during real incidents:
- Unclear roles and responsibilities
- Inaccessible or inaccurate data
- Poor communication across agencies
- Decision paralysis under time pressure
- Lack of tested fallback procedures

Rehearsals address these by:
- Creating safe-to-fail environments
- Exercising decision-making under stress
- Validating tooling and data availability
- Building relationships and trust across teams

---

## 2. Stakeholders
- **Incident commanders**: decision authority during incidents
- **On-call teams**: technical responders
- **Agency leadership**: escalation and resource allocation
- **Communications/public affairs**: public messaging
- **Legal/compliance**: regulatory obligations
- **IT/security**: tooling and infrastructure
- **External partners**: utilities, vendors, regional coordination

---

## 3. Threat model / abuse cases

### 3.1 Assets to protect
- Confidentiality of rehearsal scenarios (especially if based on real vulnerabilities)
- Integrity of findings and action items
- Availability of rehearsal infrastructure

### 3.2 Abuse/failure cases
- **Scenario leakage** revealing sensitive vulnerabilities
- **Rubber-stamping** rehearsals without meaningful engagement
- **Findings ignored** leading to repeated failures
- **Rehearsal fatigue** reducing participation

### 3.3 Controls
- Access controls for scenario materials
- Mandatory post-rehearsal reviews and action tracking
- Rotating facilitators to avoid groupthink
- Executive sponsorship and accountability

---

## 4. Reference architecture (components + data flows)

### 4.1 Components
1. **Scenario library**
   - Pre-built scenarios (cyber, infrastructure, natural disaster)
   - Custom scenarios based on recent near-misses

2. **Simulation engine**
   - Twin-based scenario execution (where applicable)
   - Injects and timeline management

3. **Rehearsal management**
   - Scheduling, participant invitations, role assignments
   - Facilitation tools and chat/communication

4. **Observation and capture**
   - Note-taking, timeline recording, decision logging
   - Performance metrics (response times, decision quality)

5. **After-action review (AAR)**
   - Findings, root causes, action items
   - Owner assignment and due dates

6. **Tracking and reporting**
   - Action item status, closure verification
   - Rehearsal metrics and trends

### 4.2 Data flows
- Scenario selection → rehearsal setup → execution
- Observations → AAR → action items
- Action items → implementation → verification
- Rehearsal data → metrics and program improvement

---

## 5. Methods / best practices

### 5.1 Types of rehearsals
- **Tabletop exercises (TTX)**: discussion-based, low-tech, focus on decision-making
- **Functional exercises**: operational, use actual tools and systems
- **Full-scale exercises**: multi-agency, realistic, resource-intensive
- **On-call drills**: test specific technical skills or procedures

### 5.2 Scenario design principles
- Realistic but not overly complex
- Clear objectives and success criteria
- Include injects to test decision-making under pressure
- Vary scenarios to cover different threat types

### 5.3 Facilitation and observation
- Neutral facilitator to manage pace and focus
- Observers to capture decisions and gaps
- Avoid “gotcha” moments; focus on learning

### 5.4 After-action review (AAR)
- What was expected vs what happened
- What went well and what didn’t
- Root causes of gaps
- Action items with owners and due dates

### 5.5 Continuous improvement
- Track action item closure rates
- Measure rehearsal participation and engagement
- Update scenarios based on real incidents and near-misses

External reference: CISA Tabletop Exercise Packages provide scenario templates and guidance for incident response exercises ([CISA TTEPs](https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages)).

---

## 6. Data requirements
- Scenario library with objectives and injects
- Participant roles and contact information
- Rehearsal schedules and attendance records
- AAR findings and action items
- Metrics: response times, decision quality, action closure rates

---

## 7. Implementation plan (phases)

### Phase 0 — Program governance
- Define rehearsal objectives and scope
- Establish executive sponsorship and accountability

### Phase 1 — Scenario library and baseline
- Develop initial scenarios for high-risk areas
- Conduct first tabletop exercises

### Phase 2 — Expand and integrate
- Add functional exercises and on-call drills
- Integrate twin-based simulations where applicable

### Phase 3 — Mature program
- Regular cadence (quarterly tabletops, annual functional exercises)
- Track metrics and demonstrate improvement
- Share lessons learned across agencies

---

## 8. Testing and validation
- Validate scenarios with subject matter experts
- Test rehearsal tools and infrastructure before exercises
- Survey participants for feedback
- Verify action items are implemented and effective

---

## 9. Observability (SLIs/SLOs)

### 9.1 SLIs
- % of scheduled rehearsals completed
- % of action items closed on time
- Participant satisfaction scores
- Time to complete AAR and assign action items

### 9.2 Example SLOs
- 100% of high-risk scenarios rehearsed annually
- 90% of action items closed within 90 days

---

## 10. Governance and compliance
- Executive sponsorship and budget
- Clear ownership of the rehearsal program
- Alignment with regulatory requirements (e.g., cybersecurity frameworks)
- Documentation for audits and reviews

---

## 11. Risks and mitigations
- **Rehearsal fatigue** → vary formats, keep sessions focused
- **Low participation** → executive mandate, schedule in advance
- **Findings ignored** → track and report to leadership
- **Scenarios outdated** → update based on real incidents

---

## 12. Costs and FinOps
- Staff time for planning, facilitation, and participation
- Tools for scenario management and AAR tracking
- External facilitators or consultants (optional)

Track:
- Cost per rehearsal
- Cost per action item closed

---

## 13. KPIs
- Reduction in incident response time (MTTD/MTTR)
- Reduction in repeat incidents
- Participant confidence and readiness scores
- Action item closure rate

---

## 14. Deliverables and checklists

### 14.1 Deliverables
- Scenario library with objectives and injects
- Rehearsal schedule and facilitation guide
- AAR templates and action tracking system
- Metrics dashboard and program reports

### 14.2 Readiness checklist
- [ ] Executive sponsorship established
- [ ] Scenario library developed
- [ ] Facilitators trained
- [ ] Action tracking system in place

---

## 15. References
### 15.1 Workspace source
- Item 14 in [`kali-task-research.md`](../kali-task-research.md:1)

### 15.2 External references (retrieved via Firecrawl MCP)
- CISA Tabletop Exercise Packages (TTEPs): https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages
- Ready.gov — Exercises: https://www.ready.gov/business/training/testing-exercise/exercises

### 15.3 Suggested further reading (not fetched)
- Incident command system (ICS) training and certification
- Cybersecurity incident response frameworks (NIST, ISO)
- After-action review best practices