# Cybersecurity Home Lab Architecture

## Overview

This project is a segmented enterprise-style cybersecurity lab designed to support practical Blue Team, Red Team, SOC and Purple Team exercises.

The environment is built in Oracle VirtualBox and uses OPNsense as the central firewall and routing platform between multiple isolated security zones.

Rather than operating as a flat home network, the lab separates systems according to function, trust level and security sensitivity.

The final architecture contains six internal security zones:

- REDNET — offensive security
- CORPNET — corporate workstations
- SERVERNET — internal servers
- SOCNET — SIEM and monitoring
- ADNET — identity infrastructure
- DMZ — externally exposed services

The environment is designed to simulate the architecture and security controls commonly found in small and medium enterprise networks.

---

# Architecture Goals

The lab was designed around several primary objectives:

1. Build a realistic segmented enterprise network.
2. Apply least-privilege firewall policies between security zones.
3. Deploy centralized endpoint and network security monitoring.
4. Build and secure an Active Directory environment.
5. Create a controlled offensive network for attack simulation.
6. Deploy a monitored DMZ web server.
7. Validate defensive controls through controlled adversarial activity.
8. Practice detection engineering and incident investigation.
9. Develop practical SOC and Purple Team skills.
10. Maintain professional technical documentation suitable for a cybersecurity portfolio.

---

# High-Level Architecture

The environment consists of seven primary virtual machines.

| System | Address | Zone | Primary Role |
|---|---|---|---|
| `LAB-FW-01` | Multiple gateway interfaces | All Zones | OPNsense firewall, routing and IDS |
| `LAB-KALI-01` | `10.10.10.10` | REDNET | Offensive security workstation |
| `LAB-WIN-01` | `10.10.20.20` | CORPNET | Windows domain workstation |
| `LAB-UBUNTU-01` | `10.10.30.30` | SERVERNET | Internal Linux server |
| `LAB-SIEM-01` | `10.10.40.10` | SOCNET | Wazuh SIEM infrastructure |
| `LAB-DC-01` | `10.10.50.10` | ADNET | Active Directory Domain Controller |
| `LAB-DMZ-01` | `10.10.60.10` | DMZ | Nginx web server |

---

# Network Zones

## REDNET

Network:

`10.10.10.0/24`

Gateway:

`10.10.10.1`

Primary host:

`LAB-KALI-01 — 10.10.10.10`

REDNET represents an adversarial or untrusted network.

Kali Linux is used for controlled attack simulation, network testing and Purple Team validation.

REDNET is intentionally isolated from sensitive internal systems.

Validated controls include:

- REDNET → DMZ HTTP/80 allowed
- REDNET → DMZ SSH/22 blocked
- REDNET → CORPNET SMB/445 blocked
- REDNET → ADNET LDAP/389 blocked

Traffic originating from REDNET is monitored by Suricata on OPNsense.

---

## CORPNET

Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

Primary host:

`LAB-WIN-01 — 10.10.20.20`

CORPNET represents the enterprise workstation environment.

`LAB-WIN-01` is joined to the Active Directory domain:

`cyberlab.internal`

The workstation receives Group Policy security controls and produces security telemetry through:

- Windows Security auditing
- Sysmon
- PowerShell Script Block Logging
- PowerShell Module Logging
- Windows Defender
- Windows Firewall auditing
- Wazuh agent monitoring

CORPNET is permitted to communicate with Active Directory only through required services.

Validated examples include:

- DNS TCP/53
- Kerberos TCP/88
- LDAP TCP/389

`LAB-WIN-01` is also the approved administrative workstation for SSH management of the DMZ server.

---

## SERVERNET

Network:

`10.10.30.0/24`

Gateway:

`10.10.30.1`

Primary host:

`LAB-UBUNTU-01 — 10.10.30.30`

SERVERNET represents protected internal Linux server infrastructure.

The Ubuntu server provides a controlled environment for Linux security monitoring, auditing and detection engineering.

Security monitoring includes:

- Wazuh agent
- Linux system logs
- auditd telemetry
- account creation and deletion monitoring
- custom Wazuh detection rules

Validated segmentation includes:

- SERVERNET → SOCNET Wazuh TCP/1514 allowed
- SERVERNET → CORPNET SMB/445 blocked
- SERVERNET → DMZ SSH/22 blocked

---

## SOCNET

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

Primary host:

`LAB-SIEM-01 — 10.10.40.10`

SOCNET contains the centralized monitoring infrastructure.

`LAB-SIEM-01` hosts the Wazuh security monitoring stack:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

The environment currently uses Wazuh version:

`v4.14.7`

Monitored agents include:

| Agent ID | Host |
|---|---|
| `001` | `LAB-WIN-01` |
| `002` | `LAB-UBUNTU-01` |
| `003` | `LAB-DC-01` |
| `004` | `LAB-DMZ-01` |

All four endpoints were confirmed Active during final infrastructure validation.

SOCNET also receives Suricata security telemetry originating from OPNsense.

---

## ADNET

Network:

`10.10.50.0/24`

Gateway:

`10.10.50.1`

Primary host:

`LAB-DC-01 — 10.10.50.10`

ADNET isolates critical identity and authentication infrastructure.

`LAB-DC-01` runs Windows Server 2025 and provides:

- Active Directory Domain Services
- DNS
- Kerberos authentication
- LDAP directory services
- Group Policy
- centralized user and computer management

Domain:

`cyberlab.internal`

The Active Directory structure includes dedicated organizational units for:

- Admins
- Users
- Groups
- Service Accounts
- Servers
- Workstations

Security groups include:

- `GG_Employees`
- `GG_Helpdesk`
- `GG_IT_Admins`
- `GG_SOC_Analysts`
- `GG_Domain_Admins`

The environment separates normal user accounts from privileged administrative accounts.

The Domain Controller is monitored using:

- Windows Security auditing
- Sysmon
- PowerShell logging
- Wazuh

Final targeted Domain Controller validation passed:

- Advertising
- SYSVOL
- NetLogons
- Services
- DNS

---

## DMZ

Network:

`10.10.60.0/24`

Gateway:

`10.10.60.1`

Primary host:

`LAB-DMZ-01 — 10.10.60.10`

The DMZ contains services intended to interact with less-trusted networks while remaining isolated from protected internal infrastructure.

`LAB-DMZ-01` runs Ubuntu Server and hosts Nginx.

Primary exposed service:

`TCP/80 — HTTP`

Administrative access:

`TCP/22 — SSH`

SSH is restricted at the firewall to the approved administrative workstation:

`LAB-WIN-01 (10.10.20.20)`

The DMZ server is hardened with:

- SSH public-key authentication
- password authentication disabled
- keyboard-interactive authentication disabled
- direct root SSH login disabled
- restricted firewall management path
- centralized Wazuh monitoring
- Nginx access-log monitoring
- Nginx error-log monitoring

Unauthorized DMZ access to protected private networks is blocked.

Outbound HTTP and HTTPS access is permitted for required package management and updates.

---

# OPNsense Firewall Architecture

`LAB-FW-01` is the central security gateway for the environment.

OPNsense provides:

- inter-zone routing
- firewall policy enforcement
- network segmentation
- DNS services for selected zones
- NTP services
- outbound Internet access
- firewall logging
- Suricata IDS

Each security zone has a dedicated OPNsense interface.

| Zone | Gateway |
|---|---|
| REDNET | `10.10.10.1` |
| CORPNET | `10.10.20.1` |
| SERVERNET | `10.10.30.1` |
| SOCNET | `10.10.40.1` |
| ADNET | `10.10.50.1` |
| DMZ | `10.10.60.1` |

Firewall policy follows a least-privilege model.

Broad communication between internal networks is not automatically permitted.

Explicit rules are created only for required services.

---

# Firewall Design Philosophy

The firewall architecture follows four core principles.

## Default Deny

Traffic that does not match an approved policy is denied.

## Explicit Allow Rules

Only required communication paths are permitted.

Examples include:

`CORPNET → ADNET`

for DNS, Kerberos and LDAP.

`Endpoints → SOCNET`

for Wazuh telemetry.

`REDNET → DMZ`

for HTTP testing.

`LAB-WIN-01 → LAB-DMZ-01`

for administrative SSH.

## Network Isolation

Sensitive systems are placed into dedicated zones rather than sharing a flat network.

## Logged Security Boundaries

Important deny rules are logged so unauthorized communication attempts can be investigated.

---

# Active Directory Architecture

The Active Directory environment was designed to resemble a small enterprise identity architecture.

Domain:

`cyberlab.internal`

Domain Controller:

`LAB-DC-01`

The Active Directory design includes:

- structured Organizational Units
- role-based security groups
- standard user accounts
- privileged administrative accounts
- domain-joined workstation
- Group Policy security baselines

The Windows workstation resides in:

`CyberLab\Workstations`

A workstation security baseline GPO is linked specifically to the workstation OU rather than modifying default domain policies.

---

# Workstation Security Baseline

The workstation Group Policy baseline includes:

- Windows Firewall enabled
- inbound traffic blocked by default
- outbound traffic allowed
- Logon auditing
- Process Creation auditing
- Special Logon auditing
- command-line process auditing
- PowerShell Script Block Logging
- PowerShell Module Logging

This configuration improves visibility for both endpoint investigations and detection engineering.

---

# Endpoint Monitoring Architecture

The environment uses Wazuh agents on the primary monitored hosts.

Telemetry flow:

`LAB-WIN-01 → LAB-SIEM-01`

`LAB-UBUNTU-01 → LAB-SIEM-01`

`LAB-DC-01 → LAB-SIEM-01`

`LAB-DMZ-01 → LAB-SIEM-01`

Endpoint monitoring provides visibility into:

- authentication
- process execution
- privilege changes
- PowerShell activity
- account changes
- filesystem changes
- Windows Defender events
- firewall events
- Linux audit events
- web-server access activity

---

# Windows Telemetry

The Windows workstation provides multiple telemetry sources.

## Sysmon

Sysmon provides detailed process and system activity.

## PowerShell Logging

Enabled telemetry includes:

- Script Block Logging
- Module Logging

## Windows Security Auditing

Advanced Windows auditing provides events related to:

- authentication
- privilege use
- process creation
- account activity
- security policy changes

## Windows Defender

Defender security events are available to Wazuh for detection and investigation.

## Windows Firewall

Windows Filtering Platform events are monitored, including blocked network activity.

---

# Domain Controller Monitoring

`LAB-DC-01` generates additional security telemetry relevant to identity attacks.

Examples include:

- Event ID 4672 — Special privileges assigned
- Event ID 4719 — System audit policy changed
- Event ID 5136 — Directory service object modified

Sysmon is also installed on the Domain Controller.

The Sysmon configuration was tuned to prioritize security-relevant processes rather than logging all process creation events.

Examples of monitored executables include:

- `powershell.exe`
- `pwsh.exe`
- `cmd.exe`
- `rundll32.exe`
- `regsvr32.exe`
- `mshta.exe`
- `certutil.exe`
- `bitsadmin.exe`
- `wscript.exe`
- `cscript.exe`
- `schtasks.exe`
- `nltest.exe`
- `ntdsutil.exe`
- `whoami.exe`
- `net.exe`
- `net1.exe`

This reduces unnecessary telemetry while preserving high-value security visibility.

---

# Linux Monitoring

`LAB-UBUNTU-01` provides Linux detection telemetry.

Monitoring includes:

- Linux system events
- auditd
- authentication
- account management
- Wazuh file integrity monitoring

Custom Wazuh rules were developed and validated for Linux account activity.

Examples:

- Rule `100200` — user creation
- Rule `100201` — user deletion

This provides experience with Linux detection engineering in addition to Windows monitoring.

---

# Suricata IDS Architecture

Suricata is deployed on OPNsense.

Primary monitored interface:

`em2`

This interface provides visibility into REDNET traffic.

Suricata uses ET Open and other IDS signatures to identify suspicious network activity.

Events are exported using EVE JSON.

Suricata event flow:

`OPNsense Suricata`

→ remote syslog

→ `LAB-SIEM-01 UDP/5514`

→ rsyslog

→ `/var/log/suricata/eve.json`

→ Wazuh logcollector

→ Wazuh detection rules

A dedicated rsyslog listener receives Suricata events on:

`UDP/5514`

The resulting JSON events are written to:

`/var/log/suricata/eve.json`

Wazuh monitors this file using JSON log collection.

---

# Suricata Detection Validation

A controlled Purple Team test was used to validate IDS visibility between REDNET and the DMZ.

`LAB-KALI-01` requested an HTTP resource hosted by `LAB-DMZ-01`.

The controlled web response contained:

`uid=0(root) gid=0(root) groups=0(root)`

Suricata detected the response using:

- Signature ID: `2100498`
- Signature: `GPL ATTACK_RESPONSE id check returned root`
- Category: `Potentially Bad Traffic`

Observed traffic:

`LAB-DMZ-01 10.10.60.10:80 → LAB-KALI-01 10.10.10.10`

Suricata generated an EVE JSON alert.

Wazuh subsequently generated:

- Rule ID: `86601`
- Description: `Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

This validated the complete network IDS monitoring path.

---

# DMZ Web Monitoring

Nginx logs are collected locally by the Wazuh agent on `LAB-DMZ-01`.

Monitored files:

`/var/log/nginx/access.log`

`/var/log/nginx/error.log`

A controlled HTTP request originating from REDNET generated a web-server security event.

Example:

`GET /wazuh-nginx-verify-01 HTTP/1.1`

Source:

`10.10.10.10`

Result:

`404`

Wazuh generated:

- Rule ID: `31101`
- Description: `Web server 400 error code.`
- Decoder: `web-accesslog`

This validated application-layer monitoring in addition to network IDS monitoring.

---

# Centralized Detection Architecture

The lab combines multiple security data sources.

## Endpoint Telemetry

Generated by:

- Windows workstation
- Linux server
- Domain Controller
- DMZ web server

## Network Telemetry

Generated by:

- OPNsense firewall
- Suricata IDS

## Identity Telemetry

Generated by:

- Active Directory
- Windows Security logs
- Group Policy
- Domain Controller Sysmon

## Application Telemetry

Generated by:

- Nginx

All relevant telemetry is centralized within Wazuh.

This enables correlation between endpoint, identity, network and application activity.

---

# Purple Team Workflow

The environment supports a complete Purple Team cycle.

Attack simulation begins from:

`LAB-KALI-01`

Traffic crosses:

`LAB-FW-01`

Targets may include:

- Windows workstation
- Linux server
- Domain infrastructure
- DMZ services

Defensive telemetry is collected by:

- endpoint agents
- Windows auditing
- Sysmon
- Linux auditd
- OPNsense firewall logs
- Suricata
- Nginx logs

Events are analyzed in:

`LAB-SIEM-01`

Detection gaps can then be addressed through:

- firewall policy improvements
- Wazuh custom rules
- Sysmon tuning
- audit-policy changes
- IDS tuning
- endpoint hardening

The same scenario can then be repeated to verify improved detection coverage.

---

# Final Network Validation

A full six-zone validation was completed after infrastructure deployment.

## REDNET

Validated:

- HTTP to DMZ allowed
- SSH to DMZ blocked
- SMB to CORPNET blocked
- LDAP to ADNET blocked

## CORPNET

Validated:

- DNS to ADNET allowed
- Kerberos to ADNET allowed
- LDAP to ADNET allowed
- Wazuh telemetry to SOCNET allowed
- administrative SSH to DMZ allowed

## SERVERNET

Validated:

- Wazuh telemetry to SOCNET allowed
- SMB to CORPNET blocked
- SSH to DMZ blocked

## ADNET

Validated:

- Wazuh telemetry to SOCNET allowed
- SSH to DMZ blocked
- Domain Controller health checks passed

## DMZ

Validated:

- Wazuh telemetry to SOCNET allowed
- access to protected CORPNET services blocked
- access to ADNET LDAP blocked
- outbound HTTPS allowed
- Nginx monitoring operational
- key-only administrative SSH operational

## SOCNET

Validated:

- Wazuh Manager active
- Wazuh Indexer active
- Wazuh Dashboard active
- Filebeat active
- all monitored agents Active
- Suricata alerts received
- DMZ web alerts received

---

# VirtualBox Architecture

Each security zone is implemented using a dedicated VirtualBox Internal Network.

This provides Layer 2 separation between zones while allowing OPNsense to act as the central Layer 3 security gateway.

`LAB-FW-01` contains multiple virtual network interfaces mapped to the individual security zones.

The firewall also maintains an upstream Internet-facing interface for package updates and controlled outbound Internet access.

This architecture allows the lab to simulate enterprise segmentation without requiring physical networking equipment.

---

# Snapshot Strategy

Snapshots were used throughout the build process before significant infrastructure, networking and security changes.

Examples included checkpoints before:

- Active Directory deployment
- Group Policy changes
- Domain joins
- firewall changes
- Suricata integration
- DMZ deployment
- Wazuh configuration
- SSH hardening

After completing the environment and validating all six zones, obsolete build snapshots were removed.

Every VM now retains a single known-good snapshot:

`Final-Six-Zone-Validated-Baseline`

This snapshot represents the final validated infrastructure state before operational SOC and Purple Team exercises begin.

---

# Security Design Principles

The final architecture demonstrates several security concepts commonly used in enterprise environments:

- defense in depth
- network segmentation
- least privilege
- centralized logging
- endpoint detection
- network intrusion detection
- identity security
- restricted administrative access
- default-deny firewall policy
- attack-surface reduction
- detection engineering
- controlled adversarial testing
- security validation

---

# Current Project State

The infrastructure build phase is complete.

The environment currently contains:

- segmented six-zone architecture
- OPNsense firewall
- Suricata IDS
- Active Directory
- domain-joined Windows workstation
- internal Linux server
- centralized Wazuh SIEM
- hardened DMZ web server
- Windows security telemetry
- Linux auditing
- Domain Controller monitoring
- firewall logging
- network IDS telemetry
- web-server telemetry
- custom Wazuh detections
- validated network security policies

The next project phase focuses on operating the environment rather than continuing to add infrastructure.

Planned activities include:

- realistic SOC investigations
- Purple Team attack simulations
- Active Directory attack scenarios
- endpoint compromise investigations
- web attack simulations
- detection engineering
- incident response
- threat hunting
- security-control validation
- incident reporting

The goal is to use the environment as an operational cybersecurity training platform rather than simply a collection of security tools.
