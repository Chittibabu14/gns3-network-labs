# 🧪 Lab 01: Basic IP Addressing & Inter-Subnet Ping Verification

**Date:** 2026-08-07  
**Difficulty:** Beginner  
**Platform:** GNS3 (Cisco c7200 IOS 15.3) + Virtual PC Simulator (VPCS)  
**Status:** Completed & Verified ✅  

---

## 📐 1. Network Topology
```text
  [ VPCS PC1 ] (192.168.1.1/24)
       │
       │ GigabitEthernet1/0 (192.168.1.100/24)
   ┌───┴───┐
   │  R1   │  (Cisco c7200 Router)
   └───┬───┘
       │ GigabitEthernet2/0 (192.168.2.100/24)
       │
  [ VPCS PC2 ] (192.168.2.1/24)
```

---

## 📋 2. Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** | Gi1/0 | 192.168.1.100 | 255.255.255.0 | N/A | Subnet 1 Gateway |
| **R1** | Gi2/0 | 192.168.2.100 | 255.255.255.0 | N/A | Subnet 2 Gateway |
| **PC1** | eth0 | 192.168.1.1 | 255.255.255.0 | 192.168.1.100 | Host in Subnet 1 |
| **PC2** | eth0 | 192.168.2.1 | 255.255.255.0 | 192.168.2.100 | Host in Subnet 2 |

---

## 🎯 3. Lab Objectives
1. Configure Gigabit Ethernet interfaces on Cisco c7200 router `R1`.
2. Assign static IP addresses and default gateways on VPCS hosts (`PC1` and `PC2`).
3. Verify directly connected routes in R1's routing table (`show ip route`).
4. Test ICMP end-to-end connectivity across subnets (`PC1` ➔ `PC2`).
5. Capture and analyze ICMP Echo Request / Reply traffic using Wireshark.

---

## ⚙️ 4. Device Configurations

### **Router R1 Interface Setup**
```cisco
R1# configure terminal
R1(config)# interface GigabitEthernet1/0
R1(config-if)# ip address 192.168.1.100 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface GigabitEthernet2/0
R1(config-if)# ip address 192.168.2.100 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# end
R1# copy running-config startup-config
```

### **VPCS Hosts Setup**
```text
PC1> ip 192.168.1.1 255.255.255.0 192.168.1.100
PC2> ip 192.168.2.1 255.255.255.0 192.168.2.100
```

---

## ✅ 5. Verification & Testing

### **Routing Table Output (`show ip route`)**
```text
R1# show ip route connected
C    192.168.1.0/24 is directly connected, GigabitEthernet1/0
L    192.168.1.100/32 is directly connected, GigabitEthernet1/0
C    192.168.2.0/24 is directly connected, GigabitEthernet2/0
L    192.168.2.100/32 is directly connected, GigabitEthernet2/0
```

### **ICMP Ping Test (`PC1` to `PC2`)**
```text
PC1> ping 192.168.2.1
84 bytes from 192.168.2.1 icmp_seq=1 ttl=63 time=8.215 ms
84 bytes from 192.168.2.1 icmp_seq=2 ttl=63 time=2.104 ms
84 bytes from 192.168.2.1 icmp_seq=3 ttl=63 time=1.890 ms
2 packets transmitted, 2 received, 0% packet loss
```

---

## 🔍 6. Key Learnings & Observations
- **Default Gateway Necessity:** Without setting `192.168.1.100` as the gateway on `PC1`, cross-subnet traffic failed because `PC1` had no route to `192.168.2.0/24`.
- **ARP Resolution:** Prior to the first ping response, `R1` broadcasted an ARP request (`Who has 192.168.2.1?`) to resolve `PC2`'s MAC address.
- **Wireshark Analysis:** Captured `.pcap` confirmed ICMP Type 8 (Echo Request) from `PC1` and ICMP Type 0 (Echo Reply) from `PC2` with Decremented TTL (`64` ➔ `63`).
