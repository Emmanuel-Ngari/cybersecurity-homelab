# Wazuh Detection Engineering

## Overview

This document records the complete evolution of detection engineering in the Cybersecurity Home Lab, from the earliest phase where security activity was observed manually through the final multi-source Wazuh detection environment.

Detection engineering did not exist as a formal discipline in the original lab.

The earliest environment focused mainly on:

- Nmap
- SSH
- Hydra
- fail2ban
- local Windows logs
- direct host observation
- manual attacker/defender validation

As the project matured, the objective changed from simply asking:

**Did the attack work?**

to asking:

**Can the defensive side reliably detect, explain, and investigate what happened?**

This led to the gradual development of:

- endpoint detections
- Linux detections
- Windows detections
- firewall correlation
- Suricata alert ingestion
- web-server detections
- decoder troubleshooting
- log-format tuning
- validation workflows
- Purple Team detection testing

The final detection platform is centered on:

`LAB-SIEM-01`

using:

`Wazuh 4.14.7`

---

# Phase 1 — Before Formal Detection Engineering

The earliest lab used:

`192.168.56.0/24`

Primary systems included:

- Kali — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu — `192.168.56.30`

At this stage, security validation was largely manual.

Examples included:

- scan a system with Nmap
- attempt SSH authentication
- trigger fail2ban
- observe the result
- check whether a port became filtered

This was useful for learning controls, but there was no central detection logic.

---

# Early Purple Team Pattern

Even before Wazuh, the lab was already beginning to follow a simple Purple Team cycle.

Example:

    Kali
      |
      v
    SSH Authentication Attack
      |
      v
    Ubuntu
      |
      v
    fail2ban
      |
      v
    Source Blocked
      |
      v
    Kali Retest

This created the first version of the lab's future detection philosophy:

    Generate Activity
          |
          v
    Observe Defensive Response
          |
          v
    Verify Result
          |
          v
    Improve Control

---

# Transition to Centralized Detection

When Wazuh was introduced, the project gained the ability to centralize security telemetry.

This changed the detection model from:

    Local Host Observation

to:

    Centralized Event Collection
             |
             v
       Detection Rules
             |
             v
        SOC Analysis

This was the beginning of formal detection engineering in the lab.

---

# Detection Engineering Objectives

The project adopted several detection goals.

## Detect Real Activity

Rules should be triggered by actual security-relevant events.

## Validate Every Detection

A rule should not be considered complete simply because it exists.

Controlled traffic or host activity must prove that it fires.

## Reduce False Positives

Rules should be tuned so normal activity does not overwhelm the analyst.

## Preserve Context

The detection should provide enough information to understand what happened.

## Use Multiple Telemetry Layers

Where possible, endpoint, firewall, IDS, and application evidence should support one another.

---

# Windows Detection Foundation

`LAB-WIN-01` became the first heavily monitored endpoint.

Its monitoring stack eventually included:

- Windows Security logs
- Sysmon
- PowerShell logging
- PowerShell transcription
- advanced auditing
- Defender telemetry
- Windows Firewall events
- File Integrity Monitoring

This created the raw telemetry needed for Windows detection engineering.

---

# Windows Process Visibility

Process activity became one of the most important security data sources.

The lab used:

- Windows Process Creation auditing
- command-line capture
- Sysmon

This made it possible to investigate:

- PowerShell
- command shells
- LOLBins
- suspicious child processes
- administrative tools

---

# PowerShell Detection Context

PowerShell telemetry included:

- Script Block Logging
- Module Logging
- transcription
- Sysmon process events
- command-line audit events

This created overlapping evidence around PowerShell execution.

A future suspicious PowerShell action could therefore be correlated across multiple sources rather than depending on a single event.

---

# Windows Firewall Detection Context

The Windows endpoint also monitored:

`Event ID 5157`

This represents a connection blocked by the Windows Filtering Platform.

This event became useful for distinguishing:

- OPNsense firewall blocks
- Windows host firewall blocks

That distinction is important during investigation.

---

# File Integrity Monitoring

Wazuh File Integrity Monitoring was introduced as another detection source.

This supports scenarios involving:

- file creation
- file modification
- configuration tampering
- persistence artifacts

FIM events can later be correlated with process and user activity.

---

# Linux Detection Foundation

`LAB-UBUNTU-01` became the primary Linux detection-engineering system.

Monitoring included:

- Linux logs
- authentication events
- auditd
- user-management activity
- Wazuh agent telemetry

This provided a platform for custom Linux rules.

---

# Native Linux User Creation Detection

A controlled user-creation action was generated on:

`LAB-UBUNTU-01`

A custom Wazuh rule was created and validated:

`Rule 100200`

Purpose:

Detect a native Linux user-add event.

The detection confirmed the full chain:

    Linux Administrative Action
             |
             v
         System Log
             |
             v
          Wazuh Agent
             |
             v
        LAB-SIEM-01
             |
             v
        Rule 100200

Status:

**VALIDATED**

---

# Native Linux User Deletion Detection

A controlled user-deletion action was also generated.

Custom Wazuh rule:

`Rule 100201`

Purpose:

Detect native Linux user deletion.

Status:

**VALIDATED**

These detections demonstrated that the lab was capable of building custom rules around normal operating-system telemetry.

---

# Why the Linux Rules Mattered

The user-add and user-delete detections were important because they proved the project had progressed beyond:

- installing agents
- collecting logs
- viewing dashboards

The lab was now actively converting telemetry into security detections.

---

# OPNsense Firewall Detection Engineering

After endpoint detections were established, OPNsense firewall telemetry was integrated into Wazuh.

This introduced network enforcement data into the detection pipeline.

---

# Firewall Telemetry Value

Firewall logs provided visibility into:

- blocked connections
- attempted lateral movement
- segmentation violations
- unauthorized service access

This added network context that endpoint logs could not always provide.

---

# Firewall Correlation Rule

A Wazuh firewall correlation rule was successfully validated:

`Rule 87702`

This confirmed that OPNsense events could trigger actionable Wazuh detections.

Status:

**VALIDATED**

---

# Firewall Detection Example

A typical firewall detection path is:

    Source Host
        |
        v
    Unauthorized Connection
        |
        v
    LAB-FW-01
        |
        v
    OPNsense Block
        |
        v
    Firewall Log
        |
        v
    Wazuh
        |
        v
    Rule 87702

This provides visibility into attempted communication that never reaches the destination host.

---

# Why Firewall Detection Matters

Without firewall telemetry, a blocked attack path may produce no endpoint evidence at all.

For example:

    REDNET → ADNET LDAP/389

If OPNsense blocks the traffic before it reaches the Domain Controller, the endpoint may never log an application event.

The firewall log therefore becomes the primary detection source.

---

# Suricata Detection Engineering

Suricata introduced network IDS alerts into the detection architecture.

Primary monitored interface:

`em2`

This corresponds to REDNET.

The goal was to detect attack patterns even when firewall policy allowed the connection.

---

# First Suricata Alert

A controlled Suricata test produced:

`Signature ID 2100498`

Signature:

`GPL ATTACK_RESPONSE id check returned root`

Category:

`Potentially Bad Traffic`

Severity:

`2`

This proved that Suricata itself could detect the test traffic.

The next challenge was getting that event into Wazuh correctly.

---

# Initial Decoder Failure

The first Suricata events did not decode correctly in Wazuh.

They fell through to:

`Rule 1002`

This showed that the event was arriving, but Wazuh did not yet understand the event structure properly.

This became an important detection-engineering troubleshooting phase.

---

# Decoder Troubleshooting

The project investigated:

- syslog formatting
- decoder matching
- JSON structure
- Wazuh test tools
- event transport

A custom decoder was introduced during this phase.

Observed decoder name:

`suricata-eve-syslog`

This helped confirm that the event could be matched more specifically.

---

# Transition to JSON

The architecture was later simplified.

Suricata events were ingested as structured JSON instead of maintaining unnecessary custom wrapper logic.

The effective decoder became:

`json`

This was a cleaner and more maintainable solution.

---

# Built-In Wazuh Suricata Detection

Once the JSON structure was processed correctly, Wazuh's built-in Suricata rules were able to handle the event.

Ruleset file:

`0475-suricata_rules.xml`

Detection:

`Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

Status:

**VALIDATED**

---

# Retiring the Custom Wrapper

Once the built-in JSON path worked, the custom Suricata wrapper decoder was no longer necessary.

This was an important engineering decision.

A good detection architecture should not maintain custom code where a reliable built-in solution already exists.

The final design therefore favors:

- simpler parsing
- structured JSON
- built-in Wazuh rules
- less custom maintenance

---

# Suricata Event Transport

A dedicated Suricata telemetry path was created using:

`UDP/5514`

Destination:

`LAB-SIEM-01 — 10.10.40.10`

rsyslog receives the events and writes them to:

`/var/log/suricata/eve.json`

Wazuh then ingests that file as JSON.

---

# Final Suricata Detection Pipeline

The final pipeline is:

    Network Traffic
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
    Wazuh Rule 86601

---

# Controlled Suricata Validation

A temporary web resource was created on:

`LAB-DMZ-01`

containing:

    uid=0(root) gid=0(root) groups=0(root)

Kali requested that resource.

Suricata generated:

    SID: 2100498
    Signature: GPL ATTACK_RESPONSE id check returned root
    Category: Potentially Bad Traffic
    Severity: 2

Wazuh generated:

`Rule 86601`

This validated the complete detection path.

---

# full_log Troubleshooting

One Suricata event in Wazuh showed:

`full_log: null`

This initially looked suspicious.

However, the structured event data was still present.

The event could be investigated using:

- rule ID
- location
- JSON fields

The lesson was:

**Do not assume a detection failed simply because one optional field is empty.**

---

# Live JSON Parsing Lesson

Direct `jq` parsing against a live log file occasionally returned errors.

Because valid Wazuh alerts were still being produced, the entire pipeline was not treated as broken.

This reinforced a key engineering principle:

**Correlate symptoms with end-to-end behaviour before changing a working pipeline.**

---

# Nginx Detection Engineering

The DMZ introduced a new detection source:

`Nginx`

The Wazuh agent on:

`LAB-DMZ-01`

monitors:

`/var/log/nginx/access.log`

using:

`apache`

log format.

It also monitors:

`/var/log/nginx/error.log`

using:

`syslog`

log format.

---

# Controlled Nginx Detection

Kali generated a controlled request:

`/wazuh-nginx-verify-01`

Nginx returned:

`404`

The access log recorded the request.

Wazuh generated:

`Rule 31101`

Description:

`Web server 400 error code.`

Event details included:

- Agent `LAB-DMZ-01`
- Source IP `10.10.10.10`
- Decoder `web-accesslog`

Status:

**VALIDATED**

---

# Why the Nginx Detection Matters

The Nginx event demonstrated application-layer visibility.

The same REDNET-to-DMZ traffic could now produce evidence at multiple layers:

    Kali
      |
      v
    OPNsense
      |
      +--> Firewall Evidence
      |
      +--> Suricata Evidence
      |
      v
    Nginx
      |
      +--> Access Log
      |
      v
    Wazuh

This is a major improvement over relying on one detection source.

---

# Active Directory Detection Context

When Active Directory was introduced, the detection surface expanded again.

`LAB-DC-01` provides events such as:

- `4672`
- `4719`
- `5136`

These support future detections around:

- privileged logons
- audit-policy changes
- Active Directory object modifications

---

# Event 4672

Event:

`4672`

represents special privileges assigned to a new logon.

This can provide context for:

- Domain Admin sessions
- privileged administration
- suspicious elevated activity

The event itself is not automatically malicious.

Its value comes from correlation.

---

# Event 4719

Event:

`4719`

represents a system audit policy change.

This is particularly important for detecting possible:

- defensive control tampering
- unauthorized audit changes
- defense evasion

---

# Event 5136

Event:

`5136`

represents Active Directory object modification.

This can support detections involving:

- account changes
- group-related modifications
- directory configuration changes
- persistence

---

# Domain Controller Sysmon

Sysmon was added to:

`LAB-DC-01`

and tuned to focus on high-value executables.

Examples include:

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

This provides process context around identity-related activity.

---

# Detection Correlation

The final architecture supports multi-source correlation.

For example:

    Event 5136
        |
        +
    Event 4672
        |
        +
    Sysmon ProcessCreate
        |
        +
    PowerShell 4104
        |
        v
    Wazuh Investigation

This is far more useful than treating each event independently.

---

# Detection Development Workflow

The project gradually established a professional detection-engineering workflow.

The final process is:

    1. Define the behaviour to detect
    2. Identify the telemetry source
    3. Generate controlled activity
    4. Confirm the raw event exists
    5. Confirm Wazuh receives it
    6. Check decoder output
    7. Build or select detection rule
    8. Trigger the behaviour again
    9. Confirm alert generation
    10. Review event context
    11. Tune noise or false positives
    12. Retest
    13. Document evidence

This workflow should be used for future detection development.

---

# Telemetry Before Rules

An important lesson was:

**Do not start by writing a rule when the required telemetry has not been confirmed.**

Before building detection logic, verify:

- the event exists
- the agent or collector receives it
- the decoder parses it correctly

Only then should detection logic be added.

This prevents time being wasted debugging rules for events that never reach the SIEM.

---

# Raw Event Validation

A professional detection workflow begins with raw evidence.

For example:

    Attack
      |
      v
    Raw Event
      |
      v
    Wazuh Ingestion
      |
      v
    Decoder
      |
      v
    Rule

Each stage must work before the next one can be trusted.

---

# Decoder Validation

Decoder troubleshooting became one of the major lessons from the Suricata phase.

A rule cannot reliably match fields that were never extracted.

This means decoder validation is a core part of detection engineering.

---

# Built-In Rules vs Custom Rules

The project uses both.

## Built-In Rules

Preferred where Wazuh already provides reliable support.

Examples:

- Suricata Rule `86601`
- Nginx Rule `31101`

## Custom Rules

Used when the lab needs detection logic that is not provided by default.

Examples:

- Linux Rule `100200`
- Linux Rule `100201`
- firewall correlation Rule `87702`

The goal is not to create custom rules unnecessarily.

---

# Detection Quality

A detection should provide enough context for an analyst to understand:

- what happened
- where it happened
- which host was involved
- which user or source was involved
- which rule fired
- what should be investigated next

A rule that generates noise without useful context is not considered mature.

---

# False Positive Reduction

Detection tuning aims to reduce unnecessary alerts while preserving meaningful activity.

Examples of noise-reduction work include:

- Sysmon ProcessCreate filtering
- Sysmon DNS filtering
- avoiding unnecessary custom Suricata wrappers
- choosing specific telemetry sources
- focusing firewall logging on meaningful boundaries

---

# Detection Engineering and Storage

Collecting every possible event can:

- consume SIEM storage
- slow searches
- increase analyst fatigue
- hide important alerts

The project therefore follows the principle:

**Collect enough to detect and investigate, not everything simply because it is available.**

---

# Detection Evidence

Important detections should be supported by evidence such as:

- raw logs
- Wazuh alerts
- source IPs
- destination IPs
- rule IDs
- timestamps
- process names
- screenshots where appropriate

This ensures the GitHub portfolio demonstrates actual validation rather than only configuration.

---

# Final Validated Detection Inventory

## Linux User Creation

Rule:

`100200`

Status:

**PASS**

---

## Linux User Deletion

Rule:

`100201`

Status:

**PASS**

---

## OPNsense Firewall Correlation

Rule:

`87702`

Status:

**PASS**

---

## Suricata Alert

Rule:

`86601`

Signature:

`GPL ATTACK_RESPONSE id check returned root`

Status:

**PASS**

---

## Nginx Web Detection

Rule:

`31101`

Description:

`Web server 400 error code.`

Status:

**PASS**

---

# Supporting Telemetry

Detection engineering is also supported by:

- Windows Security events
- Sysmon
- PowerShell
- Defender
- Windows Firewall Event 5157
- auditd
- Active Directory events
- OPNsense firewall logs
- Suricata
- Nginx

This gives the lab multiple sources for future rules.

---

# Final Purple Team Detection Model

The final environment supports the following operational model:

    Threat Behaviour
          |
          v
    Controlled Simulation
          |
          v
    Endpoint / Network / Application Telemetry
          |
          v
    Wazuh
          |
          v
    Detection
          |
          v
    SOC Investigation
          |
          v
    Detection Gap Analysis
          |
          v
    Rule / Logging Improvement
          |
          v
    Retest

This is the core Purple Team cycle for future exercises.

---

# Detection Gap Philosophy

A failed detection is not necessarily a failed exercise.

If a controlled attack produces no alert, the correct questions are:

- Was telemetry generated?
- Was it ingested?
- Was it decoded?
- Was there an existing rule?
- Was the rule too narrow?
- Was the activity expected to trigger?

A missing detection can become a detection-engineering task.

This is one of the main purposes of the lab.

---

# Final Detection Architecture

The complete architecture can be summarized as:

    REDNET / User Activity
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
        Target Systems
        |     |     |
        |     |     |
     Windows Linux  DMZ
        |     |     |
        +-----+-----+
             |
             v
         Wazuh Agents
             |
             v
        LAB-SIEM-01
             |
             v
    Decoding / Correlation
             |
             v
          Alerts
             |
             v
     SOC Investigation
             |
             v
    Detection Engineering

---

# Final Validation Summary

| Detection Capability | Result |
|---|---|
| Centralized Wazuh detection | PASS |
| Windows telemetry | PASS |
| Linux telemetry | PASS |
| auditd telemetry | PASS |
| Sysmon telemetry | PASS |
| PowerShell telemetry | PASS |
| Windows Firewall telemetry | PASS |
| OPNsense firewall telemetry | PASS |
| Linux Rule 100200 | PASS |
| Linux Rule 100201 | PASS |
| Firewall Rule 87702 | PASS |
| Suricata ingestion | PASS |
| Suricata Rule 86601 | PASS |
| Nginx ingestion | PASS |
| Nginx Rule 31101 | PASS |
| Active Directory telemetry | PASS |
| Domain Controller Sysmon | PASS |
| Multi-source detection architecture | PASS |

---

# Historical vs Current State

This document intentionally preserves the full detection-engineering evolution.

Historical stages include:

- manual local observation
- fail2ban validation
- no SIEM
- endpoint log collection
- Windows monitoring
- Linux monitoring
- first custom Linux rules
- firewall-log integration
- Suricata decoder failure
- custom decoder testing
- transition to structured JSON
- built-in Suricata detection
- Active Directory monitoring
- Nginx monitoring

The authoritative current state is:

    Detection Platform: Wazuh
    SIEM Host: LAB-SIEM-01
    Windows Monitoring: Active
    Linux Monitoring: Active
    Active Directory Monitoring: Active
    DMZ Monitoring: Active
    Firewall Monitoring: Active
    Suricata Monitoring: Active
    Application Monitoring: Active
    Custom Detection Engineering: Active

---

# Evolution Summary

Detection engineering evolved through:

    Manual Observation
          |
          v
    fail2ban Validation
          |
          v
    Wazuh Deployment
          |
          v
    Windows Telemetry
          |
          v
    Linux Telemetry
          |
          v
    Custom Linux Rules
          |
          v
    OPNsense Firewall Detection
          |
          v
    Suricata Integration
          |
          v
    Decoder Troubleshooting
          |
          v
    JSON / Built-In Rules
          |
          v
    Active Directory Monitoring
          |
          v
    Nginx Monitoring
          |
          v
    Multi-Source Detection Engineering
          |
          v
    Purple Team Validation Platform

---

# Final Status

The baseline detection-engineering phase is complete.

The Cybersecurity Home Lab now supports detection across:

- endpoint
- Linux
- Windows
- Active Directory
- firewall
- IDS
- web application

The environment has moved from basic log collection into a validated detection-engineering platform.

Future work should focus on realistic attack scenarios and answer questions such as:

- Did the attack generate telemetry?
- Did Wazuh detect it?
- Was the alert useful?
- Was important context missing?
- Did the detection create noise?
- How can the detection be improved?
- Can the improved detection survive retesting?

This detection-measurement-improvement cycle is now the foundation for future SOC and Purple Team operations in the lab.
