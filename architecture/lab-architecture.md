# Cybersecurity Homelab Architecture

## 1. Overview

This document describes the current architecture of my professional cybersecurity homelab.

The environment was originally built as a small flat VirtualBox network containing Kali Linux, Windows 11, and Ubuntu Server.

The lab has since evolved into a segmented security environment using OPNsense as the central:

- Firewall
- Router
- Security gateway
- NAT gateway
- DNS service
- NTP service
- Network-policy enforcement point
- Security logging platform

The architecture is designed to support both offensive and defensive cybersecurity training.

---

## 2. Lab Objectives

The homelab is designed to provide practical experience with:

- Networking
- Linux administration
- Windows administration
- Firewall configuration
- Network segmentation
- Routing
- NAT
- DNS
- Security monitoring
- Ethical hacking
- Penetration testing
- Vulnerability assessment
- Endpoint security
- Blue Team operations
- SOC operations
- Incident response
- Threat detection
- Detection engineering
- SIEM
- IDS/IPS
- Active Directory security
- Attack simulation

The long-term objective is to create an environment where attacks can be generated, detected, investigated, contained, and documented.

---

## 3. Virtualization Platform

The laboratory is hosted using:

`Oracle VirtualBox`

The host system runs Windows and provides the virtualization platform for all laboratory systems.

VirtualBox provides isolated virtual networks that allow multiple security zones to exist without exposing laboratory traffic directly to the physical network.

---

## 4. Current Core Virtual Machines

The current architecture contains four primary virtual machines.

| Virtual Machine | Operating System | Role |
|---|---|---|
| `LAB-FW-01` | OPNsense | Firewall / Router / Security Gateway |
| `LAB-KALI-01` | Kali Linux | Offensive Security Workstation |
| `LAB-WIN-01` | Windows 11 Enterprise | Management / Defensive Endpoint |
| `LAB-UBUNTU-01` | Ubuntu Server | Protected Linux Server |

These systems form the foundation of the offensive and defensive laboratory.

---

## 5. Architecture Evolution

The original laboratory used a flat internal network:

`192.168.56.0/24`

The original addressing included:

Kali:

`192.168.56.10`

Windows:

`192.168.56.20`

Ubuntu:

`192.168.56.30`

Each system also had a direct VirtualBox NAT adapter for Internet access.

Although this architecture was useful for learning basic networking, it allowed systems to communicate without a centralized firewall.

The laboratory was therefore redesigned using security zones.

---

## 6. Current Segmented Architecture

The current architecture is:

Internet  
↓  
VirtualBox NAT  
↓  
OPNsense WAN  
↓  
Firewall / Routing / NAT / Logging  
↓  
Security Zones

The internal security zones are:

`REDNET`

`CORPNET`

`SERVERNET`

All inter-zone communication must pass through OPNsense.

---

## 7. High-Level Network Diagram

The current logical architecture is:

Internet
|
v
VirtualBox NAT
|
v
+--------------------------------------------------+
| LAB-FW-01                                        |
| OPNsense Firewall                                |
|                                                  |
| WAN:       em0 - 10.0.2.15/24                   |
| REDNET:    em2 - 10.10.10.1/24                  |
| CORPNET:   em1 - 10.10.20.1/24                  |
| SERVERNET: em3 - 10.10.30.1/24                  |
+--------------------------------------------------+
        |                  |                 |
        |                  |                 |
        v                  v                 v

     REDNET              CORPNET          SERVERNET
  10.10.10.0/24       10.10.20.0/24    10.10.30.0/24
        |                  |                 |
        |                  |                 |
        v                  v                 v

   LAB-KALI-01         LAB-WIN-01       LAB-UBUNTU-01
   10.10.10.10         10.10.20.20      10.10.30.30
 Offensive Security      Management       Linux Server
        |                  |                 |
        +------------------+-----------------+
                           |
                     OPNsense Policy
                           |
                      Controlled Access

---

## 8. WAN Network

The OPNsense WAN interface connects to VirtualBox NAT.

Interface:

`em0`

IPv4 Address:

`10.0.2.15/24`

Default Gateway:

`10.0.2.2`

Configuration:

`DHCP`

Purpose:

- Internet connectivity
- Outbound NAT
- Software updates
- Package downloads
- External training resources

Internal laboratory systems do not have direct VirtualBox NAT connectivity anymore.

Their Internet traffic must pass through OPNsense.

---

## 9. REDNET

REDNET is the offensive-security network.

Network:

`10.10.10.0/24`

OPNsense Gateway:

`10.10.10.1`

Primary System:

`LAB-KALI-01`

Kali Address:

`10.10.10.10/24`

Default Gateway:

`10.10.10.1`

DNS:

`10.10.10.1`

Purpose:

- Penetration testing
- Network scanning
- Enumeration
- Vulnerability assessment
- Ethical hacking
- Controlled exploitation
- Red Team exercises
- Security-tool experimentation

REDNET is treated as an untrusted internal network.

---

## 10. CORPNET

CORPNET is the trusted management and endpoint network.

Network:

`10.10.20.0/24`

OPNsense Gateway:

`10.10.20.1`

Primary System:

`LAB-WIN-01`

Windows Address:

`10.10.20.20/24`

Default Gateway:

`10.10.20.1`

DNS:

`10.10.20.1`

Purpose:

- Firewall administration
- Windows security
- PowerShell
- Endpoint monitoring
- System administration
- Defensive-security exercises
- Future domain-joined workstation activity

Windows is currently the trusted OPNsense management workstation.

---

## 11. SERVERNET

SERVERNET is the protected server network.

Network:

`10.10.30.0/24`

OPNsense Gateway:

`10.10.30.1`

Primary System:

`LAB-UBUNTU-01`

Ubuntu Address:

`10.10.30.30/24`

Default Gateway:

`10.10.30.1`

DNS:

`10.10.30.1`

Purpose:

- Linux administration
- SSH
- Server hardening
- Security monitoring
- Vulnerability testing
- Controlled attack targets
- Future web applications
- Future internal services

SERVERNET is isolated from unnecessary access to other laboratory zones.

---

## 12. OPNsense Interface Mapping

The firewall uses four VirtualBox adapters.

| VirtualBox Adapter | OPNsense Interface | Network | Purpose |
|---|---|---|---|
| Adapter 1 | `em0` | NAT | WAN |
| Adapter 2 | `em1` | `CORP-NET` | Management / Endpoint |
| Adapter 3 | `em2` | `RED-NET` | Offensive Security |
| Adapter 4 | `em3` | `SERVER-NET` | Protected Servers |

The interface assignments were verified using MAC addresses before security-zone deployment.

---

## 13. OPNsense Interface Addresses

The firewall addressing is:

| Interface | Zone | Address |
|---|---|---|
| `em0` | WAN | `10.0.2.15/24` |
| `em1` | CORPNET | `10.10.20.1/24` |
| `em2` | REDNET | `10.10.10.1/24` |
| `em3` | SERVERNET | `10.10.30.1/24` |

The firewall has direct Layer 3 connectivity to each internal security zone.

---

## 14. Mandatory Gateway Architecture

The internal systems no longer use independent NAT adapters.

Final paths are:

Kali:

`10.10.10.10 → 10.10.10.1 → OPNsense → WAN → Internet`

Windows:

`10.10.20.20 → 10.10.20.1 → OPNsense → WAN → Internet`

Ubuntu:

`10.10.30.30 → 10.10.30.1 → OPNsense → WAN → Internet`

This makes OPNsense the mandatory security gateway for the laboratory.

---

## 15. Direct NAT Removal

Direct NAT access was removed from the internal virtual machines after OPNsense outbound NAT was validated.

Current state:

Kali direct NAT:

`Disabled`

Windows direct NAT:

`Disabled`

Ubuntu direct NAT:

`Disabled`

The original Windows LAB-NET adapter is also disabled.

This prevents laboratory systems from bypassing firewall controls.

---

## 16. Outbound NAT

OPNsense uses:

`Automatic Source NAT rule generation`

The automatic NAT configuration includes:

- REDNET
- CORPNET
- SERVERNET

Traffic leaving the laboratory is translated to the OPNsense WAN address.

This allows all protected systems to reach the Internet through the firewall.

---

## 17. DNS Architecture

OPNsense provides DNS service to the laboratory systems.

Kali DNS:

`10.10.10.1`

Windows DNS:

`10.10.20.1`

Ubuntu DNS:

`10.10.30.1`

DNS access is controlled through explicit firewall rules.

This allows DNS traffic to remain visible and centrally managed.

---

## 18. NTP Architecture

Internal zones are permitted to use approved NTP services on OPNsense.

NTP traffic uses:

`UDP/123`

Time synchronization is important for:

- Security logs
- Event correlation
- Incident timelines
- SIEM
- Authentication
- Digital forensics

Accurate time becomes increasingly important as centralized monitoring is added.

---

## 19. Security Trust Model

The current zones use different trust levels.

| Zone | Trust Level | Role |
|---|---|---|
| WAN | Untrusted | External network |
| REDNET | Untrusted Internal | Offensive Security |
| CORPNET | Trusted Internal | Management / Endpoints |
| SERVERNET | Protected Internal | Servers |

Trust does not provide unrestricted access.

All zone communication is controlled by firewall policy.

---

## 20. CORPNET Policy

CORPNET currently permits:

- Windows → OPNsense HTTPS
- CORPNET → OPNsense DNS
- CORPNET → OPNsense NTP
- Windows → SERVERNET SSH
- Windows → SERVERNET HTTPS
- CORPNET → Internet

CORPNET is restricted from:

- Unauthorized OPNsense services
- REDNET
- Unauthorized private networks
- Unapproved SERVERNET services

---

## 21. REDNET Policy

REDNET currently permits:

- Kali → OPNsense DNS
- Kali → OPNsense NTP
- Kali → REDNET gateway ICMP
- Kali → Ubuntu for controlled security testing
- REDNET → Internet

REDNET is restricted from:

- OPNsense Web GUI
- Firewall management services
- CORPNET
- Unauthorized private networks

This provides a controlled attacker environment.

---

## 22. SERVERNET Policy

SERVERNET currently permits:

- Ubuntu → OPNsense DNS
- Ubuntu → OPNsense NTP
- Ubuntu → SERVERNET gateway ICMP
- SERVERNET → Internet

SERVERNET is restricted from initiating connections toward:

- CORPNET
- REDNET
- OPNsense management services
- Unauthorized private networks

This reduces lateral-movement opportunities if a server becomes compromised.

---

## 23. Controlled Offensive Path

The primary authorized attack path is:

`Kali 10.10.10.10`

↓

`REDNET`

↓

`OPNsense`

↓

`SERVERNET`

↓

`Ubuntu 10.10.30.30`

This path is intentionally permitted for controlled offensive-security training.

Validated activities include:

- ICMP connectivity
- Nmap enumeration
- SSH service discovery

Nmap confirmed:

`22/tcp open ssh`

on the Ubuntu Server.

---

## 24. Controlled Administrative Path

The approved server-management path is:

`Windows 10.10.20.20`

↓

`CORPNET`

↓

`OPNsense`

↓

`SERVERNET`

↓

`Ubuntu 10.10.30.30:22`

Windows PowerShell validation confirmed:

`TcpTestSucceeded : True`

for TCP port 22.

This allows trusted server administration while maintaining segmentation.

---

## 25. Firewall Management

OPNsense management is restricted to the trusted Windows management workstation.

Management address:

`https://10.10.20.1`

Approved host:

`10.10.20.20`

Approved protocol:

`TCP/443`

Kali cannot access the OPNsense Web GUI.

Testing from REDNET produced:

`Connection timed out`

This confirmed that firewall management isolation is functioning correctly.

---

## 26. Default LAN Rules

During migration, OPNsense's default LAN allow rules were temporarily retained as a rollback mechanism.

After the custom firewall policy was validated, the following rules were disabled:

`Default allow LAN to any rule`

`Default allow LAN IPv6 to any rule`

The laboratory now operates using explicit custom rules.

---

## 27. Firewall Philosophy

The firewall architecture follows:

### Least Privilege

Only required traffic is allowed.

### Default Deny

Traffic without an explicit legitimate purpose is denied.

### Network Segmentation

Systems with different security roles are placed on different networks.

### Controlled Administration

Administrative access originates from trusted systems.

### Controlled Offensive Testing

Attack systems can only reach intentionally authorized targets.

### Stateful Filtering

Return traffic for legitimate established connections is handled through firewall state.

### Logging

Important allow and block rules are logged for later investigation.

---

## 28. Validated Connectivity

The following connectivity has been verified.

| Source | Destination | Result |
|---|---|---|
| Windows | OPNsense HTTPS | ALLOW |
| Windows | OPNsense DNS | ALLOW |
| Windows | Internet | ALLOW |
| Windows | Ubuntu SSH | ALLOW |
| Windows | Kali | BLOCK |
| Kali | OPNsense DNS | ALLOW |
| Kali | Internet | ALLOW |
| Kali | OPNsense HTTPS | BLOCK |
| Kali | Windows | BLOCK |
| Kali | Ubuntu | ALLOW |
| Ubuntu | OPNsense DNS | ALLOW |
| Ubuntu | Internet | ALLOW |
| Ubuntu | Windows | BLOCK |
| Ubuntu | Kali | BLOCK |

---

## 29. Firewall Recovery Strategy

Recovery mechanisms include:

- VirtualBox snapshots
- OPNsense configuration backups
- GitHub architecture documentation
- Network configuration documentation

Snapshots were taken:

- Before major migrations
- Before final NAT removal
- After successful cutovers
- After firewall-policy verification

OPNsense XML configuration backups are stored privately and are not committed to GitHub.

---

## 30. Legacy LAB-NET

The original laboratory network was:

`192.168.56.0/24`

It was used during the first phase of the project to develop networking and troubleshooting skills.

The original LAB-NET has been superseded by:

- REDNET
- CORPNET
- SERVERNET

The old architecture is retained only as historical documentation of the lab's development.

---

## 31. Current Infrastructure Status

| Component | Status |
|---|---|
| OPNsense | 🟢 Operational |
| WAN | 🟢 Operational |
| CORPNET | 🟢 Operational |
| REDNET | 🟢 Operational |
| SERVERNET | 🟢 Operational |
| Windows | 🟢 Operational |
| Kali Linux | 🟢 Operational |
| Ubuntu Server | 🟢 Operational |
| Outbound NAT | 🟢 Verified |
| DNS | 🟢 Verified |
| Internet Access | 🟢 Verified |
| Inter-Zone Segmentation | 🟢 Verified |
| Firewall Management Isolation | 🟢 Verified |
| Controlled Offensive Path | 🟢 Verified |
| Direct VM NAT Bypass | 🔴 Disabled |
| Default LAN Allow Rules | 🔴 Disabled |
| Original LAB-NET | 🔴 Retired |

---

## 32. Planned SOC Architecture

The next major defensive-security development will introduce centralized monitoring.

A future architecture may include:

`SOC-NET`

Possible network:

`10.10.40.0/24`

Potential systems:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Log collectors
- Security monitoring tools

Telemetry may be collected from:

- Windows
- Ubuntu
- OPNsense
- Suricata
- Authentication systems

---

## 33. Planned Active Directory Architecture

A future Active Directory environment may use:

`AD-NET`

Possible network:

`10.10.50.0/24`

Potential systems include:

- Windows Server
- Active Directory Domain Services
- Domain Controller
- DNS
- Group Policy
- Domain-joined Windows workstation

This will support:

- Identity security
- Kerberos
- Active Directory attacks
- Group Policy
- Windows logging
- Detection engineering
- Enterprise incident-response exercises

---

## 34. Planned DMZ Architecture

A future DMZ may use:

`10.10.60.0/24`

Possible systems include:

- Vulnerable web applications
- Linux web servers
- Public-facing services
- API services
- Security-testing targets

The DMZ will use strict firewall policy between:

- WAN
- CORPNET
- REDNET
- SERVERNET
- SOC-NET

---

## 35. Planned IDS/IPS

OPNsense will eventually integrate Suricata.

Initial deployment will likely use:

`IDS Mode`

The goal will be to detect activity without immediately blocking traffic.

After tuning, selected rules may be tested using:

`IPS Mode`

Example workflow:

Kali  
↓  
Attack Traffic  
↓  
OPNsense  
↓  
Suricata  
↓  
Alert  
↓  
SIEM  
↓  
Blue Team Investigation

---

## 36. Planned SIEM Integration

Wazuh is planned as the primary SIEM/XDR platform for the next defensive-security phase.

Potential telemetry sources include:

Windows:

- Security Event Logs
- Authentication Events
- PowerShell Logs
- Defender Events
- Sysmon

Ubuntu:

- Authentication Logs
- SSH Logs
- System Logs
- Application Logs

OPNsense:

- Firewall Logs
- Block Events
- Network Connections

Suricata:

- IDS Alerts
- Network Threat Events

This will allow attack activity to be correlated across multiple systems.

---

## 37. Future Offensive Security Development

The offensive environment will be expanded with:

- Vulnerable Linux targets
- Vulnerable web applications
- Active Directory attack labs
- Network service exploitation
- Privilege escalation
- Credential attacks
- Lateral movement
- Pivoting
- Post-exploitation
- Red Team simulations

All activities will remain inside authorized laboratory environments.

---

## 38. Future Defensive Security Development

Defensive capabilities will be expanded with:

- Wazuh
- Sysmon
- Windows Event Forwarding
- Suricata
- Firewall monitoring
- Endpoint telemetry
- Detection rules
- Threat hunting
- Incident response
- Security dashboards
- Alert triage
- Timeline reconstruction
- Digital forensics

The goal is to observe offensive activity from the defender's perspective.

---

## 39. Target Professional Architecture

The long-term target architecture is:

Internet
|
v
OPNsense Firewall
|
+----------------+----------------+----------------+----------------+----------------+
|                |                |                |                |
v                v                v                v                v
REDNET         CORPNET         SERVERNET        SOC-NET          AD-NET
|                |                |                |                |
Kali           Windows          Linux           SIEM          Windows Server
Attack          Endpoint        Services         Wazuh         Active Directory
Tools           Management      Targets          IDS/Logs      Domain Services
|
+---------------------------------------------------------------+
                                |
                               DMZ
                                |
                       Vulnerable Applications

All communication between zones will be controlled through firewall policy.

---

## 40. Learning Methodology

The homelab follows the methodology:

`Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat`

The environment is not intended merely to run cybersecurity tools.

Each component is designed to provide practical understanding of:

- Why the technology exists
- How it is configured
- How it fails
- How it is attacked
- How it is monitored
- How it is defended
- How incidents are investigated

---

## 41. Current Architecture Milestone

The project has successfully progressed from:

Flat VirtualBox Network

↓

Multi-VM Cybersecurity Lab

↓

Dedicated OPNsense Firewall

↓

Security-Zone Segmentation

↓

Explicit Firewall Policy

↓

Mandatory Firewall Routing

↓

Controlled Offensive and Defensive Paths

The current environment provides a strong foundation for advanced SOC, SIEM, IDS/IPS, Active Directory, penetration-testing, and incident-response exercises.

---

## Security Disclaimer

This cybersecurity homelab is used exclusively for authorized cybersecurity education, ethical hacking, defensive-security training, and incident-response exercises.

All offensive testing, network scanning, enumeration, exploitation, and attack simulation are performed only against systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, services, applications, accounts, or organizations are targeted.
