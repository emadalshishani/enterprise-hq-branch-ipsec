# IPsec VPN Verification

## 1. Purpose

This document records the verification performed after configuring the policy-based IPsec VPN between HQ-SRX and Branch-SRX.

The verification confirms that:

- IKE Phase 1 established successfully.
- IPsec Phase 2 established successfully.
- Both SRXs reported an active IPsec tunnel.
- The two SRXs identified each other using the expected WAN addresses.

---

## 2. IKE Phase 1 Verification

IKE security associations were checked on both SRXs using:

```text
show security ike security-associations
```

### Branch-SRX

The Branch-SRX reported the IKE security association as `UP`:

```text
Index   State  Initiator cookie  Responder cookie  Mode   Remote Address
1512635 UP     a296c603e4746996  e3c175f78f62f6ee  Main   10.10.30.1
```

This confirmed that Branch-SRX successfully established IKE Phase 1 with HQ-SRX.

### HQ-SRX

The HQ-SRX also reported the IKE security association as `UP`:

```text
Index   State  Initiator cookie  Responder cookie  Mode   Remote Address
5325115 UP     a296c603e4746996  e3c175f78f62f6ee  Main   10.10.30.2
```

The matching IKE cookies confirmed that both SRXs were participating in the same established IKE session.

---

## 3. IPsec Phase 2 Verification

IPsec security associations were checked using:

```text
show security ipsec security-associations
```

### Branch-SRX

Branch-SRX reported:

```text
Total active tunnels: 1
Total Ipsec sas: 1
```

The active SAs used:

```text
ESP:aes-cbc-128/sha1
```

with HQ-SRX as the remote gateway:

```text
10.10.30.1
```

This confirmed that the IPsec Phase 2 security associations were established.

### HQ-SRX

HQ-SRX reported:

```text
Total active tunnels: 1
Total Ipsec sas: 1
```

The active SAs used:

```text
ESP:aes-cbc-128/sha1
```

with Branch-SRX as the remote gateway:

```text
10.10.30.2
```

---

## 4. Verification Result

The verification confirmed the following state:

| Verification | Result |
|---|---|
| HQ-SRX IKE SA | UP |
| Branch-SRX IKE SA | UP |
| HQ-SRX IPsec SA | Active |
| Branch-SRX IPsec SA | Active |
| Active IPsec tunnels | 1 |
| VPN peer reachability | Established |

At this stage, the VPN control plane was successfully established.

The next verification stage was end-to-end traffic testing between hosts in the HQ and Branch LANs.

---

## 5. Verification Commands

The primary commands used during verification were:

```text
show security ike security-associations
show security ipsec security-associations
```

These commands were used to distinguish between IKE Phase 1 establishment and IPsec Phase 2 establishment.

---

## 6. Verification Notes

The tunnel establishment confirmed that the VPN negotiation and security associations were operational.

However, an established IPsec tunnel alone does not prove that end-to-end communication between the HQ and Branch LANs is working.

End-to-end connectivity was therefore tested separately and documented in:

```text
testing/connectivity-testing.md
```