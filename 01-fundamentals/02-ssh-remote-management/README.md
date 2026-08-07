# 🔒 Lab 02: Secure Remote Access using SSH

Welcome to Lab 02! In this lab, I configured **SSH (Secure Shell)** on Cisco routers to securely control a router remotely over the network without exposing passwords to hackers.

---

## 🤔 What is SSH & Why Do We Use It?

When network engineers want to configure a router in another city (like Bangalore or Chennai), they don't drive there — they log into it remotely over the network.

There are two main ways to log in remotely:
- ❌ **Telnet (Insecure):** Sends your username and password in **plain text**. Anyone listening on the network (using tools like Wireshark) can easily steal your password!
- ✅ **SSH (Secure):** Encrypts all traffic using **RSA encryption keys**. Even if someone captures the traffic, it looks like unreadable gibberish.

---

## 📐 Network Setup Diagram

```text
  ┌────────────────────────┐                             ┌────────────────────────┐
  │    Router Banglore     │                             │     Router Chennai     │
  │     (192.168.12.1)     ├──────────[ Network ]────────┤     (192.168.12.2)     │
  │                        │  ◄── [ Encrypted SSH ] ───  │  (Initiates SSH Log)   │
  └────────────────────────┘                             └────────────────────────┘
```

---

<img width="583" height="210" alt="SSH_Topology" src="https://github.com/user-attachments/assets/d1559606-d5ef-43b3-b55d-5180b16b02fe" />


## 📋 Device Details

| Device Name | Interface | IP Address | Domain Name | SSH User | SSH Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Banglore** | GigabitEthernet1/0 | `192.168.12.1/24` | `thnetworks` | `admin` | Enabled (Server) |
| **Chennai** | GigabitEthernet1/0 | `192.168.12.2/24` | `tbnetworks` | `admin` | Enabled (Client) |

---

## 🛠️ Step-by-Step Commands Explained

To enable SSH on a Cisco router, we must follow 5 essential steps:

### **Step 1: Set Hostname & IP Address**
```cisco
Router# configure terminal
Router(config)# hostname Banglore
Banglore(config)# interface GigabitEthernet1/0
Banglore(config-if)# ip address 192.168.12.1 255.255.255.0
Banglore(config-if)# no shutdown
```

### **Step 2: Set Domain Name & Generate RSA Encryption Keys**
*SSH requires a domain name to generate encryption keys.*
```cisco
Banglore(config)# ip domain name thnetworks
Banglore(config)# crypto key generate rsa
! When prompted for key size, enter 1024 (or 2048)
How many bits in the modulus [512]: 1024
```

### **Step 3: Create Local Admin User & Enable Password**
```cisco
! Create admin user for logging in
Banglore(config)# username admin password admin123

! Protect privileged mode (#) with an encrypted enable secret password
Banglore(config)# enable secret admin
```

### **Step 4: Lock Down VTY Lines to ONLY Accept SSH**
*This blocks old, insecure Telnet access completely.*
```cisco
Banglore(config)# line vty 0 4
Banglore(config-line)# login local           <-- Use router's username/password
Banglore(config-line)# transport input ssh   <-- Allow ONLY SSH connections!
Banglore(config-line)# transport output ssh
Banglore(config-line)# end
Banglore# copy running-config startup-config
```

---

## ✅ How We Verified it Works

From Router **`Chennai`**, we initiated an SSH connection to Router **`Banglore`**:

```text
Chennai# ssh -l admin 192.168.12.1
Password: admin123

Banglore> enable
Password: admin
Banglore# 
```

🎉 **Success!** We logged into Router `Banglore` remotely from `Chennai`.

---

## 🔍 Packet Capture (Wireshark)

The raw Wireshark traffic capture file for this lab is saved in the repository:
📁 **[`pcaps/ssh_session_traffic.pcap`](./pcaps/ssh_session_traffic.pcap)**

### What you can see in this capture file:
- **TCP 3-Way Handshake (`SYN`, `SYN-ACK`, `ACK`):** Establishes the TCP connection on Port `22`.
- **SSH Protocol Negotiation:** Routers negotiate `SSH-2.0` protocol capabilities.
- **Diffie-Hellman Key Exchange:** Exchanging keys to establish an encrypted tunnel.
- **Encrypted Payload:** All subsequent user credentials, keystrokes, and output are completely encrypted.

---

## 🎓 Simple Takeaways (What I Learned)

1. **Prerequisites for SSH:** You MUST configure a **hostname**, a **domain name**, and generate **RSA crypto keys** before SSH will turn on.
2. **`transport input ssh`:** This command is crucial for security. It turns off Telnet and ensures only encrypted SSH connections are accepted.
3. **Wireshark Proof:** When analyzing the packet capture in Wireshark, the payload showed `SSH-2.0`, proving that credentials and commands were completely encrypted.
