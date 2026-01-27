# Analysis — Digital signage

Source: [`digital-signage-deep-research.md`](../themes/digital-signage-deep-research.md:1)

## Relevance
Digital signage is moderately relevant to a city digital twin portfolio. It is a city-wide “actuation/communication” channel rather than a modeling capability, but it becomes strategically important when integrated with real-time city data (mobility, events, hazards) and with emergency management (CAP alerts). If treated as a standalone kiosk/signage project, it risks being peripheral; if treated as an output surface for twin-driven situational awareness and public communication, it is more central.

## Depth / maturity
The document is mature as a generic capability outline:
- Identifies core outputs: content management, interactive wayfinding, emergency broadcast, engagement analytics.
- Provides a reasonable component architecture (hardware, media players, network, CMS, integrations, analytics, remote management) and flows.
- Includes basic threat/abuse cases (unauthorized content, DoS, spoofing, privacy violations) and controls.
- Includes operational metrics (latency, uptime) and phased deployment.

Relative to “deep research” standards in this repo, it is not very deep: it lacks hard choices around governance, security posture, and what analytics are appropriate/ethical.

## Gaps / blind spots
1) **Emergency broadcasting governance and safety**
CAP integration is mentioned, but missing:
- priority override governance (who can trigger, under what conditions)
- testing/drills and failure modes (what if network is down)
- consistent messaging across channels (signage, SMS, app, broadcast)

2) **Security posture for edge devices**
Signage networks are a common attack surface. Missing:
- secure boot, device attestation, patch cadence
- segmentation of signage management plane
- incident response playbooks for content compromise

3) **Privacy and legality of “viewer analytics”**
The doc references demographics and engagement analytics without defining:
- whether cameras are used and how data is processed (on-device vs cloud)
- anonymization standards, opt-in/notice requirements
- what constitutes acceptable collection in public spaces

4) **Accessibility beyond WCAG**
WCAG is referenced, but physical/public interfaces require:
- multilingual support, screen height/visibility considerations
- support for low vision and hearing impairment (audio, contrast, tactile cues)

5) **Content governance and misinformation risk**
Content approval workflows are mentioned, but missing:
- editorial policies, accountability, audit trails
- safeguards against politicization or biased “targeted messaging”

6) **Integration with the digital twin platform**
It lists data feeds (traffic, weather, events) but does not specify:
- canonical data contracts and caching strategy
- “degraded mode” content behavior during outages

## Latent research needs (only as implied by the gaps)
- **Public-space analytics governance**: privacy-by-design patterns for engagement measurement that avoid biometric/demographic inference.
- **Secure edge fleet management**: reference security architecture for thousands of unattended devices with strong update and attestation discipline.
- **Emergency comms reliability**: end-to-end testing patterns and redundancy strategies integrating CAP with multiple channels.

## Contribution to collective sensemaking
This topic highlights a city-scale “information actuation” layer: how insights and alerts reach people. It can complement hazard, mobility, and resilience themes by providing a concrete public interface and a mechanism for operational transparency. However, to contribute strongly, it needs tighter coupling to the twin’s data products and much more explicit governance around security, emergency authority, and privacy.