# Multi-AS BGP Lab Reference

## Scope

This document records the current multi-autonomous-system lab design and the verified operational behavior available for documentation.

The lab uses six autonomous systems and combines Juniper SRX/Junos and Cisco IOS platforms.

## Autonomous Systems

| AS | Role |
|---|---|
| AS100 | HQ |
| AS200 | Branch |
| AS1000 | Enterprise transit/core |
| AS2000 | Enterprise transit/core |
| AS3000 | ISP1 |
| AS4000 | ISP2 |

## High-Level Relationship

- HQ (AS100) peers with AS1000 using eBGP.
- Branch (AS200) peers with AS2000 using eBGP.
- AS1000 and AS2000 provide the intermediate transit paths.
- AS1000 and AS2000 connect to both ISP1 (AS3000) and ISP2 (AS4000).
- ISP1 and ISP2 have an eBGP relationship between them.
- ISP1 and ISP2 use the upstream gateway 10.142.13.1 in the lab Internet segment.

## AS1000

Operational documentation includes R1, R3, R4, R5, and R6.

Internal links documented from the lab:
- R1-R3: 10.1.3.0/30
- R1-R4: 10.1.4.0/30
- R3-R5: 10.3.5.0/30
- R3-R4: 10.3.4.0/30
- R4-R6: 10.4.6.0/30
- R5-R6: 10.5.6.0/30

External roles:
- R1 ↔ HQ (AS100)
- R5/R6 ↔ ISP1 (AS3000)
- R5/R6 ↔ ISP2 (AS4000)

## AS2000

The Branch side uses AS2000 as the BGP transit domain between the Branch edge and the upstream ISP domains.

- Branch (AS200) peers with PE1 in AS2000.
- AS2000 has internal iBGP and external connectivity to ISP1 (AS3000) and ISP2 (AS4000).
- The observed routing state includes default-route learning through ISP2 with alternate reachability through ISP1.

## AS3000 — ISP1

Cisco IOS 15.7.

Verified external BGP relationships:
- Two peers in AS1000
- Two peers in AS2000
- One peer in AS4000

The lab ISP1 router also has a static default route toward the upstream lab Internet gateway:
- 0.0.0.0/0 → 10.142.13.1

## AS4000 — ISP2

Cisco IOS 15.7.

Verified external BGP relationships:
- Two peers in AS1000
- Two peers in AS2000
- One peer in AS3000

The lab ISP2 router also has a static default route toward:
- 10.142.13.1

## HQ and Branch

### HQ — AS100
Juniper SRX / Junos.

The current BGP edge is:
- Local: 10.10.30.1
- Peer: 10.10.30.2
- Peer AS: 1000

HQ advertises the configured VLAN10 prefix through an export policy.

### Branch — AS200
Juniper SRX / Junos.

The current BGP edge is:
- Local: 10.10.40.1
- Peer: 10.10.40.2
- Peer AS: 2000

Branch participates in OSPF internally toward its local routing domain.

## Verified BGP State

The captured verification showed established BGP sessions across the active topology and route learning across the AS boundaries.

Examples include:
- HQ receiving enterprise/ISP routes through AS1000.
- Branch receiving external routes through AS2000.
- AS1000 learning and propagating routes toward both ISP domains.
- AS2000 learning and propagating routes toward both ISP domains.
- ISP1 and ISP2 maintaining eBGP connectivity to the enterprise ASes.

## Operational Notes

This document intentionally records the current lab behavior without adding unverified design assumptions.

Detailed command output and failure evidence are maintained in the dedicated verification and failover documents.
