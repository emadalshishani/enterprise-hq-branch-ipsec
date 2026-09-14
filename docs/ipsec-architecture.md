# HQ-to-Branch IPsec Architecture

## 1. Purpose

This document describes the architecture used to integrate the Enterprise HQ and Enterprise Branch sites through a policy-based IPsec VPN.

The document focuses on the relationship between the two sites, the traffic path, and the role of each network component. Device-specific IPsec configurations are documented separately under `configs/`.

---

## 2. Site Connectivity

The VPN connects the two Juniper SRX firewalls through their WAN-facing interfaces.

| Device | Interface | Address | Role |
|---|---|---|---|
| HQ-SRX | ge-0/0/1.0 | 10.10.30.1 | HQ VPN endpoint |
| Branch-SRX | ge-0/0/1.0 | 10.10.30.2 | Branch VPN endpoint |

The two SRXs use these addresses as the IKE/IPsec peer endpoints.

The IPsec VPN is policy-based. Traffic is directed into the VPN through security policies rather than through a route-based tunnel interface.

---

## 3. Logical Architecture

```text
                         IPsec VPN
                  =======================
                  |                    |
                  |   Encrypted Path   |
                  |                    |
             10.10.30.1            10.10.30.2
               HQ-SRX              Branch-SRX
                  |                    |
             10.10.10.2          10.10.20.2 / .6
                  |                    |
             HQ L3 Switches       Branch L3 Switches
                  |                    |
              HQ VLANs            Branch VLANs
```

The HQ and Branch LANs remain independent Layer 2 domains.

The IPsec tunnel provides secure Layer 3 connectivity between the two sites.

---

## 4. HQ Site

The HQ internal network connects to HQ-SRX through the HQ Layer 3 switch.

HQ internal networks:

| VLAN | Network |
|---|---|
| VLAN 10 | 192.168.10.0/29 |
| VLAN 20 | 192.168.20.0/29 |
| VLAN 30 | 192.168.30.0/29 |

HQ-SRX provides the Layer 3 connection toward the HQ switching infrastructure through:

```text
HQ-SRX ge-0/0/0
10.10.10.1
        |
        |
10.10.10.2
L3-SW-HQ
```

The HQ Layer 3 switch provides connectivity toward the HQ VLANs.

---

## 5. Branch Site

The Branch network uses two Layer 3 switches connected to Branch-SRX.

Branch-SRX has two internal Layer 3 connections:

```text
Branch-SRX ge-0/0/2
10.10.20.1
        |
        |
10.10.20.2
L3-SW-Branch1


Branch-SRX ge-0/0/0
10.10.20.5
        |
        |
10.10.20.6
L3-SW-Branch2
```

The Branch switching infrastructure provides redundancy through the two Layer 3 switches and VRRP for the user VLAN gateways.

Branch VLANs:

| VLAN | Network |
|---|---|
| VLAN 50 | 192.168.50.0/29 |
| VLAN 60 | 192.168.60.0/29 |

OSPF is used between the Branch Layer 3 switches and Branch-SRX to provide dynamic routing and allow the network to react to link/path failures.

---

## 6. VPN Traffic Flow

The VPN provides connectivity between the private networks behind the two SRXs.

Example: Branch-PC4 communicating with HQ-PC1.

```text
Branch-PC4
    |
    v
VLAN 60
192.168.60.0/29
    |
    v
Branch L3 Switching
    |
    v
Branch-SRX
10.10.30.2
    |
    |  IPsec
    |  encrypted traffic
    v
HQ-SRX
10.10.30.1
    |
    v
L3-SW-HQ
    |
    v
VLAN 30
192.168.30.0/29
    |
    v
HQ-PC1
192.168.30.2
```

The return traffic follows the reverse path through the VPN.

---

## 7. Routing and IPsec Roles

Routing and IPsec perform different functions in this design.

### Routing

Routing determines how traffic reaches the remote site's networks.

For example, traffic destined for the HQ networks must be forwarded toward HQ-SRX.

### Security Policy

The SRX security policy identifies traffic that should be permitted through the VPN.

The relevant policy uses:

```text
source-address      any
destination-address any
application         any
then permit tunnel ipsec-vpn
```

The policy-based VPN action associates the permitted traffic with the configured IPsec VPN.

### IPsec

IPsec provides encryption and authentication for the traffic crossing between the two SRXs.

Therefore:

```text
Routing
   ↓
Traffic reaches the SRX
   ↓
Security policy matches
   ↓
Traffic is associated with IPsec VPN
   ↓
Traffic is encrypted
   ↓
Traffic crosses the VPN
   ↓
Remote SRX decrypts traffic
   ↓
Traffic is forwarded to the destination LAN
```

---

## 8. Underlay vs VPN

The design can be viewed as two separate layers.

### Underlay

The underlay provides basic IP connectivity between:

```text
HQ-SRX 10.10.30.1
        |
        |
Branch-SRX 10.10.30.2
```

This connectivity is required before the VPN can establish.

### Overlay

The IPsec VPN provides the secure communication path between the private networks behind the two SRXs.

```text
HQ LANs
   |
HQ-SRX
   |
[ IPsec Overlay ]
   |
Branch-SRX
   |
Branch LANs
```

The VPN therefore depends on the underlying IP reachability between the two VPN endpoints.

---

## 9. High-Level Design Characteristics

The completed design provides:

- Separate HQ and Branch Layer 2 domains.
- Layer 3 routing within each site.
- VRRP-based gateway redundancy at the Branch site.
- OSPF-based routing between Branch Layer 3 switches and Branch-SRX.
- Policy-based IPsec between HQ-SRX and Branch-SRX.
- Encrypted inter-site traffic.
- Security-policy-controlled VPN traffic.
- Independent documentation for the HQ and Branch SRX configurations.
- Separate testing and troubleshooting documentation for the integration layer.

---

## 10. Related Documentation

Detailed device configuration:

- `configs/hq-srx-ipsec.md`
- `configs/branch-srx-ipsec.md`

The following documents cover verification, connectivity testing, failover behavior, and troubleshooting of the integrated environment.
