# VirtualBox Networking Configuration

## 1. Overview

This document describes the VirtualBox networking configuration used to build my cybersecurity homelab.

The laboratory uses two network interfaces on each virtual machine:

1. NAT — Internet connectivity
2. LAB-NET — Isolated laboratory communication

This design allows the systems to access the Internet when required while keeping cybersecurity experiments within a controlled laboratory network.

---

## 2. Network Design

The network architecture consists of:

- NAT for Internet access
- LAB-NET for isolated VM-to-VM communication
- Three virtual machines
- Static IP addressing on LAB-NET

The three virtual machines are:

| Virtual Machine | Primary Role | LAB-NET IP |
|---|---|---|
| Kali Linux | Offensive Security | `192.168.56.10` |
| Windows 11 Enterprise | Endpoint / Defensive Security | `192.168.56.20` |
| LAB-UBUNTU-01 | Linux Server / Infrastructure | `192.168.56.30` |

---

## 3. Adapter 1 — NAT

The first network adapter on each virtual machine is configured as:

**Network Mode:** NAT

The NAT interface provides Internet connectivity to the virtual machines.

### Primary Uses

- Operating system updates
- Installing packages
- Downloading security tools
- Accessing legitimate learning resources
- General Internet connectivity

The NAT network uses the VirtualBox NAT gateway.

The default gateway observed in the laboratory is:

`10.0.2.2`

---

## 4. Adapter 2 — LAB-NET

The second network adapter is used for the isolated cybersecurity laboratory network.

**Network:** LAB-NET

**Network Range:** `192.168.56.0/24`

The LAB-NET interface does not use an Internet gateway.

Its purpose is to provide direct communication between the laboratory virtual machines.

---

## 5. LAB-NET Addressing

The following static addresses are assigned to the laboratory systems:

| System | LAB-NET Interface | IP Address | Subnet Mask |
|---|---|---|---|
| Kali Linux | `eth1` | `192.168.56.10` | `/24` |
| Windows 11 | `LAB-NET` adapter | `192.168.56.20` | `/24` |
| Ubuntu Server | `enp0s8` | `192.168.56.30` | `/24` |

The subnet is:

`192.168.56.0/24`

The LAB-NET network does not have a default gateway configured.

---

## 6. Kali Linux Configuration

Kali Linux uses two interfaces.

### NAT Interface

**Interface:** `eth0`

**Purpose:** Internet connectivity

**Example Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

### LAB-NET Interface

**Interface:** `eth1`

**IP Address:** `192.168.56.10/24`

**Purpose:** Communication with the isolated laboratory network

The routing table contains a route for:

`192.168.56.0/24`

through the LAB-NET interface.

---

## 7. Windows 11 Configuration

Windows 11 uses two network adapters.

### NAT Adapter

**Adapter Name:** `NAT`

**IP Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

**Purpose:** Internet connectivity

### LAB-NET Adapter

**Adapter Name:** `LAB-NET`

**IP Address:** `192.168.56.20/24`

**Default Gateway:** None

**DNS:** None

**Purpose:** Isolated laboratory communication

The Windows LAB-NET adapter was configured with a static IPv4 address.

---

## 8. Ubuntu Server Configuration

Ubuntu Server uses two network interfaces.

### NAT Interface

**Interface:** `enp0s3`

**IP Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

**Purpose:** Internet connectivity

### LAB-NET Interface

**Interface:** `enp0s8`

**IP Address:** `192.168.56.30/24`

**Purpose:** Isolated laboratory communication

The LAB-NET address was initially configured manually during laboratory setup.

Persistent Netplan configuration will be documented separately after the configuration is finalized.

---

## 9. Routing Design

The routing design separates Internet traffic from laboratory traffic.

### Internet Traffic

Internet-bound traffic uses the NAT interface.

Example:

`0.0.0.0/0 → 10.0.2.2`

### Laboratory Traffic

Traffic destined for the LAB-NET uses the isolated interface.

Example:

`192.168.56.0/24 → LAB-NET`

This prevents the LAB-NET interface from becoming the default route to the Internet.

---

## 10. Connectivity Testing

After configuring the network interfaces, connectivity between the laboratory systems was tested.

### Kali → Windows

Source:

`192.168.56.10`

Destination:

`192.168.56.20`

**Result:** Successful

### Kali → Ubuntu

Source:

`192.168.56.10`

Destination:

`192.168.56.30`

**Result:** Successful

### Windows → Kali

Source:

`192.168.56.20`

Destination:

`192.168.56.10`

**Result:** Successful

### Windows → Ubuntu

Source:

`192.168.56.20`

Destination:

`192.168.56.30`

**Result:** Successful

### Ubuntu → Kali

Source:

`192.168.56.30`

Destination:

`192.168.56.10`

**Result:** Successful

### Ubuntu → Windows

Source:

`192.168.56.30`

Destination:

`192.168.56.20`

**Result:** Successful

---

## 11. Connectivity Troubleshooting

During the initial configuration, connectivity between some systems did not work immediately.

One of the issues involved communication between Kali Linux and Windows 11.

The problem required investigation of:

- IP addressing
- Network interfaces
- Routing
- Windows network profile
- Windows Firewall
- ICMP connectivity
- VirtualBox adapter configuration

After troubleshooting and correcting the configuration, communication between Kali Linux and Windows was successfully established.

This demonstrated an important networking principle:

**A correct IP address alone does not guarantee connectivity.**

Network configuration, routing, firewall rules, and interface settings must all be considered.

---

## 12. Security Considerations

The LAB-NET network is intended for authorized cybersecurity experimentation.

The design provides:

- Network isolation
- Controlled VM-to-VM communication
- Separation from normal Internet traffic
- A dedicated environment for security testing

All offensive security activities performed within this network are restricted to systems that I own or have explicit authorization to test.

---

## 13. VirtualBox Network Model

The conceptual model is:

Internet
↓
VirtualBox NAT
↓
VM NAT Interfaces

And separately:

Kali Linux
`192.168.56.10`
↓
LAB-NET
↓
Windows 11
`192.168.56.20`
↓
LAB-NET
↓
Ubuntu Server
`192.168.56.30`

The two networks serve different purposes and are intentionally separated.

---

## 14. Future Improvements

Future networking improvements will include:

- Persistent Ubuntu Netplan configuration
- Network traffic monitoring
- Packet capture and analysis
- Additional isolated networks
- Active Directory network segment
- Dedicated attacker and victim VLAN-style segments
- Centralized logging
- Security monitoring
- Network intrusion detection

---

## 15. Lessons Learned

Building the VirtualBox network provided practical experience with:

- NAT networking
- Isolated networks
- Static IPv4 addressing
- Network interfaces
- Routing
- Default gateways
- Network segmentation
- Windows Firewall
- Linux networking
- Connectivity troubleshooting

The lab demonstrates how virtualization can be used to create a controlled cybersecurity environment for practical security training.

---

## 16. Security Disclaimer

This laboratory is designed for authorized cybersecurity training and experimentation.

All security testing is performed against systems within my controlled laboratory environment.

No unauthorized systems, networks, accounts, or organizations are targeted.
