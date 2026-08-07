# 🧪 Lab 01: Basic IP Addressing & Ping Testing

Hello! 👋 Welcome to my first networking lab. In this lab, I connected two virtual computers (`PC1` and `PC2`) belonging to **two different networks** using a Cisco Router (`R1`) in GNS3.

---

## 🤔 What is this Lab About?

Imagine two people speaking different languages in different rooms. They cannot talk directly. They need a translator to pass messages between them.

In networking:
- **`PC1`** is in Network A (`192.168.1.0`)
- **`PC2`** is in Network B (`192.168.2.0`)
- **Router `R1`** acts as the translator (Default Gateway) that routes data packets between Network A and Network B.

---

## 📐 Network Topology (Setup Diagram)

```text
  [ PC1 ]                                                      [ PC2 ]
(192.168.1.1)                                                (192.168.2.1)
      │                                                           │
      │ (Cable connected to g1/0)     (Cable connected to g2/0)   │
      └──────────────► ┌────────────────┐ ◄──────────────────────┘
                       │   Router R1    │
                       └────────────────┘
```

---


  <img width="593" height="559" alt="Ping_topology" src="https://github.com/user-attachments/assets/8eab63fc-a933-4841-ad3f-d8ab50fd380c" />

## 📋 Simple Addressing Table

| Device | Connected Interface | IP Address | Subnet Mask | Default Gateway | What it does |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **PC1** | eth0 | `192.168.1.1` | `255.255.255.0` | `192.168.1.100` | Host computer in Network 1 |
| **PC2** | eth0 | `192.168.2.1` | `255.255.255.0` | `192.168.2.100` | Host computer in Network 2 |
| **R1** | GigabitEthernet1/0 | `192.168.1.100` | `255.255.255.0` | None | Gateway for Network 1 |
| **R1** | GigabitEthernet2/0 | `192.168.2.100` | `255.255.255.0` | None | Gateway for Network 2 |

---

## 🛠️ Step-by-Step Commands Used

### **Step 1: Configure Router R1**

On Router R1, we give IP addresses to the two ports (interfaces) so it can talk to both networks:

```cisco
! Go into configuration mode
R1# configure terminal

! Configure Port 1 for PC1's network
R1(config)# interface GigabitEthernet1/0
R1(config-if)# ip address 192.168.1.100 255.255.255.0
R1(config-if)# no shutdown   <-- Turns ON the interface port

! Configure Port 2 for PC2's network
R1(config)# interface GigabitEthernet2/0
R1(config-if)# ip address 192.168.2.100 255.255.255.0
R1(config-if)# no shutdown   <-- Turns ON the interface port
R1(config-if)# end

! Save configuration so it isn't lost on restart
R1# copy running-config startup-config
```

### **Step 2: Configure Computers (VPCS Hosts)**

On `PC1`, set its IP address and point its gateway to Router R1's `g1/0` port:
```text
PC1> ip 192.168.1.1 255.255.255.0 192.168.1.100
```

On `PC2`, set its IP address and point its gateway to Router R1's `g2/0` port:
```text
PC2> ip 192.168.2.1 255.255.255.0 192.168.2.100
```

---

## ✅ How We Verified it Works

From `PC1`, we ran a `ping` command to send test packets to `PC2`:

```text
PC1> ping 192.168.2.1
84 bytes from 192.168.2.1 icmp_seq=1 ttl=63 time=8.215 ms
84 bytes from 192.168.2.1 icmp_seq=2 ttl=63 time=2.104 ms
2 packets transmitted, 2 received, 0% packet loss
```

🎉 **Result:** `0% packet loss` means `PC1` successfully reached `PC2` through Router `R1`!

---




## 🔍 Packet Capture (Wireshark)

The raw Wireshark traffic capture file for this lab is saved in the repository:
📁 **[`pcaps/icmp_ping_traffic.pcap`](./pcaps/icmp_ping_traffic.pcap)**

### What you can see in this capture file:
- **ARP Request/Reply:** `PC1` resolving Router `R1`'s MAC address before sending IP packets.
- **ICMP Echo Request (Type 8):** Sent from `192.168.1.1` to `192.168.2.1`.
- **ICMP Echo Reply (Type 0):** Sent back from `192.168.2.1` to `192.168.1.1`.

---

## 🎓 Simple Takeaways (What I Learned)

1. **Why `no shutdown` matters:** By default, router ports are turned OFF (`shutdown`). We must type `no shutdown` to turn them ON.
2. **Why Default Gateway matters:** If `PC1` doesn't know `PC2`'s address, it sends the packet to its **Default Gateway** (`192.168.1.100`), which forwards it to `PC2`.
3. **What Ping is:** Ping uses **ICMP** packets to test if another computer is reachable across a network.
