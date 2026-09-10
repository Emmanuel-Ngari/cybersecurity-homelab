# Cybersecurity Homelab

A hands-on offensive and defensive cybersecurity laboratory built with Oracle VirtualBox, OPNsense, Kali Linux, Windows 11 Enterprise, and Ubuntu Server.

The environment is designed to simulate a segmented enterprise-style network where offensive activity can be generated, controlled, monitored, detected, investigated, and documented.

**Learning Methodology**

Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat

---

# 1. Project Overview

This repository documents the design, deployment, configuration, troubleshooting, testing, and continuous development of my cybersecurity homelab.

The project originally started as a simple three-VM environment connected through a shared internal network.

It has since evolved into a segmented architecture using OPNsense as the central:

- Firewall
- Router
- NAT gateway
- DNS service
- NTP service
- Network security gateway
- Traffic policy enforcement point
- Security logging platform

The environment supports both:

- Offensive Security
- Defensive Security

The objective is to develop practical cybersecurity skills through building, testing, troubleshooting, attacking, defending, monitoring, and documenting real systems.

---

# 2. Current Architecture

The current laboratory consists of four core virtual machines divided across three internal security zones.

<pre>
                           INTERNET
                              |
                              v
                       VirtualBox NAT
                         10.0.2.0/24
                              |
                              v
                    +-------------------+
                    |     LAB-FW-01     |
                    |     OPNsense      |
                    |                   |
                    | Firewall          |
                    | Router            |
                    | NAT Gateway       |
                    | DNS / NTP         |
                    | Security Logging  |
                    +---------+---------+
                              |
             +----------------+----------------+
             |                |                |
             v                v                v

        +----------+      +-----------+     +------------+
        | RED-NET  |      | CORP-NET  |     | SERVER-NET |
        |10.10.10/24|     |10.10.20/24|     |10.10.30/24|
        +----+-----+      +-----+-----+     +------+-----+
             |                  |                  |
             v                  v                  v

       LAB-KALI-01         LAB-WIN-01       LAB-UBUNTU-01
       Kali Linux          Windows 11        Ubuntu Server
       10.10.10.10         10.10.20.20       10.10.30.30

       Offensive           Management /      Protected
       Security            Blue Team          Server
</pre>

### OPNsense Interfaces

| Interface | Zone | Address | Purpose |
|---|---|---|---|
| `em0` | WAN | `10.0.2.15/24` | Internet uplink |
| `em1` | CORPNET | `10.10.20.1/24` | Management / Endpoint |
| `em2` | REDNET | `10.10.10.1/24` | Offensive Security |
| `em3` | SERVERNET | `10.10.30.1/24` | Protected Servers |

All communication between internal security zones is routed through OPNsense.

The internal VMs no longer have direct VirtualBox NAT connectivity.

---

# 3. Core Virtual Machines

| System | Operating System | Security Role | Network |
|---|---|---|---|
| `LAB-FW-01` | OPNsense 26.7 | Firewall / Router / Security Gateway | All Zones |
| `LAB-KALI-01` | Kali Linux Rolling | Offensive Security Workstation | REDNET |
| `LAB-WIN-01` | Windows 11 Enterprise | Management / Defensive Endpoint | CORPNET |
| `LAB-UBUNTU-01` | Ubuntu Server | Protected Linux Server | SERVERNET |

---

# 4. Security Zones

The laboratory currently contains three internal security zones.

## REDNET

**Network:** `10.10.10.0/24`

**Gateway:** `10.10.10.1`

**Primary System:** `LAB-KALI-01`

**Host Address:** `10.10.10.10/24`

**Purpose:**

- Ethical hacking
- Penetration testing
- Network reconnaissance
- Port scanning
- Service enumeration
- Vulnerability assessment
- Controlled exploitation
- Red Team exercises

REDNET is treated as an untrusted internal network.

---

## CORPNET

**Network:** `10.10.20.0/24`

**Gateway:** `10.10.20.1`

**Primary System:** `LAB-WIN-01`

**Host Address:** `10.10.20.20/24`

**Purpose:**

- OPNsense administration
- Windows administration
- PowerShell
- Endpoint security
- Blue Team exercises
- Server administration
- Security monitoring

CORPNET is currently the trusted management network.

---

## SERVERNET

**Network:** `10.10.30.0/24`

**Gateway:** `10.10.30.1`

**Primary System:** `LAB-UBUNTU-01`

**Host Address:** `10.10.30.30/24`

**Purpose:**

- Linux administration
- SSH services
- Server hardening
- Security monitoring
- Controlled attack targets
- Future web applications
- Future internal services

SERVERNET is treated as a protected server environment.

---

# 5. OPNsense Firewall

OPNsense is the central security gateway for the homelab.

Current interfaces:

| Interface | Zone | Address |
|---|---|---|
| `em0` | WAN | `10.0.2.15/24` |
| `em1` | CORPNET | `10.10.20.1/24` |
| `em2` | REDNET | `10.10.10.1/24` |
| `em3` | SERVERNET | `10.10.30.1/24` |

WAN receives Internet connectivity through VirtualBox NAT.

The internal systems no longer connect directly to VirtualBox NAT.

---

# 6. Mandatory Firewall Routing

All internal VM Internet traffic now passes through OPNsense.

### Kali

`10.10.10.10 → 10.10.10.1 → OPNsense → WAN → Internet`

### Windows

`10.10.20.20 → 10.10.20.1 → OPNsense → WAN → Internet`

### Ubuntu

`10.10.30.30 → 10.10.30.1 → OPNsense → WAN → Internet`

Direct NAT bypasses on Kali, Windows, and Ubuntu are disabled.

---

# 7. Outbound NAT

OPNsense uses:

`Automatic Source NAT rule generation`

The firewall performs outbound NAT for:

- REDNET
- CORPNET
- SERVERNET

Outbound NAT was validated before the original direct VM NAT adapters were disabled.

Internet connectivity remained operational after final cutover.

---

# 8. DNS Architecture

Each security zone uses its local OPNsense interface as DNS.

| System | DNS Server |
|---|---|
| Kali Linux | `10.10.10.1` |
| Windows 11 | `10.10.20.1` |
| Ubuntu Server | `10.10.30.1` |

DNS functionality has been validated from all three systems.

---

# 9. NTP Architecture

Internal security zones are allowed to use approved NTP services through OPNsense.

**Protocol:** UDP

**Port:** `123`

Accurate time synchronization is important for:

- Security logs
- Authentication
- SIEM correlation
- Incident timelines
- Detection engineering
- Digital forensics

---

# 10. Firewall Security Philosophy

The firewall configuration follows:

- Least privilege
- Default deny
- Explicit access
- Network segmentation
- Controlled administration
- Controlled offensive testing
- Stateful filtering
- Security logging

The goal is not to allow every system to communicate freely.

The goal is to control:

- Who may communicate
- Which destination may be reached
- Which services may be used
- Which connections should be denied
- Which events should be logged

---

# 11. Firewall Aliases

Aliases are used to improve firewall rule readability and administration.

### Host Aliases

`ADMIN_WORKSTATION`

`10.10.20.20`

`KALI_ATTACKER`

`10.10.10.10`

`UBUNTU_SERVER`

`10.10.30.30`

### Network Aliases

`LAB_INTERNAL_NETS`

- `10.10.10.0/24`
- `10.10.20.0/24`
- `10.10.30.0/24`

`PRIVATE_NETS`

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

### Service Aliases

`SERVER_ADMIN_PORTS`

- `22`
- `443`

`DNS_PORT`

- `53`

`NTP_PORT`

- `123`

---

# 12. CORPNET Firewall Policy

CORPNET currently permits:

- Admin workstation → OPNsense HTTPS
- CORPNET → OPNsense DNS
- CORPNET → OPNsense NTP
- Admin workstation → Ubuntu SSH
- Admin workstation → Ubuntu HTTPS
- CORPNET → Internet

CORPNET blocks:

- Unauthorized OPNsense services
- REDNET
- Unauthorized private networks
- Unapproved SERVERNET traffic

The original broad OPNsense LAN allow rules have been disabled.

---

# 13. REDNET Firewall Policy

REDNET permits:

- Kali → OPNsense DNS
- Kali → OPNsense NTP
- Kali → REDNET gateway ICMP
- Kali → Ubuntu for authorized security testing
- REDNET → Internet

REDNET blocks:

- OPNsense management access
- CORPNET
- Unauthorized private networks

This gives Kali a controlled attack environment without granting unrestricted access to trusted systems.

---

# 14. SERVERNET Firewall Policy

SERVERNET permits:

- Ubuntu → OPNsense DNS
- Ubuntu → OPNsense NTP
- Ubuntu → SERVERNET gateway ICMP
- SERVERNET → Internet

SERVERNET blocks new unsolicited connections toward:

- CORPNET
- REDNET
- OPNsense management services
- Unauthorized private networks

This reduces lateral-movement opportunities if a server becomes compromised.

---

# 15. Controlled Offensive Path

The primary authorized offensive-security path is:

<pre>
LAB-KALI-01
10.10.10.10
     |
     v
   REDNET
     |
     v
  OPNsense
     |
     v
 SERVERNET
     |
     v
LAB-UBUNTU-01
10.10.30.30
</pre>

This path has been successfully validated.

Nmap testing against Ubuntu confirmed:

`22/tcp open ssh`

Future exercises will include:

- Reconnaissance
- Enumeration
- Vulnerability assessment
- SSH testing
- Web application testing
- Controlled exploitation
- Privilege escalation
- Post-exploitation analysis

---

# 16. Controlled Administrative Path

The trusted server-management path is:

<pre>
LAB-WIN-01
10.10.20.20
     |
     v
  CORPNET
     |
     v
  OPNsense
     |
     v
 SERVERNET
     |
     v
LAB-UBUNTU-01
10.10.30.30
</pre>

Approved services:

- SSH `TCP/22`
- HTTPS `TCP/443`

PowerShell testing confirmed:

`TcpTestSucceeded : True`

for Ubuntu SSH.

---

# 17. Firewall Management

OPNsense is managed from Windows on CORPNET.

**Management Address:** `https://10.10.20.1`

**Approved Source:** `10.10.20.20`

**Approved Protocol:** HTTPS

**Port:** `443`

PowerShell validation:

`Test-NetConnection 10.10.20.1 -Port 443`

Result:

`TcpTestSucceeded : True`

Kali on REDNET cannot access the OPNsense Web GUI.

---

# 18. Validated Security Matrix

| Source | Destination | Service | Result |
|---|---|---|---|
| Windows | OPNsense | HTTPS | ALLOW |
| Windows | OPNsense | DNS | ALLOW |
| Windows | Internet | Internet | ALLOW |
| Windows | Ubuntu | SSH | ALLOW |
| Windows | Kali | General | BLOCK |
| Kali | OPNsense | DNS | ALLOW |
| Kali | OPNsense | ICMP | LIMITED ALLOW |
| Kali | OPNsense | HTTPS Management | BLOCK |
| Kali | Internet | Internet | ALLOW |
| Kali | Windows | General | BLOCK |
| Kali | Ubuntu | Authorized Testing | ALLOW |
| Ubuntu | OPNsense | DNS | ALLOW |
| Ubuntu | OPNsense | ICMP | LIMITED ALLOW |
| Ubuntu | Internet | Internet | ALLOW |
| Ubuntu | Windows | New Connections | BLOCK |
| Ubuntu | Kali | New Connections | BLOCK |

---

# 19. Network Validation

The laboratory was validated using multiple operating-system and security tools.

### Linux

- `ip addr`
- `ip route`
- `ping`
- `nslookup`
- `curl`
- `ss`
- `nmcli`
- `nmap`

### Windows PowerShell

- `Get-NetAdapter`
- `Get-NetIPAddress`
- `Get-NetIPConfiguration`
- `Get-NetRoute`
- `Test-NetConnection`
- `Resolve-DnsName`
- `ping`
- `curl.exe`

### OPNsense

- Interface Overview
- Firewall Rules
- Firewall Live View
- Outbound NAT
- Routing
- Interface Diagnostics
- Configuration Backups

---

# 20. Troubleshooting Experience

Real networking and firewall problems were encountered and resolved during the build.

These included:

- Windows APIPA addressing
- Incorrect Windows LAB-NET configuration
- Windows Firewall and ICMP behavior
- Kali-to-Windows connectivity problems
- Ubuntu missing static IPv4 configuration
- Netplan YAML indentation errors
- Persistent Ubuntu addressing
- Incorrect OPNsense interface assignments
- Firewall rule action configured incorrectly
- Firewall rule ordering
- Windows route selection
- Kali interface renumbering after NAT removal
- NetworkManager profile rebinding
- Ubuntu interface persistence
- DNS troubleshooting
- Outbound NAT validation
- Stateful firewall testing

These troubleshooting exercises are documented as part of the portfolio.

---

# 21. Repository Structure

The repository is organized by technical domain so each part of the homelab can be documented independently.

<pre>
cybersecurity-homelab/
│
├── README.md
│
├── architecture/
│   ├── lab-architecture.md
│   ├── virtualbox-networking.md
│   ├── network-diagram.md
│   └── network-diagram.png
│
├── virtual-machines/
│   ├── kali.md
│   ├── windows.md
│   └── ubuntu.md
│
├── networking/
│   ├── addressing.md
│   ├── connectivity-tests.md
│   └── troubleshooting.md
│
├── firewall/
│   ├── opnsense-firewall.md
│   ├── security-zones.md
│   ├── policy-matrix.md
│   ├── firewall-rules.md
│   └── validation-testing.md
│
├── linux/
│   └── ubuntu-server.md
│
├── windows/
│   └── windows-security.md
│
├── offensive-security/
│
├── blue-team/
│
├── incident-response/
│
├── screenshots/
│
└── CHANGELOG.md
</pre>

### Documentation Areas

**architecture/**

Documents the overall topology and network architecture.

**virtual-machines/**

Documents the configuration and purpose of each VM.

**networking/**

Contains addressing, routing, connectivity, and troubleshooting documentation.

**firewall/**

Contains OPNsense configuration, security zones, policies, firewall rules, and validation testing.

**offensive-security/**

Will contain penetration-testing and Red Team exercises.

**blue-team/**

Will contain SOC, monitoring, detection, and defensive-security labs.

**incident-response/**

Will contain investigation and incident-response exercises.

**screenshots/**

Will contain sanitized evidence from completed laboratory exercises.

---

# 22. Snapshot and Recovery Strategy

VirtualBox snapshots are taken before significant configuration changes.

The standard workflow is:

Snapshot → Change → Test → Verify → Backup → Document → Continue

Snapshots were created:

- Before firewall deployment changes
- Before security-zone migration
- Before final NAT removal
- After successful host cutovers
- After explicit firewall-policy validation

OPNsense configuration backups are also exported after important firewall milestones.

Firewall XML backup files are stored privately and are not committed to the public repository.

---

# 23. Original LAB-NET

The original flat laboratory network was:

`192.168.56.0/24`

Original addresses:

| System | Original Address |
|---|---|
| Kali | `192.168.56.10` |
| Windows | `192.168.56.20` |
| Ubuntu | `192.168.56.30` |

LAB-NET was useful for learning:

- Static IP addressing
- Routing
- Windows networking
- Linux networking
- Firewall troubleshooting
- Connectivity testing

It has now been retired from the active architecture.

---

# 24. Architecture Evolution

The project has evolved through several major stages.

<pre>
Basic Virtual Machines
        |
        v
Shared LAB-NET
192.168.56.0/24
        |
        v
Networking & Troubleshooting
        |
        v
OPNsense Deployment
        |
        v
Security-Zone Segmentation
        |
        +---- REDNET
        |
        +---- CORPNET
        |
        +---- SERVERNET
        |
        v
Explicit Firewall Policies
        |
        v
Outbound NAT Through OPNsense
        |
        v
Direct NAT Bypasses Removed
        |
        v
Validated Offensive Path
        |
        v
Validated Defensive Administration
        |
        v
Professional Segmented Homelab
</pre>

The next major stage introduces centralized monitoring and detection.

---

# 25. Planned SOC Architecture

The next major defensive-security phase will introduce centralized security monitoring.

**Proposed Zone:** `SOC-NET`

**Proposed Network:** `10.10.40.0/24`

Potential systems include:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Log collectors
- Security-monitoring infrastructure
- Detection-engineering tools

Potential telemetry sources include:

- Windows
- Ubuntu
- OPNsense
- Suricata
- Future Active Directory systems

---

# 26. Planned Endpoint Telemetry

Windows telemetry will eventually include:

- Windows Security Event Log
- Sysmon
- PowerShell logging
- Microsoft Defender events
- Authentication events
- Process execution
- Network connections

Ubuntu telemetry will eventually include:

- Authentication logs
- SSH logs
- systemd journal
- Linux audit logs
- Service logs
- Network activity

---

# 27. Planned IDS/IPS

Suricata is planned for network intrusion detection and prevention.

### Initial Stage

`IDS Mode`

Purpose:

- Observe suspicious traffic
- Generate alerts
- Learn signatures
- Tune detections
- Reduce false positives

### Later Stage

`IPS Mode`

Target workflow:

<pre>
Kali Attack
    |
    v
  REDNET
    |
    v
 OPNsense
    |
    v
 Suricata
    |
    v
   Alert
    |
    v
 Wazuh SIEM
    |
    v
Blue Team Investigation
</pre>

---

# 28. Planned SIEM

Wazuh is planned as the primary centralized security-monitoring platform.

Planned uses include:

- Log collection
- Endpoint monitoring
- Security alerts
- File integrity monitoring
- Threat detection
- Vulnerability information
- Security dashboards
- Incident investigation
- Detection engineering

The long-term objective is to correlate firewall, endpoint, authentication, network, and offensive-security activity.

---

# 29. Planned Active Directory Lab

A future enterprise identity environment will include Windows Server and Active Directory.

**Proposed Zone:** `AD-NET`

**Proposed Network:** `10.10.50.0/24`

Planned technologies include:

- Windows Server
- Active Directory Domain Services
- Domain Controller
- DNS
- Group Policy
- Kerberos
- NTLM
- Domain users
- Domain computers
- Authentication logging
- Identity security

---

# 30. Planned Active Directory Security Testing

The Active Directory environment will eventually support authorized exercises involving:

- Active Directory enumeration
- BloodHound
- Kerberos security
- Password security
- Privilege escalation
- Lateral movement
- Misconfiguration analysis
- Group Policy security
- Credential attacks
- Defensive detection
- Incident investigation

All testing will remain inside the controlled homelab.

---

# 31. Planned DMZ

A future DMZ will provide a dedicated environment for public-facing and intentionally vulnerable services.

**Proposed Zone:** `DMZ-NET`

**Proposed Network:** `10.10.60.0/24`

Potential systems include:

- OWASP Juice Shop
- DVWA
- Linux web servers
- APIs
- Intentionally vulnerable applications
- Public-facing test services

The DMZ will be isolated using explicit OPNsense firewall rules.

---

# 32. Offensive Security Roadmap

Future offensive-security development includes:

- Network reconnaissance
- Nmap
- Service enumeration
- Vulnerability scanning
- Web application testing
- Burp Suite
- Metasploit
- Linux exploitation
- Windows exploitation
- Privilege escalation
- Active Directory attacks
- BloodHound
- Impacket
- Credential attacks
- Lateral movement
- Pivoting
- Post-exploitation
- Red Team simulations

---

# 33. Defensive Security Roadmap

Future defensive-security development includes:

- Windows Event Viewer
- Sysmon
- PowerShell logging
- Microsoft Defender
- Linux auditd
- SSH monitoring
- OPNsense firewall logging
- Suricata
- Wazuh
- SIEM
- Detection engineering
- Threat hunting
- Alert triage
- Incident response
- Digital forensics
- Timeline reconstruction
- Network traffic analysis

---

# 34. Incident Response Roadmap

Future incident-response exercises will follow a structured lifecycle:

Preparation → Detection → Triage → Investigation → Containment → Eradication → Recovery → Lessons Learned

Potential technologies include:

- Wazuh
- Sysmon
- Windows Event Viewer
- PowerShell
- Linux logs
- Wireshark
- Velociraptor
- OPNsense
- Suricata

---

# 35. Target Professional Architecture

The long-term goal is to expand the current three-zone environment into a larger enterprise-style offensive and defensive cybersecurity lab.

<pre>
                              INTERNET
                                 |
                                 v
                         +---------------+
                         |   LAB-FW-01   |
                         |    OPNsense   |
                         |               |
                         | Firewall      |
                         | Router        |
                         | NAT           |
                         | IDS / IPS     |
                         +-------+-------+
                                 |
       +-------------+-----------+-----------+-------------+-------------+
       |             |                       |             |             |
       v             v                       v             v             v

   +--------+    +---------+            +----------+   +---------+   +--------+
   | REDNET |    | CORPNET |            |SERVERNET |   | SOC-NET |   | AD-NET |
   |10.10.10|    |10.10.20 |            |10.10.30  |   |10.10.40 |   |10.10.50|
   +---+----+    +----+----+            +----+-----+   +----+----+   +---+----+
       |              |                      |              |            |
       v              v                      v              v            v

     Kali          Windows 11             Ubuntu          Wazuh      Windows Server
   Red Team        Endpoint /            Servers          SIEM       Active Directory
   Attack Tools    Management            Services         SOC        Domain Services
       |              |                      |              |            |
       |              |                      |              |            |
       +--------------+----------------------+--------------+------------+
                                      |
                                      v
                          Centralized Security Telemetry
                                      |
                                      v
                         Detection / Investigation / IR

                                      |
                                      v
                                  DMZ-NET
                                10.10.60.0/24
                                      |
                                      v
                         Vulnerable Applications
                         Web Servers / APIs / Targets
</pre>

### Planned Security Zones

| Zone | Proposed Network | Primary Purpose |
|---|---|---|
| REDNET | `10.10.10.0/24` | Offensive Security |
| CORPNET | `10.10.20.0/24` | Endpoint / Management |
| SERVERNET | `10.10.30.0/24` | Protected Servers |
| SOC-NET | `10.10.40.0/24` | SIEM / Monitoring |
| AD-NET | `10.10.50.0/24` | Active Directory |
| DMZ-NET | `10.10.60.0/24` | Vulnerable / Public-Facing Services |

### Target Security Workflow

<pre>
Attack Generation
      |
      v
Firewall Transit
      |
      v
Network Inspection
      |
      v
Endpoint Telemetry
      |
      v
Centralized SIEM
      |
      v
Detection
      |
      v
Alert
      |
      v
Investigation
      |
      v
Incident Response
      |
      v
Remediation
      |
      v
Retesting
</pre>

The long-term objective is to use the same environment to practice both the attacker and defender perspectives.

---

# 36. Planned Professional Tools

## Network Security

- OPNsense
- Suricata
- Wireshark
- tcpdump
- Nmap

## SIEM / SOC

- Wazuh
- Security Onion
- Centralized logging
- Detection rules
- Security dashboards

## Windows Security

- Windows Event Viewer
- Sysmon
- PowerShell
- Microsoft Defender
- Windows Event Forwarding

## Linux Security

- auditd
- systemd journal
- SSH logging
- UFW
- Fail2ban

## Incident Response

- Velociraptor
- Wazuh
- Sysmon
- Wireshark
- Endpoint logs
- Firewall logs

## Offensive Security

- Kali Linux
- Nmap
- Burp Suite
- Metasploit
- BloodHound
- Impacket
- Netcat
- Gobuster
- Hydra
- Custom scripts

## Vulnerable Targets

- OWASP Juice Shop
- DVWA
- Linux services
- Windows systems
- Active Directory
- Intentionally vulnerable hosts

---

# 37. Learning Methodology

The project follows:

Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat

Every major configuration change should be:

- Understood
- Recoverable
- Tested
- Verified
- Documented

The objective is not simply to install cybersecurity tools.

Each technology should be understood from multiple perspectives:

- Architecture
- Configuration
- Networking
- Security
- Attack
- Detection
- Troubleshooting
- Monitoring
- Incident response
- Documentation

---

# 38. Current Project Status

| Component | Status |
|---|---|
| Kali Linux | 🟢 Operational |
| Windows 11 Enterprise | 🟢 Operational |
| Ubuntu Server | 🟢 Operational |
| OPNsense | 🟢 Operational |
| REDNET | 🟢 Operational |
| CORPNET | 🟢 Operational |
| SERVERNET | 🟢 Operational |
| Firewall Rules | 🟢 Operational |
| Network Segmentation | 🟢 Verified |
| Outbound NAT | 🟢 Verified |
| DNS | 🟢 Verified |
| Internet Connectivity | 🟢 Verified |
| Controlled Attack Path | 🟢 Verified |
| Controlled Management Path | 🟢 Verified |
| Direct NAT Bypasses | 🔴 Disabled |
| Default LAN Allow Rules | 🔴 Disabled |
| Legacy LAB-NET | 🔴 Retired |
| Suricata IDS/IPS | 🟡 Planned |
| Wazuh SIEM | 🟡 Planned |
| SOC-NET | 🟡 Planned |
| Active Directory | 🟡 Planned |
| DMZ | 🟡 Planned |
| Incident Response Lab | 🟡 Planned |

---

# 39. Current Milestone

The homelab has successfully progressed through:

<pre>
Basic Virtual Machines
        |
        v
Flat Internal Network
        |
        v
Network Troubleshooting
        |
        v
Dedicated OPNsense Firewall
        |
        v
Security-Zone Segmentation
        |
        v
Least-Privilege Firewall Policies
        |
        v
Mandatory Firewall Routing
        |
        v
Controlled Offensive Path
        |
        v
Controlled Administrative Path
        |
        v
Validated Network Segmentation
</pre>

The networking and firewall foundation is now operational.

---

# 40. Next Development Phase

The next major phase will focus on defensive monitoring and security visibility.

<pre>
Centralized Logging
        |
        v
Endpoint Telemetry
        |
        v
Network Detection
        |
        v
SIEM
        |
        v
Alerting
        |
        v
Threat Hunting
        |
        v
Incident Response
</pre>

Planned next milestones include:

- SOC-NET design
- Wazuh deployment
- Windows Sysmon
- Windows security logging
- Ubuntu security logging
- OPNsense log forwarding
- Suricata IDS
- SIEM dashboards
- Detection engineering
- Attack-and-detection exercises
- Incident-response labs

---

# 41. Security Disclaimer

This cybersecurity homelab is used exclusively for authorized cybersecurity education, ethical hacking, defensive-security training, penetration-testing practice, and incident-response exercises.

All network scanning, enumeration, exploitation, attack simulation, and security testing are performed only against systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, applications, services, accounts, or organizations are targeted.

---

# Author

**Emmanuel Ngari**

Cybersecurity | Network Security | Blue Team | Red Team

**GitHub:** `Emmanuel-Ngari`
