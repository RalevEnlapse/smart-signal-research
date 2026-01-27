# Analysis — Smart lighting

Source: [`smart-lighting-deep-research.md`](../themes/smart-lighting-deep-research.md:1)

## Relevance
Smart lighting is a strong “early win” smart-city domain because it is:
- **Asset-heavy and measurable** (fixtures, poles, power draw, maintenance tickets).
- **Operationally actionable** with clear levers (dimming profiles, schedules, zonal control).
- **Cross-domain**: lighting affects public safety perceptions, night-time economy, mobility corridors, and can host sensor “mount points” for other systems.

Within a city digital twin, lighting fits naturally as a **physical infrastructure twin** (state + health + control) with direct links to energy management and public works.

## Depth / maturity
The document is relatively mature as a high-level implementation guide:
- Good architecture layers (sensing, control, comms, edge, cloud, apps, integration).
- Explicit standards/protocols (DALI, Zhaga Book 18, TALQ, IEC 62386) that matter for real deployments.
- Incorporates operational concerns (predictive maintenance, safe fallback modes).

Limitations are mainly in the “last mile” of engineering and governance:
- how to choose comms topologies and ensure reliability in dense/complex environments
- how to handle privacy when “traffic/pedestrian presence” sensing is used

## Gaps / blind spots
1) **Procurement + interoperability strategy**
Standards are listed, but missing:
- guidance on vendor selection criteria (TALQ compliance depth, OTA update capability)
- interoperability testing plans and conformance tooling

2) **Cybersecurity details for critical infrastructure**
Controls are generic (TLS/VPN/RBAC). Missing:
- key management and device identity lifecycle
- secure firmware/OTA update pipeline, rollback, and SBOM expectations
- segmentation and incident response playbooks for field networks

3) **Privacy model for sensing**
The theme uses motion/traffic sensing and mentions crime hotspots. Missing:
- explicit boundary between non-identifying sensors vs cameras
- governance for using safety/policing datasets to drive lighting policy

4) **Illumination quality and environmental impact trade-offs**
Energy savings are emphasized; missing:
- glare, color temperature, light pollution, biodiversity effects
- standards for illumination levels and community acceptability thresholds

5) **Edge vs cloud split and failure modes**
Edge is mentioned, but not specified:
- which decisions must run locally (latency-critical safety constraints)
- how to maintain consistency across zones under partial network outages

6) **Measurement and verification (M&V)**
SLOs/SLIs include savings and uptime, but missing:
- baseline methodology (pre/post, normalization, seasonality)
- independent verification and auditability for claimed savings

## Latent research needs (only as implied by the gaps)
- **Security-by-design for field IoT**: reference patterns for device identity, OTA updates, and segmentation appropriate for municipal scale.
- **Privacy-preserving sensing**: guidance on selecting sensors and processing pipelines that avoid person-identification while enabling adaptive control.
- **Environmental illumination policy**: linking smart lighting control to light pollution and biodiversity metrics.

## Contribution to collective sensemaking
This topic is a concrete example of a city twin component with clear physical assets, measurable KPIs, and established standards. It helps the collective anchor abstract “digital twin” discussions into practical infrastructure modernization.

Its biggest contribution is the standards-aware architecture framing. Its biggest opportunity is to deepen cybersecurity, privacy boundaries, and M&V rigor so the theme supports trustworthy, auditable deployments at city scale.