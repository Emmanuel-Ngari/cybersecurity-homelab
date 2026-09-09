# Cybersecurity Homelab Architecture

## 1. Overview

This document describes the architecture, network design, virtual machines, and connectivity of my cybersecurity homelab.

The environment is designed to provide an isolated and controlled platform for practicing cybersecurity concepts across offensive security, defensive security, networking, Linux administration, Windows security, and incident response.

---

## 2. Lab Objectives

The architecture is designed to provide:

- An isolated environment for security experimentation
- Internet access for system updates and software installation
- Controlled communication between laboratory systems
- Separate roles for offensive and defensive activities
- A realistic environment for attack-and-defense exercises
- A safe environment for troubleshooting and experimentation

---

## 3. Virtual Machines

The current laboratory consists of three primary virtual machines.

| Virtual Machine | Operating System | Primary Role | LAB-NET Address |
|---|---|---|---|
| Kali Linux | Kali Linux | Offensive Security | `192.168.56.10` |
| Windows 11 | Windows 11 Enterprise | Endpoint / Defensive Security | `192.168.56.20` |
| LAB-UBUNTU-01 | Ubuntu Server | Linux Server / Infrastructure | `192.168.56.30` |

---

## 4. Network Architecture

The laboratory uses two network connections on the virtual machines.

### 4.1 NAT Network

The NAT interface provides controlled Internet access.

**Purpose:**

- Operating system updates
- Installing security tools
- Downloading packages
- Accessing legitimate external learning resources

The NAT network is not used as the primary network for laboratory attack simulations.

### 4.2 LAB-NET

The LAB-NET network is the isolated laboratory network used for communication between the virtual machines.

**Network:** `192.168.56.0/24`

### LAB-NET Addressing

Kali Linux: `192.168.56.10`

Windows 11: `192.168.56.20`

Ubuntu Server: `192.168.56.30`

There is no Internet gateway configured on LAB-NET.

This helps keep laboratory traffic separated from normal Internet traffic.

---

## 5. Network Interface Design

Each virtual machine uses two network interfaces.

| Interface | Purpose |
|---|---|
| Adapter 1 | NAT / Internet connectivity |
| Adapter 2 | LAB-NET / Isolated laboratory traffic |

The LAB-NET interface is used for VM-to-VM communication and security testing.

---

## 6. Connectivity Testing

Connectivity between the systems was tested after configuring the laboratory network.

### Kali → Windows

`192.168.56.10 → 192.168.56.20`

**Result:** SUCCESS

### Kali → Ubuntu

`192.168.56.10 → 192.168.56.30`

**Result:** SUCCESS

### Windows → Kali

`192.168.56.20 → 192.168.56.10`

**Result:** SUCCESS

### Windows → Ubuntu

`192.168.56.20 → 192.168.56.30`

**Result:** SUCCESS

### Ubuntu → Kali

`192.168.56.30 → 192.168.56.10`

**Result:** SUCCESS

### Ubuntu → Windows

`192.168.56.30 → 192.168.56.20`

**Result:** SUCCESS

---

## 7. Security Boundaries

The laboratory has been designed with a separation between Internet connectivity and laboratory traffic.

The NAT interface provides Internet access, while LAB-NET is used for isolated communication between the laboratory systems.

The isolated LAB-NET provides the primary environment for security experimentation.

---

## 8. System Roles

### Kali Linux — Attacker

Kali Linux represents the offensive security workstation.

Planned activities include:

- Reconnaissance
- Network scanning
- Service enumeration
- Vulnerability assessment
- Exploitation within the lab
- Privilege escalation
- Post-exploitation analysis
- Penetration testing

### Windows 11 — Endpoint

Windows represents a typical endpoint that may be monitored, hardened, and tested.

Planned activities include:

- Windows administration
- PowerShell
- Windows Firewall
- Event logging
- User and privilege management
- Endpoint security
- Attack detection
- Hardening

### Ubuntu Server — Server

Ubuntu represents a Linux server within the laboratory network.

Planned activities include:

- Linux administration
- SSH
- User and permission management
- Service management
- Firewall configuration
- Log analysis
- Server hardening
- Security monitoring

---

## 9. Current Architecture

The current laboratory architecture consists of three virtual machines connected to an isolated LAB-NET while also maintaining NAT connectivity for Internet access.

**Internet → NAT → Virtual Machines**

**LAB-NET → Kali Linux, Windows 11, Ubuntu Server**

LAB-NET addressing:

- Kali Linux — `192.168.56.10`
- Windows 11 — `192.168.56.20`
- Ubuntu Server — `192.168.56.30`

---

## 10. Future Improvements

The homelab will continue to evolve.

Planned improvements include:

- Persistent network configuration
- Centralized logging
- SIEM deployment
- Windows event collection
- Network traffic monitoring
- Vulnerability scanning
- Active Directory environment
- Detection engineering
- Incident response scenarios
- Attack-and-defense exercises
- Security automation
- Additional Linux and Windows systems

---

## 11. Lessons Learned

Building the network provided practical experience with:

- Virtual machine networking
- NAT
- Isolated networks
- IPv4 addressing
- Network interfaces
- Routing
- Windows Firewall
- Linux networking
- Connectivity troubleshooting
- Network segmentation

A major lesson from the initial configuration was that successful network communication requires more than assigning IP addresses. Interface configuration, routing, firewall rules, and network profiles must all be considered.

---

## 12. Security Disclaimer

This environment is a controlled cybersecurity laboratory.

All offensive security activities documented in this repository are performed only against systems that I own or have explicit authorization to test.

No unauthorized systems or networks are targeted.
