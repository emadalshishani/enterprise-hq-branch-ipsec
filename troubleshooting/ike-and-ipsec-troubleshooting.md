# IKE and IPsec Troubleshooting

## 1. Purpose

This document records the troubleshooting process performed while establishing the policy-based IPsec VPN between HQ-SRX and Branch-SRX.

The purpose is to document the actual issues encountered during implementation, the verification performed, and the changes that ultimately resulted in a successful VPN establishment.

---

## 2. Initial Condition

The HQ-SRX and Branch-SRX were configured with matching high-level IKE and IPsec parameters.

However, the VPN did not initially establish.

The troubleshooting process therefore started by verifying the IKE and IPsec configuration and checking the security associations on both SRXs.

---

## 3. IKE Diffie-Hellman Group

During the initial comparison of the IKE proposals, the HQ-SRX had an explicit Diffie-Hellman group configured:

```text
set security ike proposal ike-prob dh-group group2
```

The Branch-SRX did not initially contain the corresponding IKE DH group.

The Branch configuration was updated to:

```text
set security ike proposal ike-prop dh-group group2
```

This aligned the IKE DH configuration between the two peers.

---

## 4. IPsec PFS Adjustment

During troubleshooting, the IPsec policy on both devices initially used:

```text
perfect-forward-secrecy keys group20
```

The PFS setting was changed on both SRXs to:

```text
perfect-forward-secrecy keys group2
```

This change was made as part of the troubleshooting process while aligning and verifying the Phase 2 configuration.

The change should be understood as a configuration adjustment made during troubleshooting rather than as a general requirement that the IPsec PFS group must match the IKE DH group.

---

## 5. IPsec Security Association Check

After the IKE and IPsec parameters were adjusted, the VPN was still not established.

The following commands were used on Branch-SRX:

```text
show security ike security-associations
```

No active IKE security association was present.

IPsec was also checked:

```text
show security ipsec security-associations
```

The result showed:

```text
Total active tunnels: 0
Total Ipsec sas: 0
```

At this point, the configuration parameters had been reviewed, but the VPN still had no active security associations.

---

## 6. Root Cause: Security Policy Tunnel Action

The next stage of troubleshooting focused on the security policies associated with the policy-based VPN.

The relevant policies were allowing traffic but were not yet associated with the IPsec VPN through a tunnel action.

The required action was added to the security policies:

```text
then permit tunnel ipsec-vpn ipsec-vpn
```

This was configured for the relevant internal-to-remote-zone policies on both HQ-SRX and Branch-SRX.

This was the key change that allowed the policy-based VPN traffic to be associated with the configured IPsec VPN.

---

## 7. Successful IKE Establishment

After the security policy tunnel action was configured, the IKE security association became active.

Branch-SRX:

```text
show security ike security-associations
```

Result:

```text
Index   State  Initiator cookie  Responder cookie  Mode   Remote Address
1512635 UP     a296c603e4746996  e3c175f78f62f6ee  Main   10.10.30.1
```

The IKE state changed to:

```text
UP
```

The remote peer was:

```text
10.10.30.1
```

---

## 8. Successful IPsec Establishment

The IPsec security associations were then verified.

Branch-SRX reported:

```text
Total active tunnels: 1
Total Ipsec sas: 1
```

The active SAs used:

```text
ESP:aes-cbc-128/sha1
```

with HQ-SRX as the remote gateway.

HQ-SRX also reported:

```text
Total active tunnels: 1
Total Ipsec sas: 1
```

with Branch-SRX as the remote gateway.

This confirmed successful establishment of both the IKE and IPsec security associations.

---

## 9. Troubleshooting Summary

The troubleshooting sequence was:

```text
VPN did not establish
        |
        v
Compare IKE proposals
        |
        v
Add missing IKE DH group on Branch-SRX
        |
        v
Adjust IPsec PFS during configuration verification
        |
        v
Check IKE/IPsec security associations
        |
        v
No active tunnel
        |
        v
Review policy-based VPN security policies
        |
        v
Add "permit tunnel ipsec-vpn"
        |
        v
IKE SA established
        |
        v
IPsec SA established
```

---

## 10. Lessons Learned

The troubleshooting process demonstrated several important points about policy-based IPsec on Junos:

- Matching IKE and IPsec parameters must be verified on both peers.
- IKE Phase 1 and IPsec Phase 2 are separate negotiation stages.
- IKE DH and IPsec PFS are separate parameters.
- An established VPN configuration still depends on the security policy correctly associating traffic with the IPsec VPN.
- `show security ike security-associations` and `show security ipsec security-associations` are useful for distinguishing between Phase 1 and Phase 2 problems.
- Tunnel establishment alone does not prove end-to-end LAN connectivity; routing and traffic testing must be verified separately.
