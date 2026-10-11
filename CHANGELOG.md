# Changelog

All notable changes to the Cybersecurity Home Lab are documented in this file.

This changelog preserves the major technical milestones of the project from the original flat VirtualBox lab through the final validated six-zone Purple Team environment.

---

## Final Validated Baseline

### Final Architecture

The lab reached its completed six-zone architecture with:

- REDNET — `10.10.10.0/24`
- CORPNET — `10.10.20.0/24`
- SERVERNET — `10.10.30.0/24`
- SOCNET — `10.10.40.0/24`
- ADNET — `10.10.50.0/24`
- DMZ — `10.10.60.0/24`

Primary systems:

- `LAB-FW-01` — OPNsense
- `LAB-KALI-01` — `10.10.10.10`
- `LAB-WIN-01` — `10.10.20.20`
- `LAB-UBUNTU-01` — `10.10.30.30`
- `LAB-SIEM-01` — `10.10.40.10`
- `LAB-DC-01` — `10.10.50.10`
- `LAB-DMZ-01` — `10.10.60.10`

### Final Validation

Validated:

- REDNET → DMZ HTTP/80 allowed
- REDNET → DMZ SSH/22 blocked
- REDNET → CORPNET SMB/445 blocked
- REDNET → ADNET LDAP/389 blocked
- CORPNET → ADNET DNS/53 allowed
- CORPNET → ADNET Kerberos/88 allowed
- CORPNET → ADNET LDAP/389 allowed
- CORPNET → SOCNET Wazuh/1514 allowed
- LAB-WIN-01 → DMZ SSH/22 allowed
- SERVERNET → SOCNET Wazuh/1514 allowed
- SERVERNET → CORPNET SMB/445 blocked
- SERVERNET → DMZ SSH/22 blocked
- ADNET → SOCNET Wazuh/1514 allowed
- ADNET → DMZ SSH/22 blocked
- DMZ → SOCNET Wazuh/1514 allowed
- DMZ → CORPNET SMB/445 blocked
- DMZ → ADNET LDAP/389 blocked
- DMZ → Internet HTTPS/443 allowed

### Final Wazuh State

All endpoint agents Active:

- ID 001 — `LAB-WIN-01`
- ID 002 — `LAB-UBUNTU-01`
- ID 003 — `LAB-DC-01`
- ID 004 — `LAB-DMZ-01`

Wazuh Manager version:

`v4.14.7`

Core services validated:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

### Final Snapshot State

All temporary development snapshots were removed.

Each VM retains one clean known-good snapshot:

`Final-Six-Zone-Validated-Baseline`

Snapshot UUIDs:

- LAB-FW-01 — `d0821c00-02ab-47a9-8341-b6e31001b3b6`
- LAB-KALI-01 — `dbb7b8d0-d519-48ef-9e57-7571b912ba92`
- LAB-WIN-01 — `36d9a7bc-4556-4f14-9887-959d4a46e65c`
- LAB-UBUNTU-01 — `23d97e1a-a95e-4dd9-8912-f48e312ccf74`
- LAB-SIEM-01 — `4081e1d5-fcb7-4f40-ad2a-04a081c865ca`
- LAB-DC-01 — `dd108acf-e11f-44f2-82f1-c3665d3f3691`
- LAB-DMZ-01 — `ac27c1e7-c61f-41fd-ab11-b7e84883068c`

---

## DMZ Phase

### Added

Created sixth security zone:

`DMZ — 10.10.60.0/24`

Gateway:

`10.10.60.1`

Added:

`LAB-DMZ-01 — 10.10.60.10`

Operating system:

`Ubuntu Server 26.04.1 LTS`

Primary service:

`Nginx`

### OPNsense Expansion

Added seventh VirtualBox NIC to:

`LAB-FW-01`

Configured:

- Internal Network: `DMZ-NET`
- Adapter type: `82540EM`
- OPNsense interface: `em6`

Assigned as:

`DMZ`

### DMZ Firewall Rules

Added:

- DMZ → OPNsense DNS
- DMZ → OPNsense NTP
- DMZ → Wazuh Agent Services
- DMZ → Unauthorized Private Networks block
- DMZ → HTTP/HTTPS outbound

Added alias:

`LAB_DMZ_01 = 10.10.60.10`

Added alias:

`HTTP_HTTPS_PORTS = 80,443`

### REDNET Web Exposure

Added:

`ALLOW - REDNET to LAB-DMZ-01 HTTP`

Validated HTTP/80 from Kali.

Kept SSH/22 blocked from REDNET.

### OPNsense Apply Lesson

Resolved issue where correctly configured REDNET-to-DMZ HTTP rule remained blocked because pending OPNsense changes had not been applied.

Operational lesson established:

Save → Apply → Generate fresh traffic → Verify.

### Restricted DMZ Administration

Added:

`ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH`

Source:

`LAB-WIN-01 — 10.10.20.20`

Validated SSH administration from CORPNET.

Confirmed:

- REDNET → DMZ SSH blocked
- ADNET → DMZ SSH blocked

### SSH Hardening

Implemented:

- Ed25519 key authentication
- `PermitRootLogin no`
- `PubkeyAuthentication yes`
- `PasswordAuthentication no`
- `KbdInteractiveAuthentication no`

Validated key-based login.

Validated password-only authentication failure.

### Wazuh DMZ Monitoring

Installed Wazuh Agent `4.14.7-1`.

Registered:

`ID 004 — LAB-DMZ-01`

Configured Nginx monitoring:

- `/var/log/nginx/access.log` using `apache`
- `/var/log/nginx/error.log` using `syslog`

Validated Wazuh Rule:

`31101`

from controlled Nginx 404 traffic.

### Suricata DMZ Validation

Created temporary controlled HTTP content:

`uid=0(root) gid=0(root) groups=0(root)`

Triggered:

`Suricata SID 2100498`

and:

`Wazuh Rule 86601`

Removed temporary test file after validation.

### System Maintenance

Updated approximately 23 packages.

Validated:

- SSH active
- Nginx active
- Wazuh Agent active

No reboot required.

---

## Active Directory Phase

### Added ADNET

Created:

`ADNET — 10.10.50.0/24`

Gateway:

`10.10.50.1`

Added:

`LAB-DC-01 — 10.10.50.10`

### Windows Server Deployment

Deployed:

`Windows Server 2025 Standard Evaluation`

Configured static networking.

### Active Directory Domain Services

Created new forest:

`cyberlab.internal`

NetBIOS:

`CYBERLAB`

Configured:

- Active Directory Domain Services
- DNS
- Global Catalog
- Kerberos
- LDAP
- SYSVOL
- NETLOGON

### Domain Health Validation

Validated:

- `whoami`
- `nltest /dsgetdc`
- SYSVOL
- NETLOGON
- targeted `dcdiag`
- DNS health

### Organizational Structure

Created OUs:

- Admins
- Users
- Groups
- Service Accounts
- Servers
- Workstations

Created Global Groups:

- GG_Employees
- GG_Helpdesk
- GG_IT_Admins
- GG_SOC_Analysts
- GG_Domain_Admins

### User Accounts

Created standard account:

`emmanuel.patrick@cyberlab.internal`

Created privileged account:

`emmanuel.ngari-adm@cyberlab.internal`

Separated normal user activity from Domain Admin activity.

### AD Firewall Policy

Added explicit CORPNET → Domain Controller rules for required Active Directory services.

Important TCP services included:

- 53
- 88
- 135
- 389
- 445
- 464
- 3268
- 49152:65535

Important UDP services included:

- 53
- 88
- 123
- 389
- 464

Validated:

- DNS/53
- Kerberos/88
- LDAP/389

### Workstation Domain Join

Joined:

`LAB-WIN-01`

to:

`cyberlab.internal`

### Group Policy

Created:

`GPO-Workstation-Security-Baseline`

Linked to:

`CyberLab\Workstations`

Configured:

- Windows Firewall enabled
- inbound block
- outbound allow
- Logon Success/Failure auditing
- Process Creation auditing
- Special Logon auditing
- command-line process logging
- PowerShell Script Block Logging
- PowerShell Module Logging

### Domain Controller Monitoring

Registered:

`ID 003 — LAB-DC-01`

with Wazuh.

Observed important events:

- 4672
- 4719
- 5136

### Domain Controller Sysmon

Installed:

`Sysmon 15.22`

Path:

`C:\Tools\Sysmon\Sysmon64.exe`

Configuration:

`sysmon-dc.xml`

Tuned ProcessCreate and DNS monitoring to reduce noise.

### ADNET Segmentation

Validated:

- REDNET → ADNET LDAP blocked
- ADNET → SOCNET Wazuh allowed
- ADNET → DMZ SSH blocked

---

## Suricata IDS Phase

### Added Suricata

Installed Suricata on:

`LAB-FW-01`

Monitored interface:

`em2`

Corresponding zone:

`REDNET`

Enabled ET Open rules.

### Initial Detection

Validated:

`SID 2100498`

`GPL ATTACK_RESPONSE id check returned root`

### Wazuh Ingestion Troubleshooting

Initial Suricata events fell through to:

`Rule 1002`

Identified issue as decoding/parsing rather than network transport.

### Custom Decoder Experimentation

Tested custom decoder:

`suricata-eve-syslog`

Used temporarily during troubleshooting.

### JSON Migration

Moved to structured JSON ingestion.

Effective decoder:

`json`

Retired unnecessary custom wrapper decoder.

### Dedicated Syslog Transport

Created dedicated Suricata transport:

`UDP/5514`

Configured rsyslog listener on:

`LAB-SIEM-01`

Created:

`/etc/rsyslog.d/30-suricata-remote.conf`

### EVE JSON Storage

Created:

`/var/log/suricata/eve.json`

Configured ownership:

`syslog:wazuh`

Configured Wazuh localfile ingestion using JSON format.

### Wazuh Suricata Detection

Validated built-in Wazuh rule:

`86601`

from:

`0475-suricata_rules.xml`

### Troubleshooting Lessons

Documented:

- `full_log` may be null for structured events
- live `jq` parse errors do not automatically indicate pipeline failure
- end-to-end evidence should guide troubleshooting
- built-in rules should be preferred where reliable

---

## OPNsense Firewall Telemetry Phase

### Added

Forwarded OPNsense firewall logs into Wazuh.

### Detection

Validated firewall correlation:

`Wazuh Rule 87702`

### Result

OPNsense became both:

- a firewall enforcement platform
- a network security telemetry source

---

## Linux Monitoring and Detection Engineering Phase

### Wazuh Agent

Enrolled:

`LAB-UBUNTU-01`

Final agent:

`ID 002`

### auditd

Integrated Linux `auditd` telemetry into Wazuh.

### Custom Rules

Validated:

`Rule 100200`

for native Linux user creation.

Validated:

`Rule 100201`

for native Linux user deletion.

### Connectivity Troubleshooting

Confirmed Wazuh TCP/1514 could be healthy even when ICMP failed.

Established service-level validation as preferred troubleshooting method.

### Agent-State Troubleshooting

Investigated temporary apparent agent disconnect.

Determined older error logs did not represent current state.

Confirmed agent returned to:

`Active`

---

## Windows Endpoint Monitoring Phase

### Wazuh Agent

Enrolled:

`LAB-WIN-01`

Final agent:

`ID 001`

### Sysmon

Installed and integrated Sysmon into Wazuh.

Added richer process visibility.

### PowerShell Logging

Enabled:

- Script Block Logging
- Module Logging
- transcription

### Advanced Auditing

Enabled:

- Logon Success
- Logon Failure
- Process Creation
- Special Logon
- command-line process capture

### Defender

Integrated Microsoft Defender telemetry.

### Windows Firewall

Added monitoring for:

`Event ID 5157`

### File Integrity Monitoring

Configured Wazuh FIM for selected Windows paths and changes.

### Result

Windows evolved from a standalone endpoint into a fully monitored corporate workstation.

---

## Wazuh SIEM Deployment Phase

### Added SOCNET

Created:

`SOCNET — 10.10.40.0/24`

Gateway:

`10.10.40.1`

Added:

`LAB-SIEM-01 — 10.10.40.10`

### Wazuh Stack

Deployed:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

### Result

The lab moved from local host log inspection to centralized SOC monitoring.

---

## Initial OPNsense Segmentation Phase

### Added LAB-FW-01

Introduced OPNsense as:

- router
- firewall
- segmentation platform
- Internet gateway
- DNS/NTP service

### Created REDNET

Network:

`10.10.10.0/24`

Host:

`LAB-KALI-01 — 10.10.10.10`

### Created CORPNET

Network:

`10.10.20.0/24`

Host:

`LAB-WIN-01 — 10.10.20.20`

### Created SERVERNET

Network:

`10.10.30.0/24`

Host:

`LAB-UBUNTU-01 — 10.10.30.30`

### Created SOCNET

Network:

`10.10.40.0/24`

Host:

`LAB-SIEM-01 — 10.10.40.10`

### Firewall Policy

Introduced:

- specific allow rules
- broad private-network deny rules
- restricted firewall management
- controlled REDNET attack paths
- protected SERVERNET
- SOC telemetry paths

### Result

The project transitioned from flat networking to enterprise-style trust zones.

---

## Windows Rebuild and VirtualBox Troubleshooting

### Windows Rebuild

Standardized the workstation as:

`LAB-WIN-01`

Final operating system:

`Windows 11 Enterprise Evaluation 25H2`

### Networking

Original adapters were organized and renamed during the early rebuild.

### VirtualBox Graphics

Resolved visual/login issues caused by 3D acceleration.

Restored stable full-screen operation.

### Result

Created a stable Windows endpoint suitable for security monitoring and domain integration.

---

## Original Linux Defensive Phase

### SSH

Enabled SSH on Ubuntu Server.

### Nmap

Validated:

`22/tcp open`

from Kali.

### Hydra

Used controlled Hydra authentication testing against SSH.

### fail2ban

Installed and configured fail2ban.

Validated attacker blocking.

Observed SSH becoming filtered from the attacker's perspective after repeated failures.

### Result

Created the first complete attacker/defender validation loop.

---

## Original Flat Lab

### Initial Network

Created lab around:

`192.168.56.0/24`

Primary systems:

- Kali — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu — `192.168.56.30`

VirtualBox NAT provided Internet access where required.

### Initial Skills Developed

The original lab focused on:

- networking
- Windows administration
- Linux administration
- SSH
- Nmap
- Hydra
- host firewall behaviour
- attacker/defender interaction

### Importance

This phase established the technical foundation for all later architecture and security work.

---

## Current Project State

The infrastructure phase is complete.

Current capabilities include:

- six-zone segmentation
- OPNsense firewalling
- Wazuh SIEM
- Windows endpoint monitoring
- Linux endpoint monitoring
- Active Directory
- Group Policy
- Suricata IDS
- firewall telemetry
- DMZ
- Nginx monitoring
- restricted SSH administration
- custom detection engineering
- validated allow and deny paths

The project has transitioned from infrastructure building into:

- SOC operations
- threat hunting
- incident investigation
- detection engineering
- Purple Team attack simulation
- detection-gap analysis
- response exercises

Future infrastructure changes should be driven by clear operational requirements discovered during realistic security scenarios.
