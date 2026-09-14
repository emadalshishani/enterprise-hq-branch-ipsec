# HQ-SRX IPsec Configuration

## Purpose

This document contains the IPsec-related configuration implemented on the **HQ-SRX** to establish the site-to-site VPN with the Branch SRX.

The configuration is limited to the components directly related to the VPN integration:

- IKE Phase 1
- IKE gateway
- IPsec Phase 2
- IPsec VPN
- Security policy required for the policy-based VPN

Sensitive authentication material is intentionally excluded.

---

## VPN Peer

| Parameter | Value |
|---|---|
| Local SRX | HQ-SRX |
| Local WAN Interface | ge-0/0/1.0 |
| Local Address | 10.10.30.1 |
| Remote SRX | Branch-SRX |
| Remote Address | 10.10.30.2 |

---

## IKE Phase 1

The HQ-SRX uses a pre-shared-key based IKE configuration.

### IKE Proposal

```text
set security ike proposal ike-prob authentication-method pre-shared-keys
set security ike proposal ike-prob dh-group group2
set security ike proposal ike-prob authentication-algorithm sha-256
set security ike proposal ike-prob encryption-algorithm aes-256-cbc
```

### IKE Policy

```text
set security ike policy ike-policy mode main
set security ike policy ike-policy proposals ike-prob
```

The pre-shared key is configured on the device but is intentionally not included in this repository.

### IKE Gateway

```text
set security ike gateway ike-GW ike-policy ike-policy
set security ike gateway ike-GW address 10.10.30.2
set security ike gateway ike-GW external-interface ge-0/0/1.0
set security ike gateway ike-GW local-address 10.10.30.1
```

---

## IPsec Phase 2

### IPsec Proposal

```text
set security ipsec proposal ipsec-prob protocol esp
set security ipsec proposal ipsec-prob authentication-algorithm hmac-sha1-96
set security ipsec proposal ipsec-prob encryption-algorithm aes-128-cbc
set security ipsec proposal ipsec-prob lifetime-seconds 180
```

### IPsec Policy

```text
set security ipsec policy ipsec-policy perfect-forward-secrecy keys group2
set security ipsec policy ipsec-policy proposals ipsec-prob
```

### IPsec VPN

```text
set security ipsec vpn ipsec-vpn ike gateway ike-GW
set security ipsec vpn ipsec-vpn ike ipsec-policy ipsec-policy
set security ipsec vpn ipsec-vpn establish-tunnels immediately
```

---

## Policy-Based VPN Security Policy

The VPN is policy-based, so the security policy between the HQ internal zone and the Branch zone includes the IPsec tunnel action.

```text
set security policies from-zone internal to-zone branch policy internal-branch match source-address any
set security policies from-zone internal to-zone branch policy internal-branch match destination-address any
set security policies from-zone internal to-zone branch policy internal-branch match application any
set security policies from-zone internal to-zone branch policy internal-branch then permit tunnel ipsec-vpn ipsec-vpn
```

The `then permit tunnel ipsec-vpn ipsec-vpn` action is required for the policy-based VPN traffic to use the configured IPsec tunnel.

---

## Troubleshooting Note

During the initial VPN troubleshooting, the HQ-SRX already had the following IKE Diffie-Hellman configuration:

```text
set security ike proposal ike-prob dh-group group2
```

The Branch-SRX initially did not have an IKE DH group configured. Group 2 was subsequently added to the Branch configuration.

The IPsec PFS setting was also changed from Group 20 to Group 2 on both peers during troubleshooting. This was a configuration alignment and verification step and is not documented as the proven cause of the initial VPN failure.

The VPN remained down until the required policy-based IPsec tunnel action was added to the relevant security policies on both SRX devices.

---

## Verification

After the required configuration was applied, the HQ-SRX established an IKE security association with the Branch-SRX.

Expected verification commands:

```text
show security ike security-associations
show security ipsec security-associations
```

The final verification showed:

```text
IKE State: UP
Total active tunnels: 1
Total Ipsec sas: 1
```

Detailed verification results and troubleshooting evidence are documented separately under the `testing/` and `troubleshooting/` directories.
