# Cybersecurity Home Lab

## Overview

This repository documents the complete evolution of a hands-on cybersecurity home lab from a simple flat VirtualBox network into a segmented enterprise-style Purple Team environment.

The project began with three systems:

- Kali Linux
- Windows
- Ubuntu Server

communicating over:

`192.168.56.0/24`

The original goal was to build practical skills in:

- networking
- Linux administration
- Windows administration
- SSH
- Nmap
- Hydra
- fail2ban
- attacker/defender interaction

As the project matured, the architecture was redesigned around OPNsense, centralized monitoring, endpoint telemetry, Active Directory, network IDS, and a dedicated DMZ.

The final environment now includes:

- six security zones
- OPNsense routing and firewalling
- Wazuh SIEM
- Windows Sysmon
- PowerShell logging
- advanced Windows auditing
- Microsoft Defender monitoring
- Windows Firewall telemetry
- Linux auditd
- custom Wazuh detections
- Suricata IDS
- Active Directory
- Group Policy
- Domain Controller monitoring
- Nginx application monitoring
- restricted DMZ administration
- validated network segmentation
- centralized multi-source security telemetry

The infrastructure phase is complete and the lab is now ready for realistic SOC and Purple Team operations.

---

# Project Evolution

The project progressed through several major stages:

    Flat Learning Lab
          |
          v
    Nmap / SSH / Hydra / fail2ban
          |
          v
    OPNsense Segmentation
          |
          v
    REDNET / CORPNET / SERVERNET / SOCNET
          |
          v
    Wazuh SIEM
          |
          v
    Windows and Linux Monitoring
          |
          v
    Detection Engineering
          |
          v
    OPNsense Firewall Telemetry
          |
          v
    Suricata IDS
          |
          v
    Active Directory / ADNET
          |
          v
    Group Policy / DC Monitoring
          |
          v
    DMZ / Nginx / Restricted SSH
          |
          v
    Final Six-Zone Validation
          |
          v
    Purple Team Operations

---

# Final Architecture

The final network contains six security zones.

| Zone | Network | Primary System | Role |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `LAB-KALI-01` | Offensive security |
| CORPNET | `10.10.20.0/24` | `LAB-WIN-01` | Corporate workstation |
| SERVERNET | `10.10.30.0/24` | `LAB-UBUNTU-01` | Internal Linux server |
| SOCNET | `10.10.40.0/24` | `LAB-SIEM-01` | Security monitoring |
| ADNET | `10.10.50.0/24` | `LAB-DC-01` | Active Directory |
| DMZ | `10.10.60.0/24` | `LAB-DMZ-01` | Public-facing web server |

Central firewall:

`LAB-FW-01`

Platform:

`OPNsense`

---

# Final Systems

## LAB-FW-01

Role:

- firewall
- router
- segmentation point
- DNS/NTP service
- Internet gateway
- firewall telemetry source
- Suricata IDS platform

---

## LAB-KALI-01

Address:

`10.10.10.10`

Zone:

`REDNET`

Role:

- reconnaissance
- attack simulation
- web testing
- IDS validation
- Purple Team adversary system

---

## LAB-WIN-01

Address:

`10.10.20.20`

Zone:

`CORPNET`

Role:

- corporate workstation
- domain member
- monitored Windows endpoint
- approved DMZ administrative source

Security telemetry includes:

- Sysmon
- PowerShell logging
- advanced auditing
- Defender
- Windows Firewall
- File Integrity Monitoring

---

## LAB-UBUNTU-01

Address:

`10.10.30.30`

Zone:

`SERVERNET`

Role:

- protected internal Linux server
- Linux monitoring target
- auditd source
- custom detection platform

---

## LAB-SIEM-01

Address:

`10.10.40.10`

Zone:

`SOCNET`

Platform:

`Wazuh 4.14.7`

Role:

- centralized log collection
- alert generation
- endpoint monitoring
- detection engineering
- firewall telemetry
- IDS telemetry
- SOC investigation

---

## LAB-DC-01

Address:

`10.10.50.10`

Zone:

`ADNET`

Operating system:

`Windows Server 2025 Standard Evaluation`

Role:

- Active Directory Domain Services
- DNS
- Kerberos
- LDAP
- Global Catalog
- Group Policy
- identity telemetry

Domain:

`cyberlab.internal`

---

## LAB-DMZ-01

Address:

`10.10.60.10`

Zone:

`DMZ`

Operating system:

`Ubuntu Server 26.04.1 LTS`

Role:

- Nginx web server
- controlled REDNET target
- application monitoring
- Suricata validation
- restricted SSH administration

---

# Original Lab

The project began with:

`192.168.56.0/24`

Original systems:

    Kali      192.168.56.10
    Windows   192.168.56.20
    Ubuntu    192.168.56.30

This early environment supported:

- Nmap
- SSH
- Hydra
- fail2ban
- basic Windows firewall testing
- attacker/defender exercises

The original architecture is preserved in the documentation because it represents the real beginning of the project.

---

# OPNsense and Network Segmentation

The flat network was later replaced by OPNsense-controlled segmentation.

The first segmented architecture introduced:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

Later additions included:

- ADNET
- DMZ

All inter-zone communication now passes through:

`LAB-FW-01`

This provides:

- routing
- least-privilege firewall policy
- logging
- IDS inspection

---

# Firewall Design

The firewall strategy uses:

- explicit allow rules
- broad private-network deny rules
- default-deny behaviour
- logged security boundaries
- narrow administrative exceptions

Examples include:

    REDNET → DMZ HTTP/80                ALLOW
    REDNET → DMZ SSH/22                 BLOCK
    REDNET → CORPNET SMB/445            BLOCK
    REDNET → ADNET LDAP/389             BLOCK

    CORPNET → ADNET DNS/53              ALLOW
    CORPNET → ADNET Kerberos/88         ALLOW
    CORPNET → ADNET LDAP/389            ALLOW
    LAB-WIN-01 → DMZ SSH/22             ALLOW

    SERVERNET → SOCNET Wazuh/1514       ALLOW
    SERVERNET → CORPNET SMB/445         BLOCK
    SERVERNET → DMZ SSH/22              BLOCK

    ADNET → SOCNET Wazuh/1514           ALLOW
    ADNET → DMZ SSH/22                   BLOCK

    DMZ → SOCNET Wazuh/1514             ALLOW
    DMZ → CORPNET SMB/445               BLOCK
    DMZ → ADNET LDAP/389                BLOCK
    DMZ → Internet HTTPS/443            ALLOW

---

# Wazuh SIEM

Wazuh was introduced after the initial segmented network was operational.

Final active agents:

    ID 001 — LAB-WIN-01
    ID 002 — LAB-UBUNTU-01
    ID 003 — LAB-DC-01
    ID 004 — LAB-DMZ-01

All four agents were Active during final validation.

---

# Windows Monitoring

`LAB-WIN-01` provides layered endpoint telemetry.

Monitoring includes:

- Windows Security events
- Sysmon
- PowerShell Script Block Logging
- PowerShell Module Logging
- PowerShell transcription
- Process Creation auditing
- Special Logon auditing
- Microsoft Defender
- Windows Firewall Event 5157
- File Integrity Monitoring

---

# Linux Monitoring

`LAB-UBUNTU-01` provides:

- Linux system logs
- authentication logs
- auditd
- Wazuh agent telemetry

Validated custom detections include:

`Rule 100200`

Native Linux user creation.

`Rule 100201`

Native Linux user deletion.

---

# OPNsense Firewall Monitoring

Firewall logs are forwarded into Wazuh.

A validated firewall correlation detection uses:

`Rule 87702`

This allows blocked network activity to be investigated centrally.

---

# Suricata IDS

Suricata runs on:

`LAB-FW-01`

Primary monitored interface:

`em2`

corresponding to REDNET.

ET Open rules are used.

The final Suricata telemetry path is:

    Suricata
       |
       v
    OPNsense Remote Logging
       |
       | UDP/5514
       v
    LAB-SIEM-01
       |
       v
    rsyslog
       |
       v
    /var/log/suricata/eve.json
       |
       v
    Wazuh

Validated detection:

`Rule 86601`

for:

`GPL ATTACK_RESPONSE id check returned root`

---

# Active Directory

Active Directory was added after the initial four-zone architecture and core monitoring stack were stable.

Domain:

`cyberlab.internal`

NetBIOS:

`CYBERLAB`

Primary Domain Controller:

`LAB-DC-01`

---

# Active Directory Structure

Primary OUs include:

- Admins
- Users
- Groups
- Service Accounts
- Servers
- Workstations

Security groups include:

- GG_Employees
- GG_Helpdesk
- GG_IT_Admins
- GG_SOC_Analysts
- GG_Domain_Admins

---

# Identity Separation

A standard account exists for normal use:

`emmanuel.patrick@cyberlab.internal`

A separate privileged administrative account exists:

`emmanuel.ngari-adm@cyberlab.internal`

This separates normal user activity from Domain Admin operations.

---

# Group Policy

The workstation security baseline is:

`GPO-Workstation-Security-Baseline`

Linked to:

`CyberLab\Workstations`

It includes:

- Windows Firewall
- inbound block
- outbound allow
- Logon Success/Failure auditing
- Process Creation auditing
- Special Logon auditing
- command-line process capture
- PowerShell Script Block Logging
- PowerShell Module Logging

---

# Domain Controller Monitoring

`LAB-DC-01` is monitored by Wazuh and Sysmon.

Important observed Windows events include:

- `4672`
- `4719`
- `5136`

Sysmon version:

`15.22`

The Domain Controller configuration was tuned to focus on security-relevant executables such as:

- powershell
- cmd
- rundll32
- regsvr32
- mshta
- certutil
- schtasks
- nltest
- ntdsutil
- whoami
- net

---

# DMZ

The DMZ was the final major network expansion.

`LAB-DMZ-01`

runs:

- Ubuntu Server
- Nginx
- Wazuh Agent
- SSH

The server is isolated from protected internal networks.

---

# DMZ Security

Important controls include:

- DNS only through OPNsense
- NTP through OPNsense
- Wazuh access to SOCNET
- broad private-network deny
- HTTP/HTTPS outbound only
- REDNET HTTP exposure
- restricted CORPNET SSH administration

---

# DMZ SSH Hardening

SSH administration is allowed only from:

`LAB-WIN-01`

Authentication is key-only.

Final settings include:

    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no

This provides both:

- network-level source restriction
- host-level authentication hardening

---

# Nginx Monitoring

Wazuh monitors:

`/var/log/nginx/access.log`

and:

`/var/log/nginx/error.log`

A controlled 404 request generated:

`Rule 31101`

Description:

`Web server 400 error code.`

This validated application-layer monitoring.

---

# Suricata DMZ Validation

A controlled HTTP response containing:

    uid=0(root) gid=0(root) groups=0(root)

triggered:

`Suricata SID 2100498`

and then:

`Wazuh Rule 86601`

This confirmed network IDS visibility across the REDNET-to-DMZ path.

---

# Validated Detection Inventory

| Detection | Rule | Status |
|---|---|---|
| Linux user creation | `100200` | PASS |
| Linux user deletion | `100201` | PASS |
| OPNsense firewall correlation | `87702` | PASS |
| Suricata alert | `86601` | PASS |
| Nginx web error | `31101` | PASS |

---

# Final Six-Zone Validation

The final environment passed service-level validation.

| Source | Destination | Service | Expected | Result |
|---|---|---|---|---|
| REDNET | DMZ | HTTP/80 | Allow | PASS |
| REDNET | DMZ | SSH/22 | Block | PASS |
| REDNET | CORPNET | SMB/445 | Block | PASS |
| REDNET | ADNET | LDAP/389 | Block | PASS |
| CORPNET | ADNET | DNS/53 | Allow | PASS |
| CORPNET | ADNET | Kerberos/88 | Allow | PASS |
| CORPNET | ADNET | LDAP/389 | Allow | PASS |
| CORPNET | SOCNET | Wazuh/1514 | Allow | PASS |
| LAB-WIN-01 | DMZ | SSH/22 | Allow | PASS |
| SERVERNET | SOCNET | Wazuh/1514 | Allow | PASS |
| SERVERNET | CORPNET | SMB/445 | Block | PASS |
| SERVERNET | DMZ | SSH/22 | Block | PASS |
| ADNET | SOCNET | Wazuh/1514 | Allow | PASS |
| ADNET | DMZ | SSH/22 | Block | PASS |
| DMZ | SOCNET | Wazuh/1514 | Allow | PASS |
| DMZ | CORPNET | SMB/445 | Block | PASS |
| DMZ | ADNET | LDAP/389 | Block | PASS |
| DMZ | Internet | HTTPS/443 | Allow | PASS |

---

# SIEM Health

Final Wazuh services:

- Manager — Active
- Indexer — Active
- Dashboard — Active
- Filebeat — Active

Final manager version:

`v4.14.7`

Final observed storage:

    53G total
    25G used
    26G available
    49% used

---

# Change Management

Significant infrastructure changes followed a controlled workflow:

    Snapshot
        |
        v
    Configure
        |
        v
    Verify
        |
        v
    Test
        |
        v
    Review Logs
        |
        v
    Document

Snapshots were taken before major:

- network changes
- firewall changes
- Wazuh changes
- Suricata changes
- Active Directory changes
- DMZ changes

---

# Final Snapshot Baseline

After final validation, obsolete snapshots were removed.

Every VM now retains one snapshot:

`Final-Six-Zone-Validated-Baseline`

This represents the known-good infrastructure state before operational SOC and Purple Team exercises begin.

---

# Repository Structure

## `architecture/`

Architecture and VirtualBox design.

Key files include:

- `lab-architecture.md`
- `network-diagram.md`
- `network-diagram.png`
- `virtualbox-networking.md`

---

## `networking/`

Network addressing, connectivity, and troubleshooting.

Key files include:

- `addressing.md`
- `connectivity-tests.md`
- `troubleshooting.md`

---

## `firewall/`

OPNsense policy and validation.

Key files include:

- `security-zone.md`
- `policy-matrix.md`
- `firewall-rules.md`
- `opnsense-firewall.md`
- `validation-testing.md`

---

## `virtual-machines/`

System-specific documentation.

Key files include:

- `kali.md`
- `windows.md`
- `ubuntu.md`
- `siem.md`
- `domain-controller.md`
- `dmz.md`

---

## `blue-team/`

Windows and Wazuh monitoring documentation.

Key files include:

- `wazuh-siem.md`
- `wazuh-detection-engineering.md`
- `windows-sysmon.md`
- `windows-powershell-logging.md`
- `windows-advanced-auditing.md`

---

## `monitoring/`

Linux and network monitoring.

Key files include:

- `linux-endpoint-security-monitoring.md`
- `suricata-wazuh-integration.md`

---

## `docs/`

Project-wide documentation.

Key files include:

- `active-directory.md`
- `homelab-build-journal.md`

---

## `evidence/`

Validation evidence is organized into areas such as:

- networking
- firewall
- Windows
- Linux
- Wazuh
- attack detection

---

# Key Engineering Lessons

The project produced several important lessons.

## Test the Real Service

A failed ping does not necessarily mean a required TCP service is unavailable.

## Rule Order Matters

Specific firewall exceptions must appear before broader deny rules.

## Save and Apply Are Different

OPNsense changes must be applied before testing.

## Telemetry Comes Before Detection

Confirm:

- raw event
- ingestion
- decoding

before writing detection logic.

## Historical Errors Are Not Current State

Always correlate timestamps.

## Avoid Unnecessary Changes

A stable validated system should not be modified simply for cosmetic cleanup.

## Prefer Maintainable Designs

Built-in JSON Suricata support was preferred over an unnecessary custom wrapper decoder.

## Protect Identity Infrastructure

The Domain Controller is isolated and is not used as a routine jump host.

## Validate Both Allow and Deny Paths

A firewall is not validated by testing only successful connections.

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Windows administration
- Linux administration
- TCP/IP networking
- VirtualBox
- OPNsense
- firewall policy
- network segmentation
- Wazuh
- SIEM operations
- Sysmon
- PowerShell logging
- Windows auditing
- Active Directory
- Group Policy
- auditd
- Suricata
- rsyslog
- Nginx
- SSH hardening
- detection engineering
- log analysis
- troubleshooting
- Purple Team validation

---

# Operational Phase

The baseline infrastructure is complete.

Future work should focus on realistic SOC and Purple Team scenarios rather than continued infrastructure expansion.

Planned scenario categories include:

- reconnaissance
- credential attacks
- suspicious PowerShell
- privilege escalation
- persistence
- lateral movement
- Active Directory abuse
- web attacks
- network IDS detections
- alert triage
- threat hunting
- incident response
- detection-gap analysis

The goal of each future exercise should be:

    Attack
       |
       v
    Telemetry
       |
       v
    Detection
       |
       v
    Investigation
       |
       v
    Improvement
       |
       v
    Retest

---

# Project Status

**Infrastructure Build: COMPLETE**

**Six-Zone Segmentation: VALIDATED**

**Endpoint Monitoring: ACTIVE**

**Network IDS: ACTIVE**

**Active Directory: ACTIVE**

**DMZ Monitoring: ACTIVE**

**Wazuh Agents: 4/4 ACTIVE**

**Final Baseline: CREATED**

**Next Phase: SOC / PURPLE TEAM OPERATIONS**
