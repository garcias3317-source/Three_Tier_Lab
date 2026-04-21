# 🌐 Three-Tier Hierarchical Network Lab

## 📖 Description
This project demonstrates the design and implementation of a fully functional enterprise-grade three-tier hierarchical network built in Cisco Packet Tracer. I configured Layer 3 routing with OSPF, gateway redundancy with HSRP, link aggregation with EtherChannel (LACP), inter-VLAN routing, centralized DHCP with relay, dual-ISP NAT, and a full suite of Layer 2 security features. This lab simulates a real-world corporate campus network covering the core, distribution, and access layers.

---
Topology Overview
<p align="center">
<img src="https://imgur.com/a/AQreICR/>
</p>
---

## 🛠️ Technologies & Protocols
* **Routing:** OSPF (Single Area 0), Floating Static Routes (Dual ISP Failover)
* **Redundancy:** HSRP v2 (Per-VLAN Gateway Redundancy)
* **Link Aggregation:** EtherChannel / LACP (L2 & L3 Port-channels)
* **Switching:** Rapid-PVST+, 802.1Q Trunking, VTP v2
* **Edge Services:** NAT/PAT, Centralized DHCP, DHCP Relay (`ip helper`)
* **Security:** DHCP Snooping, Dynamic ARP Inspection (DAI), Port Security, BPDU Guard, SSH v2, ACLs
* **Logging:** Centralized Syslog

---

## 🚀 Project Walk-through

### 1. Three-Tier Topology Build
The network follows the standard hierarchical design:
* **Edge:** One router (R1) connecting to dual ISPs.
* **Core:** Two switches (CSW1/CSW2) with cross-links.
* **Distribution:** Two switches (DSW1/DSW2) handling routing and SVIs.
* **Access:** Three switches (ASW1–ASW3) dual-homed for redundancy.

### 2. VLANs, VTP, and Trunking
* **VLANs:** 10 (PCs), 20 (Phones), 30 (Servers), 99 (Management).
* **VTP:** DSW1 configured as VTP Server to propagate VLANs.
* **Trunking:** 802.1Q trunks on all uplinks with an unused Native VLAN (1000) and DTP disabled via `nonegotiate`.

### 3. EtherChannel (LACP)
I configured **Port-channel 1** using LACP (mode active) for the CSW1↔CSW2 and DSW1↔DSW2 links. 
* **Core:** Routed (Layer 3) port-channels.
* **Distribution:** Layer 2 trunks.
This provides link redundancy and aggregates bandwidth.

### 4. Rapid-PVST+ & STP Root Placement
To prevent suboptimal forwarding, STP Root placement was aligned with HSRP roles:
* **DSW1:** Primary Root for VLANs 10, 99 | Secondary for 20, 30.
* **DSW2:** Primary Root for VLANs 20, 30 | Secondary for 10, 99.
* **Access:** PortFast and BPDU Guard enabled on all end-device ports.

### 5. Inter-VLAN Routing & HSRP Redundancy
Configured SVI interfaces on DSW1/DSW2 with HSRP v2 for load balancing:

| VLAN | Virtual IP | HSRP Active | Priority |
| :--- | :--- | :--- | :--- |
| **10 (PCs)** | 10.1.0.1 | DSW1 | 105 |
| **20 (Phones)** | 10.2.0.1 | DSW2 | 105 |
| **30 (Servers)** | 10.3.0.1 | DSW2 | 105 |
| **99 (Mgmt)** | 10.0.0.1 | DSW1 | 105 |

### 6. OSPF Single-Area Routing
OSPF Area 0 configured across R1, CSW1, CSW2, DSW1, and DSW2.
* **Router-ID:** Stable Loopback0 interfaces used.
* **Passive Interfaces:** Enabled on SVIs/Loopbacks to suppress unnecessary hellos.
* **Internet:** R1 injects a default route using `default-information originate`.

### 7. Centralized DHCP & IP Helper
R1 serves as the central DHCP server. I implemented `ip helper-address 10.0.0.44` on all SVIs to relay DHCP broadcasts to the router across Layer 3 boundaries.

### 8. NAT / PAT & Dual ISP Failover
* **PAT:** Configured on R1 for public internet access.
* **Failover:** Implemented using Floating Static Routes:
    * **Primary:** `0.0.0.0/0 via 100.100.100.2` (AD 1)
    * **Backup:** `0.0.0.0/0 via 100.100.100.6` (AD 5)

### 9. Layer 2 Security
* **DHCP Snooping & DAI:** Enabled on access switches to prevent rogue DHCP servers and ARP spoofing.
* **Rate Limiting:** Untrusted ports limited to 15 pps.
* **Port Security:** Restricted MAC address learning on access ports.

### 10. SSH Management & ACLs
* **SSH v2:** Enabled on all devices for secure management.
* **Access Control:** VTY lines restricted via ACL to the management subnet (10.1.0.0/24).
* **Syslog:** All system logs are forwarded to **SRV1 (10.3.0.4)**.

---

## 📊 IP Addressing Summary

| Segment | Subnet | Notes |
| :--- | :--- | :--- |
| **VLAN 10 (PCs)** | 10.1.0.0/24 | DHCP from R1, Gateway 10.1.0.1 |
| **VLAN 20 (Phones)** | 10.2.0.0/24 | DHCP from R1, Gateway 10.2.0.1 |
| **VLAN 30 (Servers)** | 10.3.0.0/24 | Static assignment, Gateway 10.3.0.1 |
| **VLAN 99 (Mgmt)** | 10.0.0.0/28 | Static per switch |
| **Core P2P** | 10.0.0.16-43/30 | Point-to-point links |
| **loopbacks** | 10.0.0.44-48/32| OSPF router-Ids|
| **NAT Pool** | 100.100.100.97/29 | PAT Overload range |

---

## 🏆 Skills
Enterprise three-tier hierarchical network design
OSPF single-area configuration, router-IDs, passive interfaces, default route injection
HSRP v2 per-VLAN active/standby with load balancing and preempt
EtherChannel (LACP) on Layer 2 and Layer 3 port-channels
Rapid-PVST+ with manual STP root alignment to HSRP roles
802.1Q trunking, VTP v2, native VLAN security, DTP disable
Voice VLAN (VLAN 20) with IP phone integration
Centralized DHCP with `ip helper-address` relay across Layer 3
NAT overload (PAT) and static NAT with dual-ISP floating static failover
DHCP Snooping, Dynamic ARP Inspection (DAI), rate limiting
Port Security (max 2 MACs, restrict violation)
PortFast and BPDU Guard on all access-facing ports
SSH v2 with ACL-based VTY restriction and local authentication
Syslog centralization to a dedicated server
Unused port shutdown across all switches
---

