# Cybersecurity Homelab

A hands-on offensive and defensive cybersecurity laboratory built using Oracle VirtualBox, OPNsense, Kali Linux, Windows 11 Enterprise, and Ubuntu Server.

The environment is designed to simulate a segmented enterprise-style network where offensive-security activity can be generated, controlled, monitored, detected, investigated, and documented.

The lab follows the methodology:

Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat

---

# 1. Project Overview

This project documents the design, deployment, configuration, troubleshooting, testing, and continuous development of my cybersecurity homelab.

The lab originally started as a simple three-VM environment connected through a shared internal network.

It has since evolved into a segmented architecture using OPNsense as the central:

- Firewall
- Router
- NAT gateway
- DNS service
- NTP service
- Network security gateway
- Traffic-policy enforcement point
- Security logging platform

The environment supports both:

- Offensive Security
- Defensive Security

The goal is to build practical cybersecurity skills rather than only study theoretical concepts.

---

# 2. Current Architecture

The current laboratory architecture is:

Internet
|
v
VirtualBox NAT
|
v
+
| LAB-FW-01                                        |
| OPNsense Firewall                                |
|                                                  |
| WAN:       10.0.2.15/24                         |
| CORPNET:   10.10.20.1/24                        |
| REDNET:    10.10.10.1/24                        |
| SERVERNET: 10.10.30.1/24                        |
+
          |                 |                 |
          |                 |                 |
          v                 v                 v

       REDNET             CORPNET          SERVERNET
    10.10.10.0/24      10.10.20.0/24    10.10.30.0/24
          |                 |                 |
          |                 |                 |
          v                 v                 v

      Kali Linux        Windows 11       Ubuntu Server
     10.10.10.10       10.10.20.20      10.10.30.30
      Offensive          Management       Protected
       Security          / Blue Team       Server

All inter-zone communication passes through OPNsense.

---

# 3. Core Virtual Machines

| System | Operating System | Security Role | Network |
|---|---|---|---|
| LAB-FW-01 | OPNsense | Firewall / Router / Security Gateway | Multiple |
| Kali Linux | Kali Linux Rolling | Offensive Security | REDNET |
| Windows Endpoint | Windows 11 Enterprise | Management / Defensive Endpoint | CORPNET |
| LAB-UBUNTU-01 | Ubuntu Server | Protected Linux Server | SERVERNET |

---

# 4. Security Zones

The laboratory currently contains three internal security zones.

## REDNET

Network:

10.10.10.0/24

Gateway:

10.10.10.1

Primary system:

Kali Linux

Address:

10.10.10.10/24

Purpose:

- Ethical hacking
- Penetration testing
- Network reconnaissance
- Port scanning
- Enumeration
- Vulnerability assessment
- Controlled exploitation
- Red Team exercises

REDNET is treated as an untrusted internal network.

---

## CORPNET

Network:

10.10.20.0/24

Gateway:

10.10.20.1

Primary system:

Windows 11 Enterprise

Address:

10.10.20.20/24

Purpose:

- Firewall management
- Windows administration
- PowerShell
- Endpoint security
- Blue Team exercises
- Server administration
- Security monitoring

CORPNET is the current trusted management network.

---

## SERVERNET

Network:

10.10.30.0/24

Gateway:

10.10.30.1

Primary system:

Ubuntu Server

Address:

10.10.30.30/24

Purpose:

- Linux administration
- Server hardening
- SSH
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
| em0 | WAN | 10.0.2.15/24 |
| em1 | CORPNET | 10.10.20.1/24 |
| em2 | REDNET | 10.10.10.1/24 |
| em3 | SERVERNET | 10.10.30.1/24 |

WAN receives connectivity through VirtualBox NAT.

The internal systems no longer have their own direct VirtualBox NAT connectivity.

---

# 6. Mandatory Firewall Routing

All internal VM Internet traffic is routed through OPNsense.

Kali:

10.10.10.10
↓
10.10.10.1
↓
OPNsense
↓
WAN
↓
Internet

Windows:

10.10.20.20
↓
10.10.20.1
↓
OPNsense
↓
WAN
↓
Internet

Ubuntu:

10.10.30.30
↓
10.10.30.1
↓
OPNsense
↓
WAN
↓
Internet

Direct NAT bypasses on the internal systems are disabled.

---

# 7. Outbound NAT

OPNsense uses:

Automatic Source NAT rule generation

The firewall performs outbound address translation for:

- REDNET
- CORPNET
- SERVERNET

Outbound NAT was tested before the original direct VM NAT adapters were disabled.

Internet connectivity remained operational after final cutover.

---

# 8. DNS Architecture

Each security zone uses its local OPNsense interface as DNS.

Kali:

10.10.10.1

Windows:

10.10.20.1

Ubuntu:

10.10.30.1

DNS functionality has been validated from all three systems.

---

# 9. Firewall Security Philosophy

The firewall configuration follows:

- Least privilege
- Default deny
- Explicit access
- Network segmentation
- Controlled administration
- Controlled offensive testing
- Stateful filtering
- Security logging

The objective is not to allow every system to communicate.

The objective is to control:

- Who may communicate
- Which destination may be reached
- Which services may be used
- Which connections should be blocked
- Which events should be logged

---

# 10. Firewall Aliases

Aliases are used to make firewall policies easier to understand and maintain.

Current aliases include:

ADMIN_WORKSTATION

10.10.20.20

KALI_ATTACKER

10.10.10.10

UBUNTU_SERVER

10.10.30.30

LAB_INTERNAL_NETS

10.10.10.0/24
10.10.20.0/24
10.10.30.0/24

PRIVATE_NETS

10.0.0.0/8
172.16.0.0/12
192.168.0.0/16

SERVER_ADMIN_PORTS

22
443

DNS_PORT

53

NTP_PORT

123

---

# 11. CORPNET Firewall Policy

CORPNET currently permits:

- Management workstation → OPNsense HTTPS
- CORPNET → OPNsense DNS
- CORPNET → OPNsense NTP
- Management workstation → Ubuntu SSH
- Management workstation → Ubuntu HTTPS
- CORPNET → Internet

CORPNET blocks:

- Unauthorized OPNsense services
- REDNET
- Unauthorized private networks
- Unapproved SERVERNET traffic

The original broad OPNsense LAN allow rules have been disabled.

---

# 12. REDNET Firewall Policy

REDNET permits:

- Kali → OPNsense DNS
- Kali → OPNsense NTP
- Kali → REDNET gateway ICMP
- Kali → Ubuntu for authorized testing
- REDNET → Internet

REDNET blocks:

- OPNsense management access
- CORPNET
- Unauthorized private networks

This gives Kali a controlled attack environment without granting unrestricted access to trusted systems.

---

# 13. SERVERNET Firewall Policy

SERVERNET permits:

- Ubuntu → OPNsense DNS
- Ubuntu → OPNsense NTP
- Ubuntu → SERVERNET gateway ICMP
- SERVERNET → Internet

SERVERNET blocks new unsolicited connections toward:

- CORPNET
- REDNET
- OPNsense management
- Unauthorized private networks

This helps reduce lateral-movement opportunities if a server becomes compromised.

---

# 14. Controlled Offensive Path

The primary authorized offensive-security path is:

Kali
↓
REDNET
↓
OPNsense
↓
SERVERNET
↓
Ubuntu

Source:

10.10.10.10

Destination:

10.10.30.30

This path has been tested successfully.

Nmap testing against Ubuntu confirmed:

22/tcp open ssh

This environment will be used for future:

- Reconnaissance
- Enumeration
- Vulnerability assessment
- SSH testing
- Web application testing
- Controlled exploitation
- Privilege escalation
- Post-exploitation analysis

---

# 15. Controlled Administrative Path

The trusted server-management path is:

Windows
↓
CORPNET
↓
OPNsense
↓
SERVERNET
↓
Ubuntu

Source:

10.10.20.20

Destination:

10.10.30.30

Approved ports:

22/TCP

443/TCP

PowerShell testing confirmed:

TcpTestSucceeded : True

for Ubuntu SSH.

---

# 16. Firewall Management

OPNsense is managed from Windows on CORPNET.

Management address:

https://10.10.20.1

Approved source:

10.10.20.20

Approved service:

TCP/443

PowerShell validation:

Test-NetConnection 10.10.20.1 -Port 443

Result:

TcpTestSucceeded : True

Kali on REDNET cannot access the OPNsense Web GUI.

---

# 17. Validated Security Matrix

| Source | Destination | Service | Result |
|---|---|---|---|
| Windows | OPNsense | HTTPS | ALLOW |
| Windows | OPNsense | DNS | ALLOW |
| Windows | Internet | HTTPS / Internet | ALLOW |
| Windows | Ubuntu | SSH | ALLOW |
| Windows | Kali | ICMP / General | BLOCK |
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

# 18. Network Validation

The network was validated using tools including:

Linux:

ip addr

ip route

ping

nslookup

curl

ss

nmcli

Nmap

Windows:

Get-NetAdapter

Get-NetIPAddress

Get-NetIPConfiguration

Get-NetRoute

Test-NetConnection

Resolve-DnsName

ping

curl.exe

OPNsense:

- Interface Overview
- Firewall Rules
- Firewall Live View
- Outbound NAT
- Routing
- Interface diagnostics
- Configuration backups

---

# 19. Troubleshooting Experience

Several real networking problems were encountered and resolved while building the environment.

These included:

- Windows APIPA addressing
- Incorrect Windows LAB-NET configuration
- Windows Firewall and ICMP behavior
- Kali-to-Windows connectivity failure
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
- DNS testing
- Outbound NAT validation
- Stateful firewall testing

These troubleshooting exercises are documented as part of the portfolio.

---

# 20. Snapshot and Recovery Strategy

VirtualBox snapshots are taken before significant changes.

The standard workflow is:

Snapshot
↓
Change
↓
Test
↓
Verify
↓
Backup
↓
Document
↓
Continue

Snapshots were created:

- Before firewall deployment changes
- Before security-zone migration
- Before final NAT removal
- After successful host cutovers
- After explicit firewall policy validation

OPNsense configuration backups are also exported after major firewall milestones.

Firewall backup XML files are stored privately and are not committed to the public repository.

---

# 21. Repository Structure

Current project structure:

cybersecurity-homelab/
|
├── README.md
|
├── architecture/
│   ├── network-diagram.md
│   ├── network-diagram.png
│   ├── lab-architecture.md
│   └── virtualbox-networking.md
|
├── virtual-machines/
│   ├── kali.md
│   ├── windows.md
│   └── ubuntu.md
|
├── networking/
│   ├── addressing.md
│   ├── connectivity-tests.md
│   └── troubleshooting.md
|
├── firewall/
│   ├── opnsense-firewall.md
│   ├── security-zones.md
│   ├── policy-matrix.md
│   ├── firewall-rules.md
│   └── validation-testing.md
|
├── linux/
│   └── ubuntu-server.md
|
├── windows/
│   └── windows-security.md
|
├── offensive-security/
|
├── blue-team/
|
├── incident-response/
|
├── screenshots/
|
└── CHANGELOG.md

The repository will continue expanding as new laboratory capabilities are deployed.

---

# 22. Architecture Evolution

The project has evolved through several stages.

Stage 1:

Three Virtual Machines

↓

Stage 2:

Shared LAB-NET

192.168.56.0/24

↓

Stage 3:

Networking and Connectivity Troubleshooting

↓

Stage 4:

OPNsense Firewall Deployment

↓

Stage 5:

Security-Zone Segmentation

↓

Stage 6:

CORPNET

REDNET

SERVERNET

↓

Stage 7:

Explicit Firewall Policies

↓

Stage 8:

Outbound NAT Through OPNsense

↓

Stage 9:

Removal of Direct NAT Bypasses

↓

Stage 10:

Validated Offensive and Defensive Paths

The next stage introduces centralized monitoring and network detection.

---

# 23. Original LAB-NET

The original flat laboratory network was:

192.168.56.0/24

Original addresses:

Kali:

192.168.56.10

Windows:

192.168.56.20

Ubuntu:

192.168.56.30

LAB-NET was useful for learning:

- Static IP addressing
- Routing
- Windows networking
- Linux networking
- Firewall troubleshooting
- Connectivity testing

It has now been retired from the active architecture.

---

# 24. Planned SOC Architecture

The next major defensive-security phase will introduce centralized security monitoring.

Proposed network:

SOC-NET

Possible subnet:

10.10.40.0/24

Planned technologies include:

- Wazuh
- Centralized logging
- Security dashboards
- Detection engineering
- Alert investigation
- Threat hunting

Potential telemetry sources include:

- Windows
- Ubuntu
- OPNsense
- Suricata
- Future Active Directory systems

---

# 25. Planned Endpoint Telemetry

Windows telemetry will eventually include:

- Windows Security Event Log
- Sysmon
- PowerShell logging
- Microsoft Defender events
- Authentication events
- Process execution
- Network connections

Ubuntu telemetry will include:

- Authentication logs
- SSH logs
- systemd journal
- Linux audit logs
- Service logs
- Network activity

---

# 26. Planned IDS/IPS

Suricata is planned for network intrusion detection and prevention.

Initial deployment:

IDS Mode

Purpose:

- Observe suspicious traffic
- Generate alerts
- Learn signatures
- Tune detections
- Reduce false positives

Future testing:

IPS Mode

Target workflow:

Kali
↓
Attack
↓
OPNsense
↓
Suricata
↓
Alert
↓
SIEM
↓
Investigation

---

# 27. Planned SIEM

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

The long-term objective is to correlate firewall, endpoint, network, authentication, and attack activity.

---

# 28. Planned Active Directory Lab

A future enterprise identity environment will include Windows Server and Active Directory.

Possible future zone:

AD-NET

Possible subnet:

10.10.50.0/24

Planned technologies and concepts include:

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

# 29. Planned Active Directory Security Testing

The Active Directory environment will eventually support authorized exercises involving:

- Enumeration
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

These exercises will remain entirely within the controlled homelab.

---

# 30. Planned DMZ

A future DMZ may use:

10.10.60.0/24

Potential systems include:

- OWASP Juice Shop
- DVWA
- Linux web servers
- APIs
- Intentionally vulnerable applications
- Public-facing test services

The DMZ will be isolated using OPNsense firewall policy.

---

# 31. Offensive Security Roadmap

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

# 32. Defensive Security Roadmap

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

# 33. Network Analysis Roadmap

Planned network-security tools include:

- Wireshark
- tcpdump
- Nmap
- Netcat
- Suricata
- OPNsense
- Network-flow analysis
- DNS analysis
- TCP/IP analysis
- HTTP/HTTPS analysis
- Firewall-log analysis

---

# 34. Incident Response Roadmap

Future incident-response exercises will include:

Preparation
↓
Detection
↓
Triage
↓
Investigation
↓
Containment
↓
Eradication
↓
Recovery
↓
Lessons Learned

Potential tools include:

- Wazuh
- Sysmon
- Event Viewer
- PowerShell
- Linux logs
- Wireshark
- Velociraptor
- OPNsense
- Suricata

---

# 35. Future Architecture

The long-term architecture is expected to evolve toward:

Internet
|
v
OPNsense
|
+-------------+-------------+-------------+-------------+-------------+
|             |             |             |             |
v             v             v             v             v
REDNET      CORPNET      SERVERNET      SOC-NET       AD-NET
|             |             |             |             |
Kali        Windows        Linux         Wazuh       Windows Server
Attack      Endpoint       Servers       SIEM        Active Directory
Tools       Management     Targets       Monitoring   Domain Services
|
+------------------------ Attack Telemetry ------------------------+
                                                                  |
                                                                  v
                                                         Detection / Response

Future:

DMZ-NET
|
Vulnerable Applications

All zones will be controlled through OPNsense.

---

# 36. Learning Methodology

The project follows:

Learn
↓
Build
↓
Break
↓
Troubleshoot
↓
Fix
↓
Test
↓
Document
↓
Repeat

Every major configuration change should be:

- Understood
- Tested
- Verified
- Documented
- Recoverable

---

# 37. Current Project Status

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

# 38. Current Milestone

The homelab has successfully progressed from:

Basic Virtual Machines

↓

Flat Network

↓

Network Troubleshooting

↓

Professional Firewall Deployment

↓

Security-Zone Segmentation

↓

Least-Privilege Firewall Policies

↓

Mandatory Firewall Routing

↓

Validated Offensive Security Path

↓

Validated Defensive Administration Path

The networking and firewall foundation is now operational.

---

# 39. Next Phase

The next major project phase will focus on:

Centralized Logging
↓
Endpoint Telemetry
↓
Network Detection
↓
SIEM
↓
Alerting
↓
Threat Hunting
↓
Incident Response

This will transform the network-security lab into a more complete offensive-and-defensive cybersecurity environment.

---

# 40. Security Disclaimer

This cybersecurity homelab is used exclusively for authorized cybersecurity education, ethical hacking, defensive-security training, penetration-testing practice, and incident-response exercises.

All network scanning, enumeration, exploitation, attack simulation, and security testing are performed only against systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, applications, services, accounts, or organizations are targeted.

---

# Author

Emmanuel Ngari

Cybersecurity | Network Security | Blue Team | Red Team

GitHub:

Emmanuel-Ngari
