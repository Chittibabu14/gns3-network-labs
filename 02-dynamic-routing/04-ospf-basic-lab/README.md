# Lab 04: Single-Area OSPFv2 Basic Configuration

Welcome to Lab 04! In this lab, I configured **OSPF (Open Shortest Path First)** across three Cisco routers in a linear topology using GNS3. All routers are in a single OSPF **Area 0** and use **Loopback interfaces** as stable Router IDs.

---

## What is OSPF & Why Do We Use It?

In small networks, static routes can handle traffic forwarding. But in real enterprise and service-provider networks with dozens of routers and subnets, manually configuring and maintaining static routes becomes unscalable and error-prone.

**OSPF (Open Shortest Path First)** is a **Link-State Routing Protocol** that solves this by:
1. **Automatic Route Discovery:** Routers exchange LSAs (Link-State Advertisements) to build a complete map of the network.
2. **Shortest Path Calculation:** Uses Dijkstra's SPF algorithm to compute the best path to every destination.
3. **Fast Convergence:** Detects topology changes quickly and recalculates routes in seconds.
4. **Scalability:** Supports hierarchical design via areas (Area 0 backbone + stub/NSSA areas).

---

## Network Topology

```text
   Loopback0              Loopback0              Loopback0
    1.1.1.1                2.2.2.2                3.3.3.3
       |                      |                      |
  +---------+           +---------+           +---------+
  |   R1    |           |   R2    |           |   R3    |
  |         |  g1/0     |  g1/0   |  g2/0     |  g1/0   |
  |  .1     |-----------|  .2     |-----------|  .2     |
  +---------+           +---------+           +---------+
         192.168.1.0/24              192.168.2.0/24

              All routers: OSPF Process 110, Area 0
```
![Uploading image.png…]()

---

## Addressing Table

| Device | Interface | IP Address | Subnet Mask | OSPF Area |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Loopback0 | 1.1.1.1 | 255.255.255.255 | Area 0 |
| **R1** | GigabitEthernet1/0 | 192.168.1.1 | 255.255.255.0 | Area 0 |
| **R2** | Loopback0 | 2.2.2.2 | 255.255.255.255 | Area 0 |
| **R2** | GigabitEthernet1/0 | 192.168.1.2 | 255.255.255.0 | Area 0 |
| **R2** | GigabitEthernet2/0 | 192.168.2.1 | 255.255.255.0 | Area 0 |
| **R3** | Loopback0 | 3.3.3.3 | 255.255.255.255 | Area 0 |
| **R3** | GigabitEthernet1/0 | 192.168.2.2 | 255.255.255.0 | Area 0 |

---

## Step-by-Step Cisco IOS Commands

### Step 1: Configure IP Addressing on All Routers

**R1:**
```cisco
R1# configure terminal
R1(config)# interface Loopback0
R1(config-if)# ip address 1.1.1.1 255.255.255.255
R1(config-if)# exit
R1(config)# interface GigabitEthernet1/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

**R2:**
```cisco
R2# configure terminal
R2(config)# interface Loopback0
R2(config-if)# ip address 2.2.2.2 255.255.255.255
R2(config-if)# exit
R2(config)# interface GigabitEthernet1/0
R2(config-if)# ip address 192.168.1.2 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface GigabitEthernet2/0
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit
```

**R3:**
```cisco
R3# configure terminal
R3(config)# interface Loopback0
R3(config-if)# ip address 3.3.3.3 255.255.255.255
R3(config-if)# exit
R3(config)# interface GigabitEthernet1/0
R3(config-if)# ip address 192.168.2.2 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# exit
```

### Step 2: Enable OSPF and Advertise Networks

**R1:**
```cisco
R1(config)# router ospf 110
R1(config-router)# network 1.1.1.1 0.0.0.0 area 0
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# end
```

**R2:**
```cisco
R2(config)# router ospf 110
R2(config-router)# network 2.2.2.2 0.0.0.0 area 0
R2(config-router)# network 192.168.1.0 0.0.0.255 area 0
R2(config-router)# network 192.168.2.0 0.0.0.255 area 0
R2(config-router)# end
```

**R3:**
```cisco
R3(config)# router ospf 110
R3(config-router)# network 3.3.3.3 0.0.0.0 area 0
R3(config-router)# network 192.168.2.0 0.0.0.255 area 0
R3(config-router)# end
```

---

## How We Verified It Works

### 1. Verify OSPF Neighbor Adjacency (`show ip ospf neighbor`)

**R2** (connected to both R1 and R3) should show two FULL adjacencies:
```text
R2# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:33    192.168.1.1     GigabitEthernet1/0
3.3.3.3           1   FULL/BDR        00:00:31    192.168.2.2     GigabitEthernet2/0
```

### 2. Verify OSPF-Learned Routes (`show ip route ospf`)

**R1** should learn R2's loopback, R3's loopback, and the 192.168.2.0/24 subnet via OSPF:
```text
R1# show ip route ospf
      2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/2] via 192.168.1.2, GigabitEthernet1/0
      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/3] via 192.168.1.2, GigabitEthernet1/0
O     192.168.2.0/24 [110/2] via 192.168.1.2, GigabitEthernet1/0
```

### 3. Verify End-to-End Reachability (Ping Test)

From **R1**, ping R3's loopback (3.3.3.3) to confirm full OSPF-routed connectivity across both subnets:
```text
R1# ping 3.3.3.3 source 1.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.1
!!!!!
Success rate is 100 percent (5/5)
```

### 4. Verify OSPF Interface Details (`show ip ospf interface brief`)
```text
R2# show ip ospf interface brief
Interface    PID   Area   IP Address/Mask    Cost  State Nbrs F/C
Lo0          110   0      2.2.2.2/32         1     LOOP  0/0
Gi1/0        110   0      192.168.1.2/24     1     DR    1/1
Gi2/0        110   0      192.168.2.1/24     1     DR    1/1
```

---

## Key OSPF Concepts Demonstrated

| Concept | What It Means |
| :--- | :--- |
| **OSPF Process ID (110)** | Locally significant identifier; does not need to match across routers (but we used the same for clarity) |
| **Area 0 (Backbone)** | All OSPF routers must connect to Area 0 either directly or via a virtual link |
| **Wildcard Mask** | Inverse of the subnet mask, used in `network` statements (e.g., `0.0.0.255` = match the first 3 octets) |
| **Router ID** | Highest loopback IP is automatically elected as the OSPF Router ID (R1=1.1.1.1, R2=2.2.2.2, R3=3.3.3.3) |
| **DR/BDR Election** | On multi-access segments, OSPF elects a Designated Router (DR) and Backup DR (BDR) to reduce LSA flooding |
| **FULL State** | Indicates complete LSDB synchronization between neighbors |
| **Administrative Distance** | OSPF has AD of 110 (lower = more trusted; directly connected = 0, static = 1, OSPF = 110) |

---

## Simple Takeaways (What I Learned)

1. **Loopback as Router ID:** Using a Loopback interface guarantees a stable Router ID that never goes down, unlike physical interfaces.
2. **Wildcard masks are the inverse of subnet masks:** `255.255.255.0` subnet mask becomes `0.0.0.255` wildcard mask in OSPF `network` commands.
3. **R2 is the transit router:** Since R1 and R3 are not directly connected, all traffic between them traverses R2 — OSPF automatically computes this path.
4. **OSPF is classless:** It carries subnet mask information in its LSAs, supporting VLSM and CIDR.
