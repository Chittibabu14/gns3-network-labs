# 🌐 GNS3 Network Engineering & Security Labs

**Maintained by:** Chittibabu Bavisetti ([@Chittibabu14](https://github.com/Chittibabu14))  
**LinkedIn:** [chittibabu-bavisetti-6a444525b](https://linkedin.com/in/chittibabu-bavisetti-6a444525b)  

Welcome to my **GNS3 Network Engineering Portfolio**! This repository serves as a documented, daily log of network topologies, Cisco device configurations, packet analyses, and security hardening built using **GNS3**, **Cisco IOS**, and **Wireshark**.

---

## 🗺️ Master Lab Index

### 📂 Phase 1: Network Fundamentals & Device Management
| Lab # | Lab Title | Key Concepts / Protocols | Link |
| :--- | :--- | :--- | :--- |
| **Lab 01** | Basic IP Addressing & Ping Verification | VPCS, GigabitEthernet Interfaces, ICMP, Subnetting | [View Lab](./01-fundamentals/01-basic-router-ping/) |
| **Lab 02** | Secure SSH Remote Management & Device Hardening | Domain Name, RSA Key Pairs, SSHv2, VTY Login Local, Secret Enable | [View Lab](./01-fundamentals/02-ssh-remote-management/) |
| **Lab 03** | Cisco IOS DHCP Server & DORA Process Analysis | IP DHCP Pool, Excluded Addresses, DORA Handshake, UDP 67/68 | [View Lab](./01-fundamentals/03-cisco-dhcp-server/) |

---

### 📂 Phase 2: Switching, VLANs & Layer 2 Security *(In Progress)*
| Lab # | Lab Title | Key Concepts / Protocols | Link |
| :--- | :--- | :--- | :--- |
| **Lab 04** | Inter-VLAN Routing (Router-on-a-Stick) | 802.1Q Trunking, Subinterfaces, Encapsulation | *Coming Soon* |
| **Lab 05** | Switchport Security & DHCP Snooping | MAC Limit, Sticky MAC, Rogue DHCP Prevention, DAI | *Coming Soon* |
| **Lab 06** | Spanning Tree Protocol (STP) Tuning | Root Bridge Election, BPDU Guard, PortFast | *Coming Soon* |

---

### 📂 Phase 3: Dynamic Routing Protocols *(Upcoming)*
| Lab # | Lab Title | Key Concepts / Protocols | Link |
| :--- | :--- | :--- | :--- |
| **Lab 07** | Single-Area OSPFv2 Configuration | Router ID, Wildcard Masks, Passive Interfaces, Adjacency | *Coming Soon* |
| **Lab 08** | Multi-Area OSPF & ABR Summarization | Area 0 (Backbone), Inter-Area Routing, LSA Types | *Coming Soon* |
| **Lab 09** | BGP Basics & Dual-Homed ISP Connectivity | eBGP, AS Numbers, Route Advertisements | *Coming Soon* |

---

### 📂 Phase 4: Network Security, ACLs & VPNs *(Upcoming)*
| Lab # | Lab Title | Key Concepts / Protocols | Link |
| :--- | :--- | :--- | :--- |
| **Lab 10** | Standard & Extended Access Control Lists (ACLs) | Packet Filtering, Inbound/Outbound ACLs, Port Blocking | *Coming Soon* |
| **Lab 11** | Dynamic NAT & PAT (Port Address Translation) | Inside/Outside Interfaces, Public IP Overload | *Coming Soon* |
| **Lab 12** | Site-to-Site IPsec VPN Tunnel | IKEv1/v2 Phase 1 & 2, Crypto Maps, AES Encryption | *Coming Soon* |

---

## 🛠️ Repository File Structure

Each lab directory follows a standardized, professional structure:

```text
lab-name/
├── README.md               <-- Detailed Lab Documentation (Topology, Objectives, Verification)
├── topology.png            <-- GNS3 Diagram Screenshot
├── configs/                <-- Cisco IOS Running Configurations (.cfg / .txt)
└── pcaps/                  <-- Wireshark Traffic Captures (.pcap)
```

---

## 🧰 Tools & Images Used
- **Simulator:** GNS3 v2.2+
- **Router Image:** Cisco c7200 (IOS 15.3 ADVENTERPRISEK9)
- **Hosts:** VPCS (Virtual PC Simulator) & Alpine Linux
- **Packet Analyzer:** Wireshark
