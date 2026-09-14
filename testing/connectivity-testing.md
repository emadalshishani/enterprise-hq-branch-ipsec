# Inter-Site Connectivity Testing

## 1. Purpose

This document records the end-to-end connectivity testing performed after the HQ-to-Branch IPsec VPN was established.

The testing was used to verify that hosts behind the two SRXs could communicate across the encrypted inter-site connection.

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

This indicated that the problem was not limited to IKE or IPsec negotiation and required investigation of the routing path toward the Branch networks.

---

## 4. Routing Issue

The HQ-SRX initially contained a broad static route:

```text
set routing-options static route 192.168.0.0/16 next-hop 10.10.10.2
```

This route was removed:

```text
delete routing-options static route 192.168.0.0/16 next-hop 10.10.10.2
```

Specific routes for the HQ LAN networks were then configured:

```text
set routing-options static route 192.168.10.0/29 next-hop 10.10.10.2
set routing-options static route 192.168.20.0/29 next-hop 10.10.10.2
set routing-options static route 192.168.30.0/29 next-hop 10.10.10.2
```

The purpose of this change was to keep the HQ LAN routes specific to the actual networks behind the HQ Layer 3 switch.

---

## 5. Retest After Routing Correction

After the routing change, the HQ-to-Branch connectivity tests were repeated.

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

Both Branch networks were reachable successfully.

---

## 6. End-to-End Result

The connectivity testing confirmed bidirectional communication across the HQ-to-Branch VPN.

| Direction | Destination | Result |
|---|---|---|
| Branch → HQ | 192.168.10.2 | Successful |
| Branch → HQ | 192.168.10.3 | Successful |
| Branch → HQ | 192.168.20.2 | Successful |
| HQ → Branch | 192.168.50.3 | 100% |
| HQ → Branch | 192.168.60.3 | 100% |

The testing demonstrated that:

- The IPsec tunnel was operational.
- Branch hosts could reach HQ networks.
- HQ hosts could reach Branch networks.
- The initial reverse-direction failure was resolved through routing correction.
- Multiple LANs on both sites were reachable across the VPN.

---

## 7. Additional SRX Flow Verification

HQ-SRX flow sessions were also inspected using:

```text
show security flow session
```

The output showed active traffic associated with the inter-site communication, including:

- ESP traffic on `ge-0/0/1.0`
- IKE traffic using UDP/500
- ICMP sessions associated with the Branch security policy
- Traffic entering the SRX through the VPN-facing interface and being forwarded toward the internal interface

This provided additional evidence that the traffic was being processed by the SRX and associated with the inter-site VPN.

---

## 8. Next Test

After successful end-to-end connectivity was established, a path-failure test was performed at the Branch site.

The failover test is documented separately in:

```text
testing/failover-testing.md
```