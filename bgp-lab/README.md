# Enterprise Multi-AS BGP Lab

## Overview

This section documents the expanded enterprise/service-provider style lab built around six autonomous systems, multi-vendor routing platforms, BGP interconnection, IGP domains, and failure testing.

The current topology combines Juniper SRX/Junos and Cisco IOS platforms.

## Autonomous Systems

| AS | Role | Platform |
|---|---|---|
| AS100 | HQ | Juniper SRX / Junos |
| AS200 | Branch | Juniper SRX / Junos |
| AS1000 | Enterprise transit/core | Juniper SRX / Junos |
| AS2000 | Enterprise transit/core | Cisco IOS |
| AS3000 | ISP1 | Cisco IOS |
| AS4000 | ISP2 | Cisco IOS |

## Routing Model

- HQ (AS100) ↔ AS1000: eBGP
- Branch (AS200) ↔ AS2000: eBGP
- AS1000 internal routing: IS-IS + iBGP
- AS2000 internal routing: OSPF + iBGP
- AS1000 ↔ AS3000/AS4000: eBGP
- AS2000 ↔ AS3000/AS4000: eBGP
- AS3000 ↔ AS4000: eBGP
- ISP1 and ISP2 use the upstream lab Internet gateway 10.142.13.1

## Repository Contents

### Configurations
Sanitized device configurations are stored under `configs/`.

Credentials and encrypted authentication material are intentionally redacted before publication.

The retired AS1000 R2 references were removed from the published AS1000 configurations, including the R2-related 10.2.3.0/30 and 10.2.4.0/30 interfaces and 2.2.2.2 iBGP peer references.

### Design Documentation
- `docs/architecture.md`
- `docs/addressing.md`
- `docs/routing-design.md`
- `docs/lab-status.md`

### Testing
- `testing/bgp-verification.md`
- `testing/failover-testing.md`

## Verified Failure Testing

The current evidence includes:

1. R5 ↔ ISP1 eBGP link failure
2. R6 ↔ ISP1 eBGP link failure
3. End-to-end ISP2 failure with traffic convergence to ISP1

The end-to-end test used continuous ICMP from HQ PC2 (192.168.10.3) to 8.8.8.8 and a pre/post traceroute comparison.

No claim of hitless failover is made; the documented ISP2 failure test observed packet loss during convergence.

## Related Work

The multi-AS lab builds on the earlier:
- Enterprise HQ
- Enterprise Branch
- HQ–Branch policy-based IPsec VPN

Those projects remain in the repository as their own historical stages.
