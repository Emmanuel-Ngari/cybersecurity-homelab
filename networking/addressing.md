# Homelab IP Addressing and Network Configuration

## 1. Overview

This document describes the IP addressing scheme used throughout my cybersecurity homelab.

The laboratory uses two separate network segments:

- NAT network for Internet connectivity
- LAB-NET for isolated communication between cybersecurity laboratory systems

The separation allows the laboratory systems to access required Internet resources while keeping security testing traffic isolated from the host's normal network environment.

---

## 2. Network Architecture

The homelab consists of three primary virtual machines:

| Virtual Machine | Role | NAT IP | LAB-NET IP |
|---|---|---|---|
| Kali Linux | Offensive Security | `10.0.2.15/24` | `192.168.56.10/24` |
| Windows 11 Enterprise | Windows Endpoint / Blue Team | `10.0.2.15/24` | `192.168.56.20/24` |
| Ubuntu Server | Linux Server / Blue Team | `10.0.2.15/24` | `192.168.56.30/24` |

The identical NAT addresses are expected because each VirtualBox VM has its own isolated NAT network.

---

## 3. NAT Network

The NAT interface provides Internet connectivity to the virtual machines.

**Network:** `10.0.2.0/24`

**Default Gateway:** `10.0.2.2`

**DNS:** `10.0.2.3`

The NAT interface is primarily used for:

- Operating system updates
- Package installation
- Downloading legitimate security tools
- Accessing cybersecurity learning resources
- Internet connectivity when required

The NAT network is not used as the primary communication network for laboratory security testing.

---

## 4. LAB-NET

LAB-NET is the isolated network used for communication between the cybersecurity laboratory systems.

**Network:** `192.168.56.0/24`

**Subnet Mask:** `255.255.255.0`

**Default Gateway:** None

The LAB-NET network is used for:

- Security testing
- Network scanning
- Service enumeration
- Blue Team monitoring
- Incident response exercises
- Linux and Windows security testing
- Communication between laboratory systems

The LAB-NET interface does not require an Internet gateway.

---

## 5. Kali Linux Addressing

Kali Linux is the primary offensive security workstation.

### NAT Interface

**Interface:** `eth0`

**IP Address:** `10.0.2.15/24`

**Gateway:** `10.0.2.2`

### LAB-NET Interface

**Interface:** `eth1`

**IP Address:** `192.168.56.10/24`

**Gateway:** None

Kali uses LAB-NET to communicate with the Windows and Ubuntu systems during authorized security testing.

---

## 6. Windows 11 Addressing

Windows 11 Enterprise is the primary Windows endpoint in the laboratory.

### NAT Adapter

**Adapter:** `NAT`

**IP Address:** `10.0.2.15/24`

**Gateway:** `10.0.2.2`

**DNS:** `10.0.2.3`

### LAB-NET Adapter

**Adapter:** `LAB-NET`

**IP Address:** `192.168.56.20/24`

**Gateway:** None

**DNS:** None

Windows uses LAB-NET for isolated communication with Kali Linux and Ubuntu Server.

---

## 7. Ubuntu Server Addressing

Ubuntu Server is the Linux server within the laboratory.

### NAT Interface

**Interface:** `enp0s3`

**IP Address:** `10.0.2.15/24`

**Gateway:** `10.0.2.2`

### LAB-NET Interface

**Interface:** `enp0s8`

**IP Address:** `192.168.56.30/24`

**Gateway:** None

The LAB-NET address was initially configured manually and was later converted to a persistent Netplan configuration.

The configuration was verified after reboot.

---

## 8. Complete Addressing Table

| System | Interface | Network | IPv4 Address | Gateway | Purpose |
|---|---|---|---|---|---|
| Kali Linux | `eth0` | NAT | `10.0.2.15/24` | `10.0.2.2` | Internet |
| Kali Linux | `eth1` | LAB-NET | `192.168.56.10/24` | None | Lab traffic |
| Windows 11 | `NAT` | NAT | `10.0.2.15/24` | `10.0.2.2` | Internet |
| Windows 11 | `LAB-NET` | LAB-NET | `192.168.56.20/24` | None | Lab traffic |
| Ubuntu Server | `enp0s3` | NAT | `10.0.2.15/24` | `10.0.2.2` | Internet |
| Ubuntu Server | `enp0s8` | LAB-NET | `192.168.56.30/24` | None | Lab traffic |

---

## 9. Routing Design

The laboratory uses separate routes for Internet traffic and isolated laboratory traffic.

### NAT Default Route

Internet-bound traffic uses:

`10.0.2.2`

as the default gateway.

### LAB-NET Route

Traffic destined for:

`192.168.56.0/24`

is sent through the LAB-NET interface.

This allows the systems to communicate directly without sending laboratory traffic through the NAT gateway.

---

## 10. Connectivity Matrix

The laboratory systems were tested for connectivity across LAB-NET.

| Source | Destination | Address | Result |
|---|---|---|---|
| Kali | Windows | `192.168.56.20` | 🟢 Successful |
| Kali | Ubuntu | `192.168.56.30` | 🟢 Successful |
| Windows | Kali | `192.168.56.10` | 🟢 Successful |
| Windows | Ubuntu | `192.168.56.30` | 🟢 Successful |
| Ubuntu | Kali | `192.168.56.10` | 🟢 Successful |
| Ubuntu | Windows | `192.168.56.20` | 🟢 Successful |

The successful connectivity tests confirmed that the LAB-NET network is functioning correctly.

---

## 11. Ubuntu Persistence Verification

Ubuntu's LAB-NET configuration was tested to ensure that the static address remained available after reboot.

The expected address:

`192.168.56.30/24`

was still present after the system restarted.

This confirmed that the Netplan configuration was persistent.

---

## 12. Network Troubleshooting

The laboratory required troubleshooting during initial network configuration.

Windows initially received an APIPA address in the:

`169.254.0.0/16`

range instead of the required LAB-NET address.

The Windows LAB-NET adapter was subsequently configured with:

`192.168.56.20/24`

Kali-to-Windows communication initially failed and required investigation of:

- IP addressing
- Network adapter configuration
- VirtualBox networking
- Windows network profile
- Windows Firewall
- ICMP traffic
- Routing

After troubleshooting, Kali and Windows successfully communicated across LAB-NET.

---

## 13. Network Isolation

LAB-NET provides an isolated communication environment for cybersecurity exercises.

The network is intended to keep offensive and defensive testing traffic within the controlled laboratory environment.

Examples of activities that can be performed on LAB-NET include:

- Port scanning
- Network discovery
- Service enumeration
- Vulnerability assessment
- Authentication testing
- Packet analysis
- Detection engineering
- Incident response exercises

All activities are restricted to authorized laboratory systems.

---

## 14. Why Two Network Interfaces Are Used

Using two network interfaces provides a practical separation between:

**Internet connectivity**

and

**Cybersecurity laboratory traffic**

The NAT interface allows the virtual machines to reach the Internet when required.

The LAB-NET interface allows the virtual machines to communicate with each other without using the Internet-facing route.

This architecture provides a useful foundation for offensive and defensive cybersecurity training.

---

## 15. Security Benefits

The two-network architecture provides several security benefits:

- Separates laboratory traffic from normal Internet traffic
- Reduces accidental interaction with external systems
- Provides a predictable private IP addressing scheme
- Makes network monitoring easier
- Supports controlled offensive security exercises
- Supports Blue Team monitoring
- Simplifies troubleshooting
- Provides a realistic multi-host environment

---

## 16. Network Verification Commands

The following commands are useful for verifying the network configuration.

### Kali Linux

`ip addr`

`ip route`

`ping -c 4 192.168.56.20`

`ping -c 4 192.168.56.30`

### Ubuntu Server

`ip addr`

`ip route`

`ping -c 4 192.168.56.10`

`ping -c 4 192.168.56.20`

### Windows PowerShell

`Get-NetAdapter`

`Get-NetIPConfiguration`

`Get-NetIPAddress`

`Get-NetRoute`

`ping 192.168.56.10`

`ping 192.168.56.30`

These commands are used to verify interface state, addressing, routing, and connectivity.

---

## 17. Network Troubleshooting Methodology

When a connectivity problem occurs, the following troubleshooting sequence will be used:

1. Verify that the VM is running.
2. Verify the VirtualBox adapter configuration.
3. Check the network interface state.
4. Check the assigned IPv4 address.
5. Check the subnet mask.
6. Check the routing table.
7. Check the destination host.
8. Test connectivity with ICMP.
9. Check firewall configuration.
10. Check network profiles.
11. Review relevant logs.
12. Retest connectivity.
13. Document the problem and resolution.

This creates a repeatable troubleshooting methodology rather than relying on trial and error.

---

## 18. Planned Network Security Labs

The addressing architecture will support future cybersecurity exercises including:

### Network Discovery

- Host discovery
- ARP analysis
- Network mapping

### Network Scanning

- Port scanning
- Service discovery
- Version detection

### Packet Analysis

- Wireshark
- TCP/IP analysis
- DNS analysis
- HTTP traffic analysis

### Defensive Security

- Firewall configuration
- Network monitoring
- Suspicious traffic detection
- Intrusion detection

### Offensive Security

- Enumeration
- Vulnerability assessment
- Controlled exploitation
- Privilege escalation

All offensive activities will remain inside the isolated laboratory.

---

## 19. Current Network Status

**Status:** 🟢 Operational

The current laboratory addressing scheme is:

**NAT**

`10.0.2.0/24`

↓  

**LAB-NET**

`192.168.56.0/24`

### Current LAB-NET Hosts

`192.168.56.10` → Kali Linux

`192.168.56.20` → Windows 11

`192.168.56.30` → Ubuntu Server

All three systems are currently operational and can communicate across LAB-NET.

Ubuntu's LAB-NET configuration has also been verified to persist after reboot.

---

## 20. Security Disclaimer

This network architecture is designed exclusively for authorized cybersecurity education, testing, and experimentation.

All scanning, enumeration, exploitation, monitoring, and other security activities are performed only against systems that I own or have explicit authorization to test.

No unauthorized external systems or networks are targeted.

The homelab is maintained as a controlled environment for developing practical cybersecurity skills.
