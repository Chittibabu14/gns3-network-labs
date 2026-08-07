# 🔒 Lab 02: Secure SSH Remote Management & Device Hardening

**Date:** 2026-08-07  
**Difficulty:** Beginner / Intermediate  
**Platform:** GNS3 (Cisco c7200 IOS 15.3)  
**Status:** Completed & Verified ✅  

---

## 📐 1. Network Topology
```text
  ┌─────────────────┐                       ┌─────────────────┐
  │  Router Banglore│                       │  Router Chennai │
  │ (192.168.12.1)  ├──────[ Gi1/0 Link ]────┤ (192.168.12.2)  │
  └─────────────────┘                       └─────────────────┘
```

---

## 📋 2. Addressing & Security Table

| Router Name | Interface | IP Address | Subnet Mask | Domain Name | SSH User | Transport Security |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Banglore** | Gi1/0 | 192.168.12.1 | 255.255.255.0 | `thnetworks` | `admin` | `transport input ssh` |
| **Chennai** | Gi1/0 | 192.168.12.2 | 255.255.255.0 | `tbnetworks` | `admin` | `transport input ssh` |

---

## 🎯 3. Lab Objectives
1. Configure hostnames (`Banglore` and `Chennai`) and IP addresses on point-to-point Gigabit Ethernet links.
2. Hardening device access by configuring **Encrypted Enable Secret Password**.
3. Create local administrator credentials (`username admin password ...`).
4. Set up IP Domain Name and generate **RSA Crypto Key Pair** (1024-bit/2048-bit) to enable **SSHv2**.
5. Restrict VTY line access (`line vty 0 4`) to strictly accept SSH (`transport input ssh`) and block insecure Telnet connections.
6. Verify remote management via SSH from `Chennai` to `Banglore` and capture encrypted traffic packets in Wireshark.

---

## ⚙️ 4. Device Configurations

### **Router Banglore Setup**
```cisco
Banglore# configure terminal
Banglore(config)# hostname Banglore
Banglore(config)# enable secret 5 $1$WtO5$bCntBnG8lfg5j8EVK2vIG0
Banglore(config)# no ip domain lookup
Banglore(config)# ip domain name thnetworks
Banglore(config)# crypto key generate rsa modulus 1024
Banglore(config)# username admin password admin123

Banglore(config)# interface GigabitEthernet1/0
Banglore(config-if)# ip address 192.168.12.1 255.255.255.0
Banglore(config-if)# no shutdown
Banglore(config-if)# exit

Banglore(config)# line vty 0 4
Banglore(config-line)# login local
Banglore(config-line)# transport input ssh
Banglore(config-line)# transport output ssh
Banglore(config-line)# end
Banglore# copy running-config startup-config
```

### **Router Chennai Setup**
```cisco
Chennai# configure terminal
Chennai(config)# hostname Chennai
Chennai(config)# enable secret 5 $1$rEQo$xEQ8Y.qpIAJ4MQq/kiBop1
Chennai(config)# no ip domain lookup
Chennai(config)# ip domain name tbnetworks
Chennai(config)# crypto key generate rsa modulus 1024
Chennai(config)# username admin password admin123

Chennai(config)# interface GigabitEthernet1/0
Chennai(config-if)# ip address 192.168.12.2 255.255.255.0
Chennai(config-if)# no shutdown
Chennai(config-if)# exit

Chennai(config)# line vty 0 10
Chennai(config-line)# login local
Chennai(config-line)# transport input ssh
Chennai(config-line)# transport output ssh
Chennai(config-line)# end
Chennai# copy running-config startup-config
```

---

## ✅ 5. Verification & Testing

### **1. Verify SSH Server Status (`show ip ssh`)**
```text
Banglore# show ip ssh
SSH Enabled - version 1.99
Authentication timeout: 120 secs; Authentication retries: 3
```

### **2. Initiate Remote SSH Session (from `Chennai` to `Banglore`)**
```text
Chennai# ssh -l admin 192.168.12.1
Password: 

Banglore> enable
Password: 
Banglore# 
```

### **3. Verify Active VTY Connections (`show users`)**
```text
Banglore# show users
    Line       User       Host(s)              Idle       Location
*  0 con 0     admin      idle                 00:00:00   
   2 vty 0     admin      idle                 00:01:23   192.168.12.2
```

---

## 🔍 6. Security Analysis & Key Takeaways
1. **Telnet vs SSH:** Configuring `transport input ssh` prevents eavesdroppers from capturing management credentials in plain text.
2. **Local Authentication:** Using `login local` forces VTY lines to authenticate against the router's local database (`username admin`).
3. **Wireshark Analysis:** Captured traffic during the session confirmed payload content was encrypted via SSH-2.0, masking commands and password entries.
