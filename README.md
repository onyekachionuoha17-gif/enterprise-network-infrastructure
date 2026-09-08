# Enterprise Network Infrastructure Lab

## Project Overview

This project demonstrates the design, configuration, security, and troubleshooting of a multi-layer enterprise network built in Cisco Packet Tracer.

The environment was designed to simulate a real enterprise infrastructure using a hierarchical MDF/BDF/IDF architecture. The network includes VLAN segmentation, Layer 3 switching, inter-VLAN routing, DHCP, DNS, SSH management, guest network isolation, switch hardening, Spanning Tree Protocol (STP), redundant uplinks, an ASA firewall, and NAT/PAT.

The goal of this project was not only to build a functioning network, but also to practice troubleshooting, documentation, redundancy, network security, and validation techniques used by Network Engineers and Data Center Technicians.

---

## Network Topology

![Enterprise Network Infrastructure](INFRASTRUCTURE%20CLOSE-UP.png)

### Architecture

The network follows a hierarchical design:

- **MDF / Core Layer:** CORE-SW
- **BDF / Distribution Layer:** BDF-SW
- **IDF1 / Access Layer:** Corporate and Operations
- **IDF2 / Access Layer:** Servers and Guest Wireless
- **IDF3 / Access Layer:** Network Administration and Security
- **Edge:** R1 and Cisco ASA 5505
- **External Network:** ISP-R1 and simulated Internet Server

Two uplinks connect the Core and BDF switches to provide Layer 2 redundancy.

---

## VLAN and IP Addressing Plan

| VLAN | Name | Network | Default Gateway |
|---|---|---|---|
| 10 | CORPORATE | 10.10.10.0/24 | 10.10.10.1 |
| 20 | SERVERS | 10.10.20.0/24 | 10.10.20.1 |
| 30 | NETWORK | 10.10.30.0/24 | 10.10.30.1 |
| 40 | OPERATIONS | 10.10.40.0/24 | 10.10.40.1 |
| 50 | GUEST | 10.10.50.0/24 | 10.10.50.1 |
| 60 | SECURITY | 10.10.60.0/24 | 10.10.60.1 |
| 99 | MANAGEMENT | 10.10.99.0/24 | 10.10.99.1 |

### Management Addresses

| Device | Management IP |
|---|---|
| CORE-SW | 10.10.99.1 |
| BDF-SW | 10.10.99.2 |
| IDF1-SW | 10.10.99.11 |
| IDF2-SW | 10.10.99.12 |
| IDF3-SW | 10.10.99.13 |

---

## Layer 3 Routing

CORE-SW operates as the Layer 3 core switch.

Switch Virtual Interfaces (SVIs) were configured for each enterprise VLAN, allowing CORE-SW to provide the default gateways and route traffic between VLANs.

The Core also contains a default route toward R1:

```text
0.0.0.0/0 → 172.16.1.2
```

R1 uses a summarized route back toward the enterprise VLANs:

```text
10.10.0.0/16 → 172.16.1.1
```

![Layer 3 Routing](LAYER%203%20ROUTING%20CORE-SW.png)

---

## VLAN Configuration

Seven production and management VLANs were implemented across the switching infrastructure.

![VLAN Configuration](VLAN%20CONFIG%20CORE-SW.png)

---

## 802.1Q Trunking

802.1Q trunks carry multiple VLANs between the Core, BDF, and IDF switches.

The BDF provides connectivity between the Core and the three access-layer IDFs.

![802.1Q Trunking](802.1Q%20TRUNKING%20BDF-SW.png)

---

## Spanning Tree and Redundancy

Redundant Layer 2 uplinks were configured between CORE-SW and BDF-SW.

CORE-SW was intentionally configured as the STP root bridge for the enterprise production VLANs.

During normal operation:

- Primary uplink: **Forwarding**
- Backup uplink: **Alternate / Blocking**

This prevents a Layer 2 switching loop while maintaining a redundant path.

A failover test was also performed by administratively shutting down the primary uplink. STP transitioned the alternate link to the forwarding state and maintained the Layer 2 path. The primary connection was then restored.

### STP Root Bridge

![STP Root](CORE-SW%20STP%20ROOT.png)

### Redundant Uplinks

![STP Redundancy](STP%20REDUNDANCY%20BDF-SW.png)

---

## DHCP

CORE-SW provides DHCP services to the client VLANs.

Address exclusions were configured to reserve infrastructure addresses, and clients receive the appropriate:

- IPv4 address
- Subnet mask
- Default gateway
- DNS server

CORP-PC1 successfully received:

```text
IP Address:      10.10.10.21
Subnet Mask:     255.255.255.0
Default Gateway: 10.10.10.1
```

### DHCP Binding

![DHCP Binding](DHCP%20BINDING%20CORE-SW.png)

### DHCP Client Verification

![DHCP Client](CLIENT%20DHCP%20PROOF.png)

---

## DNS

SERVER1 provides internal DNS services.

An internal DNS record was created:

```text
intranet.enterprise.local → 10.10.20.10
```

CORP-PC1 successfully resolved the hostname and communicated with the server.

![DNS Test](DNS%20PROOF.png)

---

## Inter-VLAN Routing

Connectivity testing verified routing between the internal enterprise VLANs.

From CORP-PC1 in VLAN 10, successful ICMP tests were performed to systems in:

- VLAN 20 — Servers
- VLAN 30 — Network
- VLAN 40 — Operations
- VLAN 60 — Security

![Inter-VLAN Connectivity](INTER-VLAN%20CONNECTIVITY.png)

---

## Guest Network Isolation

The wireless guest network uses:

```text
SSID: Enterprise-Guest
VLAN: 50
Network: 10.10.50.0/24
```

An extended ACL was applied at the IDF2 Guest access port to prevent Guest VLAN devices from accessing internal enterprise networks.

Testing confirmed:

- Guest → Guest Gateway: **Allowed**
- Guest → Corporate: **Blocked**
- Guest → Servers: **Blocked**
- Guest → Network Administration: **Blocked**
- Guest → Operations: **Blocked**
- Guest → Security: **Blocked**
- Guest → Management: **Blocked**

![Guest Isolation](GUEST%20VLAN%20ISOLATION.png)

---

## Secure Network Management

A dedicated VLAN 99 management network was implemented.

SSH was configured on the BDF and IDF switches using:

- Local authentication
- RSA keys
- SSH version 2
- VTY lines restricted to SSH

Remote management was successfully tested from NETADMIN-PC.

![SSH Management](SSH%20MANAGEMENT.png)

---

## Switch Hardening

Several Layer 2 security practices were implemented:

- Unused switch ports administratively disabled
- Unused ports labeled for documentation
- Port security on endpoint access ports
- Sticky MAC learning
- Maximum of one MAC address on applicable wired endpoint ports
- Port-security violation mode set to restrict
- PortFast on endpoint-facing access ports
- BPDU Guard on endpoint-facing access ports
- Dedicated management VLAN

The Guest WAP port was intentionally not limited to a single sticky MAC because multiple wireless clients may communicate through the access point.

---

## Firewall and NAT/PAT

A Cisco ASA 5505 was deployed between the enterprise edge router and ISP.

### ASA Interfaces

| Interface | Address | Security Level |
|---|---|---:|
| INSIDE | 172.16.0.1/30 | 100 |
| OUTSIDE | 203.0.113.2/30 | 0 |

Dynamic PAT translates enterprise private addresses in the `10.10.0.0/16` network to the ASA outside interface address.

Testing confirmed active translations from:

```text
10.10.10.21 → 203.0.113.2
```

![ASA NAT PAT](ASA%20NAT%20PAT.png)

---

## Troubleshooting Case Study

During end-to-end Internet testing, CORP-PC1 could reach the enterprise Core, R1, and the ASA inside interface, but the complete ICMP exchange to the simulated Internet network did not complete.

Packet Tracer Simulation Mode was used to trace the packet hop-by-hop.

Testing confirmed:

1. CORP-PC1 forwarded traffic through the IDF, BDF, Core, and R1.
2. The ASA received the internal packet.
3. The inside ACL permitted the traffic.
4. Dynamic PAT translated `10.10.10.21` to `203.0.113.2`.
5. ISP-R1 received the translated packet.
6. ISP-R1 generated the ICMP Echo Reply.
7. EDGE-FW received the reply.
8. The ASA reverse-translated the destination back to `10.10.10.21`.
9. In this Packet Tracer simulation, the return packet stopped at the ASA instead of continuing toward R1.

Because outbound forwarding, PAT, return traffic, and reverse translation were individually verified, the behavior was documented as a Packet Tracer ASA simulation limitation rather than repeatedly altering otherwise validated configurations.

This troubleshooting process demonstrates the use of packet-level analysis to isolate a problem to a specific point in the network path.

---

## Skills Demonstrated

- Enterprise network design
- MDF / BDF / IDF architecture
- Cisco IOS CLI
- VLAN configuration
- 802.1Q trunking
- Layer 3 switching
- Switch Virtual Interfaces (SVIs)
- Inter-VLAN routing
- Static and default routing
- DHCP
- DNS
- IPv4 subnetting
- ACL implementation
- Guest network segmentation
- Cisco ASA firewall configuration
- NAT/PAT
- SSH remote administration
- Port security
- Switch hardening
- PortFast
- BPDU Guard
- Spanning Tree Protocol
- Layer 2 redundancy
- Failover testing
- Network troubleshooting
- Packet Tracer Simulation Mode
- Technical documentation

---

## Project File

The complete Cisco Packet Tracer `.pkt` file is included in this repository so the topology and configurations can be reviewed.

---

## Author

**Onyekachi Onuoha**

Enterprise Networking / Data Center Infrastructure Portfolio Project
