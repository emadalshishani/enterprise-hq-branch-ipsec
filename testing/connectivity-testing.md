# Inter-Site Connectivity Testing

## 1. Purpose

This document records the end-to-end connectivity testing performed after establishing the HQ-to-Branch policy-based IPsec VPN.

The testing was used to verify that hosts behind the two SRXs could communicate across the encrypted inter-site connection and to identify and resolve a routing issue encountered during the initial HQ-to-Branch tests.

---

## 2. Branch-to-HQ Connectivity

Connectivity was first tested from a Branch host toward hosts in the HQ LAN.

The test was performed from Branch-PC4.

Successful ping tests were recorded toward the following HQ hosts:

```text
192.168.10.2
192.168.10.3
192.168.20.2
```

All tested destinations responded successfully.

This confirmed that traffic originating from the Branch LAN could cross the IPsec connection and reach multiple HQ networks.

---

## 3. Initial HQ-to-Branch Test

The reverse direction was then tested from the HQ Layer 3 switch toward Branch hosts.

The initial tests failed:

```text
L3-SW-HQ# do ping 192.168.60.3
Success rate is 0 percent (0/5)

L3-SW-HQ# do ping 192.168.50.3
Success rate is 0 percent (0/5)
```

At this stage, the IPsec tunnel itself was already established.

This indicated that the problem was not with IKE or IPsec tunnel establishment and required investigation of the routing path toward the Branch networks.

---

## 4. Routing Issue

The routing table on HQ-SRX was investigated to determine why traffic destined for the Branch networks was not being forwarded through the VPN.

### Initial HQ-SRX Routing

HQ-SRX had the following routes:

```text
192.168.0.0/16 → 10.10.10.2
0.0.0.0/0      → 10.10.30.2
```

The `192.168.0.0/16` route was intended to provide reachability toward the internal HQ networks through L3-SW-HQ.

However, the Branch networks:

```text
192.168.50.0/29
192.168.60.0/29
```

are also contained within the `192.168.0.0/16` address space.

### Routing Decision

When HQ-SRX received a packet destined for a Branch host such as:

```text
192.168.50.3
```

the destination matched both routes:

```text
192.168.0.0/16
0.0.0.0/0
```

Junos uses longest-prefix match when selecting a route. Therefore, the `/16` route was preferred because it is more specific than the default `/0` route.

The routing decision was effectively:

```text
Destination: 192.168.50.3
        |
        +--> 192.168.0.0/16  ✓
        |
        +--> 0.0.0.0/0       ✓
                 |
                 v
       Longest Prefix Match
                 |
                 v
        Select 192.168.0.0/16
                 |
                 v
        Next-hop 10.10.10.2
                 |
                 v
            L3-SW-HQ
```

The same routing decision applied to traffic destined for `192.168.60.3`.

As a result, HQ-SRX forwarded the traffic back toward L3-SW-HQ through `10.10.10.2` instead of forwarding it toward the Branch-SRX VPN peer at `10.10.30.2`.

The IPsec tunnel itself was therefore operational, but the traffic was being directed toward the wrong next hop by the routing table.

### Routing Correction

The broad `/16` route was removed:

```text
delete routing-options static route 192.168.0.0/16 next-hop 10.10.10.2
```

It was replaced with specific routes for the actual HQ LAN networks:

```text
set routing-options static route 192.168.10.0/29 next-hop 10.10.10.2
set routing-options static route 192.168.20.0/29 next-hop 10.10.10.2
set routing-options static route 192.168.30.0/29 next-hop 10.10.10.2
```

This maintained reachability to the HQ LANs while removing the overlapping route that also matched the Branch networks.

With the `/16` route removed, traffic destined for the Branch networks no longer matched a more-specific HQ route and could use the default route toward:

```text
10.10.30.2
```

which is the Branch-SRX VPN peer.

This allowed the traffic to be forwarded toward the policy-based IPsec VPN.

---

## 5. Retest After Routing Correction

After the routing correction, the HQ-to-Branch connectivity tests were repeated.

### Branch VLAN 50

```text
L3-SW-HQ# do ping 192.168.50.3
Success rate is 100 percent (5/5)
round-trip min/avg/max = 2/3/5 ms
```

### Branch VLAN 60

```text
L3-SW-HQ# do ping 192.168.60.3
Success rate is 100 percent (5/5)
round-trip min/avg/max = 3/4/7 ms
```

Both Branch networks were successfully reachable from the HQ Layer 3 switch.

---

## 6. End-to-End Result

The connectivity testing confirmed bidirectional communication across the HQ-to-Branch VPN.

| Direction   | Destination  | Result     |
| ----------- | ------------ | ---------- |
| Branch → HQ | 192.168.10.2 | Successful |
| Branch → HQ | 192.168.10.3 | Successful |
| Branch → HQ | 192.168.20.2 | Successful |
| HQ → Branch | 192.168.50.3 | 100%       |
| HQ → Branch | 192.168.60.3 | 100%       |

The testing demonstrated that:

* The IPsec tunnel was operational.
* Branch hosts could reach multiple HQ networks.
* HQ hosts could reach multiple Branch networks.
* The initial HQ-to-Branch failure was caused by an overlapping broad static route on HQ-SRX.
* The routing issue was resolved by replacing the broad `/16` route with specific HQ LAN routes.
* Multiple LANs on both sites were reachable across the VPN.

---

## 7. Additional SRX Flow Verification

HQ-SRX flow sessions were also inspected using:

```text
show security flow session
```

The output showed active traffic associated with the inter-site communication, including:

* ESP traffic on `ge-0/0/1.0`
* IKE traffic using UDP/500
* ICMP sessions associated with the Branch security policy
* Traffic entering the SRX through the VPN-facing interface and being forwarded toward the internal interface

This provided additional evidence that the traffic was being processed by the SRX and associated with the inter-site VPN.

---

## 8. Next Test

After successful end-to-end connectivity was established, a path-failure test was performed at the Branch site.

The failover test is documented separately in:

```text
testing/failover-testing.md
```
