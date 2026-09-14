# Branch Path Failover Testing

## 1. Purpose

This document records the Branch path-failure test performed after establishing end-to-end HQ-to-Branch connectivity.

The objective was to verify that Branch traffic could recover through the alternate Layer 3 path when the primary Branch-SRX connection was unavailable.

---

## 2. Test Scenario

The test was performed using a continuous ping from Branch-PC4 toward an HQ host:

```text
Branch-PC4> ping 192.168.30.2 -t
```

During the test, the connection between L3-SW-Branch2 and Branch-SRX was intentionally shut down.

The affected interface was:

```text
L3-SW-Branch2
Ethernet0/0
```

This interface connects L3-SW-Branch2 toward Branch-SRX.

---

## 3. Failure Injection

The interface was administratively disabled:

```text
interface ethernet 0/0
shutdown
```

The switch reported the interface going down:

```text
%LINK-5-CHANGED: Interface Ethernet0/0, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/0, changed state to down
```

The OSPF adjacency with the neighboring Branch device also transitioned from `FULL` to `DOWN`:

```text
%OSPF-5-ADJCHG: Process 1, Nbr 10.10.20.1 on Ethernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
```

This confirmed that the failure was detected by the routing layer.

---

## 4. Connectivity During Failure

Before the failure, the continuous ping was successful:

```text
84 bytes from 192.168.30.2 icmp_seq=1 ttl=60 time=8.663 ms
84 bytes from 192.168.30.2 icmp_seq=2 ttl=60 time=4.780 ms
84 bytes from 192.168.30.2 icmp_seq=3 ttl=60 time=3.637 ms
```

During convergence, three consecutive packets timed out:

```text
192.168.30.2 icmp_seq=4 timeout
192.168.30.2 icmp_seq=5 timeout
192.168.30.2 icmp_seq=6 timeout
```

Connectivity then recovered:

```text
84 bytes from 192.168.30.2 icmp_seq=7 ttl=60 time=4.537 ms
```

Additional successful replies followed after the path recovered.

---

## 5. Recovery

The failed interface was restored using:

```text
interface ethernet 0/0
no shutdown
```

The network returned to its normal state after the interface was restored.

The observed behavior demonstrated that the Branch network could recover from the loss of the tested primary path and continue forwarding traffic through the available alternate path.

---

## 6. Test Result

| Test Item | Result |
|---|---|
| Continuous ping before failure | Successful |
| Branch-SRX link failure | Detected |
| OSPF adjacency | FULL → DOWN |
| Packet loss during convergence | 3 packets |
| Connectivity after convergence | Recovered |
| Interface restoration | Successful |

### Result: PASS

The test successfully demonstrated path failover and recovery.

However, the failover was **not hitless** because three ICMP packets were lost during routing convergence.

---

## 7. Important Scope

This test validates **Branch Layer 3 path failover**.

It does not represent a failure of the IPsec tunnel endpoints themselves.

The tested failure was:

```text
L3-SW-Branch2
      X
Branch-SRX
```

The IPsec VPN remained the inter-site security mechanism while the Branch internal path was being tested.

---

## 8. Observed Routing Behavior

During recovery, the ping output included:

```text
Redirect Network, gateway 192.168.60.1 -> 192.168.60.6
```

This output was observed during the test and is recorded as part of the captured behavior.

No additional conclusion is drawn from the ICMP redirect message without further packet-level investigation.

---

## 9. Conclusion

The Branch redundancy design successfully recovered inter-site connectivity after the tested Branch-SRX path became unavailable.

The test provided practical evidence that:

- OSPF detected the failed path.
- The routing topology reconverged.
- Traffic recovered through an alternate path.
- End-to-end HQ connectivity was restored.
- A short convergence-related packet loss occurred during failover.