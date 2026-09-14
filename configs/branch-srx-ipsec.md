# Branch-SRX IPsec Configuration

## Purpose

This document contains the IPsec-related configuration implemented on the **Branch-SRX** to establish the site-to-site VPN with the HQ SRX.

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
| Local SRX | Branch-SRX |
| Local WAN Interface | ge-0/0/1.0 |
| Local Address | 10.10.30.2 |
| Remote SRX | HQ-SRX |
| Remote Address | 10.10.30.1 |

---

## IKE Phase 1

The Branch-SRX uses a pre-shared-key based IKE configuration.

### IKE Proposal

```text
set security ike proposal ike-prop authentication-method pre-shared-keys
set security ike proposal ike-prop dh-group group2
set security ike proposal ike-prop authentication-algorithm sha-256
set security ike proposal ike-prop encryption-algorithm aes-256-cbc
```

### IKE Policy

```text
set security ike policy ike-policy mode main
set security ike policy ike-policy proposals ike-prop
```

The pre-shared key is configured on the device but is intentionally not included in this repository.

### IKE Gateway

```text
set security ike gateway ike-GW ike-policy ike-policy
set security ike gateway ike-GW address 10.10.30.1
set security ike gateway ike-GW external-interface ge-0/0/1.0
set security ike gateway ike-GW local-address 10.10.30.2
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

## Security Zone

The Branch WAN-facing interface is assigned to the security zone used for the HQ VPN traffic.

```text
set security zones security-zone hq host-inbound-traffic system-services ping
set security zones security-zone hq host-inbound-traffic system-services ike
set security zones security-zone hq interfaces ge-0/0/1.0
```

IKE traffic is explicitly allowed as host-inbound traffic on the VPN-facing zone.

---

## Policy-Based VPN Security Policy

The Branch security policy between the internal zone and the HQ zone includes the IPsec tunnel action.

```text
set security policies from-zone internal to-zone hq policy internal-hq match source-address any
set security policies from-zone internal to-zone hq policy internal-hq match destination-address any
set security policies from-zone internal to-zone hq policy internal-hq match application any
set security policies from-zone internal to-zone hq policy internal-hq then permit tunnel ipsec-vpn ipsec-vpn
```

The `then permit tunnel ipsec-vpn ipsec-vpn` action is required for the policy-based VPN traffic to use the configured IPsec tunnel.

---

## Troubleshooting Note

The initial Branch IKE proposal did not include a Diffie-Hellman group, while the HQ-SRX was already configured with Group 2.

Group 2 was subsequently added to the Branch IKE proposal:

```text
set security ike proposal ike-prop dh-group group2
```

The IPsec PFS setting was initially configured as Group 20 on both peers.

During troubleshooting, PFS was changed from Group 20 to Group 2 on both devices as a configuration alignment and verification step. This change is not documented as the proven cause of the initial VPN establishment failure.

The VPN remained down after these changes.

The actual configuration issue identified during troubleshooting was the missing policy-based IPsec tunnel action in the relevant security policies.

After the tunnel action was added to both peers, the IKE and IPsec security associations were successfully established.

---

## Verification

After the required configuration was applied, the Branch-SRX established an IKE security association with the HQ-SRX.

Verification commands:

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
