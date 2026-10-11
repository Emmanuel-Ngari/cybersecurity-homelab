# Active Directory

## Overview

This document records the complete Active Directory project phase in the Cybersecurity Home Lab, from the period when the environment had no centralized identity infrastructure through the final validated `cyberlab.internal` deployment.

Active Directory was not part of the original lab.

The earliest environment consisted mainly of:

- Kali Linux
- Windows
- Ubuntu Server

on:

`192.168.56.0/24`

The Windows system operated as a standalone endpoint.

At that stage, the lab had no:

- Domain Controller
- centralized authentication
- Group Policy
- enterprise user structure
- privileged-account separation
- domain DNS
- identity-focused monitoring

As the lab matured into a more realistic enterprise environment, centralized identity became necessary.

This led to the introduction of:

- ADNET
- Windows Server 2025
- Active Directory Domain Services
- DNS
- Kerberos
- LDAP
- Global Catalog
- Organizational Units
- security groups
- standard and privileged accounts
- domain-joined workstation
- Group Policy
- advanced auditing
- Wazuh monitoring
- Sysmon on the Domain Controller

The final domain is:

`cyberlab.internal`

NetBIOS name:

`CYBERLAB`

Primary Domain Controller:

`LAB-DC-01`

Address:

`10.10.50.10`

Zone:

`ADNET`

---

# Phase 1 — Before Active Directory

The original Windows system was not domain joined.

Authentication and system configuration were handled locally.

This was sufficient for early work involving:

- Windows administration
- networking
- local firewall configuration
- Wazuh
- Sysmon
- PowerShell logging

However, it did not represent how Windows systems are typically managed in an enterprise.

The environment lacked:

- centralized user identity
- centralized policy
- domain authentication
- enterprise account separation
- directory monitoring

---

# Why Active Directory Was Added

The lab needed an identity platform capable of supporting:

- domain authentication
- Kerberos
- LDAP
- centralized user management
- Group Policy
- workstation management
- privileged administration
- identity telemetry
- future Active Directory attacks
- Purple Team exercises

Active Directory was therefore introduced as a major infrastructure expansion.

---

# ADNET Creation

A dedicated identity network was created:

`ADNET`

Network:

`10.10.50.0/24`

Gateway:

`10.10.50.1`

Domain Controller:

`LAB-DC-01 — 10.10.50.10`

This created a separate trust zone for identity infrastructure.

---

# Why ADNET Was Separate

The Domain Controller was intentionally not placed in CORPNET.

A separate network provides:

- stronger segmentation
- reduced lateral-movement exposure
- clearer firewall policy
- dedicated identity protection
- easier monitoring

The Domain Controller is treated as high-value infrastructure.

---

# Windows Server Deployment

The Domain Controller was deployed using:

`Windows Server 2025 Standard Evaluation`

Final host:

`LAB-DC-01`

Final address:

`10.10.50.10`

Final gateway:

`10.10.50.1`

The server used a static address because Active Directory infrastructure requires stable addressing.

---

# Active Directory Domain Services

The Active Directory Domain Services role was installed.

The server was promoted to a Domain Controller for a new forest.

Domain:

`cyberlab.internal`

NetBIOS:

`CYBERLAB`

Forest and domain functional levels were configured for the Windows Server 2025 environment.

---

# DNS Integration

DNS is a core dependency of Active Directory.

`LAB-DC-01` provides DNS for:

`cyberlab.internal`

This allows clients to locate:

- Domain Controllers
- Kerberos services
- LDAP
- Global Catalog
- domain resources

DNS therefore became one of the most important CORPNET-to-ADNET services.

---

# Global Catalog

`LAB-DC-01` operates as a Global Catalog server.

This supports:

- directory searches
- authentication
- forest-wide identity lookups

The associated service includes:

`TCP/3268`

---

# Core Domain Services

The Active Directory environment relies on services including:

- DNS
- Kerberos
- LDAP
- SMB
- RPC
- NTP
- Global Catalog

These services were incorporated into OPNsense firewall policy.

---

# Active Directory Firewall Design

The project avoided a broad rule such as:

`CORPNET → ADNET Any`

Instead, required services were explicitly permitted to the Domain Controller.

Important TCP services include:

- `53`
- `88`
- `135`
- `389`
- `445`
- `464`
- `3268`
- `49152:65535`

Important UDP services include:

- `53`
- `88`
- `123`
- `389`
- `464`

This supports domain functionality while preserving least privilege.

---

# CORPNET to ADNET Validation

The corporate workstation was used to test key services.

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

These service checks were supplemented by successful domain join and Group Policy operation.

---

# Domain Validation

The server was validated using:

    whoami

Result:

`cyberlab\administrator`

This confirmed the Domain Controller was operating in the new domain context.

---

# Domain Controller Discovery

The following was tested:

    nltest /dsgetdc:cyberlab.internal

The command returned the Domain Controller successfully.

Status:

**PASS**

This validated:

- DNS discovery
- domain registration
- DC availability

---

# SYSVOL and NETLOGON

The standard Active Directory shares were present:

- `SYSVOL`
- `NETLOGON`

These are required for:

- Group Policy
- logon scripts
- domain operations

Their availability confirmed core domain functionality.

---

# Domain Controller Health

Targeted health checks included:

    dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

No output was returned.

For `/q`, this indicated no detected failures in those tests.

Additional DNS health validation also passed.

Status:

**PASS**

---

# Organizational Unit Design

A structured OU hierarchy was created under:

`CyberLab`

Primary OUs include:

- Admins
- Users
- Groups
- Service Accounts
- Servers
- Workstations

This provides logical separation and supports targeted Group Policy.

---

# Why OUs Were Used

The OU structure allows:

- workstation policy targeting
- administrative organization
- future delegation
- cleaner directory management
- role-based grouping

This is more realistic than leaving objects in default containers.

---

# Security Groups

The domain includes role-based Global Groups.

Examples:

- `GG_Employees`
- `GG_Helpdesk`
- `GG_IT_Admins`
- `GG_SOC_Analysts`
- `GG_Domain_Admins`

The `GG_` prefix represents:

`Global Group`

---

# GG_Domain_Admins Clarification

`GG_Domain_Admins`

is a custom classification group.

It is not itself equivalent to the built-in:

`Domain Admins`

group.

Actual privileged access is granted separately.

This prevents confusion between:

- role classification
- real administrative privilege

---

# Standard User Account

A standard domain account was created:

`emmanuel.patrick@cyberlab.internal`

Membership includes:

- Domain Users
- GG_Employees
- GG_SOC_Analysts

This account represents normal day-to-day identity.

---

# Privileged Administrative Account

A separate privileged account was created:

`emmanuel.ngari-adm@cyberlab.internal`

Membership includes:

- Domain Users
- GG_IT_Admins
- GG_Domain_Admins
- Domain Admins

This account is intended for privileged administration.

---

# Privileged Account Separation

The lab deliberately separates:

- normal user activity
- administrative activity

This supports the enterprise principle:

**Do not perform everyday work using Domain Admin privileges.**

This also improves future detection scenarios because privileged actions can be attributed to a dedicated account.

---

# Workstation Domain Join

`LAB-WIN-01`

was successfully joined to:

`cyberlab.internal`

This transformed the endpoint into a managed enterprise workstation.

Successful domain join validated multiple services, including:

- DNS
- Kerberos
- LDAP
- SMB
- RPC

---

# Workstation OU

The domain-joined workstation is managed under:

`CyberLab\Workstations`

This allows workstation-specific policies to be applied without affecting:

- servers
- the Domain Controller
- other infrastructure

---

# Group Policy Introduction

A workstation security baseline was created:

`GPO-Workstation-Security-Baseline`

Linked to:

`CyberLab\Workstations`

This centralized several Windows security controls.

---

# Windows Firewall Policy

The GPO configures Windows Firewall.

Final behaviour:

    Firewall: Enabled
    Inbound: Block
    Outbound: Allow

This adds host-level protection in addition to OPNsense segmentation.

---

# Audit Policy

The workstation baseline includes:

## Logon

    Success
    Failure

## Process Creation

    Success

## Special Logon

    Success

This improves security visibility.

---

# Command-Line Process Auditing

The GPO also enables:

`Include command line in process creation events`

This allows analysts to see how a process was launched.

This is especially useful for:

- PowerShell
- cmd
- attacker tooling
- administrative commands

---

# PowerShell Group Policy

The workstation baseline enables:

- Script Block Logging
- Module Logging

Module logging scope:

`*`

This centralizes PowerShell visibility.

---

# Group Policy Validation

The policy successfully applied to:

`LAB-WIN-01`

Status:

**PASS**

This confirmed:

- domain join
- OU placement
- Group Policy processing
- domain communication

---

# Active Directory Auditing

Security auditing on the Domain Controller was expanded to provide identity-focused telemetry.

Important observed events include:

- `4672`
- `4719`
- `5136`

---

# Event ID 4672

Event `4672` indicates:

`Special privileges assigned to new logon`

This provides useful context for privileged sessions.

Possible use cases include:

- Domain Admin activity
- administrator logons
- privileged automation

---

# Event ID 4719

Event `4719` indicates:

`System audit policy was changed`

This is security-relevant because attackers may attempt to weaken logging.

It can help detect:

- audit tampering
- unauthorized policy changes
- possible defense evasion

---

# Event ID 5136

Event `5136` records:

Active Directory object modification.

This can help monitor:

- account changes
- group-related changes
- directory configuration
- identity persistence

---

# Wazuh Integration

The Domain Controller was enrolled into Wazuh.

Final agent:

    ID: 003
    Name: LAB-DC-01
    Status: Active

Destination:

`LAB-SIEM-01`

Address:

`10.10.40.10`

---

# ADNET to SOCNET Validation

Test:

    Test-NetConnection 10.10.40.10 -Port 1514

Result:

    TcpTestSucceeded : True

Status:

**PASS**

This confirmed the Domain Controller could send telemetry to the SIEM.

---

# Sysmon on the Domain Controller

Sysmon was installed on:

`LAB-DC-01`

Version:

`15.22`

Binary:

`C:\Tools\Sysmon\Sysmon64.exe`

Configuration:

`sysmon-dc.xml`

Wazuh successfully received Sysmon telemetry.

---

# Why Sysmon Was Added

The Domain Controller produces useful standard Windows security events, but Sysmon adds process-level context.

This supports monitoring of:

- PowerShell
- command shells
- LOLBins
- domain reconnaissance tools
- suspicious administrative utilities

---

# Sysmon Tuning

The initial configuration collected broader process and DNS telemetry.

The configuration was later tuned to reduce noise.

Important monitored executables include:

- powershell
- pwsh
- cmd
- rundll32
- regsvr32
- mshta
- certutil
- bitsadmin
- wscript
- cscript
- schtasks
- nltest
- ntdsutil
- whoami
- net
- net1

DNS monitoring was also focused on security-relevant processes.

---

# Why Tuning Was Important

A Domain Controller generates large volumes of normal activity.

Collecting every event would:

- increase noise
- consume SIEM storage
- make investigations harder

The goal was to preserve high-value visibility.

---

# REDNET Isolation

REDNET was not granted normal access to Active Directory.

A final test checked:

`REDNET → ADNET LDAP/389`

Result:

Connection timed out.

Status:

**PASS**

This confirms attacker-network isolation.

---

# ADNET to DMZ Isolation

The Domain Controller was also prevented from becoming a general-purpose administration host.

Test:

`LAB-DC-01 → LAB-DMZ-01 TCP/22`

Result:

    TcpTestSucceeded : False

Status:

**PASS**

---

# Why the DC Is Not a Jump Host

The Domain Controller should not be used for routine administration of lower-trust systems.

Doing so would:

- expose privileged credentials
- increase attack surface
- mix identity and administration roles

DMZ administration therefore occurs from:

`LAB-WIN-01`

instead.

---

# Identity Security Architecture

The final identity architecture can be represented as:

    LAB-WIN-01
      CORPNET
         |
         | AD Services
         v
    LAB-FW-01
      OPNsense
         |
         v
    LAB-DC-01
      ADNET
         |
         +--> AD DS
         |
         +--> DNS
         |
         +--> Kerberos
         |
         +--> LDAP
         |
         +--> Group Policy
         |
         +--> Windows Auditing
         |
         +--> Sysmon
         |
         v
      Wazuh Agent
         |
         v
    LAB-SIEM-01
      SOCNET

---

# Security Model

The Active Directory design uses multiple layers of protection.

## Network Layer

ADNET is separated by OPNsense.

## Identity Layer

Normal and privileged accounts are separated.

## Policy Layer

Group Policy enforces workstation security.

## Endpoint Layer

Windows auditing and Sysmon provide telemetry.

## SOC Layer

Wazuh centralizes identity events.

---

# Future Purple Team Use Cases

The environment is now capable of supporting controlled scenarios involving:

- domain reconnaissance
- suspicious privileged logons
- account manipulation
- group changes
- PowerShell abuse
- scheduled tasks
- credential access simulation
- lateral movement
- persistence
- audit-policy tampering

These scenarios can be correlated across:

- Windows Security logs
- Sysmon
- PowerShell
- firewall logs
- Wazuh

---

# Final Validation Summary

| Validation | Result |
|---|---|
| ADNET created | PASS |
| Windows Server 2025 deployed | PASS |
| AD DS installed | PASS |
| `cyberlab.internal` created | PASS |
| DNS active | PASS |
| Global Catalog active | PASS |
| SYSVOL available | PASS |
| NETLOGON available | PASS |
| `nltest /dsgetdc` | PASS |
| Targeted `dcdiag` | PASS |
| OU hierarchy created | PASS |
| Security groups created | PASS |
| Standard user created | PASS |
| Privileged admin created | PASS |
| Workstation domain join | PASS |
| CORPNET → DNS/53 | PASS |
| CORPNET → Kerberos/88 | PASS |
| CORPNET → LDAP/389 | PASS |
| Workstation GPO | PASS |
| Advanced auditing | PASS |
| Event 4672 observed | PASS |
| Event 4719 observed | PASS |
| Event 5136 observed | PASS |
| Wazuh agent active | PASS |
| Sysmon active | PASS |
| Sysmon tuning | PASS |
| REDNET → LDAP blocked | PASS |
| ADNET → DMZ SSH blocked | PASS |

---

# Snapshot and Change Management

Snapshots were taken before significant Active Directory changes.

Major stages included:

- server deployment
- network configuration
- AD DS promotion
- domain creation
- OU and group creation
- user creation
- firewall changes
- workstation domain join
- Group Policy
- Wazuh onboarding
- Sysmon deployment
- Sysmon tuning

After final six-zone validation, obsolete intermediate snapshots were removed.

The final Domain Controller snapshot is:

`Final-Six-Zone-Validated-Baseline`

---

# Historical vs Current State

This document intentionally preserves the entire identity-security evolution.

Historical state included:

- standalone Windows
- local accounts
- no domain
- no Group Policy
- no ADNET
- no Domain Controller

The final authoritative state is:

    Domain:
    cyberlab.internal

    NetBIOS:
    CYBERLAB

    Domain Controller:
    LAB-DC-01

    IP:
    10.10.50.10

    Zone:
    ADNET

    DNS:
    Active

    Global Catalog:
    Active

    Wazuh:
    Active

    Sysmon:
    Active

    Workstation:
    LAB-WIN-01 joined

    Group Policy:
    GPO-Workstation-Security-Baseline

---

# Evolution Summary

The Active Directory project evolved through:

    Standalone Windows
           |
           v
    Need for Central Identity
           |
           v
    ADNET Creation
           |
           v
    Windows Server 2025
           |
           v
    AD DS / DNS
           |
           v
    cyberlab.internal
           |
           v
    OU Structure
           |
           v
    Security Groups
           |
           v
    Standard / Privileged Accounts
           |
           v
    Workstation Domain Join
           |
           v
    Group Policy
           |
           v
    Advanced Auditing
           |
           v
    Wazuh Monitoring
           |
           v
    Sysmon and Tuning
           |
           v
    Final Identity Security Platform

---

# Final Status

The Active Directory infrastructure phase is complete.

The lab now has a centralized identity environment providing:

- domain authentication
- DNS
- Kerberos
- LDAP
- Group Policy
- role-based user and group structure
- privileged-account separation
- identity auditing
- Wazuh monitoring
- Sysmon visibility
- firewall-enforced ADNET isolation

The identity platform is stable and ready for realistic Active Directory SOC and Purple Team exercises.

Future work should focus on controlled identity attack simulation, alert investigation, detection engineering, threat hunting, and privilege-abuse scenarios rather than continuing to alter the baseline architecture without a specific operational requirement.
