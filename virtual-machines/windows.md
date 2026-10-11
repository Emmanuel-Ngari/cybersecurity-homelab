# LAB-WIN-01 — Windows Corporate Workstation

## Overview

This document records the complete lifecycle of the Windows workstation used throughout the Cybersecurity Home Lab, from the earliest flat-network phase through its final role as `LAB-WIN-01` in the six-zone architecture.

The Windows system has been part of the project since the beginning.

Its role evolved from a basic Windows endpoint used for connectivity and attacker/defender exercises into a monitored, domain-joined corporate workstation with:

- Wazuh agent monitoring
- Sysmon
- PowerShell logging
- advanced Windows auditing
- Defender telemetry
- Windows Firewall monitoring
- Active Directory integration
- Group Policy
- restricted administrative access to the DMZ

Final host:

`LAB-WIN-01`

Final address:

`10.10.20.20`

Final zone:

`CORPNET`

Final role:

Corporate Windows workstation and approved administrative endpoint

---

# Phase 1 — Original Windows Lab System

The earliest version of the lab used a Windows endpoint on:

`192.168.56.0/24`

Original Windows address:

`192.168.56.20`

The other primary systems included:

- Kali — `192.168.56.10`
- Ubuntu — `192.168.56.30`

At this stage the Windows system was mainly used for:

- basic host connectivity
- Windows administration
- testing connectivity to Kali and Ubuntu
- firewall behaviour
- endpoint-security exercises
- early attacker/defender interaction

The lab did not yet contain:

- OPNsense segmentation
- SOCNET
- Active Directory
- DMZ
- centralized enterprise-style logging

---

# Early Windows Networking

During the earlier lab phase, Windows used two VirtualBox adapters:

- NAT
- Internal Network

The interfaces were later renamed for clarity:

`Ethernet` → `NAT`

`Ethernet 2` → `LAB-NET`

The internal interface used:

`192.168.56.20/24`

The NAT adapter used:

- gateway `10.0.2.2`
- DNS `10.0.2.3`

This provided:

- Internet access through VirtualBox NAT
- local lab communication through the internal adapter

---

# Early Windows-to-Kali Validation

Connectivity between Windows and Kali was tested using:

`192.168.56.20`

to:

`192.168.56.10`

Ping succeeded after the Windows firewall rules were reviewed.

This phase included examination of:

- Windows network profiles
- File and Printer Sharing rules
- ICMP behaviour
- Public network profile restrictions

The Windows network profile for the internal network initially appeared as:

`Unidentified Network`

with profile:

`Public`

This became useful early experience in understanding how Windows firewall behaviour changes according to network profile.

---

# Windows Version Evolution

During the earlier build, the system showed different version references at different points.

PowerShell initially reported an older Windows 10 Enterprise Evaluation identity, while the rebuilt system ultimately ran:

`Windows 11 Enterprise Evaluation`

Final observed version:

`25H2`

OS build:

`26200.6584`

The rebuilt Windows 11 Enterprise VM became the authoritative endpoint.

Historical Windows version references in earlier project material represent earlier stages and should not be interpreted as the final system state.

---

# VirtualBox Graphics Troubleshooting

During the Windows rebuild, display issues occurred.

A significant issue involved:

`3D Acceleration`

When enabled, it caused visual or login-screen artifacts.

The configuration was adjusted and the issue was resolved.

Full-screen support was later restored successfully.

This reinforced an important lab-building lesson:

Not every problem in a security VM is caused by the guest operating system itself; VirtualBox hardware settings can create misleading symptoms.

---

# Transition to Segmented Architecture

When OPNsense was introduced, the Windows workstation moved from the original flat network into:

`CORPNET`

Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

Final Windows address:

`10.10.20.20`

The host became:

`LAB-WIN-01`

---

# CORPNET Role

CORPNET represents the corporate-user workstation environment.

`LAB-WIN-01` became the primary endpoint in this zone.

The workstation was designed to behave like an enterprise employee or administrator workstation rather than an unrestricted lab VM.

Its final role includes:

- corporate endpoint
- Active Directory domain member
- Wazuh-monitored endpoint
- security telemetry source
- approved DMZ administration source
- Purple Team target endpoint

---

# OPNsense Integration

Once the Windows workstation moved into CORPNET, its network traffic became subject to OPNsense firewall policy.

This created a much more realistic security model.

Instead of communicating freely with every lab system, Windows was given access only where required.

Important permitted paths eventually included:

- CORPNET → Active Directory services
- CORPNET → Wazuh
- administrative access to OPNsense where approved
- restricted SSH access to the DMZ
- approved Internet access

Unauthorized paths remained restricted.

---

# Wazuh Endpoint Monitoring

`LAB-WIN-01` became the first major Windows endpoint integrated with Wazuh.

The Wazuh agent forwards endpoint telemetry to:

`LAB-SIEM-01`

Address:

`10.10.40.10`

Primary communication:

`TCP/1514`

Agent enrollment may use:

`TCP/1515`

Final Wazuh agent identity:

`ID 001 — LAB-WIN-01`

Final state:

`Active`

---

# Windows Security Monitoring Evolution

Monitoring was expanded progressively rather than being enabled all at once.

The Windows telemetry stack eventually included:

- Windows Security event logs
- Sysmon
- PowerShell logging
- PowerShell transcription
- File Integrity Monitoring
- advanced Windows auditing
- Microsoft Defender monitoring
- Windows Firewall events
- custom Wazuh detection rules

This transformed `LAB-WIN-01` into a realistic monitored enterprise endpoint.

---

# Sysmon Deployment

Sysmon was installed to provide richer endpoint visibility than standard Windows logs alone.

The deployment focused on events useful for:

- process creation
- suspicious command execution
- network-related activity
- parent/child process relationships
- attacker behaviour
- detection engineering

Sysmon logs were ingested into Wazuh.

This allowed process-level activity to be correlated with other security telemetry.

---

# PowerShell Logging

PowerShell visibility was deliberately strengthened.

Monitoring included:

- Script Block Logging
- Module Logging
- PowerShell transcription

This was important because PowerShell is commonly used both by administrators and attackers.

The goal was to ensure that future PowerShell-based attack simulations could be investigated through the SIEM.

---

# PowerShell Transcription

PowerShell transcription was enabled to provide additional command and session visibility.

This helped create a more complete record of administrative or offensive PowerShell activity.

The output could then be monitored or used as investigation evidence during Purple Team scenarios.

---

# Advanced Windows Auditing

Advanced audit policy was enabled to increase endpoint visibility.

Important categories included events related to:

- logon activity
- process creation
- privilege use
- account activity
- security-policy changes

This provided richer telemetry for Wazuh and future SOC investigations.

---

# Process Creation Auditing

Process-creation auditing was configured to include command-line information.

This significantly improved detection value because it allowed analysts to understand not only which executable launched but also how it was invoked.

This later aligned with the Active Directory Group Policy baseline applied to the workstation.

---

# Microsoft Defender Monitoring

Microsoft Defender was included in the endpoint-monitoring strategy.

The objective was to provide visibility into:

- malware detections
- security actions
- Defender operational events

This complemented Sysmon and standard Windows auditing.

---

# Windows Firewall Monitoring

Windows Firewall telemetry was also integrated into the monitoring design.

One important event used was:

`Event ID 5157`

This records connections blocked by the Windows Filtering Platform.

Including this data helped distinguish:

- OPNsense network blocks
- host firewall blocks

This improved troubleshooting and detection accuracy.

---

# File Integrity Monitoring

File Integrity Monitoring was configured through Wazuh.

This provided visibility into selected file and configuration changes.

The goal was to support future detection scenarios involving:

- unauthorized file modification
- persistence
- configuration tampering
- attacker-created artifacts

---

# Windows Detection Engineering

The workstation was used for custom Wazuh detection development.

Rather than relying only on default rules, the lab introduced custom detection logic for important endpoint behaviour.

This marked the transition from simply collecting logs to actively engineering detections.

---

# Windows as a Blue Team Endpoint

By this stage, `LAB-WIN-01` had become an important Blue Team system.

Its telemetry could be used to investigate:

- command execution
- privilege use
- suspicious processes
- PowerShell
- firewall blocks
- endpoint changes
- authentication events
- Defender alerts

This provided the foundation for later Purple Team operations.

---

# Active Directory Introduction

After the initial four-zone architecture was stable, Active Directory was added.

A dedicated AD network was created:

`ADNET — 10.10.50.0/24`

Domain Controller:

`LAB-DC-01 — 10.10.50.10`

Domain:

`cyberlab.internal`

`LAB-WIN-01` became the primary domain-joined workstation.

---

# Active Directory Network Requirements

The workstation required access to several Domain Controller services.

Important services included:

TCP:

- `53` DNS
- `88` Kerberos
- `135` RPC Endpoint Mapper
- `389` LDAP
- `445` SMB
- `464` Kerberos password operations
- `3268` Global Catalog
- `49152:65535` Dynamic RPC

UDP:

- `53` DNS
- `88` Kerberos
- `123` NTP
- `389` LDAP
- `464` Kerberos password operations

These services supported:

- domain join
- authentication
- Group Policy
- SYSVOL
- LDAP queries
- Kerberos
- RPC-based domain functions

---

# Active Directory Connectivity Validation

Before and after domain integration, service-level connectivity was tested.

## DNS

    Test-NetConnection 10.10.50.10 -Port 53

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

## Kerberos

    Test-NetConnection 10.10.50.10 -Port 88

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

## LDAP

    Test-NetConnection 10.10.50.10 -Port 389

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

# Domain Join

`LAB-WIN-01` was successfully joined to:

`cyberlab.internal`

This transformed the workstation from a standalone Windows endpoint into an enterprise-style domain member.

Domain integration enabled:

- centralized identity
- user logon
- Group Policy
- security baselines
- centralized administrative control

---

# Active Directory Users

The workstation supports use of domain identities including:

`emmanuel.patrick@cyberlab.internal`

Normal user role.

Membership includes:

- Domain Users
- GG_Employees
- GG_SOC_Analysts

A separate privileged account was created:

`emmanuel.ngari-adm@cyberlab.internal`

This supports separation between:

- normal user activity
- privileged administration

---

# Group Policy

A workstation security baseline was created:

`GPO-Workstation-Security-Baseline`

The GPO is linked only to:

`CyberLab\Workstations`

This prevents workstation-specific settings from being applied indiscriminately to servers or the Domain Controller.

---

# Windows Firewall Baseline

The Group Policy baseline enforces Windows Firewall behaviour.

Configured direction:

    Inbound: Block
    Outbound: Allow

This provides host-level protection in addition to OPNsense segmentation.

---

# Audit Policy Baseline

The workstation GPO enables important auditing including:

- Logon — Success and Failure
- Process Creation — Success
- Special Logon — Success

Command-line information is included in process-creation events.

This directly supports endpoint detection and SOC investigation.

---

# PowerShell GPO Settings

The workstation security baseline enables:

- PowerShell Script Block Logging
- PowerShell Module Logging

Module logging includes:

`*`

This ensures broad PowerShell module visibility.

---

# Group Policy Validation

After domain join, Group Policy was applied and verified on `LAB-WIN-01`.

This confirmed the workstation was correctly receiving centralized security configuration.

The Windows endpoint was therefore no longer dependent solely on local settings.

---

# CORPNET to SOCNET

The workstation must send telemetry to Wazuh.

Validation:

    Test-NetConnection 10.10.40.10 -Port 1514

Result:

    TcpTestSucceeded : True

Status:

**PASS**

This confirmed the monitoring path remained healthy after Active Directory integration.

---

# DMZ Introduction

When the DMZ was introduced, `LAB-WIN-01` gained an additional role.

It became the approved administrative workstation for:

`LAB-DMZ-01`

DMZ address:

`10.10.60.10`

---

# Initial DMZ SSH Test

Before a firewall exception existed, the Windows workstation attempted:

`TCP/22`

to:

`10.10.60.10`

The connection timed out.

This confirmed that CORPNET did not have unrestricted access to the DMZ.

---

# ADMIN_WORKSTATION Alias

The firewall uses:

`ADMIN_WORKSTATION`

Address:

`10.10.20.20`

This alias represents:

`LAB-WIN-01`

Using a specific host alias avoided allowing all of CORPNET to administer the DMZ.

---

# Restricted DMZ SSH Rule

The following path was explicitly allowed:

    ADMIN_WORKSTATION
          |
          | TCP/22
          v
    LAB-DMZ-01

The rule permits only the approved Windows workstation to manage the DMZ server.

After the rule was applied:

SSH succeeded.

Status:

**PASS**

---

# DMZ SSH Key Authentication

Administrative access was later hardened.

An Ed25519 SSH key was generated on:

`LAB-WIN-01`

for the Windows user performing DMZ administration.

The public key was installed for the Ubuntu user:

`emmanuel`

on:

`LAB-DMZ-01`

---

# Key-Only SSH Validation

Final DMZ SSH policy included:

    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no

Key-based SSH from `LAB-WIN-01` succeeded.

Password-only authentication was tested and returned:

    Permission denied (publickey).

This created two layers of control:

    OPNsense Source Restriction
          +
    SSH Public-Key Authentication

---

# Why LAB-WIN-01 Is the Administrative Source

The Windows workstation was deliberately selected instead of the Domain Controller.

Using the DC as a general-purpose management or jump host would weaken the architecture by mixing:

- identity services
- privileged infrastructure
- routine remote administration

Using `LAB-WIN-01` as the approved source better reflects enterprise separation of roles.

---

# Final CORPNET Segmentation

Final validated Windows-related paths include:

## CORPNET → ADNET DNS

Port:

`TCP/53`

Result:

**ALLOW — PASS**

---

## CORPNET → ADNET Kerberos

Port:

`TCP/88`

Result:

**ALLOW — PASS**

---

## CORPNET → ADNET LDAP

Port:

`TCP/389`

Result:

**ALLOW — PASS**

---

## CORPNET → SOCNET Wazuh

Port:

`TCP/1514`

Result:

**ALLOW — PASS**

---

## LAB-WIN-01 → DMZ SSH

Port:

`TCP/22`

Result:

**ALLOW — PASS**

---

# Windows Security Architecture

The final workstation is protected and monitored at multiple layers.

    User Activity
        |
        v
    Windows 11 Enterprise
        |
        +--> Windows Security Logs
        |
        +--> Sysmon
        |
        +--> PowerShell Logging
        |
        +--> Defender
        |
        +--> Windows Firewall
        |
        +--> File Integrity Monitoring
        |
        v
    Wazuh Agent
        |
        v
    LAB-SIEM-01

Network traffic is additionally controlled by:

`LAB-FW-01`

This provides endpoint plus network defense in depth.

---

# Windows Role in Purple Team Exercises

`LAB-WIN-01` is designed to support future scenarios such as:

- suspicious PowerShell execution
- malicious process creation
- credential-related activity
- Windows Firewall events
- Defender detections
- lateral movement
- domain-user activity
- privilege escalation
- persistence
- command-and-control simulation
- detection engineering

The infrastructure is already in place to observe these actions.

---

# Snapshot History

Snapshots were taken before significant Windows configuration changes throughout the project.

These protected the environment during:

- network rebuild
- Wazuh deployment
- Sysmon installation
- PowerShell logging
- advanced auditing
- Active Directory domain join
- Group Policy deployment
- DMZ administration configuration

After final six-zone validation, temporary snapshots were removed.

The remaining snapshot is:

`Final-Six-Zone-Validated-Baseline`

Snapshot UUID:

`36d9a7bc-4556-4f14-9887-959d4a46e65c`

---

# Historical vs Current Windows State

This document intentionally preserves the full lifecycle of the Windows endpoint.

Historical states include:

- original `192.168.56.20`
- NAT plus LAB-NET adapters
- standalone Windows operation
- early firewall and ICMP testing
- pre-Wazuh endpoint
- pre-domain workstation

The authoritative current state is:

    Hostname: LAB-WIN-01
    OS: Windows 11 Enterprise Evaluation
    Version: 25H2
    IP: 10.10.20.20
    Zone: CORPNET
    Gateway: 10.10.20.1
    Domain: cyberlab.internal
    Wazuh Agent: Active
    Role: Corporate Workstation / Administrative Endpoint

The earlier stages remain documented because they form part of the engineering history of the lab.

---

# Final Validation Summary

| Validation | Result |
|---|---|
| CORPNET → ADNET DNS/53 | PASS |
| CORPNET → ADNET Kerberos/88 | PASS |
| CORPNET → ADNET LDAP/389 | PASS |
| CORPNET → SOCNET Wazuh/1514 | PASS |
| LAB-WIN-01 → DMZ SSH/22 | PASS |
| Domain join | PASS |
| Group Policy application | PASS |
| Wazuh agent active | PASS |
| Sysmon monitoring | PASS |
| PowerShell logging | PASS |
| Advanced audit policy | PASS |
| Defender monitoring | PASS |
| Windows Firewall monitoring | PASS |
| DMZ public-key SSH | PASS |
| DMZ password-only SSH blocked | PASS |

---

# Final Role

`LAB-WIN-01` now serves as the primary corporate endpoint in the Cybersecurity Home Lab.

It combines:

- enterprise workstation behaviour
- centralized identity
- endpoint security monitoring
- controlled administrative privileges
- firewall-enforced segmentation
- SIEM visibility
- Group Policy
- Purple Team readiness

The workstation infrastructure phase is complete.

Future work should use this endpoint for realistic attack, detection, investigation, and response scenarios rather than continued infrastructure modification.
