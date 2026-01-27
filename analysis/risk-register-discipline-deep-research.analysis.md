# Analysis — Risk register discipline

Source: [`risk-register-discipline-deep-research.md`](../themes/risk-register-discipline-deep-research.md:1)

## Relevance
Risk register discipline is highly relevant for a city digital twin program because it is a multi-year, multi-stakeholder initiative with technical, operational, political, and ethical risks. Without a living risk management process, risks remain invisible until they materialize as outages, privacy incidents, governance failures, or loss of funding/public trust. This topic supports resilience and accountability across all other themes.

## Depth / maturity
The document is a straightforward operational governance template:
- Defines risk register purpose beyond “a spreadsheet” and emphasizes ownership, cadence, mitigation verification, and integration with decision-making.
- Covers stakeholders (risk owners, risk manager, leadership, auditors).
- Includes a basic threat model (underreporting, overreporting, stale data, unverified completion).
- Provides a simple architecture and integration points (incident management, project management, audits).
- Includes a phase plan, SLIs/SLOs, KPIs.

Maturity is good for program management basics, but it does not connect the risk register to the specific risk landscape of city twins (privacy, inference risk, vendor lock-in, model failures, cross-agency governance).

## Gaps / blind spots
1) **Risk taxonomy tailored to city digital twins**
It lists generic categories (technical/operational/strategic) without a twin-specific taxonomy:
- privacy and civil liberties risks (linkability, function creep)
- model risk and automation harms
- cross-agency data sharing/legal risks
- vendor and procurement risks (lock-in, escrow, exit)

2) **Quantification and thresholds**
Likelihood/impact scoring is described, but missing:
- explicit definitions per score level
- risk appetite statements and escalation criteria beyond a single example

3) **Verification rigor**
It mentions independent verification but not:
- what evidence is required to close a mitigation (tests, audits, drills)
- how to prevent “paper compliance”

4) **Integration with observability and incident response**
It links to incident management conceptually, but missing:
- feedback loops from SLO breaches and incidents into new risks
- how to tie risks to measurable indicators (leading indicators)

5) **Governance alignment**
No linkage to decision bodies:
- data governance council, security board, ethics board
- how risk decisions are documented in ADRs

## Latent research needs (only as implied by the gaps)
- **Twin-specific risk taxonomy and templates** (privacy, model risk, vendor risk, governance risk) that teams can reuse.
- **Evidence-based closure criteria**: what proof is required to consider a risk mitigated.
- **Leading indicator mapping**: connect risks to observable metrics and rehearsal outcomes.

## Contribution to collective sensemaking
This topic contributes a shared operational discipline that complements technical themes. It helps translate “we should care about X” into accountable actions with owners and deadlines.

In the collective portfolio, its value increases when cross-referenced with governance/privacy, observability, and incident rehearsals: those provide concrete risk sources and measurable signals; the risk register provides the program mechanism to ensure they are addressed and not forgotten.