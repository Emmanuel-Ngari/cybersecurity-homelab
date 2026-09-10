# OPNsense Firewall — LAB-FW-01

## 1. Overview

This document describes the deployment and initial configuration of OPNsense as the central firewall, router, and security gateway for my cybersecurity homelab.

The firewall was introduced to replace the original flat laboratory design with a segmented architecture that provides controlled communication between offensive, defensive, corporate, and server systems.

The firewall is deployed as a dedicated VirtualBox virtual machine named:

`LAB-FW-01`

---

## 2. Firewall Platform

**Platform:** OPNsense

**Version:** OPNsense 26.7

**Virtualization Platform:** Oracle VirtualBox

**Primary Functions:**

- Stateful firewalling
- Network segmentation
- Routing
- NAT
- Access control
- Security logging
- DNS services
- NTP services
- Future IDS/IPS integration
- Future VPN integration

OPNsense acts as the central security enforcement point for the homelab.

---

## 3. Virtual Machine Configuration

The firewall VM was created with the following resources:

| Resource | Configuration |
|---|---|
| VM Name | `LAB-FW-01` |
| Operating System Type | FreeBSD 64-bit |
| vCPU | 2 |
| Memory | 4096 MB |
| Virtual Disk | 32 GB |
| Disk Type | VDI |
| Storage | Dynamically allocated |

The OPNsense DVD ISO was used to install the firewall onto the virtual disk.

---

## 4. Firewall Architecture

The target architecture is:

Internet  
↓  
VirtualBox NAT  
↓  
OPNsense WAN  
↓  
Firewall / Routing / Security Policy  
↓  
Multiple isolated security zones

The firewall separates the laboratory into multiple networks rather than placing every system on a single flat subnet.

---

## 5. VirtualBox Network Adapters

LAB-FW-01 uses four VirtualBox network adapters.

| Adapter | VirtualBox Network | OPNsense Interface | Security Role |
|---|---|---|---|
| Adapter 1 | NAT | `em0` | WAN |
| Adapter 2 | `CORP-NET` | `em1` | Corporate / Management |
| Adapter 3 | `RED-NET` | `em2` | Offensive Security |
| Adapter 4 | `SERVER-NET` | `em3` | Protected Servers |

The interface mappings were verified using MAC addresses before final assignment.

---

## 6. MAC Address Verification

Interface assignment was verified instead of assuming that VirtualBox adapter numbers would automatically match FreeBSD interface names.

The verified mappings were:

| Interface | MAC Address | Role |
|---|---|---|
| `em0` | `08:00:27:3B:19:9B` | WAN |
| `em1` | `08:00:27:04:74:7C` | CORPNET |
| `em2` | `08:00:27:6C:10:29` | REDNET |
| `em3` | `08:00:27:93:06:8C` | SERVERNET |

This prevented accidental assignment of an internal network as WAN or an attacker network as the management interface.

---

## 7. WAN Configuration

The WAN interface is:

`em0`

The interface connects to:

`VirtualBox NAT`

Current addressing:

**IPv4 Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

**Configuration Method:** DHCP

The WAN interface provides Internet connectivity to OPNsense.

Later, internal laboratory systems will access the Internet through:

Internal VM  
↓  
OPNsense  
↓  
WAN  
↓  
VirtualBox NAT  
↓  
Internet

This will prevent laboratory systems from bypassing the firewall using their own direct NAT adapters.

---

## 8. CORPNET Configuration

The trusted management and corporate network is:

`CORPNET`

**Interface:** `em1`

**IPv4 Address:** `10.10.20.1/24`

**Network:** `10.10.20.0/24`

**Gateway:** None

CORPNET is intended to contain trusted endpoint and management systems.

The current management workstation is Windows 11:

`10.10.20.20`

The Windows workstation is being used to access the OPNsense Web GUI securely using HTTPS.

---

## 9. REDNET Configuration

The offensive security network is:

`REDNET`

**Interface:** `em2`

**IPv4 Address:** `10.10.10.1/24`

**Network:** `10.10.10.0/24`

**Gateway:** None

REDNET will contain Kali Linux and other offensive security systems.

Future Kali address:

`10.10.10.10`

The attacker network will not be treated as a trusted management network.

Access from REDNET to other zones will be explicitly controlled using firewall policies.

---

## 10. SERVERNET Configuration

The protected server network is:

`SERVERNET`

**Interface:** `em3`

**IPv4 Address:** `10.10.30.1/24`

**Network:** `10.10.30.0/24`

**Gateway:** None

SERVERNET will contain Linux servers and other protected services.

Future Ubuntu Server address:

`10.10.30.30`

Traffic entering and leaving SERVERNET will be controlled by OPNsense firewall rules.

---

## 11. Interface Routing

The OPNsense routing table was verified from the FreeBSD shell.

Connected networks include:

`10.10.10.0/24 → em2`

`10.10.20.0/24 → em1`

`10.10.30.0/24 → em3`

The default Internet route uses:

`10.0.2.2`

through:

`em0`

This confirms that OPNsense has direct Layer 3 connectivity to all internal security zones.

---

## 12. Management Access

The OPNsense Web GUI is currently accessed from the Windows management workstation.

Firewall management address:

`https://10.10.20.1`

Management workstation:

`10.10.20.20`

An explicit firewall rule was created to allow:

`ADMIN_WORKSTATION → This Firewall → TCP/443`

The rule is named:

`ALLOW - Admin Workstation to OPNsense HTTPS`

Management access was tested from Windows using:

`Test-NetConnection 10.10.20.1 -Port 443`

The result returned:

`TcpTestSucceeded : True`

This verified HTTPS connectivity between the management workstation and OPNsense.

---

## 13. Firewall Aliases

Aliases were created to simplify firewall administration and improve readability.

### ADMIN_WORKSTATION

**Type:** Host(s)

**Value:**

`10.10.20.20`

Purpose:

Identifies the trusted Windows management workstation.

### LAB_INTERNAL_NETS

**Type:** Network(s)

Values:

`10.10.10.0/24`

`10.10.20.0/24`

`10.10.30.0/24`

Purpose:

Represents the primary laboratory security zones.

### PRIVATE_NETS

**Type:** Network(s)

Values:

`10.0.0.0/8`

`172.16.0.0/12`

`192.168.0.0/16`

Purpose:

Represents RFC1918 private addressing and is used to help prevent unauthorized inter-network access.

### SERVER_ADMIN_PORTS

**Type:** Port(s)

Values:

`22`

`443`

Purpose:

Represents approved server administration services:

- SSH
- HTTPS

### DNS_PORT

**Type:** Port(s)

Value:

`53`

Purpose:

DNS traffic.

### NTP_PORT

**Type:** Port(s)

Value:

`123`

Purpose:

NTP traffic.

---

## 14. CORPNET Firewall Policy

The initial CORPNET policy was designed around least privilege.

The explicit rules include:

### Rule 1

`ALLOW - Admin Workstation to OPNsense HTTPS`

Allows:

`10.10.20.20 → OPNsense → TCP/443`

Purpose:

Secure firewall management.

### Rule 2

`ALLOW - CORPNET to OPNsense DNS`

Allows CORPNET systems to query the firewall for DNS.

### Rule 3

`ALLOW - CORPNET to OPNsense NTP`

Allows CORPNET systems to use the firewall for time synchronization.

### Rule 4

`BLOCK - Unauthorized CORPNET Access to Firewall`

Blocks other CORPNET traffic attempting to access services directly on OPNsense unless explicitly allowed by an earlier rule.

### Rule 5

`ALLOW - Admin Workstation to SERVERNET Admin Services`

Allows the trusted management workstation to access approved administration services in SERVERNET.

Approved ports:

`22`

`443`

### Rule 6

`BLOCK - CORPNET to Unauthorized Private Networks`

Blocks access from CORPNET to private networks unless an earlier explicit allow rule permits the traffic.

### Rule 7

`ALLOW - CORPNET to Internet`

Allows remaining permitted CORPNET traffic to access external networks.

---

## 15. Firewall Rule Ordering

Firewall rule ordering is important because the rules are configured using quick processing.

The intended CORPNET order is:

1. `ALLOW - Admin Workstation to OPNsense HTTPS`
2. `ALLOW - CORPNET to OPNsense DNS`
3. `ALLOW - CORPNET to OPNsense NTP`
4. `BLOCK - Unauthorized CORPNET Access to Firewall`
5. `ALLOW - Admin Workstation to SERVERNET Admin Services`
6. `BLOCK - CORPNET to Unauthorized Private Networks`
7. `ALLOW - CORPNET to Internet`
8. Existing default LAN allow rule during migration
9. Existing default IPv6 LAN rule during migration

The broad default rules are temporarily retained while the explicit policy is being built and tested.

They will be removed or disabled only after migration testing confirms that the custom policy works correctly.

---

## 16. Current Security-Zone Policy

The planned security policy is:

| Source | Destination | Policy |
|---|---|---|
| Internet | Internal Networks | Block |
| CORPNET | OPNsense HTTPS | Allow from admin workstation |
| CORPNET | OPNsense DNS | Allow |
| CORPNET | OPNsense NTP | Allow |
| CORPNET | SERVERNET | Limited administrative access |
| CORPNET | REDNET | Block by default |
| CORPNET | Internet | Allow |
| REDNET | CORPNET | Block |
| REDNET | OPNsense Management | Block |
| REDNET | SERVERNET | Controlled offensive testing |
| REDNET | Internet | Controlled allow |
| SERVERNET | CORPNET | Block unsolicited traffic |
| SERVERNET | REDNET | Block unsolicited traffic |
| SERVERNET | Internet | Controlled access |

This design follows a default-deny and explicit-access approach.

---

## 17. Configuration Backup

A manual OPNsense configuration backup was exported after completing the initial firewall and network-zone configuration.

The backup is stored privately outside the public GitHub repository.

Firewall configuration backups are not committed to public source control because they may contain sensitive configuration information.

Recovery options include:

- OPNsense configuration backup
- VirtualBox snapshots
- Manual configuration documentation

---

## 18. Security Principles

The firewall architecture follows several important security principles.

### Network Segmentation

Systems with different security roles are placed on separate networks.

### Least Privilege

Only explicitly required communication should be permitted between zones.

### Default Deny

Traffic that has not been explicitly approved should eventually be denied.

### Dedicated Management

Firewall administration is performed from a trusted management network rather than from the attacker network.

### Centralized Routing

Inter-zone communication must pass through OPNsense.

### Logging

Important firewall rules have logging enabled so traffic decisions can be reviewed and investigated.

### Controlled Offensive Testing

Kali Linux will be able to perform authorized testing against designated laboratory systems without receiving unrestricted access to trusted networks.

---

## 19. Planned Firewall Improvements

Future improvements include:

- Complete REDNET firewall policy
- Complete SERVERNET firewall policy
- Remove direct NAT adapters from internal VMs
- Route all internal VM Internet traffic through OPNsense
- DHCP configuration
- DNS architecture improvements
- Detailed firewall logging
- Suricata IDS
- Suricata IPS
- Threat detection rules
- GeoIP experimentation
- VLAN experimentation
- VPN configuration
- DMZ network
- Dedicated SOC network
- Active Directory network
- Centralized SIEM logging
- Firewall log forwarding to Wazuh
- Security monitoring dashboards
- Attack detection exercises
- Incident response exercises

---

## 20. Current Status

**Firewall Status:** 🟢 Operational

Current interface architecture:

WAN:

`em0 → 10.0.2.15/24`

CORPNET:

`em1 → 10.10.20.1/24`

REDNET:

`em2 → 10.10.10.1/24`

SERVERNET:

`em3 → 10.10.30.1/24`

Windows management workstation:

`10.10.20.20`

OPNsense HTTPS management connectivity has been successfully tested.

The CORPNET explicit firewall-policy foundation has been created.

REDNET and SERVERNET policies will be configured before Kali Linux and Ubuntu Server are fully migrated behind the firewall.

---

## Security Disclaimer

This firewall and cybersecurity homelab are used exclusively for authorized cybersecurity education, experimentation, defensive security, and ethical hacking.

All scanning, enumeration, exploitation, monitoring, firewall testing, and attack simulations are performed only against systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, accounts, or organizations are targeted.
