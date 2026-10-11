# LAB-SIEM-01 — Wazuh Security Information and Event Management Server

## Overview

This document records the complete lifecycle of the SIEM platform used throughout the Cybersecurity Home Lab, from its initial deployment through its final role as `LAB-SIEM-01` in the validated six-zone architecture.

The SIEM did not exist in the earliest flat-network version of the lab.

It was introduced later when the project began moving beyond isolated endpoint testing toward centralized monitoring, detection engineering, network telemetry, and realistic SOC workflows.

Final host:

`LAB-SIEM-01`

Final address:

`10.10.40.10`

Final zone:

`SOCNET`

Primary platform:

`Wazuh`

Final Wazuh version:

`4.14.7`

Primary role:

Centralized security monitoring, event collection, detection, correlation, and SOC analysis

---

# Phase 1 — Before the SIEM

The earliest version of the lab used a simple network based around:

`192.168.56.0/24`

Primary systems included:

- Kali Linux — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu Server — `192.168.56.30`

Security testing at this stage focused mainly on:

- Nmap
- SSH
- Hydra
- fail2ban
- connectivity
- local host logs
- basic attacker/defender interaction

There was no centralized SIEM.

Evidence had to be examined directly on each system.

This became increasingly limiting as the lab matured.

---

# Why a SIEM Was Added

As the project expanded, there was a need to centralize security telemetry from multiple sources.

The lab needed a platform capable of collecting and analyzing:

- Windows security events
- Sysmon
- PowerShell activity
- Linux logs
- auditd
- Microsoft Defender events
- Windows Firewall events
- OPNsense firewall logs
- Suricata IDS alerts
- Nginx access and error logs
- Active Directory events

Wazuh was introduced to provide this capability.

This marked the transition from individual host security testing into a true SOC-oriented lab.

---

# SOCNET Introduction

The SIEM was placed in its own dedicated security zone:

`SOCNET`

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

SIEM address:

`10.10.40.10`

This design separated monitoring infrastructure from:

- attacker systems
- corporate endpoints
- internal servers
- identity infrastructure
- public-facing systems

---

# LAB-SIEM-01 Role

The SIEM performs several core functions:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat
- endpoint agent management
- security event ingestion
- detection rules
- log correlation
- firewall telemetry ingestion
- Suricata ingestion
- centralized investigation

---

# Initial Wazuh Deployment

The Wazuh stack was deployed on Ubuntu Server.

The core components included:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

The system was configured as the central monitoring destination for the rest of the lab.

---

# Initial Network Position

Once SOCNET was created, the SIEM used:

`10.10.40.10`

This became the fixed destination used throughout the lab for security telemetry.

Important Wazuh communication ports included:

- TCP `1514`
- TCP `1515`

These were later incorporated into OPNsense firewall aliases.

---

# WAZUH_SERVER Alias

OPNsense uses the alias:

`WAZUH_SERVER`

Address:

`10.10.40.10`

This allows firewall rules to reference the SIEM by function rather than by raw IP address.

---

# WAZUH_AGENT_PORTS Alias

The firewall also uses:

`WAZUH_AGENT_PORTS`

Ports:

- `1514`
- `1515`

This provides a readable and reusable representation of the required Wazuh communication services.

---

# Windows Agent Onboarding

`LAB-WIN-01` became the first major endpoint integrated with the SIEM.

Final Windows agent:

    ID: 001
    Name: LAB-WIN-01
    Status: Active

The Windows endpoint later supplied telemetry including:

- Security event logs
- Sysmon
- PowerShell
- Defender
- Windows Firewall
- File Integrity Monitoring
- advanced auditing

This made Windows the first major endpoint source for detection engineering.

---

# Linux Agent Onboarding

`LAB-UBUNTU-01` was also integrated into Wazuh.

Final Linux agent:

    ID: 002
    Name: LAB-UBUNTU-01
    Status: Active

Linux monitoring eventually included:

- system logs
- authentication logs
- auditd
- user-management events
- custom detection rules

---

# Linux Detection Engineering

The SIEM was used to detect native Linux administrative events.

Custom detections included:

`Rule 100200`

for native user creation.

And:

`Rule 100201`

for native user deletion.

These detections proved that the SIEM could support custom Linux detection engineering rather than relying only on built-in rules.

---

# Windows Detection Engineering

The Windows endpoint was also used to develop and validate security monitoring.

Important monitored categories included:

- process creation
- PowerShell
- privilege use
- Defender events
- Windows Filtering Platform events
- file integrity changes

This helped transform the SIEM from a simple log collector into an active detection platform.

---

# Windows Firewall Event Monitoring

Windows Firewall telemetry included:

`Event ID 5157`

This event represents a connection blocked by the Windows Filtering Platform.

Monitoring it helped distinguish:

- endpoint firewall blocks
- OPNsense firewall blocks

This improved investigation quality.

---

# Sysmon Integration

Sysmon telemetry from Windows was collected through the Wazuh agent.

The data helped provide:

- process lineage
- command execution
- suspicious binary execution
- richer host activity context

Sysmon later became important both on the workstation and the Domain Controller.

---

# PowerShell Logging Integration

The SIEM collected enhanced PowerShell telemetry including:

- Script Block Logging
- Module Logging
- transcription-related evidence

This provided visibility into administrative and potentially malicious PowerShell execution.

---

# OPNsense Firewall Log Integration

After endpoint monitoring was established, the firewall itself became a security data source.

OPNsense logs were forwarded into the SIEM.

This provided centralized visibility into:

- blocked traffic
- segmentation violations
- unauthorized inter-zone attempts
- firewall rule behaviour

---

# Firewall Correlation Rule

A Wazuh firewall correlation rule was successfully validated:

`Rule 87702`

This proved that the SIEM could process firewall telemetry and generate meaningful detections from network enforcement events.

---

# Suricata Introduction

Suricata IDS was introduced later on OPNsense.

This added network-layer detection to the lab.

The primary monitored interface was:

`em2`

which corresponds to REDNET.

Suricata generated EVE JSON events.

The next challenge was to transport those events reliably into Wazuh.

---

# Initial Suricata Decoder Troubleshooting

At first, Suricata events did not decode correctly in Wazuh.

The events initially fell through to:

`Rule 1002`

because no suitable decoder matched the event format.

A custom Suricata decoder was then introduced.

During testing, decoder names observed included:

`suricata-eve-syslog`

and later:

`json`

This phase involved multiple iterations to determine the cleanest ingestion architecture.

---

# Built-In Suricata Rule

Once Wazuh correctly parsed the event, it generated:

`Rule 86601`

This came from:

`0475-suricata_rules.xml`

The alert corresponded to:

`GPL ATTACK_RESPONSE id check returned root`

This demonstrated that built-in Wazuh Suricata detection logic could be used without maintaining unnecessary custom wrappers.

---

# Retirement of the Custom Wrapper Decoder

Once JSON parsing worked properly, the custom Suricata wrapper approach was no longer required.

The architecture was simplified so that Suricata events could be ingested directly as JSON.

This reduced complexity and made the solution easier to maintain.

---

# Dedicated Suricata Syslog Port

A dedicated UDP port was created for Suricata telemetry:

`UDP/5514`

This kept Suricata traffic separate from other syslog sources.

The SIEM was configured to listen using rsyslog.

---

# rsyslog Configuration

A dedicated rsyslog configuration was created:

`/etc/rsyslog.d/30-suricata-remote.conf`

The configuration used:

`imudp`

on:

`5514`

Events matching the Suricata program name were written to:

`/var/log/suricata/eve.json`

---

# Suricata Log Directory

The directory:

`/var/log/suricata`

was created.

Ownership:

`syslog:wazuh`

Permissions:

`750`

The EVE file used permissions appropriate for rsyslog writing and Wazuh reading.

---

# rsyslog Listener Validation

The SIEM was confirmed listening on:

`0.0.0.0:5514`

and:

`[::]:5514`

This proved the receiver was ready before final OPNsense forwarding validation.

---

# Final Suricata Telemetry Path

The final architecture became:

    LAB-FW-01
       |
       v
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
    Wazuh Logcollector
       |
       v
    JSON Decoder
       |
       v
    Wazuh Detection

This is the authoritative Suricata ingestion architecture.

---

# Wazuh Localfile Configuration

Wazuh was configured to monitor:

`/var/log/suricata/eve.json`

using:

`json`

log format.

This allowed the event fields to be parsed directly.

---

# Suricata Detection Validation

A controlled test generated:

`Signature ID 2100498`

Signature:

`GPL ATTACK_RESPONSE id check returned root`

Wazuh generated:

`Rule 86601`

This confirmed the complete network detection pipeline.

---

# Important alerts.json Lesson

During investigation, the Suricata alert in Wazuh showed:

`full_log: null`

This initially appeared unusual.

However, the event still contained the structured JSON fields required for detection.

The correct search method therefore became:

- search by rule ID
- search by location
- search by structured event fields

rather than assuming `full_log` must always contain the original message.

---

# Live JSON Parsing Lesson

Direct `jq` testing against actively written log files occasionally produced parsing errors.

This did not automatically indicate corruption.

The likely explanation was reading the file while a line was still being written or encountering a malformed/partial live record.

Because valid Wazuh alerts continued to be generated, the pipeline itself was not considered broken.

This reinforced an important troubleshooting principle:

**Do not treat one parsing symptom as proof of an end-to-end ingestion failure when validated events are still reaching the SIEM.**

---

# Active Directory Integration

After ADNET was created, the Domain Controller was enrolled into Wazuh.

Final agent:

    ID: 003
    Name: LAB-DC-01
    Status: Active

The SIEM gained visibility into:

- Windows Security events
- privileged activity
- Active Directory changes
- Sysmon events
- Group Policy-related activity
- security auditing

---

# Domain Controller Event Monitoring

Important events observed included:

- Event ID `5136`
- Event ID `4719`
- Event ID `4672`

These events contributed visibility into:

- directory modifications
- audit policy changes
- privileged logon activity

---

# Domain Controller Sysmon

Sysmon was later installed on:

`LAB-DC-01`

Version observed:

`15.22`

Binary:

`C:\Tools\Sysmon\Sysmon64.exe`

Configuration file:

`sysmon-dc.xml`

The SIEM successfully received Domain Controller Sysmon telemetry.

---

# Domain Controller Sysmon Tuning

The Sysmon configuration was tuned to reduce noise and focus on security-relevant execution.

Process-create monitoring focused on tools such as:

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

DNS monitoring was similarly focused on relevant executables.

This reduced unnecessary event volume while preserving useful detection value.

---

# DMZ Integration

The final endpoint added to Wazuh was:

`LAB-DMZ-01`

Final agent:

    ID: 004
    Name: LAB-DMZ-01
    Status: Active

The DMZ agent provided visibility into:

- Linux host events
- Nginx access logs
- Nginx error logs
- web activity

---

# Pre-Installation Network Validation

Before installing the Wazuh agent on the DMZ server, connectivity was tested to:

`10.10.40.10`

Ports:

- `1514`
- `1515`

Both succeeded.

This confirmed firewall policy before adding the agent.

---

# Nginx Log Integration

The DMZ server forwarded Nginx logs through the Wazuh agent.

Configured log files included:

`/var/log/nginx/access.log`

using:

`apache`

log format.

And:

`/var/log/nginx/error.log`

using:

`syslog`

log format.

---

# Nginx Detection Validation

A controlled request was generated:

`GET /wazuh-nginx-verify-01 HTTP/1.1`

Source:

`10.10.10.10`

The request returned:

`404`

Wazuh generated:

`Rule 31101`

Description:

`Web server 400 error code.`

Additional event details included:

- Agent ID: `004`
- Agent: `LAB-DMZ-01`
- Source IP: `10.10.10.10`
- Decoder: `web-accesslog`
- Location: Nginx access log

This proved the web monitoring pipeline.

---

# Final Agent Inventory

The final SIEM agent state is:

    ID 001 — LAB-WIN-01 — Active
    ID 002 — LAB-UBUNTU-01 — Active
    ID 003 — LAB-DC-01 — Active
    ID 004 — LAB-DMZ-01 — Active

This confirms monitoring coverage across:

- corporate workstation
- internal Linux server
- Domain Controller
- DMZ server

---

# SIEM Recovery Event

At one stage, the SIEM experienced a storage-related problem and required recovery.

The system was successfully restored to a healthy bootable state.

The LVM layout included:

`/dev/mapper/ubuntu--vg-ubuntu--lv`

An observed healthy state showed:

    Total: 53G
    Used: approximately 21G
    Free: approximately 30G

Later final validation showed:

    Total: 53G
    Used: 25G
    Available: 26G
    Usage: 49%

This confirmed adequate remaining capacity.

---

# Recovery Validation

After recovery, the SIEM was checked for:

- filesystem health
- LVM state
- failed services
- Wazuh stack status

The volume group and logical volume were active.

No failed systemd services remained.

The system booted normally.

---

# Snapshot Decision After Recovery

An important decision was made:

**Do not restore the older SIEM snapshot.**

The recovered live system was healthier and more current than the old snapshot.

Restoring an older snapshot would have risked:

- losing later configuration
- losing detection work
- reintroducing outdated state

The recovered system therefore became the continuing authoritative SIEM.

---

# Wazuh Stack Validation

The final stack was checked and confirmed active.

Services:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Status:

**ACTIVE**

---

# Wazuh Manager Version

Final Wazuh Manager version:

`v4.14.7`

This became the reference version when installing the DMZ agent.

Version compatibility was checked before deployment.

---

# SOCNET Firewall Role

SOCNET is intentionally not used as a general-purpose internal network.

Its purpose is security monitoring.

Approved traffic primarily includes telemetry to:

`LAB-SIEM-01`

Examples:

    CORPNET → SOCNET Wazuh
    SERVERNET → SOCNET Wazuh
    ADNET → SOCNET Wazuh
    DMZ → SOCNET Wazuh
    OPNsense → SOCNET firewall telemetry
    OPNsense → SOCNET Suricata telemetry

This helps keep monitoring infrastructure isolated.

---

# Service-Level Validation

The project avoided relying only on ping.

For example, Ubuntu at one stage could not ping the SIEM but could reach:

`TCP/1514`

The Wazuh path was therefore considered healthy.

This reinforced:

**Test the service you actually require.**

---

# Agent Status Troubleshooting

At one stage, the Ubuntu agent appeared disconnected.

Investigation compared:

- current manager status
- timestamps
- recent agent logs
- network connectivity

Recent logs showed successful connection and the manager later reported the agent as Active.

This demonstrated the danger of interpreting old log messages as current failures.

---

# Final Suricata Validation

The final controlled REDNET-to-DMZ test generated the known response:

`uid=0(root) gid=0(root) groups=0(root)`

Suricata detected the response using:

`SID 2100498`

Wazuh subsequently generated:

`Rule 86601`

This proved the complete IDS-to-SIEM chain.

---

# Final Firewall Monitoring Validation

OPNsense firewall telemetry had previously been validated through:

`Wazuh Rule 87702`

This ensured that the SIEM could observe both:

- allowed/detected IDS traffic
- blocked firewall traffic

---

# Final Monitoring Coverage

The SIEM now receives telemetry from multiple security layers.

## Endpoint

- Windows
- Linux
- Domain Controller
- DMZ

## Host Security

- Sysmon
- PowerShell
- auditd
- Defender
- Windows Firewall
- File Integrity Monitoring

## Network Security

- OPNsense firewall logs
- Suricata IDS

## Application

- Nginx access logs
- Nginx error logs

## Identity

- Active Directory security events

This produces a much richer SOC environment than the original endpoint-only lab.

---

# Defense-in-Depth Visibility

The final monitoring architecture can be represented as:

    Attacker / User Activity
              |
              v
        Network Traffic
              |
              v
        LAB-FW-01
        OPNsense
          |      |
          |      +--> Firewall Logs
          |
          +--> Suricata IDS
                    |
                    v
               UDP/5514
                    |
                    v
             LAB-SIEM-01
                    ^
                    |
    +---------------+---------------+
    |               |               |
    |               |               |
LAB-WIN-01   LAB-UBUNTU-01    LAB-DC-01
    |               |               |
    +---------------+---------------+
                    |
                    |
              LAB-DMZ-01
                    |
                    v
              Wazuh Agents

This centralizes host, network, identity, and application telemetry.

---

# Final Validation Summary

| Validation | Result |
|---|---|
| Wazuh Manager active | PASS |
| Wazuh Indexer active | PASS |
| Wazuh Dashboard active | PASS |
| Filebeat active | PASS |
| Windows agent active | PASS |
| Ubuntu agent active | PASS |
| Domain Controller agent active | PASS |
| DMZ agent active | PASS |
| Firewall logs → Wazuh | PASS |
| Firewall Rule 87702 | PASS |
| Suricata UDP/5514 listener | PASS |
| Suricata EVE JSON ingestion | PASS |
| Suricata Rule 86601 | PASS |
| Linux user-add Rule 100200 | PASS |
| Linux user-delete Rule 100201 | PASS |
| Nginx Rule 31101 | PASS |
| Domain Controller telemetry | PASS |
| SIEM storage health | PASS |
| No failed systemd services after recovery | PASS |

---

# Snapshot History

The SIEM accumulated multiple snapshots throughout the project because of its central role.

Snapshots were especially important before:

- Wazuh configuration changes
- storage changes
- firewall telemetry integration
- Suricata ingestion work
- decoder modifications
- rsyslog configuration
- Active Directory onboarding
- DMZ onboarding

However, after the SIEM recovery, the live recovered system became authoritative.

The project explicitly avoided restoring an older SIEM snapshot.

Once final six-zone validation was completed, obsolete development snapshots were removed.

The remaining snapshot is:

`Final-Six-Zone-Validated-Baseline`

Snapshot UUID:

`4081e1d5-fcb7-4f40-ad2a-04a081c865ca`

---

# Historical vs Current SIEM State

This document intentionally preserves the full monitoring evolution.

Historical phases include:

- no SIEM
- endpoint-only Wazuh monitoring
- Windows telemetry
- Linux telemetry
- firewall-log integration
- early Suricata decoder failures
- custom Suricata decoder testing
- JSON decoder migration
- rsyslog UDP/5514
- Active Directory integration
- DMZ integration
- SIEM storage recovery

The authoritative current state is:

    Hostname: LAB-SIEM-01
    IP: 10.10.40.10
    Zone: SOCNET
    Wazuh Manager: v4.14.7
    Manager: Active
    Indexer: Active
    Dashboard: Active
    Filebeat: Active
    Endpoint Agents: 4 Active
    Suricata Ingestion: Active
    Firewall Telemetry: Active
    Nginx Monitoring: Active

Earlier stages remain documented because they represent the actual engineering evolution of the project.

---

# Evolution Summary

The SIEM evolved through the following maturity path:

    No Central Monitoring
          |
          v
    Wazuh Deployment
          |
          v
    Windows Monitoring
          |
          v
    Sysmon / PowerShell / Defender
          |
          v
    Linux Monitoring
          |
          v
    auditd / Custom Linux Rules
          |
          v
    OPNsense Firewall Logs
          |
          v
    Suricata IDS
          |
          v
    JSON / rsyslog Integration
          |
          v
    Active Directory Monitoring
          |
          v
    DMZ / Nginx Monitoring
          |
          v
    Full SOC Telemetry Platform

---

# Final Status

The SIEM infrastructure phase is complete.

`LAB-SIEM-01` now provides centralized monitoring for:

- Windows endpoints
- Linux servers
- Active Directory
- DMZ systems
- firewall events
- Suricata IDS
- Nginx web activity

The platform is healthy, validated, and operating as the central SOC system for the Cybersecurity Home Lab.

Future work should focus on:

- realistic SOC investigations
- Purple Team attack simulations
- detection engineering
- alert triage
- correlation
- incident response

rather than further baseline SIEM infrastructure changes.
