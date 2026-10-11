# Windows PowerShell Logging

## Overview

This document records the complete evolution of PowerShell logging in the Cybersecurity Home Lab, from the earliest standalone Windows phase through the final domain-managed monitoring configuration.

PowerShell logging was not fully configured at the beginning of the project.

The original Windows lab focused primarily on:

- basic networking
- local administration
- Windows Firewall
- early endpoint testing
- attacker/defender interaction

As the project matured, PowerShell became an important telemetry source because it is widely used for both:

- legitimate administration
- offensive security activity

The lab therefore introduced progressively stronger PowerShell logging and integrated that telemetry into Wazuh.

The final PowerShell monitoring design includes:

- Script Block Logging
- Module Logging
- PowerShell transcription
- Wazuh ingestion
- Group Policy enforcement
- complementary Sysmon process visibility
- centralized SOC analysis

Primary monitored workstation:

`LAB-WIN-01`

Final address:

`10.10.20.20`

Zone:

`CORPNET`

Domain:

`cyberlab.internal`

---

# Phase 1 — Before Enhanced PowerShell Logging

The original Windows endpoint operated on:

`192.168.56.20`

in the early flat network.

At this stage, PowerShell could be used for:

- normal Windows administration
- connectivity tests
- system inspection
- configuration

However, detailed PowerShell security visibility had not yet been implemented.

Investigation relied mainly on:

- terminal output
- standard Windows logs
- manual observation

This was insufficient for a mature SOC environment.

---

# Why PowerShell Required Additional Monitoring

PowerShell is a legitimate Windows administration framework.

It is also commonly abused during attacks for:

- reconnaissance
- payload execution
- file operations
- persistence
- remote administration
- credential-related activity
- defense evasion
- post-exploitation

Simply detecting that:

`powershell.exe`

ran is not always enough.

The SOC may also need to understand:

- what commands were executed
- which script blocks ran
- which modules were loaded
- what session activity occurred
- which user executed the commands

This led to the introduction of enhanced PowerShell logging.

---

# Transition to Centralized Monitoring

When Wazuh was introduced, Windows endpoint telemetry began moving from local-only analysis to centralized SOC monitoring.

`LAB-WIN-01` became the primary monitored Windows endpoint.

Final workstation address:

`10.10.20.20`

Final zone:

`CORPNET`

Wazuh agent:

`ID 001 — LAB-WIN-01`

Status:

`Active`

PowerShell logging became one component of the broader Windows endpoint-monitoring strategy.

---

# PowerShell Monitoring Architecture

The final monitoring path is:

    PowerShell Activity
          |
          v
    Windows PowerShell Logs
          |
          +--> Script Block Logging
          |
          +--> Module Logging
          |
          +--> Transcription
          |
          v
    Wazuh Agent
          |
          v
    LAB-SIEM-01
          |
          v
    SOC Investigation

Sysmon provides an additional process-level view.

---

# Script Block Logging

Script Block Logging was enabled to capture PowerShell code as it is processed.

This provides deeper visibility than simply recording the launch of:

`powershell.exe`

or:

`pwsh.exe`

It can help expose:

- executed commands
- script content
- encoded or reconstructed logic
- administrative scripts
- attacker-created PowerShell activity

---

# Why Script Block Logging Matters

Consider an event where only process execution is recorded:

    powershell.exe

This tells the analyst that PowerShell ran.

Script Block Logging can provide additional context about what PowerShell actually processed.

The combination therefore provides much stronger investigation value.

---

# Script Block Event Context

PowerShell Script Block Logging commonly produces events in the PowerShell Operational event channel.

An important event associated with this capability is:

`Event ID 4104`

This event can contain PowerShell script-block content.

For future SOC exercises, this can help identify:

- suspicious commands
- obfuscated PowerShell
- reconnaissance
- download activity
- persistence-related commands

---

# Module Logging

PowerShell Module Logging was also enabled.

The final configuration includes:

`*`

as the module scope.

This provides broad visibility into commands executed through PowerShell modules.

---

# Why Module Logging Matters

PowerShell commands may execute through modules rather than through standalone scripts.

Module Logging helps provide additional information around:

- cmdlets
- module-based administration
- command execution

This complements Script Block Logging.

---

# Module Logging Event Context

PowerShell Module Logging is commonly associated with:

`Event ID 4103`

This can provide details about module-based command execution.

Together:

    Event 4103
        +
    Event 4104

provide valuable PowerShell investigation context.

---

# PowerShell Transcription

PowerShell transcription was introduced as another layer of visibility.

Transcription records session output and command activity in text-based transcript files.

This provides an additional investigation source outside normal Windows event logging.

---

# Why Transcription Was Added

Event logs provide structured telemetry.

Transcripts can provide a more human-readable record of an interactive PowerShell session.

This can help during:

- incident review
- administrative troubleshooting
- attack reconstruction
- command-sequence analysis

---

# Transcription and Event Logging

PowerShell transcription does not replace:

- Script Block Logging
- Module Logging
- Sysmon
- Windows process auditing

Instead, it provides another independent source of evidence.

The design intentionally uses multiple overlapping telemetry sources.

---

# Defense-in-Depth Logging

The PowerShell monitoring approach can be represented as:

    powershell.exe
          |
          +--> Sysmon ProcessCreate
          |
          +--> Windows Process Creation Audit
          |
          +--> PowerShell Module Logging
          |
          +--> PowerShell Script Block Logging
          |
          +--> PowerShell Transcript
          |
          v
    Wazuh / SOC Investigation

A single attacker action can therefore produce multiple pieces of evidence.

---

# Wazuh Integration

PowerShell telemetry from the Windows endpoint is collected through the Wazuh agent.

Destination:

`LAB-SIEM-01`

Address:

`10.10.40.10`

Primary communication:

`TCP/1514`

This makes PowerShell events available for centralized investigation.

---

# PowerShell Event Visibility in Wazuh

The lab was designed so PowerShell telemetry could be reviewed alongside:

- Sysmon
- Security events
- Windows Firewall events
- Defender telemetry
- File Integrity Monitoring

This allows an analyst to reconstruct activity across multiple data sources.

---

# Example Investigation Relationship

A suspicious PowerShell sequence could produce:

    User Logon
        |
        v
    Event 4672 / Logon Context
        |
        v
    powershell.exe Process Creation
        |
        v
    Sysmon Event
        |
        v
    PowerShell Event 4103 / 4104
        |
        v
    Network Activity
        |
        v
    Firewall or Suricata Evidence
        |
        v
    Wazuh Investigation

This is the kind of multi-source correlation the lab was built to support.

---

# Advanced Windows Auditing Relationship

PowerShell logging was implemented alongside advanced Windows auditing.

The workstation audit baseline includes:

- Logon — Success and Failure
- Process Creation — Success
- Special Logon — Success

Command-line process creation was also enabled.

This provides additional evidence around PowerShell execution.

---

# Command-Line Process Auditing

Command-line process auditing helps capture how PowerShell was launched.

For example:

    powershell.exe -NoProfile ...

This information can complement Script Block Logging.

Even if PowerShell activity is incomplete in one telemetry source, another source may preserve useful context.

---

# Sysmon Relationship

Sysmon also monitors PowerShell execution.

Important executables include:

- `powershell`
- `pwsh`

Sysmon adds process-level context such as:

- parent process
- process execution
- executable path
- surrounding process activity

This is complementary to the content-oriented PowerShell logs.

---

# Microsoft Defender Relationship

Microsoft Defender telemetry was also included in the Windows monitoring stack.

If PowerShell activity triggers a security detection, Wazuh may therefore have evidence from:

- PowerShell
- Sysmon
- Defender
- Windows Security logs

This improves confidence during investigation.

---

# File Integrity Monitoring Relationship

PowerShell can also be used to create or modify files.

Wazuh File Integrity Monitoring may provide another source of evidence if monitored files or directories are changed.

Potential investigation chain:

    PowerShell
        |
        v
    File Creation / Modification
        |
        v
    Wazuh FIM
        |
        v
    Detection / Investigation

---

# Initial Local Configuration Phase

Before Active Directory Group Policy was introduced, enhanced PowerShell logging was configured directly on the Windows endpoint.

This allowed the telemetry design to be:

- tested
- verified
- integrated into Wazuh

before central policy management existed.

This was an important intermediate phase.

---

# Active Directory Introduction

Active Directory was added later in the project.

Domain:

`cyberlab.internal`

Domain Controller:

`LAB-DC-01`

The Windows workstation became a domain member.

This created an opportunity to move security settings from local-only configuration into centrally managed Group Policy.

---

# Workstation Security Baseline

A Group Policy Object was created:

`GPO-Workstation-Security-Baseline`

It was linked to:

`CyberLab\Workstations`

The GPO included PowerShell logging controls.

---

# Script Block Logging Through Group Policy

The workstation baseline enables:

`PowerShell Script Block Logging`

This ensures the setting is centrally enforced rather than depending only on local workstation configuration.

---

# Module Logging Through Group Policy

The workstation baseline also enables:

`PowerShell Module Logging`

Module coverage:

`*`

This provides broad monitoring of PowerShell module activity.

---

# Why Group Policy Improved the Design

Moving PowerShell controls into Group Policy provided several benefits.

## Centralization

Security settings are controlled from Active Directory.

## Consistency

A workstation cannot silently drift from the desired baseline as easily.

## Scalability

Additional domain workstations could receive the same policy.

## Realism

This mirrors enterprise security administration more closely than configuring every workstation manually.

---

# Group Policy Validation

The GPO was successfully applied to:

`LAB-WIN-01`

This confirmed:

- domain communication
- correct workstation OU placement
- functional Group Policy
- PowerShell monitoring policy delivery

Status:

**PASS**

---

# Local Configuration vs Domain Policy

The history of the project therefore includes two phases.

## Earlier Phase

PowerShell logging was configured locally as part of endpoint-hardening work.

## Final Phase

PowerShell Script Block Logging and Module Logging became part of the domain-managed workstation baseline.

The final Group Policy configuration is the authoritative security baseline.

---

# PowerShell Transcription in the Final Design

PowerShell transcription remains part of the endpoint-monitoring strategy.

Unlike Script Block and Module Logging, transcription produces text-based session records.

This provides a useful secondary evidence source for future investigations.

---

# PowerShell and Purple Team Operations

The final PowerShell monitoring environment is designed to support future simulations involving:

- reconnaissance
- encoded PowerShell
- suspicious downloads
- user and group enumeration
- persistence
- process execution
- administrative abuse
- attacker scripting

The purpose is not simply to generate logs.

The objective is to determine whether security analysts can:

1. detect the activity
2. reconstruct what happened
3. identify the user and process
4. correlate the event with other telemetry
5. improve detection coverage

---

# Potential Detection Workflow

A future Purple Team scenario could follow:

    Controlled PowerShell Attack
              |
              v
        Process Execution
              |
              +--> Sysmon
              |
              +--> Windows Audit
              |
              +--> Module Logging
              |
              +--> Script Block Logging
              |
              +--> Transcript
              |
              v
          Wazuh SIEM
              |
              v
        Alert Investigation
              |
              v
        Detection Tuning

This reflects the final purpose of the logging infrastructure.

---

# Security Value

Enhanced PowerShell logging provides several important defensive benefits.

## Visibility

Analysts can see more than just the PowerShell executable.

## Attribution

Events can be connected to user and host activity.

## Investigation

Commands and script content can help reconstruct an incident.

## Detection Engineering

Real telemetry can be used to create Wazuh rules.

## Threat Hunting

Historical PowerShell activity can be searched for suspicious patterns.

---

# Operational Considerations

Enhanced PowerShell logging increases telemetry volume.

The objective is not to enable every possible Windows event indiscriminately.

The project uses layered logging so that useful context can be collected without relying on one source.

As realistic simulations begin, PowerShell logging may be tuned further based on:

- detection value
- noise
- SIEM storage
- analyst usefulness

---

# Final Monitoring Stack on LAB-WIN-01

The final workstation security telemetry includes:

    LAB-WIN-01
        |
        +--> Windows Security Logs
        |
        +--> Advanced Auditing
        |
        +--> Command-Line Process Auditing
        |
        +--> Sysmon
        |
        +--> PowerShell Script Block Logging
        |
        +--> PowerShell Module Logging
        |
        +--> PowerShell Transcription
        |
        +--> Microsoft Defender
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

---

# Final Validation Summary

| Validation | Result |
|---|---|
| PowerShell logging introduced | PASS |
| Script Block Logging enabled | PASS |
| Module Logging enabled | PASS |
| Module scope `*` configured | PASS |
| PowerShell transcription enabled | PASS |
| Wazuh agent active | PASS |
| PowerShell telemetry available to SOC | PASS |
| Sysmon PowerShell process visibility | PASS |
| Command-line process auditing | PASS |
| Domain join completed | PASS |
| Workstation security GPO applied | PASS |
| Script Block Logging managed by GPO | PASS |
| Module Logging managed by GPO | PASS |

---

# Historical vs Current State

This document intentionally preserves the full PowerShell logging lifecycle.

Historical stages include:

- early Windows endpoint with limited PowerShell visibility
- local-only administration
- introduction of Wazuh
- local enhanced PowerShell logging
- Script Block Logging
- Module Logging
- transcription
- integration with Sysmon and advanced auditing
- Active Directory domain join
- Group Policy enforcement

The authoritative current state is:

    Host: LAB-WIN-01
    Zone: CORPNET
    Domain: cyberlab.internal
    Script Block Logging: Enabled
    Module Logging: Enabled
    Module Scope: *
    Transcription: Enabled
    Wazuh Monitoring: Active
    Sysmon Monitoring: Active
    Group Policy: GPO-Workstation-Security-Baseline

---

# Evolution Summary

PowerShell monitoring evolved through:

    Basic Local PowerShell Usage
             |
             v
    Need for Better Visibility
             |
             v
    Wazuh Endpoint Monitoring
             |
             v
    Script Block Logging
             |
             v
    Module Logging
             |
             v
    PowerShell Transcription
             |
             v
    Sysmon / Advanced Audit Correlation
             |
             v
    Active Directory Domain Join
             |
             v
    Group Policy Enforcement
             |
             v
    Final SOC-Ready PowerShell Monitoring

---

# Final Status

The PowerShell logging infrastructure phase is complete.

`LAB-WIN-01` now provides layered PowerShell telemetry through:

- Script Block Logging
- Module Logging
- transcription
- process auditing
- Sysmon
- Wazuh

The configuration is centrally reinforced through Active Directory Group Policy.

The environment is ready for realistic PowerShell-based attack simulations, SOC investigations, threat hunting, and detection engineering.

Future work should focus on generating controlled PowerShell attack behaviours and measuring whether the existing telemetry produces useful detections rather than expanding the baseline without a specific detection requirement.
