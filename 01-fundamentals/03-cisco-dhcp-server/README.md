# 📡 Lab 03: Cisco IOS DHCP Server Configuration & DORA Process Analysis

Welcome to Lab 03! In this lab, I configured a Cisco Router (`DHCP-Server`) to act as a centralized **DHCP (Dynamic Host Configuration Protocol) Server** in GNS3, dynamically provisioning IP addresses, default gateways, and DNS servers to multiple LAN hosts (`PC1`, `PC2`) and a Cisco Router client (`R2`).

---

## 🤔 What is DHCP & Why Do We Use It?

In small networks with 2-3 devices, manually setting static IP addresses is easy. But in real-world enterprise networks with hundreds or thousands of laptops, phones, and workstations, configuring each device by hand is slow, error-prone, and leads to IP address conflicts.

**DHCP (Dynamic Host Configuration Protocol)** solves this problem by automatically assigning:
1. **IP Address** & **Subnet Mask**
2. **Default Gateway** (Router IP)
3. **DNS Servers** (Domain Name System servers)
4. **Lease Time** (Duration the client can keep the IP)

---

## 🔄 The D.O.R.A. Process Explained

When a client device connects to the network, it negotiates its IP address with the DHCP server using a 4-step handshake known as **DORA**:

```text
    Client (PC / Host)                                    DHCP Server (Router)
   (UDP Source Port: 68)                                  (UDP Dest Port: 67)
           │                                                       │
           │ ────────── 1. DHCPDISCOVER (Broadcast) ─────────────► │ "Is there a DHCP server available?"
           │                                                       │
           │ ◄───────── 2. DHCPOFFER    (Unicast/Bcast) ────────── │ "Yes, here is IP 192.168.1.1 for you!"
           │                                                       │
           │ ────────── 3. DHCPREQUEST  (Broadcast) ─────────────► │ "I would like to accept that IP address."
           │                                                       │
           │ ◄───────── 4. DHCPACK      (Unicast/Bcast) ────────── │ "Acknowledged! The IP is leased to you."
           │                                                       │
```

1. **D - Discover:** Client broadcasts `DHCPDISCOVER` from `0.0.0.0:68` to `255.255.255.255:67` asking for an available DHCP server.
2. **O - Offer:** The DHCP Server reserves an IP address and unicasts/broadcasts a `DHCPOFFER` containing the proposed IP, mask, gateway, and DNS.
3. **R - Request:** Client broadcasts `DHCPREQUEST` confirming acceptance of the offered IP configuration.
4. **A - Acknowledge:** The Server sends `DHCPACK` to finalize the lease and record the MAC-to-IP binding in its DHCP database.

---

## 📐 Network Setup Diagram

```text
                               ┌────────────────────────┐
                               │   DHCP-Server (Cisco)  │
                               │     (192.168.1.100)    │
                               └───────────┬────────────┘
                                           │ (g1/0)
                                           │
                                  ┌────────┴────────┐
                                  │  Switch1 (L2)   │
                                  └───┬────┬────┬───┘
                    (e1) ┌────────────┘    │    └────────────┐ (e3)
                         │                 │ (e2)            │
                         ▼                 ▼                 ▼
                  ┌────────────┐    ┌────────────┐    ┌────────────┐
                  │    PC1     │    │    PC2     │    │  Router R2 │
                  │   (VPCS)   │    │   (VPCS)   │    │  (Client)  │
                  │   [DHCP]   │    │   [DHCP]   │    │   [DHCP]   │
                  └────────────┘    └────────────┘    └────────────┘
```
<img width="938" height="637" alt="image" src="https://github.com/user-attachments/assets/5c6b3921-6f98-4a09-91b3-59871953aa42" />

---

## 📋 Addressing & DHCP Pool Specification

| Parameter | Value / Range | Description |
| :--- | :--- | :--- |
| **DHCP Pool Name** | `SBI-BANK` | Name of the address pool |
| **Network Subnet** | `192.168.1.0 /24` | Entire LAN address space |
| **Excluded Range** | `192.168.1.100 - 192.168.1.110` | Reserved for Server & static infrastructure |
| **Default Router (Gateway)** | `192.168.1.100` | Gateway IP provided to clients |
| **DNS Servers** | `8.8.8.8`, `8.8.4.4` | Google Public DNS servers |
| **DHCP Server IP** | `192.168.1.100/24` (g1/0) | Static IP configured on Server interface |

---

## 🛠️ Step-by-Step Cisco IOS Commands

### **Step 1: Reserve Static IPs (Excluded Addresses)**
*Best Practice:* Exclude static IP addresses (such as router interfaces, printers, and servers) **before** activating the DHCP pool to avoid assigning duplicate IPs.

```cisco
DHCP-Server# configure terminal
DHCP-Server(config)# ip dhcp excluded-address 192.168.1.100 192.168.1.110
```

### **Step 2: Create and Configure the DHCP Address Pool**
```cisco
! Create pool named 'SBI-BANK'
DHCP-Server(config)# ip dhcp pool SBI-BANK

! Specify network address and subnet mask
DHCP-Server(dhcp-config)# network 192.168.1.0 255.255.255.0

! Specify default gateway for clients
DHCP-Server(dhcp-config)# default-router 192.168.1.100

! Specify DNS servers
DHCP-Server(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
DHCP-Server(dhcp-config)# exit
```

### **Step 3: Configure the Router LAN Interface**
```cisco
DHCP-Server(config)# interface GigabitEthernet1/0
DHCP-Server(config-if)# description Connected to Switch1 LAN
DHCP-Server(config-if)# ip address 192.168.1.100 255.255.255.0
DHCP-Server(config-if)# no shutdown
DHCP-Server(config-if)# end
DHCP-Server# copy running-config startup-config
```

### **Step 4: Request DHCP on VPCS Hosts (`PC1` and `PC2`)**
On VPCS hosts, execute the `dhcp` command to start the DORA negotiation:

```text
PC1> dhcp
DDORA IP 192.168.1.1/24 GW 192.168.1.100

PC2> dhcp
DDORA IP 192.168.1.2/24 GW 192.168.1.100
```

### **Step 5: Configure Cisco Router `R2` as a DHCP Client**
Cisco routers can also acquire an IP dynamically via DHCP on an interface:

```cisco
R2# configure terminal
R2(config)# interface GigabitEthernet1/0
R2(config-if)# ip address dhcp
R2(config-if)# no shutdown
R2(config-if)# end
```

---

## ✅ How We Verified It Works

### **1. Verify Active Leases on DHCP Server (`show ip dhcp binding`)**
```text
DHCP-Server# show ip dhcp binding
Bindings from all pools :
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.1.1         0100.5079.6668.00       Aug 11 2026 04:26 PM    Automatic
192.168.1.2         0100.5079.6668.01       Aug 11 2026 04:26 PM    Automatic
192.168.1.3         0063.6101.11d0.0001     Aug 11 2026 04:27 PM    Automatic
```

### **2. Verify DHCP Pool Status (`show ip dhcp pool`)**
```text
DHCP-Server# show ip dhcp pool SBI-BANK

Pool SBI-BANK :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (total/usable)     : 254 / 243
 Cause of last bind             : Dynamic / Excluded
 Total addresses                : 254
 Leased addresses               : 3
 Excluded addresses             : 11
 Pending event                  : none
```

### **3. Verify Host IP Configuration on VPCS (`show ip`)**
```text
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 192.168.1.1/24
GATEWAY     : 192.168.1.100
DNS         : 8.8.8.8  8.8.4.4
DHCP SERVER : 192.168.1.100
DHCP LEASE  : 86400, 86400/43200/75600
MAC         : 00:50:79:66:68:00
```

### **4. Verify End-to-End Connectivity (Ping Test)**
From `PC1`, we ping both the DHCP Gateway (`192.168.1.100`) and fellow client `PC2` (`192.168.1.2`):

```text
PC1> ping 192.168.1.100
84 bytes from 192.168.1.100 icmp_seq=1 ttl=255 time=6.120 ms
84 bytes from 192.168.1.100 icmp_seq=2 ttl=255 time=3.412 ms
2 packets transmitted, 2 received, 0% packet loss

PC1> ping 192.168.1.2
84 bytes from 192.168.1.2 icmp_seq=1 ttl=64 time=1.234 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=64 time=1.112 ms
2 packets transmitted, 2 received, 0% packet loss
```

---

## 🔍 Packet Capture Analysis (Wireshark)

The Wireshark traffic capture files for this lab are saved in the repository:
- 📁 **[`pcaps/dhcp_dora_traffic.pcap`](./pcaps/dhcp_dora_traffic.pcap)** (VPCS Client DORA)
- 📁 **[`pcaps/dhcp_router_client_traffic.pcap`](./pcaps/dhcp_router_client_traffic.pcap)** (Cisco Router Client DORA)

### What you can see in these capture files:
1. **Bootstrap Protocol (DHCP) Layer:**
   - **UDP Ports:** Client Port `68` (Source) $\leftrightarrow$ Server Port `67` (Destination).
   - **Transaction ID (XID):** Unique 32-bit ID used to match request and reply messages.
2. **DHCP Options:**
   - **Option 53 (DHCP Message Type):** Shows `Discover (1)`, `Offer (2)`, `Request (3)`, `ACK (5)`.
   - **Option 1 (Subnet Mask):** `255.255.255.0`
   - **Option 3 (Router / Default Gateway):** `192.168.1.100`
   - **Option 6 (Domain Name Server):** `8.8.8.8, 8.8.4.4`
   - **Option 51 (IP Address Lease Time):** 1 day (86400s)

---

## 🎓 Simple Takeaways (What I Learned)

1. **Why `excluded-address` is critical:** Always configure `ip dhcp excluded-address` *before* configuring the pool. Otherwise, the DHCP server might lease out the gateway's own IP address (`192.168.1.100`), causing a network outage.
2. **Understanding DORA:** The client begins with IP `0.0.0.0` and sends a broadcast (`255.255.255.255`). The server responds with an offer, the client requests it, and the server acknowledges it.
3. **Cisco Router Flexibility:** A Cisco router can function as a **DHCP Server** (`ip dhcp pool`), a **DHCP Client** (`ip address dhcp`), or a **DHCP Relay Agent** (`ip helper-address`).
