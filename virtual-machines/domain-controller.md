# LAB-DC-01 — Active Directory Domain Controller

## Overview

This document records the complete lifecycle of the Active Directory environment in the Cybersecurity Home Lab, from the period before centralized identity existed through the final validated `LAB-DC-01` deployment in ADNET.

Active Directory was not part of the original lab.

The earliest environment consisted primarily of:

- Kali Linux
- Windows
- Ubuntu Server

running on the original:

`192.168.56.0/24`

lab network.

As the project matured into a more realistic enterprise-style environment, centralized identity, authentication, Group Policy, Windows domain services, privileged-account separation, and identity-focused monitoring became necessary.

This led to the introduction of:

`LAB-DC-01`

Final role:

Active Directory Domain Controller and DNS server

Final operating system:

`Windows Server 2025 Standard Evaluation`

Final address:

`10.10.50.10`

Final zone:

`ADNET`

Domain:

`cyberlab.internal`

NetBIOS domain:

`CYBERLAB`

---

# Phase 1 — Before Active Directory

The original lab did not contain:

- a Domain Controller
- centralized authentication
- Active Directory
- Group Policy
- organizational units
- enterprise user accounts
- privileged-account separation
- domain DNS

The Windows system operated as a standalone endpoint.

Authentication and security settings were configured locally.

This was sufficient for early exercises involving:

- connectivity
- firewall configuration
- endpoint logging
- Sysmon
- PowerShell
- Wazuh

but it did not accurately represent a modern enterprise Windows environment.

---

# Why Active Directory Was Added

As the lab matured, Active Directory was needed to support:

- centralized identity
- domain authentication
- Kerberos
- LDAP
- Group Policy
- organizational structure
- user and administrator separation
- Windows domain telemetry
- identity-based detection engineering
- future lateral-movement simulations
- privilege-escalation scenarios
- Active Directory attack and defense exercises

The Domain Controller therefore became a major expansion of the original architecture.

---

# ADNET Introduction

Rather than placing the Domain Controller directly in CORPNET, a dedicated identity-security zone was created.

Network:

`ADNET`

Subnet:

`10.10.50.0/24`

Gateway:

`10.10.50.1`

Domain Controller:

`LAB-DC-01 — 10.10.50.10`

This created separation between:

- user workstations
- server systems
- attacker infrastructure
- security monitoring
- identity infrastructure

---

# Why ADNET Was Separated

The Domain Controller is one of the highest-value systems in an enterprise environment.

Placing it on a dedicated network provides several advantages:

- tighter firewall policy
- reduced lateral-movement exposure
- clearer trust boundaries
- easier monitoring
- reduced attacker access
- more realistic enterprise architecture

The design therefore avoids treating the Domain Controller as an ordinary workstation or general-purpose server.

---

# LAB-DC-01 Deployment

The Domain Controller was deployed using:

`Windows Server 2025 Standard Evaluation`

Final addressing:

    Hostname: LAB-DC-01
    IP Address: 10.10.50.10
    Prefix: /24
    Gateway: 10.10.50.1
    Zone: ADNET

The system was configured with a static address because identity infrastructure must remain predictable and stable.

---

# Active Directory Domain Services

The Active Directory Domain Services role was installed.

The server was promoted to a Domain Controller for:

`cyberlab.internal`

NetBIOS name:

`CYBERLAB`

The environment was created as a new forest.

---

# Forest and Domain

Final domain configuration:

    Forest: cyberlab.internal
    Domain: cyberlab.internal
    NetBIOS: CYBERLAB

The environment uses Windows Server 2025 functional levels.

---

# Domain Controller Roles

`LAB-DC-01` provides:

- Active Directory Domain Services
- DNS
- Global Catalog
- Kerberos authentication
- LDAP
- Group Policy infrastructure
- SYSVOL
- NETLOGON
- centralized identity services

---

# DNS Role

DNS is critical to Active Directory.

The Domain Controller provides domain DNS functionality for:

`cyberlab.internal`

This allows domain members to locate:

- Domain Controllers
- Kerberos services
- LDAP services
- Global Catalog
- domain resources

DNS therefore became an essential part of CORPNET-to-ADNET firewall policy.

---

# Global Catalog

`LAB-DC-01` also operates as a Global Catalog server.

This supports:

- directory searches
- authentication
- forest-wide identity lookup

The Global Catalog service uses:

`TCP/3268`

as part of the Active Directory firewall requirements.

---

# Active Directory Firewall Requirements

Unlike simple application services, Active Directory requires multiple network services.

CORPNET therefore received explicit access to the Domain Controller rather than a broad:

`CORPNET → ADNET Any`

rule.

Important TCP services include:

- `53` — DNS
- `88` — Kerberos
- `135` — RPC Endpoint Mapper
- `389` — LDAP
- `445` — SMB
- `464` — Kerberos password operations
- `3268` — Global Catalog
- `49152:65535` — Dynamic RPC

Important UDP services include:

- `53` — DNS
- `88` — Kerberos
- `123` — NTP
- `389` — LDAP
- `464` — Kerberos password operations

These rules support:

- domain join
- authentication
- Group Policy
- SYSVOL
- directory queries
- time synchronization
- RPC
- Kerberos

---

# CORPNET Connectivity Validation

The Windows workstation was used to verify access to important Domain Controller services.

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

These spot checks were combined with successful domain join and Group Policy operation to validate the broader AD service set.

---

# Domain Validation

The Domain Controller was validated using:

    whoami

Result:

`cyberlab\administrator`

This confirmed that the server was operating within the new domain context.

---

# Domain Controller Discovery Validation

The following command was used:

    nltest /dsgetdc:cyberlab.internal

The command successfully located the Domain Controller.

Status:

**PASS**

This confirmed:

- DNS-based DC discovery
- domain availability
- domain service registration

---

# SYSVOL and NETLOGON

The Domain Controller successfully published the standard Active Directory shares:

- `SYSVOL`
- `NETLOGON`

These shares are required for:

- Group Policy
- logon scripts
- domain operations

Their availability was part of the domain health validation.

---

# Domain Controller Health Checks

Targeted `dcdiag` testing included:

    dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

Result:

No output.

For these `/q` tests, no output indicated no detected failures.

Status:

**PASS**

DNS-specific Domain Controller health testing was also completed successfully.

---

# Organizational Unit Design

A structured Active Directory hierarchy was created under:

`CyberLab`

The primary organizational units include:

- Admins
- Users
- Groups
- Service Accounts
- Servers
- Workstations

This structure allows policies and identities to be organized according to role.

---

# Why Organizational Units Were Used

The OU structure supports:

- Group Policy targeting
- administrative separation
- logical organization
- future delegation
- easier investigation
- scalable identity management

The design is more realistic than placing every object inside default Active Directory containers.

---

# Security Groups

Several groups were created to represent enterprise roles.

These include:

- `GG_Employees`
- `GG_Helpdesk`
- `GG_IT_Admins`
- `GG_SOC_Analysts`
- `GG_Domain_Admins`

---

# Group Naming

The `GG_` prefix represents:

`Global Group`

This provides consistent naming and makes the directory structure easier to understand.

---

# GG_Domain_Admins Clarification

`GG_Domain_Admins` is a lab classification group.

It does not itself provide built-in Domain Admin privileges.

Actual privileged membership is controlled separately through the built-in:

`Domain Admins`

group.

This distinction prevents a custom group name from being mistaken for the actual privileged Active Directory group.

---

# Standard User Account

A normal user account was created:

`emmanuel.patrick@cyberlab.internal`

Membership includes:

- Domain Users
- GG_Employees
- GG_SOC_Analysts

This account represents normal day-to-day activity.

It is not intended to function as a permanent privileged administrative identity.

---

# Privileged Administrative Account

A separate privileged identity was created:

`emmanuel.ngari-adm@cyberlab.internal`

Membership includes:

- Domain Users
- GG_IT_Admins
- GG_Domain_Admins
- Domain Admins

This account is intended for privileged administrative work.

---

# Privileged Account Separation

Creating separate standard and privileged identities supports an important enterprise security principle:

**Do not perform normal user activity with Domain Admin credentials.**

The separation allows future scenarios involving:

- normal user compromise
- privileged account use
- privilege escalation
- administrative monitoring

to be investigated more realistically.

---

# Workstation Domain Join

`LAB-WIN-01`

was successfully joined to:

`cyberlab.internal`

This transformed the workstation from a standalone machine into a domain member.

The successful domain join indirectly validated several services, including:

- DNS
- Kerberos
- LDAP
- SMB
- RPC

---

# Workstation Organizational Unit

The workstation is managed through the:

`CyberLab\Workstations`

OU.

This allows workstation-specific Group Policy to be applied without unintentionally affecting:

- Domain Controllers
- servers
- other infrastructure

---

# Group Policy Introduction

A dedicated workstation security baseline was created:

`GPO-Workstation-Security-Baseline`

The GPO is linked only to:

`CyberLab\Workstations`

This provides centralized security configuration for the corporate Windows endpoint.

---

# Windows Firewall GPO

The workstation baseline configures Windows Firewall.

Final behaviour includes:

    Firewall: Enabled
    Inbound: Block
    Outbound: Allow

This provides host-level security in addition to OPNsense network segmentation.

---

# Audit Policy GPO

The workstation security baseline enables important auditing.

Configured categories include:

## Logon

    Success
    Failure

## Process Creation

    Success

## Special Logon

    Success

These events provide important visibility into user and administrative activity.

---

# Command-Line Process Auditing

Process creation was configured to include command-line information.

This allows SOC analysts to see not only:

`powershell.exe`

or:

`cmd.exe`

but also the arguments used during execution.

This significantly increases detection value.

---

# PowerShell Group Policy

The baseline also enables:

- PowerShell Script Block Logging
- PowerShell Module Logging

Module logging includes:

`*`

This supports detailed PowerShell visibility during future attack simulations.

---

# Group Policy Validation

The policy was successfully applied to:

`LAB-WIN-01`

This confirmed:

- successful domain membership
- correct OU placement
- functional Group Policy
- working AD network communication

Status:

**PASS**

---

# Domain Controller Security Auditing

Security auditing was expanded on the Domain Controller itself.

Important events observed included:

`Event ID 5136`

Active Directory object modification.

---

`Event ID 4719`

System audit policy change.

---

`Event ID 4672`

Special privileges assigned to a new logon.

These events provide valuable identity-security telemetry.

---

# Wazuh Integration

The Domain Controller was enrolled into Wazuh.

Final agent identity:

    ID: 003
    Name: LAB-DC-01
    Status: Active

Telemetry is sent to:

`LAB-SIEM-01 — 10.10.40.10`

Primary port:

`TCP/1514`

Enrollment may use:

`TCP/1515`

---

# ADNET to SOCNET Validation

Test:

    Test-NetConnection 10.10.40.10 -Port 1514

Result:

    TcpTestSucceeded : True

Status:

**PASS**

This confirmed the Domain Controller could send security telemetry to the SIEM.

---

# Domain Controller Sysmon

Sysmon was installed on the Domain Controller to provide richer endpoint visibility.

Version:

`15.22`

Binary:

`C:\Tools\Sysmon\Sysmon64.exe`

Configuration:

`sysmon-dc.xml`

The deployment was verified and Sysmon events were successfully ingested into Wazuh.

---

# Why Sysmon Was Added to the Domain Controller

Standard Windows auditing provides significant visibility, but Sysmon adds additional process and activity context.

This is useful for detecting activity such as:

- suspicious PowerShell
- command shells
- administrative tools
- LOLBins
- reconnaissance
- scripted attacks
- privilege abuse

---

# Initial Sysmon Configuration

The initial Domain Controller Sysmon configuration collected process and DNS activity.

However, collecting everything would create unnecessary noise.

The configuration was therefore tuned.

---

# Sysmon ProcessCreate Tuning

Final ProcessCreate monitoring focused on security-relevant executables such as:

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

This provides strong visibility into common administrative and attacker tools without collecting every normal process.

---

# Sysmon DNS Tuning

DNS monitoring was similarly focused on relevant processes such as:

- powershell
- pwsh
- cmd
- certutil
- mshta
- wscript
- cscript

The objective was to preserve useful detection value while reducing event volume.

---

# Wazuh Sysmon Validation

After tuning, Sysmon telemetry continued reaching the SIEM.

This confirmed that:

- Sysmon was active
- the configuration loaded correctly
- Wazuh collected the events
- filtering reduced noise without breaking monitoring

---

# REDNET Isolation

The attacker network was not given unrestricted access to the Domain Controller.

A final test from Kali checked:

`TCP/389`

against:

`10.10.50.10`

Expected:

`BLOCK`

Result:

Connection timed out.

Status:

**PASS**

This confirmed that REDNET could not directly access LDAP.

---

# ADNET to DMZ Isolation

The Domain Controller was intentionally prevented from functioning as a general-purpose DMZ management system.

Test:

`LAB-DC-01 → LAB-DMZ-01 TCP/22`

Expected:

`BLOCK`

Result:

    TcpTestSucceeded : False

Status:

**PASS**

---

# Why the DC Is Not a Jump Host

Using the Domain Controller to administer DMZ systems would create unnecessary risk.

The DC contains:

- privileged credentials
- identity infrastructure
- directory services
- authentication functions

Using it as a routine management workstation would expand its attack surface.

DMZ administration is therefore performed from:

`LAB-WIN-01`

through a specific firewall rule.

---

# Active Directory Monitoring Architecture

The final identity-monitoring path is:

    Domain Activity
         |
         v
    LAB-DC-01
         |
         +--> Windows Security Logs
         |
         +--> Active Directory Events
         |
         +--> Sysmon
         |
         +--> Advanced Auditing
         |
         v
    Wazuh Agent
         |
         v
    LAB-SIEM-01

This provides centralized identity and privileged-activity visibility.

---

# Active Directory Security Model

The final domain architecture combines multiple security controls.

## Network Layer

OPNsense isolates ADNET.

## Identity Layer

Standard and privileged accounts are separated.

## Policy Layer

Group Policy controls workstation security.

## Endpoint Layer

Windows auditing and Sysmon provide telemetry.

## SOC Layer

Wazuh centralizes and analyzes events.

Together, these controls create a realistic identity-security environment.

---

# Final Active Directory Validation

The final environment successfully validated:

| Validation | Result |
|---|---|
| Domain creation | PASS |
| Domain Controller promotion | PASS |
| DNS | PASS |
| Global Catalog | PASS |
| SYSVOL | PASS |
| NETLOGON | PASS |
| `nltest /dsgetdc` | PASS |
| Targeted `dcdiag` | PASS |
| CORPNET → DNS/53 | PASS |
| CORPNET → Kerberos/88 | PASS |
| CORPNET → LDAP/389 | PASS |
| Workstation domain join | PASS |
| OU structure | PASS |
| Security groups | PASS |
| Standard user account | PASS |
| Privileged admin account | PASS |
| Workstation GPO | PASS |
| Advanced auditing | PASS |
| Wazuh agent | PASS |
| Sysmon | PASS |
| ADNET → SOCNET Wazuh/1514 | PASS |
| REDNET → ADNET LDAP blocked | PASS |
| ADNET → DMZ SSH blocked | PASS |

---

# Snapshot History

Snapshots were used before major Active Directory changes.

Important stages included:

- Windows Server deployment
- network configuration
- Domain Controller promotion
- domain configuration
- OU and group creation
- user creation
- firewall-rule development
- workstation domain join
- Group Policy deployment
- Wazuh onboarding
- Sysmon deployment
- Sysmon tuning

Once the full six-zone environment passed validation, temporary development snapshots were removed.

The remaining snapshot is:

`Final-Six-Zone-Validated-Baseline`

Snapshot UUID:

`dd108acf-e11f-44f2-82f1-c3665d3f3691`

---

# Historical vs Current Active Directory State

This repository intentionally preserves the full project evolution.

Earlier stages of the lab had:

- no domain
- no Domain Controller
- local Windows accounts
- no Group Policy
- no ADNET
- no centralized identity

The authoritative current state is:

    Hostname: LAB-DC-01
    OS: Windows Server 2025 Standard Evaluation
    IP: 10.10.50.10
    Zone: ADNET
    Gateway: 10.10.50.1
    Domain: cyberlab.internal
    NetBIOS: CYBERLAB
    DNS: Active
    Global Catalog: Active
    Wazuh Agent: Active
    Sysmon: Active

The earlier stages remain part of the documentation because they represent the real progression of the lab.

---

# Evolution Summary

The Active Directory environment evolved through the following stages:

    Standalone Windows Environment
             |
             v
    Need for Centralized Identity
             |
             v
    ADNET Creation
             |
             v
    Windows Server 2025 Deployment
             |
             v
    AD DS / DNS Installation
             |
             v
    cyberlab.internal Forest
             |
             v
    OU / Group Structure
             |
             v
    Standard and Privileged Accounts
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
    Sysmon Deployment and Tuning
             |
             v
    Final Identity Security Platform

---

# Final Role

`LAB-DC-01` now serves as the centralized identity platform for the Cybersecurity Home Lab.

Its responsibilities include:

- Active Directory
- DNS
- Kerberos
- LDAP
- Group Policy
- domain authentication
- centralized identity management
- privileged-account management
- identity telemetry
- future Active Directory attack and defense scenarios

---

# Current Status

The Active Directory infrastructure phase is complete.

`LAB-DC-01` is:

- healthy
- domain-authoritative
- correctly segmented
- monitored by Wazuh
- monitored with Sysmon
- protected from REDNET
- prevented from unnecessary DMZ administration
- supporting domain authentication
- successfully applying Group Policy
- validated as part of the final six-zone architecture

Future work should use the environment for realistic identity-focused SOC and Purple Team exercises rather than continuing to modify the baseline infrastructure.
