# Windows Advanced Auditing

## Overview

This document records the complete evolution of Windows advanced auditing in the Cybersecurity Home Lab, from the earliest standalone Windows endpoint through the final domain-managed, Wazuh-integrated monitoring architecture.

Advanced auditing was not part of the original flat lab.

The earliest Windows environment focused primarily on:

- basic connectivity
- Windows Firewall
- local event logs
- endpoint administration
- attacker/defender testing

As the project matured, the need for richer security telemetry became clear.

The Windows environment therefore evolved to include:

- advanced audit policy
- logon auditing
- process creation auditing
- command-line process capture
- special logon auditing
- Windows Firewall event monitoring
- privilege-related events
- Active Directory change auditing
- Group Policy enforcement
- Wazuh ingestion

The final audited Windows systems are:

- `LAB-WIN-01`
- `LAB-DC-01`

Both provide security telemetry to:

`LAB-SIEM-01`

---

# Phase 1 — Before Advanced Auditing

The original Windows system operated on:

`192.168.56.20`

within the early flat lab.

At this stage, Windows logging relied mainly on:

- standard Security event logs
- basic Windows Firewall behaviour
- manual Event Viewer review
- direct endpoint observation

This was sufficient for introductory work but not for a mature SOC environment.

Important questions could not always be answered clearly, such as:

- Which process executed?
- Which user launched it?
- Was the logon privileged?
- What command line was used?
- Was a security policy changed?
- Was an Active Directory object modified?

This created the need for enhanced auditing.

---

# Why Advanced Auditing Was Added

As Wazuh became the central SIEM, Windows needed to generate higher-value security events.

The objectives included:

- improved logon visibility
- process execution visibility
- privilege monitoring
- audit-policy change detection
- firewall-block visibility
- Active Directory change auditing
- better SOC investigation context
- support for Purple Team exercises

The goal was not simply to generate more events.

The goal was to generate more useful events.

---

# Transition to LAB-WIN-01

The Windows endpoint was rebuilt and moved into:

`CORPNET`

Final host:

`LAB-WIN-01`

Final address:

`10.10.20.20`

Final role:

Corporate Windows workstation

The endpoint was enrolled into Wazuh as:

    ID: 001
    Name: LAB-WIN-01
    Status: Active

This created the foundation for centralized auditing.

---

# Advanced Audit Policy

Advanced auditing was introduced to provide more granular security visibility than the default Windows audit configuration.

The final workstation baseline includes important categories such as:

- Logon
- Process Creation
- Special Logon

These categories were selected because they provide strong value during investigation.

---

# Logon Auditing

Logon auditing was configured for:

- Success
- Failure

This provides visibility into:

- successful user sessions
- failed authentication attempts
- suspicious logon behaviour
- repeated failures
- potentially compromised accounts

This is particularly useful when correlating:

- user activity
- process execution
- network behaviour

---

# Why Failed Logons Matter

Repeated failed authentication attempts may indicate:

- password guessing
- brute-force attempts
- stale credentials
- misconfigured services
- user error

Centralizing these events in Wazuh allows the SOC to distinguish between normal mistakes and suspicious patterns.

---

# Why Successful Logons Matter

Successful logons provide context for later activity.

For example:

    Successful Logon
          |
          v
    Privileged Session
          |
          v
    Process Execution
          |
          v
    Suspicious PowerShell
          |
          v
    Wazuh Investigation

Without the logon event, process activity may lack user context.

---

# Special Logon Auditing

Special Logon auditing was enabled.

An important event in this category is:

`Event ID 4672`

Description:

`Special privileges assigned to new logon`

This event can indicate that a user session received elevated privileges.

---

# Event ID 4672

Event `4672` is particularly valuable when investigating:

- administrators
- Domain Admins
- privileged service accounts
- elevated sessions
- lateral movement

It does not automatically indicate malicious activity.

Its value comes from correlation with other events.

---

# Process Creation Auditing

Process creation auditing was enabled.

This allows Windows to record when new processes start.

Process execution is one of the most valuable sources of endpoint telemetry because many attacker actions ultimately involve launching a process.

---

# Command-Line Process Creation

The lab also enabled:

`Include command line in process creation events`

This significantly improves investigation value.

Instead of seeing only:

    powershell.exe

an analyst may see how PowerShell was actually launched.

Likewise, instead of only:

    cmd.exe

the command-line arguments provide additional context.

---

# Why Command-Line Capture Matters

Executable names alone can be ambiguous.

For example:

    powershell.exe

could represent:

- normal administration
- a security script
- reconnaissance
- malware execution

Command-line arguments can help distinguish these possibilities.

---

# Process Auditing and Sysmon

Windows process auditing was not used as a replacement for Sysmon.

Both were retained.

Windows auditing provides one source.

Sysmon provides another.

This creates overlapping evidence.

Example:

    Process Execution
        |
        +--> Windows Security Event
        |
        +--> Sysmon Event
        |
        v
    Wazuh

This allows analysts to validate suspicious activity using multiple sources.

---

# PowerShell Correlation

Process auditing became particularly useful when combined with PowerShell logging.

A suspicious PowerShell sequence can produce:

- process creation event
- command-line information
- Script Block Logging
- Module Logging
- Sysmon process event
- transcription

This greatly improves investigation quality.

---

# Windows Firewall Monitoring

Windows Firewall telemetry was also incorporated into the auditing strategy.

One important event is:

`Event ID 5157`

Description:

`The Windows Filtering Platform has blocked a connection`

This event indicates that the host firewall blocked traffic.

---

# Why Event 5157 Matters

In a segmented environment, failed connectivity can be caused by multiple controls.

Possible causes include:

- OPNsense
- Windows Firewall
- application state
- routing
- service failure

Event `5157` helps identify when the Windows endpoint itself blocked the connection.

---

# Network vs Host Firewall Visibility

The final lab therefore has visibility at two firewall layers.

## Network Layer

`LAB-FW-01`

OPNsense

## Host Layer

Windows Filtering Platform

This allows analysts to determine whether a blocked connection failed:

- before reaching the host
- at the host itself

---

# Windows Defender Relationship

Microsoft Defender monitoring was included alongside advanced auditing.

This means suspicious activity may generate evidence from:

- Windows Security logs
- Sysmon
- PowerShell
- Defender
- Windows Firewall

This layered approach improves confidence during investigation.

---

# File Integrity Monitoring Relationship

Wazuh File Integrity Monitoring was also enabled on the Windows endpoint.

A suspicious process may therefore be correlated with:

- file creation
- file modification
- registry or configuration changes
- process events

This provides stronger detection context.

---

# Wazuh Integration

Windows audit events are collected through the Wazuh agent.

Final workstation agent:

    ID: 001
    Name: LAB-WIN-01
    Status: Active

Destination:

`LAB-SIEM-01`

Address:

`10.10.40.10`

Primary communication:

`TCP/1514`

---

# Workstation Audit Architecture

The final workstation monitoring path is:

    User / Process Activity
            |
            v
      LAB-WIN-01
            |
            +--> Security Logs
            |
            +--> Advanced Audit Events
            |
            +--> Process Creation
            |
            +--> Special Logon
            |
            +--> Windows Firewall
            |
            +--> Sysmon
            |
            +--> PowerShell
            |
            +--> Defender
            |
            v
        Wazuh Agent
            |
            v
       LAB-SIEM-01

---

# Active Directory Introduction

Active Directory was introduced later in the project.

Domain:

`cyberlab.internal`

Domain Controller:

`LAB-DC-01`

The workstation became domain joined.

This allowed workstation auditing settings to move from local configuration into centralized Group Policy.

---

# Group Policy Baseline

The security baseline created for the workstation is:

`GPO-Workstation-Security-Baseline`

Linked to:

`CyberLab\Workstations`

The baseline includes auditing settings.

---

# Group Policy Audit Settings

The final GPO includes:

## Logon

    Success
    Failure

## Process Creation

    Success

## Special Logon

    Success

It also enables command-line process creation visibility.

---

# Why Group Policy Was Important

Before Active Directory, security settings had to be configured directly on the endpoint.

Group Policy improved the design by providing:

- centralized administration
- consistent configuration
- easier scaling
- reduced configuration drift
- enterprise realism

The final GPO configuration became the authoritative workstation baseline.

---

# GPO Validation

The policy was successfully applied to:

`LAB-WIN-01`

This confirmed:

- domain connectivity
- correct OU placement
- functioning Group Policy
- active audit policy

Status:

**PASS**

---

# Domain Controller Auditing

When `LAB-DC-01` was deployed, auditing expanded into identity infrastructure.

The Domain Controller generates events that do not normally exist on a standalone workstation.

Important validated examples include:

- `4672`
- `4719`
- `5136`

---

# Event ID 4719

Event:

`4719`

Description:

`System audit policy was changed`

This is important because attackers or administrators may alter audit settings.

A malicious actor may attempt to reduce logging before performing other actions.

Monitoring audit policy changes therefore helps detect possible defense evasion.

---

# Why Event 4719 Matters

If auditing changes unexpectedly, the SOC should investigate:

- who made the change
- what setting changed
- whether it was authorized
- what happened immediately before and after

This event becomes more valuable when correlated with privileged logons and process execution.

---

# Event ID 5136

Event:

`5136`

represents modification of an Active Directory object.

This is important for monitoring changes such as:

- user attributes
- group-related objects
- directory configuration

The exact security meaning depends on which object and attribute changed.

---

# Why Event 5136 Matters

Active Directory changes can be security-critical.

Potential scenarios include:

- unauthorized account modification
- privilege changes
- group membership manipulation
- persistence
- administrative misconfiguration

Event `5136` provides directory-change evidence for investigation.

---

# Event 4672 on the Domain Controller

Event `4672` was also observed on the Domain Controller.

This provides visibility into privileged logons involving sensitive domain accounts.

This is especially useful for monitoring:

- Domain Admin activity
- administrative sessions
- privileged automation

---

# Standard and Privileged Accounts

The Active Directory environment separates normal and privileged identities.

Standard account:

`emmanuel.patrick@cyberlab.internal`

Privileged account:

`emmanuel.ngari-adm@cyberlab.internal`

This improves the value of auditing because privileged operations can be more clearly attributed to a dedicated administrative identity.

---

# Domain Controller Wazuh Integration

The Domain Controller was enrolled into Wazuh.

Final agent:

    ID: 003
    Name: LAB-DC-01
    Status: Active

This allows Active Directory audit events to be investigated centrally.

---

# Domain Controller Monitoring Architecture

The final identity-monitoring path is:

    Domain Activity
         |
         v
     LAB-DC-01
         |
         +--> Windows Security Events
         |
         +--> AD Change Events
         |
         +--> Audit Policy Events
         |
         +--> Privileged Logon Events
         |
         +--> Sysmon
         |
         v
      Wazuh Agent
         |
         v
     LAB-SIEM-01

---

# Sysmon Relationship on the Domain Controller

Sysmon was later installed on:

`LAB-DC-01`

Version:

`15.22`

This provides additional process-level context around audited domain activity.

For example:

    Event 5136
        |
        +
    Sysmon Process Event
        |
        v
    Wazuh Investigation

This may help identify what process was active around a directory change.

---

# Auditing and Detection Engineering

The advanced auditing infrastructure supports future detection engineering.

Potential use cases include:

- repeated failed logons
- suspicious privileged sessions
- unexpected audit-policy changes
- unusual process creation
- unauthorized AD modification
- host firewall blocks
- suspicious PowerShell
- lateral movement

---

# Purple Team Workflow

The auditing environment supports a Purple Team model:

    Controlled Attack
          |
          v
    Windows / AD Activity
          |
          v
    Security Events
          |
          v
    Wazuh
          |
          v
    Analyst Investigation
          |
          v
    Detection Improvement

This turns Windows auditing into an operational security capability rather than a compliance-only setting.

---

# Multi-Source Correlation

One of the strongest aspects of the final architecture is that Windows auditing does not operate alone.

A single attack may produce evidence from:

- Windows Security
- Sysmon
- PowerShell
- Defender
- Windows Firewall
- OPNsense
- Suricata
- Wazuh

This allows analysts to reconstruct activity across:

- identity
- endpoint
- process
- network
- application layers

---

# Example Investigation Chain

A future scenario might produce:

    Failed Logons
        |
        v
    Successful Logon
        |
        v
    Event 4672
        |
        v
    Suspicious Process Creation
        |
        v
    PowerShell 4104
        |
        v
    Sysmon
        |
        v
    Network Connection
        |
        v
    Firewall / Suricata
        |
        v
    Wazuh Investigation

This is the kind of telemetry chain the lab was designed to support.

---

# Audit Noise Considerations

Advanced auditing can create significant event volume.

The objective is not simply to enable every category.

The selected audit categories prioritize:

- authentication
- privilege use
- process execution
- policy changes
- identity changes

Additional auditing should be introduced only when it supports a specific detection or investigation requirement.

---

# Final Workstation Audit State

`LAB-WIN-01`

Final auditing includes:

- Logon Success
- Logon Failure
- Process Creation Success
- Special Logon Success
- command-line process capture
- Windows Firewall monitoring
- PowerShell logging
- Sysmon
- Defender
- Wazuh

---

# Final Domain Controller Audit State

`LAB-DC-01`

Final monitoring includes:

- Windows Security events
- privileged logon events
- audit-policy change events
- Active Directory modification events
- Sysmon
- Wazuh

Important observed events include:

- `4672`
- `4719`
- `5136`

---

# Final Validation Summary

| Validation | Result |
|---|---|
| Advanced auditing introduced | PASS |
| Logon Success auditing | PASS |
| Logon Failure auditing | PASS |
| Process Creation auditing | PASS |
| Command-line process capture | PASS |
| Special Logon auditing | PASS |
| Windows Firewall Event 5157 monitoring | PASS |
| Wazuh Windows agent active | PASS |
| Group Policy baseline applied | PASS |
| Audit settings managed through GPO | PASS |
| Domain Controller auditing | PASS |
| Event 4672 observed | PASS |
| Event 4719 observed | PASS |
| Event 5136 observed | PASS |
| Domain Controller Wazuh agent active | PASS |
| Sysmon correlation available | PASS |

---

# Historical vs Current State

This document intentionally preserves the full auditing evolution.

Historical stages include:

- basic Windows logging
- standalone endpoint auditing
- local security-event review
- introduction of Wazuh
- advanced audit policy
- process creation logging
- command-line capture
- Windows Firewall monitoring
- Active Directory introduction
- Group Policy enforcement
- Domain Controller auditing
- identity-change monitoring

The authoritative current state is:

    LAB-WIN-01:
      Logon Auditing: Success / Failure
      Process Creation: Success
      Command Line: Enabled
      Special Logon: Success
      Windows Firewall Monitoring: Active
      Wazuh: Active

    LAB-DC-01:
      Security Auditing: Active
      Privileged Logon Monitoring: Active
      Audit Policy Change Monitoring: Active
      AD Change Monitoring: Active
      Sysmon: Active
      Wazuh: Active

---

# Evolution Summary

Windows auditing evolved through:

    Basic Windows Logs
           |
           v
    Need for Better Security Visibility
           |
           v
    Wazuh Endpoint Monitoring
           |
           v
    Advanced Audit Policy
           |
           v
    Process Creation
           |
           v
    Command-Line Capture
           |
           v
    Windows Firewall Monitoring
           |
           v
    Active Directory
           |
           v
    Group Policy Enforcement
           |
           v
    Domain Controller Auditing
           |
           v
    AD Change / Privilege Monitoring
           |
           v
    Final SOC-Ready Windows Auditing

---

# Final Status

The Windows advanced auditing infrastructure phase is complete.

The lab now provides centralized visibility into:

- user authentication
- failed logons
- process creation
- command-line execution
- privileged sessions
- host firewall blocks
- audit-policy changes
- Active Directory modifications

These events are available alongside Sysmon, PowerShell, Defender, firewall, and IDS telemetry in Wazuh.

Future work should focus on generating realistic Windows and Active Directory attack scenarios and using these audit sources for detection engineering, threat hunting, incident investigation, and Purple Team validation rather than enabling additional audit categories without a clear operational reason.
