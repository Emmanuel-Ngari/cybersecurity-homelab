# Cybersecurity Homelab Network Diagram

## 1. Current Architecture

The current cybersecurity homelab uses OPNsense as the central firewall, router, NAT gateway, DNS service, and policy enforcement point.

Internet
|
v
VirtualBox NAT
|
v
LAB-FW-01
OPNsense
|
+----------------------+----------------------+----------------------+
|                      |                      |
v                      v                      v
RED-NET               CORP-NET               SERVER-NET
10.10.10.0/24         10.10.20.0/24          10.10.30.0/24
|                      |                      |
v                      v                      v
LAB-KALI-01           LAB-WIN-01             LAB-UBUNTU-01
10.10.10.10           10.10.20.20            10.10.30.30
Offensive Security    Management / Blue Team Protected Linux Server

All inter-zone traffic passes through OPNsense.

---

## 2. OPNsense Firewall Interfaces

LAB-FW-01 uses four network interfaces.

WAN

Interface:

em0

Address:

10.0.2.15/24

Gateway:

10.0.2.2

Purpose:

Internet connectivity through VirtualBox NAT.

CORPNET

Interface:

em1

Address:

10.10.20.1/24

Network:

10.10.20.0/24

Purpose:

Trusted management and endpoint network.

REDNET

Interface:

em2

Address:

10.10.10.1/24

Network:

10.10.10.0/24

Purpose:

Offensive-security network.

SERVERNET

Interface:

em3

Address:

10.10.30.1/24

Network:

10.10.30.0/24

Purpose:

Protected server network.

---

## 3. Security Zones

The current environment contains three internal security zones.

| Zone | Subnet | Gateway | Primary Role |
|---|---|---|---|
| REDNET | 10.10.10.0/24 | 10.10.10.1 | Offensive Security |
| CORPNET | 10.10.20.0/24 | 10.10.20.1 | Management / Endpoint |
| SERVERNET | 10.10.30.0/24 | 10.10.30.1 | Protected Servers |

---

## 4. Core Systems

| Host | Operating System | Zone | IPv4 Address | Role |
|---|---|---|---|---|
| LAB-FW-01 | OPNsense 26.7 | All Zones | Multiple | Firewall / Router |
| LAB-KALI-01 | Kali Linux | REDNET | 10.10.10.10 | Offensive Security |
| LAB-WIN-01 | Windows 11 Enterprise | CORPNET | 10.10.20.20 | Management / Defensive Endpoint |
| LAB-UBUNTU-01 | Ubuntu Server | SERVERNET | 10.10.30.30 | Protected Linux Server |

---

## 5. Internet Traffic Flow

Kali Linux:

LAB-KALI-01
↓
REDNET
↓
10.10.10.1
↓
OPNsense
↓
WAN
↓
VirtualBox NAT
↓
Internet

Windows:

LAB-WIN-01
↓
CORPNET
↓
10.10.20.1
↓
OPNsense
↓
WAN
↓
VirtualBox NAT
↓
Internet

Ubuntu:

LAB-UBUNTU-01
↓
SERVERNET
↓
10.10.30.1
↓
OPNsense
↓
WAN
↓
VirtualBox NAT
↓
Internet

Direct VirtualBox NAT adapters on Kali, Windows, and Ubuntu are disabled.

Only OPNsense has direct VirtualBox NAT connectivity.

---

## 6. REDNET Architecture

REDNET is the offensive-security zone.

Network:

10.10.10.0/24

Gateway:

10.10.10.1

Primary Host:

LAB-KALI-01

Address:

10.10.10.10/24

DNS:

10.10.10.1

Primary functions include:

- Network reconnaissance
- Port scanning
- Service enumeration
- Vulnerability assessment
- Penetration testing
- Ethical hacking
- Exploitation labs
- Red Team simulations

REDNET is treated as an untrusted internal network.

---

## 7. CORPNET Architecture

CORPNET is the trusted management and endpoint zone.

Network:

10.10.20.0/24

Gateway:

10.10.20.1

Primary Host:

LAB-WIN-01

Address:

10.10.20.20/24

DNS:

10.10.20.1

Primary functions include:

- OPNsense management
- Windows administration
- PowerShell
- Endpoint security
- Blue Team exercises
- Server administration
- Defensive-security testing

CORPNET does not have unrestricted access to REDNET.

---

## 8. SERVERNET Architecture

SERVERNET is the protected server zone.

Network:

10.10.30.0/24

Gateway:

10.10.30.1

Primary Host:

LAB-UBUNTU-01

Address:

10.10.30.30/24

DNS:

10.10.30.1

Primary functions include:

- Linux administration
- SSH services
- Server hardening
- Security monitoring
- Vulnerability testing
- Controlled attack targets
- Future application hosting
- Future internal services

SERVERNET is restricted from initiating unnecessary connections to CORPNET and REDNET.

---

## 9. Approved Security Paths

### Windows to OPNsense

Source:

10.10.20.20

Destination:

10.10.20.1

Service:

HTTPS TCP/443

Result:

ALLOW

Purpose:

Firewall administration.

---

### Windows to Ubuntu

Source:

10.10.20.20

Destination:

10.10.30.30

Approved Services:

SSH TCP/22

HTTPS TCP/443

Result:

ALLOW

Purpose:

Server administration.

---

### Kali to Ubuntu

Source:

10.10.10.10

Destination:

10.10.30.30

Result:

ALLOW

Purpose:

Authorized offensive-security testing.

This is the primary attack path:

Kali
↓
REDNET
↓
OPNsense
↓
SERVERNET
↓
Ubuntu

---

## 10. Blocked Security Paths

### Kali to Windows

REDNET
↓
CORPNET

Result:

BLOCK

Purpose:

Prevent unrestricted attacker access to trusted management systems.

---

### Windows to Kali

CORPNET
↓
REDNET

Result:

BLOCK

Purpose:

Maintain separation between trusted and offensive-security systems.

---

### Ubuntu to Windows

SERVERNET
↓
CORPNET

Result:

BLOCK

Purpose:

Reduce lateral movement opportunities from compromised server systems.

---

### Ubuntu to Kali

SERVERNET
↓
REDNET

Result:

BLOCK

Purpose:

Prevent servers from initiating unnecessary connections toward attacker infrastructure.

---

### Kali to OPNsense Management

REDNET
↓
OPNsense HTTPS

Result:

BLOCK

Purpose:

Prevent the attacker network from managing the firewall.

---

## 11. Firewall Management Security

OPNsense is managed from the trusted Windows workstation.

Management Workstation:

10.10.20.20

Firewall Management Address:

10.10.20.1

Protocol:

HTTPS

Port:

443

Validated command:

Test-NetConnection 10.10.20.1 -Port 443

Result:

TcpTestSucceeded : True

Kali cannot access the OPNsense Web GUI.

---

## 12. DNS Architecture

Each zone uses its local OPNsense interface as DNS.

Kali:

10.10.10.1

Windows:

10.10.20.1

Ubuntu:

10.10.30.1

DNS traffic is controlled using explicit firewall rules.

---

## 13. NTP Architecture

NTP is permitted to OPNsense using:

UDP/123

Accurate time synchronization is important for:

- Log correlation
- SIEM
- Authentication
- Incident timelines
- Digital forensics
- Detection engineering

---

## 14. Stateful Firewalling

OPNsense uses stateful firewalling.

For example:

Kali initiates:

10.10.10.10 → 10.10.30.30:22

If the REDNET firewall policy permits the connection, Ubuntu can send response traffic through the existing firewall state.

Ubuntu does not require a broad rule allowing new connections toward REDNET.

This provides controlled bidirectional communication while maintaining directional security policy.

---

## 15. Layer 2 Segmentation

VirtualBox provides separate Internal Networks:

RED-NET

CORP-NET

SERVER-NET

These networks are separate Layer 2 broadcast domains.

Systems in different Internal Networks cannot communicate directly without routing.

---

## 16. Layer 3 Security Enforcement

OPNsense provides Layer 3 routing between the security zones.

Traffic crossing between:

REDNET

CORPNET

SERVERNET

must pass through OPNsense.

Firewall rules determine whether that traffic is:

ALLOW

or

BLOCK

The architecture therefore combines:

Layer 2 Isolation

+

Layer 3 Firewall Enforcement

---

## 17. Final Access Matrix

| Source | Destination | Service | Policy |
|---|---|---|---|
| CORPNET | OPNsense | HTTPS | ALLOW |
| CORPNET | OPNsense | DNS | ALLOW |
| CORPNET | OPNsense | NTP | ALLOW |
| CORPNET | Internet | Required outbound traffic | ALLOW |
| CORPNET | SERVERNET | SSH / HTTPS | ALLOW |
| CORPNET | REDNET | Any | BLOCK |
| REDNET | OPNsense | DNS | ALLOW |
| REDNET | OPNsense | NTP | ALLOW |
| REDNET | OPNsense | ICMP | LIMITED ALLOW |
| REDNET | OPNsense | Management | BLOCK |
| REDNET | CORPNET | Any | BLOCK |
| REDNET | SERVERNET | Authorized lab testing | ALLOW |
| REDNET | Internet | Required outbound traffic | ALLOW |
| SERVERNET | OPNsense | DNS | ALLOW |
| SERVERNET | OPNsense | NTP | ALLOW |
| SERVERNET | OPNsense | ICMP | LIMITED ALLOW |
| SERVERNET | CORPNET | New unsolicited traffic | BLOCK |
| SERVERNET | REDNET | New unsolicited traffic | BLOCK |
| SERVERNET | Internet | Required outbound traffic | ALLOW |

---

## 18. Outbound NAT

OPNsense uses:

Automatic Source NAT rule generation

The NAT configuration includes:

10.10.10.0/24

10.10.20.0/24

10.10.30.0/24

Internal addresses are translated to the OPNsense WAN address before reaching the Internet.

---

## 19. Direct NAT Bypass Removal

Originally, Kali, Windows, and Ubuntu each had direct VirtualBox NAT access.

Those adapters were disabled after OPNsense outbound NAT was successfully tested.

Current status:

Kali Direct NAT:

Disabled

Windows Direct NAT:

Disabled

Ubuntu Direct NAT:

Disabled

OPNsense WAN NAT:

Enabled

This makes OPNsense the mandatory Internet gateway.

---

## 20. Legacy LAB-NET

The original flat network was:

LAB-NET

Subnet:

192.168.56.0/24

Original addresses:

Kali:

192.168.56.10

Windows:

192.168.56.20

Ubuntu:

192.168.56.30

This network is no longer part of the active architecture.

It has been replaced by:

REDNET

CORPNET

SERVERNET

The old network remains part of the project history to show how the architecture evolved.

---

## 21. Firewall Policy Model

The firewall follows:

Specific Allow
↓
Specific Block
↓
Private Network Restrictions
↓
Controlled Internet Access
↓
Default Deny

The policy is designed around:

- Least privilege
- Network segmentation
- Explicit access
- Controlled administration
- Controlled offensive testing
- Logging
- Default deny

---

## 22. Validated Security Tests

The following behavior has been verified.

Windows → OPNsense HTTPS:

ALLOW

Windows → OPNsense DNS:

ALLOW

Windows → Internet:

ALLOW

Windows → Ubuntu SSH:

ALLOW

Windows → Kali:

BLOCK

Kali → OPNsense DNS:

ALLOW

Kali → Internet:

ALLOW

Kali → OPNsense HTTPS:

BLOCK

Kali → Windows:

BLOCK

Kali → Ubuntu:

ALLOW

Ubuntu → OPNsense DNS:

ALLOW

Ubuntu → Internet:

ALLOW

Ubuntu → Windows:

BLOCK

Ubuntu → Kali:

BLOCK

---

## 23. Offensive Security Workflow

The current controlled offensive workflow is:

LAB-KALI-01
↓
REDNET
↓
OPNsense
↓
Firewall Logging
↓
SERVERNET
↓
LAB-UBUNTU-01

Examples of future exercises include:

- Host discovery
- Port scanning
- Service enumeration
- SSH enumeration
- Vulnerability scanning
- Web application testing
- Controlled exploitation
- Privilege escalation
- Post-exploitation analysis

---

## 24. Defensive Security Workflow

The defensive workflow will evolve toward:

Attack Traffic
↓
OPNsense
↓
Firewall Logs
↓
IDS
↓
Endpoint Logs
↓
SIEM
↓
Alert
↓
Investigation
↓
Incident Response
↓
Remediation

This allows offensive actions to generate defensive telemetry.

---

## 25. Planned SOC-NET

A future SOC security zone may use:

10.10.40.0/24

Possible components include:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Log collectors
- Security-monitoring tools
- Detection-engineering infrastructure

Potential data sources:

- OPNsense
- Windows
- Ubuntu
- Suricata
- Future Active Directory systems

---

## 26. Planned AD-NET

A future Active Directory zone may use:

10.10.50.0/24

Potential systems include:

- Windows Server
- Active Directory Domain Services
- DNS
- Group Policy
- Domain Controller
- Domain-joined endpoints

This environment will support:

- Active Directory administration
- Kerberos
- Identity security
- AD attack techniques
- Detection engineering
- Windows logging
- Incident response

---

## 27. Planned DMZ-NET

A future DMZ may use:

10.10.60.0/24

Potential workloads include:

- Web servers
- Vulnerable applications
- APIs
- Public-facing laboratory services
- Security-testing targets

The DMZ will be isolated from trusted internal zones through explicit firewall rules.

---

## 28. Planned IDS/IPS Architecture

Suricata will later be added to the environment.

Initial deployment:

IDS Mode

Future testing:

IPS Mode

Target workflow:

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
Wazuh SIEM
↓
Blue Team Investigation

---

## 29. Planned SIEM Architecture

Wazuh is planned as the primary centralized security-monitoring platform.

Potential telemetry includes:

Windows:

- Security Event Logs
- Sysmon
- PowerShell Logs
- Microsoft Defender events
- Authentication events

Ubuntu:

- Authentication logs
- SSH logs
- System logs
- Application logs
- Audit logs

OPNsense:

- Firewall logs
- Allowed connections
- Blocked connections
- Network activity

Suricata:

- IDS alerts
- Network threat events

---

## 30. Target Professional Architecture

The long-term architecture is expected to evolve toward:

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
Kali           Windows          Linux           Wazuh          Windows Server
Attack         Endpoint         Servers         SIEM           Active Directory
Tools          Management       Services        Monitoring      Domain Services
|                                                |
|                                                |
+---------------- Attack Telemetry ---------------+
                                                 |
                                                 v
                                         Detection / Response

Additional future zone:

DMZ-NET

Purpose:

Vulnerable and public-facing laboratory services.

---

## 31. Future Professional Tools

Planned defensive tools include:

- OPNsense
- Suricata
- Wazuh
- Sysmon
- Windows Event Viewer
- PowerShell
- Linux auditd
- Wireshark
- tcpdump
- Velociraptor
- Security Onion

Planned offensive tools include:

- Kali Linux
- Nmap
- Burp Suite
- Metasploit
- BloodHound
- Impacket
- Netcat
- Gobuster
- Hydra
- Wireshark
- Custom scripts

Planned vulnerable targets may include:

- OWASP Juice Shop
- DVWA
- Linux services
- Windows systems
- Active Directory
- Intentionally vulnerable hosts

---

## 32. Current Architecture Status

OPNsense:

Operational

WAN:

Operational

REDNET:

Operational

CORPNET:

Operational

SERVERNET:

Operational

Kali:

Operational

Windows:

Operational

Ubuntu:

Operational

Outbound NAT:

Verified

DNS:

Verified

Internet Access:

Verified

Firewall Segmentation:

Verified

Controlled Attack Path:

Verified

Controlled Administrative Path:

Verified

Direct VM NAT Bypass:

Disabled

Default LAN Allow Rules:

Disabled

Original LAB-NET:

Retired

SOC-NET:

Planned

Suricata:

Planned

Wazuh:

Planned

Active Directory:

Planned

DMZ:

Planned

---

## 33. Architecture Evolution

The project has evolved through the following stages:

Flat VirtualBox Network
↓
Shared LAB-NET
↓
Three-VM Cybersecurity Lab
↓
Dedicated OPNsense Firewall
↓
Multiple Security Zones
↓
Explicit Firewall Rules
↓
Mandatory Firewall Routing
↓
Controlled Attack Paths
↓
Controlled Administrative Paths
↓
Validated Segmentation
↓
Foundation for SOC and Detection Engineering

---

## 34. Learning Methodology

The homelab follows:

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

The objective is not simply to install cybersecurity tools.

Each technology should be understood from:

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

## 35. Security Disclaimer

This cybersecurity homelab is used exclusively for authorized cybersecurity education, ethical hacking, defensive-security training, penetration-testing practice, and incident-response exercises.

All offensive-security activity is performed only against systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, applications, services, accounts, or organizations are targeted.
