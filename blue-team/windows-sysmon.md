# Windows Sysmon Monitoring

## Overview

This document records the complete evolution of Sysmon monitoring in the Cybersecurity Home Lab, from the period when the Windows environment relied mainly on standard Windows logs through the final monitored state on both the corporate workstation and the Domain Controller.

Sysmon was not part of the original flat lab.

The earliest Windows environment focused on:

- connectivity
- Windows Firewall
- local event logs
- basic endpoint configuration
- attacker/defender interaction

As the project matured, standard Windows logging was no longer sufficient for the level of endpoint visibility required for SOC and Purple Team work.

Sysmon was introduced to provide richer telemetry around:

- process creation
- command execution
- parent-child process relationships
- suspicious binaries
- DNS activity
- attacker tooling
- administrative utilities
- future persistence and lateral-movement scenarios

The final Sysmon deployment covers:

- `LAB-WIN-01`
- `LAB-DC-01`

with telemetry ingested into Wazuh.

---

# Phase 1 — Before Sysmon

The original lab used Windows as a standalone endpoint on:

`192.168.56.20`

The environment relied primarily on:

- Windows Event Viewer
- standard Security logs
- Windows Firewall logs
- local observation
- basic PowerShell output

This was useful for foundational work, but standard logging alone did not provide the depth needed for realistic endpoint detection.

Examples of missing context included:

- rich parent-child process relationships
- detailed process execution visibility
- security-focused process telemetry
- stronger insight into attacker utilities

---

# Why Sysmon Was Added

As the lab moved toward Wazuh-based monitoring, Sysmon became valuable because it provides security-oriented Windows telemetry beyond the normal event set.

The primary goals were:

- improve endpoint process visibility
- support detection engineering
- observe suspicious command execution
- provide better evidence for SOC investigations
- monitor attacker and administrative tools
- correlate Windows activity with Wazuh alerts

---

# First Sysmon Deployment

The first major Sysmon deployment was on:

`LAB-WIN-01`

This system had already evolved from the original standalone Windows endpoint into the corporate workstation located in:

`CORPNET`

Address:

`10.10.20.20`

Sysmon was installed as part of the broader Windows endpoint-monitoring phase.

---

# LAB-WIN-01 Monitoring Context

By the time Sysmon was introduced, the Windows workstation was also being monitored through:

- Windows Security logs
- Wazuh Agent
- PowerShell logging
- Defender
- Windows Firewall
- File Integrity Monitoring
- advanced auditing

Sysmon complemented these sources rather than replacing them.

---

# Sysmon Event Channel

Sysmon records events under:

`Microsoft-Windows-Sysmon/Operational`

Wazuh was configured to ingest this event channel from the Windows endpoint.

This allowed Sysmon events to be analyzed centrally in:

`LAB-SIEM-01`

---

# Initial Sysmon Objectives

The first configuration focused on collecting useful security telemetry without depending only on default Windows auditing.

Important use cases included:

- process creation
- suspicious command-line activity
- LOLBin execution
- PowerShell
- command shells
- attacker tooling
- administrative utilities

---

# Process Creation Visibility

One of the most important Sysmon capabilities is process creation monitoring.

This helps analysts answer questions such as:

- What process executed?
- Which parent launched it?
- Which user context was involved?
- Was the binary expected?
- Was the command part of a suspicious sequence?

This significantly improves investigation depth compared with only knowing that a user logged on.

---

# Command-Line Visibility

Sysmon process telemetry works particularly well when combined with Windows audit policies that also preserve command-line information.

This creates multiple sources for validating suspicious execution.

For example:

    powershell.exe
        |
        v
    suspicious command line
        |
        v
    Sysmon ProcessCreate
        |
        v
    Windows Process Auditing
        |
        v
    Wazuh

This provides stronger evidence than a single event source.

---

# PowerShell Monitoring Relationship

PowerShell was already a major logging focus in the Windows endpoint.

Sysmon added an additional layer by recording the execution of:

- `powershell.exe`
- `pwsh.exe`

while Windows PowerShell logging captured:

- script blocks
- modules
- transcripts

Together, these provide broader visibility into PowerShell abuse.

---

# LOLBin Monitoring

Sysmon is particularly useful for monitoring legitimate Windows binaries that attackers often abuse.

Examples include:

- `rundll32.exe`
- `regsvr32.exe`
- `mshta.exe`
- `certutil.exe`
- `bitsadmin.exe`
- `wscript.exe`
- `cscript.exe`

These binaries may be legitimate, but their execution can become security-relevant depending on context.

---

# Command Shell Monitoring

Important command interpreters include:

- `cmd.exe`
- `powershell.exe`
- `pwsh.exe`

Monitoring these provides visibility into many attack and administrative workflows.

---

# Scheduled Task Monitoring Context

The lab also treats:

`schtasks.exe`

as a security-relevant utility.

This is important because scheduled tasks can be used for:

- legitimate administration
- persistence
- remote execution
- privilege escalation workflows

Sysmon process visibility supports later detection engineering around this activity.

---

# Network and DNS Context

Sysmon can also provide visibility into network-related activity.

In the lab, DNS telemetry became particularly useful on the Domain Controller.

This allowed process-linked DNS behaviour to be monitored rather than looking only at raw DNS requests.

---

# Wazuh Ingestion

Sysmon events from `LAB-WIN-01` were successfully ingested into Wazuh.

This confirmed the pipeline:

    Windows Activity
         |
         v
    Sysmon
         |
         v
    Sysmon Operational Log
         |
         v
    Wazuh Agent
         |
         v
    LAB-SIEM-01

---

# Sysmon and Detection Engineering

Once Sysmon events reached Wazuh, they became useful for:

- investigation
- custom rules
- correlation
- future Purple Team testing

This represented a major improvement over the original Windows environment.

---

# Windows Endpoint Security Stack

The workstation monitoring architecture eventually became:

    LAB-WIN-01
        |
        +--> Windows Security Logs
        |
        +--> Sysmon
        |
        +--> PowerShell Logs
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

Sysmon became one layer within a broader endpoint telemetry strategy.

---

# Active Directory Expansion

When Active Directory was introduced, a new high-value Windows system appeared:

`LAB-DC-01`

Address:

`10.10.50.10`

Zone:

`ADNET`

Because the Domain Controller handles privileged identity operations, Sysmon was later deployed there as well.

---

# Why Sysmon Was Added to the Domain Controller

The Domain Controller is one of the most sensitive systems in the lab.

Monitoring it with only standard Windows logs would leave gaps in process-level visibility.

Sysmon was added to improve detection of activity involving:

- PowerShell
- command shells
- administrative tools
- reconnaissance
- LOLBins
- domain utilities
- suspicious process execution

---

# LAB-DC-01 Sysmon Installation

Sysmon version:

`15.22`

Binary:

`C:\Tools\Sysmon\Sysmon64.exe`

Configuration file:

`sysmon-dc.xml`

The installation was validated successfully.

---

# Initial Domain Controller Sysmon Configuration

The initial configuration collected process and DNS activity.

The goal was to provide broad visibility first, then tune the configuration based on observed noise.

This followed the lab’s normal workflow:

    Enable
       |
       v
    Observe
       |
       v
    Validate
       |
       v
    Tune
       |
       v
    Revalidate

---

# Why Tuning Was Necessary

Collecting every process event on a Domain Controller can generate a large volume of telemetry.

Much of that activity is legitimate Windows background behaviour.

A high-volume configuration would create:

- excessive noise
- harder investigations
- unnecessary SIEM storage use
- less useful dashboards
- lower analyst efficiency

The configuration was therefore tuned to focus on high-value activity.

---

# Final ProcessCreate Focus

The tuned Domain Controller Sysmon configuration focuses on security-relevant executables.

Examples include:

- `powershell`
- `pwsh`
- `cmd`
- `rundll32`
- `regsvr32`
- `mshta`
- `certutil`
- `bitsadmin`
- `wscript`
- `cscript`
- `schtasks`
- `nltest`
- `ntdsutil`
- `whoami`
- `net`
- `net1`

---

# PowerShell

Monitored executables include:

`powershell`

and:

`pwsh`

These can be used for:

- administration
- scripting
- reconnaissance
- execution
- post-exploitation activity

Because PowerShell is already heavily instrumented elsewhere in the lab, Sysmon adds process context around that activity.

---

# CMD

`cmd`

is included because it remains a common shell for:

- administrative commands
- batch scripts
- attacker execution
- chained utilities

---

# rundll32

`rundll32`

is monitored because it can execute DLL exports and is frequently abused as a living-off-the-land binary.

---

# regsvr32

`regsvr32`

is monitored because it can be abused to execute code through legitimate Windows components.

---

# mshta

`mshta`

is monitored because it can execute HTML Application content and is frequently associated with LOLBin abuse.

---

# certutil

`certutil`

is monitored because, beyond its legitimate certificate functions, it can be abused for:

- file transfer
- encoding
- decoding
- attacker staging

---

# bitsadmin

`bitsadmin`

is monitored because Background Intelligent Transfer Service functionality has historically been abused for file transfer and persistence-related activity.

---

# Script Hosts

The configuration monitors:

- `wscript`
- `cscript`

These are important because attackers may use Windows Script Host to execute:

- VBScript
- JScript
- other script-based payloads

---

# Scheduled Tasks

`schtasks`

is monitored because scheduled tasks can be used for:

- persistence
- execution
- remote administration
- privilege-related activity

---

# nltest

`nltest`

is included because it can be used for legitimate Active Directory troubleshooting and also domain reconnaissance.

Monitoring it is especially valuable on the Domain Controller.

---

# ntdsutil

`ntdsutil`

is particularly security-sensitive on a Domain Controller.

It is a legitimate Active Directory administration utility but can interact with highly sensitive directory data.

Its execution should therefore be visible to the SOC.

---

# whoami

`whoami`

is monitored because it is frequently used during reconnaissance to determine:

- current user
- groups
- privileges

While common, it can become important when correlated with suspicious activity.

---

# net and net1

`net`

and:

`net1`

are monitored because they can be used for:

- account enumeration
- group enumeration
- service administration
- share discovery
- user management

These are common both in administration and attacker reconnaissance.

---

# DNS Monitoring Tuning

DNS telemetry was also tuned.

Rather than collecting every DNS query indiscriminately, the configuration focuses on DNS activity associated with security-relevant executables.

Examples include:

- powershell
- pwsh
- cmd
- certutil
- mshta
- wscript
- cscript

This helps identify suspicious process-driven DNS activity while reducing ordinary system noise.

---

# Why Process-Aware DNS Matters

Raw DNS logs can show which domain was queried.

Sysmon can add process context.

For example:

    powershell.exe
         |
         v
    DNS Query
         |
         v
    suspicious-domain.example

This provides much more investigation value than a DNS query without process attribution.

---

# Domain Controller Wazuh Validation

After Sysmon deployment, `LAB-DC-01` remained connected to Wazuh.

Final agent:

    ID: 003
    Name: LAB-DC-01
    Status: Active

Sysmon events were successfully ingested.

Status:

**PASS**

---

# Post-Tuning Validation

After the Sysmon configuration was tuned, monitoring was tested again.

The objective was to confirm that:

- Sysmon remained active
- the configuration loaded successfully
- targeted process events were still collected
- DNS telemetry remained available
- Wazuh ingestion continued

The tuned configuration remained functional.

Status:

**PASS**

---

# Sysmon and Active Directory Auditing

Sysmon does not replace Active Directory security auditing.

The Domain Controller also produces important standard Windows events such as:

- `5136`
- `4719`
- `4672`

The combination provides broader visibility.

For example:

    Active Directory Change
           |
           +--> Windows Security Event
           |
           +--> Sysmon Process Context
           |
           v
        Wazuh

This can help identify not only that a change occurred but also which process activity surrounded it.

---

# LAB-WIN-01 vs LAB-DC-01 Monitoring

The two systems use Sysmon for different operational roles.

## LAB-WIN-01

Focus:

- user endpoint behaviour
- PowerShell
- suspicious processes
- attacker activity
- corporate workstation scenarios

## LAB-DC-01

Focus:

- privileged process execution
- domain utilities
- identity reconnaissance
- directory administration
- high-value LOLBin activity

Both feed into the same SIEM.

---

# Purple Team Value

Sysmon makes it possible to run realistic simulations and evaluate whether the defensive side can observe attacker behaviour.

Potential future scenarios include:

- suspicious PowerShell
- LOLBin execution
- scheduled-task persistence
- command-shell reconnaissance
- certutil file transfer
- domain enumeration
- ntdsutil execution
- script-host execution

The attacker action can then be correlated with:

- Sysmon
- Windows Security events
- PowerShell logs
- firewall activity
- Wazuh alerts

---

# Detection Workflow

A typical future Sysmon-driven detection workflow is:

    Attacker Action
         |
         v
    Windows Process Execution
         |
         v
    Sysmon Event
         |
         v
    Wazuh Agent
         |
         v
    LAB-SIEM-01
         |
         v
    Alert / Investigation
         |
         v
    Detection Tuning

This reflects the Purple Team methodology used throughout the lab.

---

# Noise Reduction Philosophy

The final Sysmon design follows an important principle:

**More telemetry is not automatically better telemetry.**

The objective is not to collect every possible event.

The objective is to collect enough high-quality data to support:

- detection
- investigation
- threat hunting
- incident response

without overwhelming the SIEM with low-value noise.

---

# Final Sysmon Coverage

Final monitored Windows systems:

| System | Role | Sysmon Status |
|---|---|---|
| `LAB-WIN-01` | Corporate Workstation | Active |
| `LAB-DC-01` | Domain Controller | Active |

Both send telemetry to:

`LAB-SIEM-01`

---

# Final Validation Summary

| Validation | Result |
|---|---|
| Sysmon installed on LAB-WIN-01 | PASS |
| Sysmon telemetry to Wazuh | PASS |
| Process monitoring | PASS |
| PowerShell process visibility | PASS |
| Sysmon installed on LAB-DC-01 | PASS |
| Sysmon 15.22 operational | PASS |
| `sysmon-dc.xml` loaded | PASS |
| Domain Controller telemetry to Wazuh | PASS |
| ProcessCreate tuning | PASS |
| DNS tuning | PASS |
| Security-relevant tools retained | PASS |
| Wazuh agent remained Active | PASS |

---

# Historical vs Current State

This document intentionally preserves the complete Sysmon evolution.

Historical stages include:

- no Sysmon
- standard Windows logs only
- first workstation Sysmon deployment
- Wazuh Sysmon ingestion
- Active Directory introduction
- Domain Controller Sysmon deployment
- broad initial DC telemetry
- tuning to reduce noise

The authoritative current state is:

    LAB-WIN-01:
      Sysmon: Active
      Wazuh Ingestion: Active

    LAB-DC-01:
      Sysmon Version: 15.22
      Configuration: sysmon-dc.xml
      ProcessCreate Filtering: Tuned
      DNS Filtering: Tuned
      Wazuh Ingestion: Active

---

# Evolution Summary

Sysmon monitoring evolved through:

    Standard Windows Logs
          |
          v
    Need for Richer Endpoint Visibility
          |
          v
    LAB-WIN-01 Sysmon
          |
          v
    Wazuh Ingestion
          |
          v
    Endpoint Detection Engineering
          |
          v
    Active Directory Deployment
          |
          v
    LAB-DC-01 Sysmon
          |
          v
    Broad Initial Collection
          |
          v
    Noise Analysis
          |
          v
    Process and DNS Tuning
          |
          v
    Final Dual-System Sysmon Monitoring

---

# Final Status

The Sysmon infrastructure phase is complete.

The Cybersecurity Home Lab now has enhanced Windows process visibility on both:

- the corporate workstation
- the Domain Controller

Sysmon events are centrally available in Wazuh and are ready to support:

- SOC investigations
- threat hunting
- detection engineering
- Active Directory attack detection
- endpoint attack simulation
- Purple Team exercises

Future work should focus on generating realistic attacker behaviours and building or tuning detections around the existing telemetry rather than expanding baseline Sysmon infrastructure without a specific operational requirement.
