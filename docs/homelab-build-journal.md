# Cybersecurity Home Lab Build Journal

## Overview

This document records the complete chronological evolution of the Cybersecurity Home Lab from its earliest flat-network stage through the final validated six-zone architecture.

The project began as a simple hands-on environment for learning networking, Linux administration, Windows administration, scanning, SSH, Hydra, and fail2ban.

Over time, it evolved into a structured enterprise-style CyberLab with:

- OPNsense
- network segmentation
- Wazuh SIEM
- Sysmon
- PowerShell logging
- advanced Windows auditing
- Defender monitoring
- Windows Firewall telemetry
- Linux auditd
- custom detection engineering
- Suricata IDS
- Active Directory
- Group Policy
- Domain Controller monitoring
- DMZ infrastructure
- Nginx
- restricted administrative access
- centralized telemetry
- final six-zone validation

The lab is now considered infrastructure-complete and ready for realistic SOC and Purple Team operations.

---

# Phase 1 — Original Flat Lab

## Initial Objective

The project began as a practical cybersecurity learning environment.

The earliest goals were to understand:

- basic networking
- Linux administration
- Windows administration
- service discovery
- SSH
- host firewalls
- brute-force testing
- defensive blocking
- attacker/defender interaction

The lab was intentionally simple.

---

# Original Network

The early environment used:

`192.168.56.0/24`

Primary systems included:

- Kali Linux — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu Server — `192.168.56.30`

VirtualBox NAT was also used to provide Internet connectivity where required.

At this stage, there was no central OPNsense firewall.

The machines could communicate relatively freely across the internal network.

---

# Original Kali Configuration

Kali acted as the offensive workstation.

A typical configuration included:

`eth0`

for NAT:

    IP: 10.0.2.15/24
    Gateway: 10.0.2.2

and:

`eth1`

for the lab:

    IP: 192.168.56.10/24

Routes included:

    default via 10.0.2.2
    192.168.56.0/24 via eth1

This allowed Kali to:

- reach the Internet
- communicate with lab systems
- perform offensive testing

---

# Original Windows Configuration

The Windows system used:

`192.168.56.20/24`

The VirtualBox adapters were later renamed for clarity:

`Ethernet` → `NAT`

`Ethernet 2` → `LAB-NET`

The NAT adapter used:

    Gateway: 10.0.2.2
    DNS: 10.0.2.3

The internal interface handled lab traffic.

---

# Original Ubuntu Configuration

Ubuntu Server used:

`192.168.56.30`

It became the first major Linux target in the environment.

The server was used for:

- SSH
- Linux administration
- attacker testing
- defensive controls

---

# Early Connectivity Testing

The first stage focused on ensuring the machines could communicate.

Tests included:

- ping
- SSH
- Nmap
- Windows firewall rules
- VirtualBox adapter troubleshooting

Windows-to-Kali connectivity was eventually confirmed.

This provided the foundation for later attack and defense exercises.

---

# Phase 2 — Early Offensive and Defensive Exercises

## Nmap

Kali used Nmap to scan Ubuntu and identify exposed services.

An early result showed:

`22/tcp open`

This confirmed that:

- Ubuntu was reachable
- SSH was listening
- Kali could enumerate services

This was one of the first practical attacker-side validations.

---

# SSH

Ubuntu SSH was enabled.

Kali could connect to the server.

This created the first realistic attacker-to-target service path in the lab.

SSH became the focus of several early defensive exercises.

---

# Hydra

Hydra was used against the Ubuntu SSH service in a controlled lab scenario.

The goal was not simply to guess passwords.

The exercise was used to observe:

- repeated authentication failures
- SSH logs
- defensive response
- attacker behaviour after blocking

---

# fail2ban

fail2ban was installed and configured on Ubuntu.

The sequence became:

    Kali
      |
      v
    Repeated SSH Failures
      |
      v
    Ubuntu Authentication Logs
      |
      v
    fail2ban
      |
      v
    Kali Source Blocked

Kali then retested the server.

An earlier:

`22/tcp open`

state could later appear:

`filtered`

This became one of the first complete attack-detect-block-retest cycles in the project.

---

# Early Purple Team Mindset

Even before the project used Wazuh or Suricata, a basic Purple Team methodology was beginning to form.

The workflow was:

    Generate Activity
         |
         v
    Observe Defensive Response
         |
         v
    Retest
         |
         v
    Confirm Outcome

This principle remained important throughout the rest of the project.

---

# Phase 3 — Windows and VirtualBox Troubleshooting

As the environment became more complex, several host and virtualization issues had to be solved.

These troubleshooting experiences became part of the engineering process.

---

# Windows Network Profiles

The internal Windows interface initially appeared as:

`Unidentified Network`

with profile:

`Public`

This affected firewall behaviour.

The project reviewed:

- File and Printer Sharing rules
- ICMP behaviour
- Public profile restrictions

This was useful experience in understanding Windows host firewall logic.

---

# Windows Version Evolution

At different points, the Windows guest reported different version information.

Earlier PowerShell output referenced an older Windows 10 Enterprise Evaluation state.

The rebuilt authoritative environment eventually became:

`Windows 11 Enterprise Evaluation`

Version:

`25H2`

Build:

`26200.6584`

Earlier references remain part of the historical lab lifecycle.

---

# VirtualBox 3D Acceleration Issue

During the Windows rebuild, 3D acceleration caused login or display artifacts.

The setting was adjusted.

The problem was resolved.

This became an important troubleshooting lesson:

**Not every guest OS issue originates inside the guest OS.**

Virtualization settings can create misleading symptoms.

---

# Full-Screen and Display Stabilization

Display configuration was later stabilized successfully.

This made the workstation more practical for continued lab use.

---

# Why the Original Architecture Became Insufficient

As the project matured, the flat `192.168.56.0/24` environment became unrealistic.

The main limitations were:

- no trust zones
- no central firewall
- no east-west policy
- no dedicated SOC network
- no identity network
- no DMZ
- no network IDS visibility

The decision was made to rebuild the architecture around segmentation.

---

# Phase 4 — OPNsense Introduction

OPNsense was introduced as:

`LAB-FW-01`

Its role became:

- central firewall
- router
- network segmentation point
- DNS service
- NTP service
- Internet gateway
- log source
- later Suricata IDS platform

This was the largest architectural change in the project up to that point.

---

# First Segmented Architecture

The first OPNsense design introduced four internal zones:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

These replaced the earlier flat-network trust model.

---

# REDNET

Network:

`10.10.10.0/24`

Gateway:

`10.10.10.1`

Primary system:

`LAB-KALI-01 — 10.10.10.10`

Role:

Offensive security network.

---

# CORPNET

Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

Primary system:

`LAB-WIN-01 — 10.10.20.20`

Role:

Corporate workstation network.

---

# SERVERNET

Network:

`10.10.30.0/24`

Gateway:

`10.10.30.1`

Primary system:

`LAB-UBUNTU-01 — 10.10.30.30`

Role:

Protected internal Linux server network.

---

# SOCNET

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

Primary system:

`LAB-SIEM-01 — 10.10.40.10`

Role:

Security monitoring network.

---

# Initial OPNsense Rule Philosophy

The rule design evolved around several principles:

- least privilege
- specific allows
- broad denies
- logging at security boundaries
- explicit service validation

Important early concepts included:

- CORPNET administration
- REDNET containment
- SERVERNET isolation
- SOCNET telemetry

---

# Early CORPNET Rules

CORPNET required access to:

- OPNsense management
- DNS
- selected SERVERNET services
- Internet

Windows was not given unrestricted access to REDNET.

---

# Early REDNET Rules

REDNET retained:

- DNS
- Internet access
- selected attack paths toward Ubuntu

but was prevented from freely reaching:

- CORPNET
- firewall management
- protected services

This was the first time attacker access became an explicit policy choice rather than flat-network behaviour.

---

# Early SERVERNET Rules

SERVERNET retained:

- DNS
- NTP
- Internet access
- selected administration

while unnecessary access to:

- CORPNET
- REDNET

was restricted.

---

# Phase 5 — Wazuh SIEM Deployment

The next major capability was centralized monitoring.

`LAB-SIEM-01`

was deployed in:

`SOCNET`

Address:

`10.10.40.10`

Wazuh became the central monitoring platform.

---

# Wazuh Stack

The SIEM included:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

This represented the transition from local log inspection to SOC-style monitoring.

---

# Windows Wazuh Onboarding

`LAB-WIN-01`

became the first major monitored endpoint.

Final agent identity:

    ID: 001
    Name: LAB-WIN-01

The Windows monitoring capability expanded progressively.

---

# Windows Security Monitoring

The endpoint eventually provided:

- Security logs
- Sysmon
- PowerShell logging
- PowerShell transcription
- advanced auditing
- Microsoft Defender events
- Windows Firewall events
- File Integrity Monitoring

This turned Windows into a realistic monitored corporate endpoint.

---

# Sysmon on Windows

Sysmon was introduced to improve process visibility.

The goal was to observe:

- process execution
- parent-child relationships
- command activity
- attacker utilities
- suspicious binaries

Wazuh successfully ingested Sysmon events.

---

# PowerShell Logging

PowerShell visibility was expanded using:

- Script Block Logging
- Module Logging
- transcription

This allowed the SOC to see more than just the execution of `powershell.exe`.

---

# Advanced Windows Auditing

Important audit categories included:

- Logon Success
- Logon Failure
- Process Creation Success
- Special Logon Success

Command-line process creation was enabled.

This improved investigation context.

---

# Windows Firewall Event Monitoring

Windows Firewall telemetry included:

`Event ID 5157`

This helped distinguish:

- host firewall blocks
- OPNsense firewall blocks

---

# Defender Monitoring

Microsoft Defender became another endpoint telemetry source.

This provided additional defensive visibility alongside:

- Sysmon
- Security events
- PowerShell

---

# File Integrity Monitoring

Wazuh FIM was also configured.

This provided visibility into selected file and configuration changes.

---

# Phase 6 — Linux Monitoring and Detection Engineering

`LAB-UBUNTU-01`

was enrolled into Wazuh.

Final agent:

    ID: 002
    Name: LAB-UBUNTU-01

The server moved from local-only monitoring into centralized SOC monitoring.

---

# auditd

`auditd`

was integrated into Wazuh.

This provided deeper visibility into:

- account activity
- privileged changes
- administrative actions

---

# Linux User Creation Detection

A custom rule was validated:

`Rule 100200`

Purpose:

Detect native Linux user creation.

---

# Linux User Deletion Detection

Another custom rule was validated:

`Rule 100201`

Purpose:

Detect native Linux user deletion.

These were among the first formal custom detections in the project.

---

# Wazuh Service-Level Troubleshooting

At one stage, ICMP to the SIEM failed while:

`TCP/1514`

worked.

This became an important lesson:

**Ping failure does not necessarily mean the monitoring path is broken.**

The actual required service should be tested.

---

# Ubuntu Agent Connectivity Troubleshooting

At one point, the Ubuntu agent appeared disconnected.

Older logs showed errors.

Recent logs showed successful connection.

The manager later reported the agent:

`Active`

This reinforced another lesson:

**Historical errors should not be treated as current failures without checking timestamps and current state.**

---

# Phase 7 — OPNsense Firewall Telemetry

After endpoint monitoring was established, OPNsense logs were integrated into Wazuh.

The firewall became both:

- an enforcement device
- a telemetry source

---

# Firewall Detection

A Wazuh firewall correlation rule was validated:

`Rule 87702`

This added network-policy violation visibility to the SOC.

---

# Why Firewall Logs Mattered

A blocked packet may never reach the destination host.

Without firewall telemetry, some attack attempts would have no endpoint evidence.

OPNsense logging therefore became an important detection source.

---

# Phase 8 — Suricata IDS

Suricata was added to OPNsense later.

Primary monitored interface:

`em2`

This corresponds to REDNET.

ET Open rules were enabled.

---

# First Suricata Alert

A controlled test generated:

`SID 2100498`

Signature:

`GPL ATTACK_RESPONSE id check returned root`

Category:

`Potentially Bad Traffic`

Severity:

`2`

This confirmed Suricata itself was working.

---

# Initial Wazuh Suricata Problem

The first events reached Wazuh but did not decode correctly.

They fell through to:

`Rule 1002`

This showed that:

- transport worked
- parsing did not

The problem moved from networking into decoder troubleshooting.

---

# Custom Decoder Experimentation

A custom decoder was introduced.

One observed decoder name was:

`suricata-eve-syslog`

This helped confirm message structure.

However, the custom wrapper was later retired.

---

# Transition to JSON

The architecture was simplified around structured JSON.

The effective decoder became:

`json`

Wazuh's built-in Suricata rules could then process the events.

---

# Suricata Rule 86601

Built-in rule:

`86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

This confirmed correct end-to-end processing.

---

# Dedicated Suricata Transport

A dedicated syslog port was configured:

`UDP/5514`

Destination:

`LAB-SIEM-01`

rsyslog was configured using:

`/etc/rsyslog.d/30-suricata-remote.conf`

---

# Suricata Log File

Events were written to:

`/var/log/suricata/eve.json`

Directory ownership:

`syslog:wazuh`

Permissions:

`750`

Wazuh monitored the file using JSON format.

---

# rsyslog Listener Validation

The SIEM was verified listening on:

`0.0.0.0:5514`

and:

`[::]:5514`

This validated the transport layer.

---

# Final Suricata Pipeline

The final path became:

    LAB-FW-01
      Suricata
         |
         v
    Remote Logging
         |
       UDP/5514
         |
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
    Wazuh Logcollector
         |
         v
       JSON Decoder
         |
         v
      Rule 86601

---

# Suricata Troubleshooting Lessons

Several lessons came from this phase.

## Arrival Does Not Equal Decoding

Rule 1002 proved the event could arrive without being parsed properly.

## JSON Was Cleaner

Structured parsing reduced custom complexity.

## Built-In Rules Were Preferable

Custom decoder logic was retired when built-in support worked.

## Live jq Errors Did Not Mean Pipeline Failure

Direct `jq` parsing on an actively written file occasionally errored, but valid alerts continued.

## full_log Could Be Null

A Wazuh event could still be valid even when:

`full_log: null`

Structured fields remained available.

---

# Phase 9 — SIEM Storage Recovery

At one point, `LAB-SIEM-01` experienced a storage-related problem.

The VM was recovered successfully.

The main logical volume was:

`/dev/mapper/ubuntu--vg-ubuntu--lv`

A healthy state showed approximately:

    53G total
    21G used
    30G free

Later final validation showed:

    53G total
    25G used
    26G available
    49% used

---

# SIEM Recovery Validation

After recovery:

- LVM was active
- the system booted normally
- no failed systemd services remained
- Wazuh services were healthy

The recovered live system became authoritative.

---

# Important Snapshot Decision

An older SIEM snapshot existed.

The decision was made:

**Do not restore it.**

Restoring it would have risked losing newer:

- Wazuh configuration
- detections
- Suricata integration
- agent onboarding

The live recovered state was retained.

---

# Phase 10 — Active Directory Planning

Once the original four-zone architecture and monitoring stack were stable, the next major goal was centralized identity.

A dedicated identity zone was planned instead of placing the Domain Controller inside CORPNET.

This became:

`ADNET`

---

# ADNET

Network:

`10.10.50.0/24`

Gateway:

`10.10.50.1`

Domain Controller:

`LAB-DC-01 — 10.10.50.10`

---

# Windows Server Deployment

`LAB-DC-01`

was deployed using:

`Windows Server 2025 Standard Evaluation`

A static address was assigned.

The server was prepared for Active Directory Domain Services.

---

# Active Directory Deployment

The server was promoted to Domain Controller.

Domain:

`cyberlab.internal`

NetBIOS:

`CYBERLAB`

The environment included:

- AD DS
- DNS
- Global Catalog
- Kerberos
- LDAP

---

# AD Health Validation

Validation included:

    whoami

Result:

`cyberlab\administrator`

and:

    nltest /dsgetdc:cyberlab.internal

which succeeded.

SYSVOL and NETLOGON were available.

---

# dcdiag Validation

Targeted health checks included:

    dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

No output was returned.

DNS validation also passed.

---

# Active Directory OU Structure

A structured hierarchy was created under:

`CyberLab`

OUs included:

- Admins
- Users
- Groups
- Service Accounts
- Servers
- Workstations

This supported cleaner identity organization and Group Policy targeting.

---

# Active Directory Groups

Role-based groups included:

- GG_Employees
- GG_Helpdesk
- GG_IT_Admins
- GG_SOC_Analysts
- GG_Domain_Admins

`GG_Domain_Admins`

was treated as a classification group rather than the built-in privileged group.

---

# Standard User Account

Account:

`emmanuel.patrick@cyberlab.internal`

Membership included:

- Domain Users
- GG_Employees
- GG_SOC_Analysts

---

# Privileged Account

Account:

`emmanuel.ngari-adm@cyberlab.internal`

Membership included:

- Domain Users
- GG_IT_Admins
- GG_Domain_Admins
- Domain Admins

This created separation between normal and privileged activity.

---

# Active Directory Firewall Rules

CORPNET was given access to required domain services.

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

The design avoided a broad:

`CORPNET → ADNET Any`

rule.

---

# Active Directory Connectivity Testing

From `LAB-WIN-01`:

DNS:

    Test-NetConnection 10.10.50.10 -Port 53

PASS.

Kerberos:

    Test-NetConnection 10.10.50.10 -Port 88

PASS.

LDAP:

    Test-NetConnection 10.10.50.10 -Port 389

PASS.

---

# Workstation Domain Join

`LAB-WIN-01`

successfully joined:

`cyberlab.internal`

This converted the endpoint from standalone Windows into a managed domain workstation.

---

# Group Policy

A workstation baseline was created:

`GPO-Workstation-Security-Baseline`

Linked to:

`CyberLab\Workstations`

---

# GPO Security Settings

The baseline included:

- Windows Firewall enabled
- inbound block
- outbound allow
- Logon Success/Failure
- Process Creation Success
- Special Logon Success
- command-line process capture
- PowerShell Script Block Logging
- PowerShell Module Logging

Module logging scope:

`*`

---

# Group Policy Validation

The GPO successfully applied to:

`LAB-WIN-01`

This validated:

- domain membership
- OU placement
- Group Policy processing

---

# Domain Controller Wazuh Onboarding

`LAB-DC-01`

was enrolled into Wazuh.

Final agent:

    ID: 003
    Name: LAB-DC-01

Status:

`Active`

---

# Domain Controller Auditing

Important observed events included:

- 4672
- 4719
- 5136

These provided visibility into:

- privileged logons
- audit policy changes
- Active Directory object changes

---

# Domain Controller Sysmon

Sysmon was installed on:

`LAB-DC-01`

Version:

`15.22`

Binary:

`C:\Tools\Sysmon\Sysmon64.exe`

Configuration:

`sysmon-dc.xml`

---

# Sysmon Tuning

The Domain Controller configuration was tuned to focus on high-value executables.

Examples included:

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

DNS monitoring was also tuned.

This reduced unnecessary SIEM noise.

---

# AD Isolation Validation

REDNET → ADNET LDAP/389:

Blocked.

ADNET → DMZ SSH/22:

Blocked.

This confirmed the identity infrastructure was not broadly exposed.

---

# Phase 11 — DMZ Planning

After:

- OPNsense
- Wazuh
- Suricata
- Active Directory

were already functioning, the final major architecture component was planned:

a dedicated DMZ.

The purpose was to create an intentionally exposed web server network without weakening internal segmentation.

---

# DMZ Network

Network:

`10.10.60.0/24`

Gateway:

`10.10.60.1`

Server:

`LAB-DMZ-01 — 10.10.60.10`

---

# OPNsense NIC 7

The firewall needed an additional network adapter.

NIC 7 was added while the VM was powered off.

Configuration:

    Mode: Internal Network
    Network: DMZ-NET
    Adapter: Intel PRO/1000 MT Desktop (82540EM)
    Cable Connected: Yes

OPNsense detected the interface as:

`em6`

It was assigned as:

`DMZ`

---

# DMZ Server Deployment

A new VM was created using:

`Ubuntu Server 26.04.1 LTS`

Resources:

    RAM: 2 GB
    CPUs: 2
    Disk: 25 GB dynamically allocated VDI

LVM was used.

The root logical volume was manually expanded to approximately:

`20G`

---

# DMZ Static Addressing

Final configuration:

    Hostname: lab-dmz-01
    IP: 10.10.60.10
    Prefix: /24
    Gateway: 10.10.60.1
    DNS: 10.10.60.1

Interface:

`enp0s3`

---

# DMZ Firewall Policy

The final DMZ rule order became:

    1. ALLOW - DMZ to OPNsense DNS
    2. ALLOW - DMZ to OPNsense NTP
    3. ALLOW - DMZ to Wazuh Agent Services
    4. BLOCK - DMZ to Unauthorized Private Networks
    5. ALLOW - DMZ HTTP HTTPS Outbound

---

# DMZ Wazuh Connectivity

Before installing the agent, ports were tested:

    nc -zv -w 3 10.10.40.10 1514
    nc -zv -w 3 10.10.40.10 1515

Both succeeded.

This confirmed network policy before software deployment.

---

# DMZ Internal Isolation

DMZ → CORPNET SMB/445:

Blocked.

DMZ → ADNET LDAP/389:

Blocked.

This validated the broad private-network deny policy.

---

# DMZ Internet Validation

A test to:

`https://packages.wazuh.com`

returned:

`HTTP/2 403`

This was correctly interpreted as:

- DNS success
- TCP success
- TLS success
- application response

Therefore:

**Network connectivity passed.**

---

# Nginx Deployment

Nginx was installed.

Observed version:

`1.28.3 (Ubuntu)`

Service:

`Active`

Port:

`TCP/80`

This became the intentional REDNET attack surface.

---

# REDNET to DMZ HTTP Rule

A specific rule was created:

`ALLOW - REDNET to LAB-DMZ-01 HTTP`

This rule was placed above the broad REDNET private-network block.

---

# Apply Changes Incident

The rule was:

- correct
- saved
- correctly ordered

but traffic still failed.

OPNsense Live View showed the packet still matching the broad deny.

The actual problem:

**The firewall changes had not been applied.**

The red Apply button was still visible.

After applying the configuration:

HTTP succeeded.

This became one of the most important operational lessons in the lab.

---

# REDNET to DMZ HTTP Validation

From Kali:

`HTTP/80`

Result:

`200`

PASS.

---

# REDNET to DMZ SSH Validation

Kali attempted:

`TCP/22`

Result:

Timed out.

OPNsense confirmed the broad REDNET block.

PASS.

---

# Restricted DMZ Administration

A management path was needed.

Initial CORPNET → DMZ SSH failed as expected.

A narrow firewall rule was created:

`ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH`

Source:

`LAB-WIN-01 — 10.10.20.20`

Destination:

`LAB-DMZ-01 — 10.10.60.10`

Port:

`TCP/22`

The rule was placed above the broad CORPNET private-network deny.

SSH then succeeded.

---

# Why the Domain Controller Was Not Used

Using `LAB-DC-01` as a jump host was considered and rejected.

The Domain Controller was kept out of routine DMZ administration.

Final ADNET → DMZ SSH test:

Blocked.

This preserved identity isolation.

---

# SSH Hardening

An Ed25519 key was generated on:

`LAB-WIN-01`

The public key was installed for the Ubuntu user:

`emmanuel`

Final effective SSH settings:

    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no

Override file:

`/etc/ssh/sshd_config.d/00-dmz-hardening.conf`

---

# SSH Validation

Key-based login from `LAB-WIN-01`:

PASS.

Password-only login:

`Permission denied (publickey).`

PASS.

---

# Wazuh Agent on DMZ

Agent version:

`4.14.7-1`

Manager:

`10.10.40.10`

Agent name:

`LAB-DMZ-01`

Final agent:

    ID: 004
    Name: LAB-DMZ-01
    Status: Active

---

# Nginx Wazuh Integration

Wazuh monitored:

`/var/log/nginx/access.log`

using:

`apache`

and:

`/var/log/nginx/error.log`

using:

`syslog`

---

# Nginx Detection Validation

Kali requested:

`/wazuh-nginx-verify-01`

Nginx returned:

`404`

Wazuh generated:

`Rule 31101`

Description:

`Web server 400 error code.`

Source IP:

`10.10.10.10`

PASS.

---

# Final Suricata DMZ Validation

A temporary file was created:

`/var/www/html/suricata-test.txt`

Content:

    uid=0(root) gid=0(root) groups=0(root)

Kali requested it.

Suricata generated:

`SID 2100498`

with:

    Source: 10.10.60.10:80
    Destination: 10.10.10.10
    Direction: to_client

Wazuh generated:

`Rule 86601`

PASS.

The test file was then removed.

---

# DMZ Updates

The server was updated using:

`apt update`

and:

`apt upgrade -y`

Approximately:

`23 packages`

were upgraded.

Afterward:

- SSH active
- Nginx active
- Wazuh agent active

No reboot was required.

---

# Why apt autoremove Was Avoided

The system was already stable.

Old packages or kernels were not removed simply for cosmetic cleanup.

The lab followed the principle:

**Do not introduce unnecessary change immediately before final validation.**

---

# Phase 12 — Final Six-Zone Validation

At this stage, the final architecture contained:

- REDNET
- CORPNET
- SERVERNET
- SOCNET
- ADNET
- DMZ

A full service-level validation was performed.

---

# REDNET Final Validation

REDNET → DMZ HTTP/80:

ALLOW — PASS.

REDNET → DMZ SSH/22:

BLOCK — PASS.

REDNET → CORPNET SMB/445:

BLOCK — PASS.

REDNET → ADNET LDAP/389:

BLOCK — PASS.

---

# CORPNET Final Validation

CORPNET → ADNET DNS/53:

PASS.

CORPNET → ADNET Kerberos/88:

PASS.

CORPNET → ADNET LDAP/389:

PASS.

CORPNET → SOCNET Wazuh/1514:

PASS.

LAB-WIN-01 → DMZ SSH/22:

PASS.

---

# SERVERNET Final Validation

SERVERNET → SOCNET Wazuh/1514:

PASS.

SERVERNET → CORPNET SMB/445:

BLOCK — PASS.

SERVERNET → DMZ SSH/22:

BLOCK — PASS.

---

# ADNET Final Validation

ADNET → SOCNET Wazuh/1514:

PASS.

ADNET → DMZ SSH/22:

BLOCK — PASS.

Domain Controller health:

PASS.

---

# DMZ Final Validation

DMZ → SOCNET Wazuh/1514:

PASS.

DMZ → CORPNET SMB/445:

BLOCK — PASS.

DMZ → ADNET LDAP/389:

BLOCK — PASS.

DMZ → Internet HTTPS/443:

PASS.

---

# Wazuh Final Agent State

Final agents:

    ID 001 — LAB-WIN-01 — Active
    ID 002 — LAB-UBUNTU-01 — Active
    ID 003 — LAB-DC-01 — Active
    ID 004 — LAB-DMZ-01 — Active

---

# Wazuh Stack Final State

Final services:

- Wazuh Manager — Active
- Wazuh Indexer — Active
- Wazuh Dashboard — Active
- Filebeat — Active

Manager version:

`v4.14.7`

---

# SIEM Storage Final State

Final observed storage:

    Filesystem:
    /dev/mapper/ubuntu--vg-ubuntu--lv

    Total:
    53G

    Used:
    25G

    Available:
    26G

    Usage:
    49%

This confirmed the SIEM remained healthy.

---

# Final Detection Coverage

Validated detection capabilities included:

- Linux user creation — Rule 100200
- Linux user deletion — Rule 100201
- OPNsense firewall correlation — Rule 87702
- Suricata alert — Rule 86601
- Nginx web event — Rule 31101
- Windows endpoint telemetry
- Domain Controller security telemetry
- Sysmon
- PowerShell
- auditd
- firewall logs

---

# Phase 13 — Final Shutdown and Cleanup

Once validation completed, the VMs were shut down.

A final shutdown issue occurred on `LAB-UBUNTU-01`.

Shutdown was initially inhibited by:

`unattended-upgrade-shutdown --wait-for-signal`

Process inspection showed no active apt or dpkg package operation.

The unattended-upgrades service was safely stopped.

The server then powered off cleanly.

---

# Soft Lockup Messages

Ubuntu also displayed soft-lockup-style messages during some shutdown activity.

The VM remained responsive.

The symptoms were treated as likely VirtualBox host CPU contention rather than immediate evidence of guest corruption.

Unnecessary kernel changes were avoided.

---

# Snapshot Cleanup

The project had accumulated multiple development snapshots.

Once the lab passed complete six-zone validation, obsolete snapshots were deleted and merged.

The goal was to keep one clean known-good baseline per VM.

---

# Final Snapshot Name

Every VM now retains exactly one snapshot:

`Final-Six-Zone-Validated-Baseline`

---

# Final Snapshot UUIDs

## LAB-FW-01

`d0821c00-02ab-47a9-8341-b6e31001b3b6`

## LAB-KALI-01

`dbb7b8d0-d519-48ef-9e57-7571b912ba92`

## LAB-WIN-01

`36d9a7bc-4556-4f14-9887-959d4a46e65c`

## LAB-UBUNTU-01

`23d97e1a-a95e-4dd9-8912-f48e312ccf74`

## LAB-SIEM-01

`4081e1d5-fcb7-4f40-ad2a-04a081c865ca`

## LAB-DC-01

`dd108acf-e11f-44f2-82f1-c3665d3f3691`

## LAB-DMZ-01

`ac27c1e7-c61f-41fd-ab11-b7e84883068c`

---

# Final Architecture

The final environment is:

    Internet
       |
       v
    VirtualBox NAT
       |
       v
    +----------------------+
    |      LAB-FW-01       |
    |       OPNsense       |
    | Firewall / Routing   |
    |    Suricata IDS      |
    +----------------------+
      |     |      |      |      |      |
      v     v      v      v      v      v
    RED   CORP   SERVER   SOC    AD     DMZ
    NET    NET     NET     NET    NET    NET

Primary systems:

    LAB-KALI-01     10.10.10.10
    LAB-WIN-01      10.10.20.20
    LAB-UBUNTU-01   10.10.30.30
    LAB-SIEM-01     10.10.40.10
    LAB-DC-01       10.10.50.10
    LAB-DMZ-01      10.10.60.10

---

# Final Security Architecture

The final lab provides:

## Network Security

- OPNsense
- six security zones
- least-privilege firewall policy
- logged deny rules
- controlled Internet access

## Endpoint Security

- Sysmon
- PowerShell logging
- advanced Windows auditing
- Defender
- Windows Firewall telemetry
- Linux auditd

## Identity Security

- Active Directory
- Group Policy
- privileged-account separation
- ADNET isolation

## Network Detection

- Suricata
- ET Open rules
- Wazuh integration

## Application Monitoring

- Nginx access logs
- Nginx error logs

## SIEM

- Wazuh
- endpoint agents
- firewall logs
- IDS alerts
- custom detections

---

# Major Engineering Lessons

## Start Simple, Then Segment

The original flat lab was useful for foundational learning.

It was later replaced when it no longer matched the project goals.

---

## Test the Required Service

Ping alone is not enough.

Use:

- TCP port tests
- real application requests
- actual service validation

---

## Rule Order Matters

Specific firewall allows must appear before broader denies.

---

## Save Is Not Apply

In OPNsense, saving a rule does not guarantee the running ruleset has changed.

Always:

- save
- apply
- verify pending changes disappear
- generate fresh traffic

---

## Telemetry Comes Before Detection

Before writing a Wazuh rule:

- confirm the raw event exists
- confirm ingestion
- confirm decoder output

Then build the detection.

---

## Historical Logs Can Mislead

Old connection errors do not automatically represent current state.

Always correlate timestamps.

---

## Do Not Overreact to One Symptom

A live `jq` error did not justify rebuilding a working Suricata pipeline.

A soft-lockup message did not justify immediate kernel changes.

Use end-to-end evidence.

---

## Prefer Built-In Functionality Where Reliable

The Suricata custom decoder was retired when structured JSON and built-in Wazuh rules worked correctly.

---

## Protect High-Value Identity Systems

The Domain Controller was:

- isolated in ADNET
- not used as a DMZ jump host
- monitored with Wazuh
- monitored with Sysmon

---

## Use Defense in Depth

DMZ SSH requires:

- approved firewall source
- TCP/22 allowance
- public-key authentication
- no password authentication
- no root login

---

## Snapshots Are Change-Control Tools

Snapshots were taken before major infrastructure changes.

Once the environment became stable, intermediate snapshots were removed.

---

# Project Maturity Evolution

The complete project can be summarized as:

    Flat Learning Lab
          |
          v
    Nmap / SSH / Hydra
          |
          v
       fail2ban
          |
          v
    Windows / Linux Administration
          |
          v
      OPNsense
          |
          v
    Four-Zone Segmentation
          |
          v
      Wazuh SIEM
          |
          v
    Windows Monitoring
          |
          v
    Linux auditd / Custom Rules
          |
          v
    Firewall Telemetry
          |
          v
      Suricata IDS
          |
          v
    Decoder / rsyslog Engineering
          |
          v
    Active Directory
          |
          v
    Group Policy / DC Monitoring
          |
          v
         DMZ
          |
          v
    Nginx / SSH Hardening / Wazuh
          |
          v
    Six-Zone Validation
          |
          v
    Final Baseline
          |
          v
    SOC / Purple Team Operations

---

# Final Status

The infrastructure build phase is complete.

The lab now provides a realistic environment for:

- SOC analysis
- threat hunting
- detection engineering
- incident investigation
- firewall analysis
- network IDS investigation
- Windows attack simulation
- Linux attack simulation
- Active Directory attack simulation
- DMZ web attack simulation
- Purple Team exercises

The project should no longer be treated as an unfinished infrastructure build.

The next phase is operational.

Future work should focus on realistic scenarios such as:

- reconnaissance
- initial access
- PowerShell abuse
- credential attacks
- privilege escalation
- persistence
- lateral movement
- identity abuse
- web attacks
- detection gaps
- alert triage
- incident response

The infrastructure should only be modified when a future scenario reveals a clear operational requirement.
