# Wazuh SIEM

## Overview

This document records the complete evolution of Wazuh in the Cybersecurity Home Lab, from the period before centralized monitoring existed through the final validated SOC architecture.

The original lab did not have a SIEM.

The earliest environment consisted mainly of:

- Kali Linux
- Windows
- Ubuntu Server

on:

`192.168.56.0/24`

Security validation at that stage relied heavily on:

- local logs
- terminal output
- service state
- attacker retesting
- direct host inspection

As the project matured, this approach became insufficient.

The lab needed centralized visibility across:

- Windows endpoints
- Linux systems
- firewall activity
- network IDS alerts
- Active Directory
- application logs
- endpoint security controls

Wazuh was introduced to provide:

- centralized log collection
- endpoint monitoring
- detection engineering
- event correlation
- alerting
- SOC investigation capability

The final platform runs on:

`LAB-SIEM-01`

Final address:

`10.10.40.10`

Zone:

`SOCNET`

Final Wazuh Manager version:

`v4.14.7`

---

# Phase 1 — Before Wazuh

The earliest lab focused on individual attacker and defender exercises.

Examples included:

- Nmap
- SSH
- Hydra
- fail2ban
- Windows Firewall testing
- connectivity validation

Evidence was reviewed directly on the affected hosts.

For example:

- SSH logs were inspected on Ubuntu
- fail2ban status was checked locally
- Windows firewall behaviour was checked locally
- Nmap results were interpreted from Kali

This was valuable for foundational learning but did not represent how a SOC operates.

---

# Why Wazuh Was Introduced

The project needed a central platform capable of receiving security telemetry from multiple hosts and technologies.

The objectives included:

- centralized endpoint visibility
- log normalization
- custom detection rules
- network-security telemetry
- alert correlation
- incident investigation
- Purple Team validation

Wazuh was selected as the SIEM/XDR platform for the lab.

---

# SOCNET Introduction

A dedicated monitoring zone was created:

`SOCNET`

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

SIEM:

`LAB-SIEM-01 — 10.10.40.10`

This separated the monitoring platform from:

- REDNET
- CORPNET
- SERVERNET
- later ADNET
- later DMZ

SOCNET was intentionally designed as a monitoring destination rather than a general-purpose transit network.

---

# Wazuh Stack

The SIEM stack includes:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

These services collectively provide:

- event ingestion
- alert generation
- indexing
- visualization
- analysis

---

# Initial Endpoint Monitoring

The first major monitoring phase focused on endpoint telemetry.

The initial monitored systems were:

- `LAB-WIN-01`
- `LAB-UBUNTU-01`

The objective was to move from local log review to centralized security analysis.

---

# Windows Agent Integration

The Windows workstation was enrolled into Wazuh.

Final agent identity:

    ID: 001
    Name: LAB-WIN-01
    Status: Active

The Windows endpoint became a major source of telemetry.

---

# Windows Security Logs

Wazuh collected Windows Security events.

This provided visibility into:

- authentication
- privilege use
- account activity
- audit-policy events
- process creation

These logs formed the foundation for Windows monitoring.

---

# Sysmon Integration

Sysmon was added to improve endpoint visibility.

Compared with standard Windows logs, Sysmon provides richer detail around:

- process creation
- parent-child relationships
- suspicious executables
- command activity
- network-related process behaviour

Wazuh successfully ingested Sysmon events.

---

# PowerShell Monitoring

PowerShell logging was expanded to include:

- Script Block Logging
- Module Logging
- transcription

This increased visibility into both legitimate administrative actions and potentially malicious PowerShell execution.

The telemetry was integrated into Wazuh for future Purple Team scenarios.

---

# Advanced Windows Auditing

Advanced auditing was enabled to improve the quality of Windows telemetry.

Important categories included:

- Logon
- Process Creation
- Special Logon

Command-line process auditing was also enabled.

This provided greater context during investigation.

---

# Microsoft Defender Monitoring

Microsoft Defender telemetry was included in the Windows monitoring strategy.

This provided visibility into:

- Defender detections
- remediation events
- operational security events

This complemented Sysmon and standard Windows auditing.

---

# Windows Firewall Monitoring

Windows Firewall events were also incorporated into Wazuh.

One important event was:

`Event ID 5157`

This represents:

`The Windows Filtering Platform has blocked a connection`

Monitoring this event helped distinguish host-level blocking from network-level OPNsense blocking.

---

# File Integrity Monitoring

Wazuh File Integrity Monitoring was configured on the Windows endpoint.

This enabled detection of selected file and configuration changes.

The purpose was to support later scenarios involving:

- persistence
- unauthorized modification
- configuration tampering
- attacker-created artifacts

---

# Linux Agent Integration

The Ubuntu server was enrolled into Wazuh.

Final agent identity:

    ID: 002
    Name: LAB-UBUNTU-01
    Status: Active

This expanded monitoring beyond Windows.

---

# Linux Log Monitoring

The Linux server provided telemetry including:

- authentication logs
- system logs
- administrative activity
- audit events

The system became a monitored enterprise-style Linux asset rather than only a basic attack target.

---

# auditd Integration

`auditd` was enabled and integrated into Wazuh.

This provided deeper Linux visibility into:

- privileged actions
- account changes
- system administration
- security-relevant activity

---

# Linux Detection Engineering

Custom detections were created for native Linux account-management activity.

User creation:

`Rule 100200`

User deletion:

`Rule 100201`

These detections demonstrated that the lab had moved beyond simple log collection into custom detection engineering.

---

# Early Detection Engineering Maturity

At this stage, the lab monitoring architecture had progressed from:

    Local Logs
        |
        v
    Centralized Wazuh
        |
        v
    Endpoint Telemetry
        |
        v
    Custom Detection Rules

This was the first major SOC maturity milestone.

---

# OPNsense Firewall Telemetry

After endpoint monitoring was established, the firewall became a telemetry source.

OPNsense firewall logs were forwarded into Wazuh.

This provided visibility into:

- blocked connections
- inter-zone violations
- suspicious network behaviour
- firewall enforcement

---

# Firewall Correlation Detection

A firewall correlation rule was validated:

`Wazuh Rule 87702`

This demonstrated that the SIEM could process OPNsense firewall events and turn them into actionable detections.

---

# Why Firewall Logs Mattered

Before firewall integration, Wazuh could observe endpoint behaviour but did not have complete visibility into rejected network paths.

Adding OPNsense telemetry made it possible to correlate:

- endpoint events
- firewall blocks
- attacker source addresses
- destination services

This improved investigation depth.

---

# Suricata IDS Introduction

Suricata was deployed on OPNsense after the firewall architecture and Wazuh monitoring were already functional.

This added network-based intrusion detection.

Primary monitored interface:

`em2`

This corresponds to REDNET.

---

# Initial Suricata Ingestion Challenge

Suricata events initially did not decode cleanly in Wazuh.

The first events fell back to:

`Rule 1002`

because no appropriate decoder matched.

This triggered a dedicated troubleshooting phase.

---

# Custom Suricata Decoder Phase

A custom decoder was introduced during troubleshooting.

Observed decoder names included:

`suricata-eve-syslog`

This allowed the project to confirm that Suricata events were reaching Wazuh, but the architecture still required refinement.

---

# Transition to JSON Decoding

The ingestion approach was later simplified.

Suricata events were handled using the built-in:

`json`

decoder.

This reduced unnecessary custom parsing logic.

---

# Built-In Suricata Detection

Wazuh's built-in Suricata ruleset successfully processed the event.

Relevant ruleset file:

`0475-suricata_rules.xml`

Alert:

`Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

This confirmed that the custom wrapper decoder was no longer necessary.

---

# Dedicated Suricata Transport

A dedicated network path was created for Suricata events.

Protocol:

`UDP`

Port:

`5514`

Destination:

`LAB-SIEM-01 — 10.10.40.10`

This separated Suricata telemetry from other log sources.

---

# rsyslog Configuration

The SIEM uses rsyslog to receive the Suricata stream.

Configuration file:

`/etc/rsyslog.d/30-suricata-remote.conf`

The configuration uses:

`imudp`

on:

`5514`

and writes Suricata events to:

`/var/log/suricata/eve.json`

---

# Suricata Log Directory

Directory:

`/var/log/suricata`

Ownership:

`syslog:wazuh`

Permissions:

`750`

This allows:

- rsyslog to write
- Wazuh to read

---

# UDP Listener Validation

The SIEM was confirmed listening on:

`0.0.0.0:5514`

and:

`[::]:5514`

This validated the transport layer before full end-to-end testing.

---

# Final Suricata Pipeline

The final architecture is:

    LAB-FW-01
       |
       v
    Suricata
       |
       v
    Remote Logging
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
    Wazuh Logcollector
       |
       v
    JSON Decoder
       |
       v
    Wazuh Rules

---

# Suricata Test Signature

A controlled IDS test produced:

`Signature ID 2100498`

Signature:

`GPL ATTACK_RESPONSE id check returned root`

Category:

`Potentially Bad Traffic`

Severity:

`2`

This became the primary end-to-end validation signature.

---

# Wazuh Suricata Validation

Wazuh generated:

`Rule 86601`

for the controlled Suricata event.

This confirmed:

- IDS detection
- event forwarding
- rsyslog reception
- JSON ingestion
- Wazuh decoding
- alert generation

---

# full_log Behaviour

During investigation, a Suricata Wazuh alert showed:

`full_log: null`

This did not mean the event was missing.

Structured fields were still present and usable.

The correct investigation approach became:

- search by rule ID
- search by location
- inspect parsed JSON fields

rather than depending exclusively on `full_log`.

---

# Live JSON Parsing Lesson

Direct `jq` operations against actively written JSON files occasionally returned parsing errors.

This did not automatically indicate that the ingestion pipeline was broken.

Because valid Wazuh alerts continued to appear, the issue was treated as a live-file parsing condition rather than evidence of total corruption.

This reinforced a broader troubleshooting principle:

**Validate the full pipeline before diagnosing failure from one isolated symptom.**

---

# Active Directory Monitoring

After ADNET was introduced, the Domain Controller was enrolled into Wazuh.

Final agent:

    ID: 003
    Name: LAB-DC-01
    Status: Active

This expanded SIEM visibility into identity infrastructure.

---

# Active Directory Security Events

Important observed events included:

- `5136`
- `4719`
- `4672`

These provided visibility into:

- directory modifications
- audit-policy changes
- privileged logons

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

Wazuh successfully ingested the Domain Controller Sysmon logs.

---

# Domain Controller Sysmon Tuning

The configuration was tuned to focus on security-relevant executables.

Process-create coverage included:

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

DNS monitoring was similarly narrowed to relevant processes.

This reduced noise while maintaining high-value telemetry.

---

# DMZ Monitoring

The final endpoint added to Wazuh was:

`LAB-DMZ-01`

Agent:

    ID: 004
    Name: LAB-DMZ-01
    Status: Active

The DMZ added application and public-facing server telemetry to the SIEM.

---

# DMZ Wazuh Connectivity Validation

Before agent installation, the required network paths were tested.

Ports:

- `1514`
- `1515`

Both were reachable from the DMZ.

This ensured the firewall path was correct before software deployment.

---

# Nginx Monitoring

The Wazuh agent on the DMZ monitors:

`/var/log/nginx/access.log`

with:

`apache`

log format.

It also monitors:

`/var/log/nginx/error.log`

with:

`syslog`

log format.

---

# Nginx Detection Validation

A controlled request generated:

`GET /wazuh-nginx-verify-01 HTTP/1.1`

Source:

`10.10.10.10`

Response:

`404`

Wazuh generated:

`Rule 31101`

Description:

`Web server 400 error code.`

The event included:

- agent `LAB-DMZ-01`
- source IP `10.10.10.10`
- decoder `web-accesslog`

This confirmed application-layer monitoring.

---

# SIEM Recovery

At one point, `LAB-SIEM-01` experienced a storage-related issue.

The system required recovery.

The SIEM successfully returned to a healthy bootable state.

---

# Storage Architecture

The main logical volume is:

`/dev/mapper/ubuntu--vg-ubuntu--lv`

An observed healthy recovery state showed approximately:

    Total: 53G
    Used: 21G
    Free: 30G

Later final validation showed:

    Total: 53G
    Used: 25G
    Available: 26G
    Usage: 49%

This confirmed sufficient storage remained.

---

# System Health After Recovery

After recovery:

- LVM was active
- the logical volume was healthy
- the system booted normally
- no failed systemd services remained
- Wazuh services were operational

---

# Snapshot Recovery Decision

An older SIEM snapshot existed.

However, restoring it would have rolled back later configuration and telemetry work.

The decision was made:

**Do not restore the old SIEM snapshot.**

The recovered current system became authoritative.

This avoided losing:

- later Wazuh configuration
- Suricata integration
- detection engineering
- agent onboarding

---

# Wazuh Stack Validation

Final service status:

- Wazuh Manager — Active
- Wazuh Indexer — Active
- Wazuh Dashboard — Active
- Filebeat — Active

This confirmed the complete platform was operational.

---

# Agent Connectivity Troubleshooting

At one point, the Ubuntu agent appeared disconnected.

Rather than assuming a current outage, the investigation compared:

- log timestamps
- current agent state
- service connectivity
- manager status

Recent logs showed successful connection.

The manager later displayed:

`Active`

This reinforced:

**Historical errors are not necessarily current errors.**

---

# Service-Level Connectivity

The lab deliberately moved away from relying only on ping.

For example:

- ICMP could fail
- TCP/1514 could still succeed

If Wazuh's required service port was reachable, the telemetry path could still be healthy.

This became a standard validation principle.

---

# Final Agent Inventory

The final Wazuh environment contains:

    ID 001 — LAB-WIN-01 — Active
    ID 002 — LAB-UBUNTU-01 — Active
    ID 003 — LAB-DC-01 — Active
    ID 004 — LAB-DMZ-01 — Active

This provides monitoring coverage across all major enterprise asset types in the lab.

---

# Final Telemetry Sources

Wazuh now receives data from multiple layers.

## Windows Endpoint

- Security events
- Sysmon
- PowerShell
- Defender
- Windows Firewall
- File Integrity Monitoring

## Linux Server

- Linux logs
- authentication
- auditd
- account-change events

## Domain Controller

- Security events
- Active Directory changes
- privileged logons
- Sysmon

## DMZ

- Linux logs
- Nginx access logs
- Nginx error logs

## Network Security

- OPNsense firewall logs
- Suricata IDS events

This provides broad SOC visibility.

---

# Detection Examples

Validated detections include:

## Firewall

`Rule 87702`

Purpose:

OPNsense firewall correlation.

---

## Suricata

`Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

---

## Linux User Creation

`Rule 100200`

---

## Linux User Deletion

`Rule 100201`

---

## Nginx Web Error

`Rule 31101`

Description:

`Web server 400 error code.`

---

# Detection Architecture

The lab can now correlate activity across multiple layers.

Example:

    Attacker
       |
       v
    OPNsense
       |
       +--> Firewall Log
       |
       +--> Suricata Alert
       |
       v
    Target
       |
       +--> Endpoint Log
       |
       +--> Application Log
       |
       v
    Wazuh
       |
       v
    Detection / Investigation

This is one of the key differences between the original lab and the final SOC environment.

---

# Final Wazuh Validation

| Validation | Result |
|---|---|
| Manager active | PASS |
| Indexer active | PASS |
| Dashboard active | PASS |
| Filebeat active | PASS |
| LAB-WIN-01 active | PASS |
| LAB-UBUNTU-01 active | PASS |
| LAB-DC-01 active | PASS |
| LAB-DMZ-01 active | PASS |
| Windows telemetry | PASS |
| Linux telemetry | PASS |
| auditd | PASS |
| Sysmon | PASS |
| PowerShell logging | PASS |
| Defender monitoring | PASS |
| Firewall logs | PASS |
| Firewall Rule 87702 | PASS |
| Suricata UDP/5514 | PASS |
| Suricata Rule 86601 | PASS |
| Linux Rule 100200 | PASS |
| Linux Rule 100201 | PASS |
| Nginx Rule 31101 | PASS |
| AD telemetry | PASS |
| SIEM storage health | PASS |

---

# Snapshot History

Wazuh configuration changed significantly throughout the project.

Snapshots were used before:

- initial deployment
- endpoint onboarding
- detection engineering
- firewall integration
- Suricata decoder work
- rsyslog changes
- Active Directory onboarding
- DMZ onboarding

After the storage recovery, the live SIEM state became authoritative.

Once the entire lab passed final validation, obsolete snapshots were removed.

The final remaining snapshot is:

`Final-Six-Zone-Validated-Baseline`

---

# Historical vs Current State

The Wazuh documentation intentionally preserves every major stage.

Historical stages include:

- no SIEM
- Windows-only monitoring
- Linux monitoring
- early detection engineering
- firewall-log integration
- failed Suricata decoding
- custom decoder experimentation
- transition to JSON
- Active Directory integration
- DMZ and Nginx integration
- SIEM recovery

The authoritative current state is:

    Server: LAB-SIEM-01
    IP: 10.10.40.10
    Zone: SOCNET
    Manager Version: v4.14.7
    Agents: 4 Active
    Firewall Telemetry: Active
    Suricata Telemetry: Active
    Nginx Monitoring: Active
    Endpoint Monitoring: Active
    Identity Monitoring: Active

---

# Evolution Summary

The SIEM capability evolved through:

    No Central Monitoring
          |
          v
    Wazuh Deployment
          |
          v
    Windows Endpoint Monitoring
          |
          v
    Sysmon / PowerShell / Defender
          |
          v
    Linux Monitoring
          |
          v
    auditd / Custom Rules
          |
          v
    OPNsense Firewall Telemetry
          |
          v
    Suricata IDS Integration
          |
          v
    Active Directory Monitoring
          |
          v
    DMZ / Nginx Monitoring
          |
          v
    Full SOC Monitoring Platform

---

# Final Status

The Wazuh SIEM phase is complete.

The platform now provides centralized visibility into:

- endpoints
- servers
- identity systems
- firewall activity
- IDS alerts
- web-server activity

The SIEM is healthy, validated, and ready for operational use.

Future work should focus on:

- alert triage
- threat hunting
- incident investigation
- detection engineering
- Purple Team exercises
- attack correlation
- response workflows

rather than further baseline Wazuh infrastructure changes.
