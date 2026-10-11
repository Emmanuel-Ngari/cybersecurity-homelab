# Linux Endpoint Security Monitoring

## Overview

This document records the complete evolution of Linux endpoint security monitoring in the Cybersecurity Home Lab, from the earliest Ubuntu SSH/fail2ban phase through the final monitored `LAB-UBUNTU-01` deployment in SERVERNET.

Linux monitoring did not begin with Wazuh.

The earliest Ubuntu environment was monitored mainly through:

- SSH logs
- local system logs
- fail2ban
- manual log inspection
- attacker retesting from Kali

As the project matured, Linux monitoring evolved into a centralized SOC capability using:

- Wazuh Agent
- Linux authentication logs
- system logs
- auditd
- custom Wazuh rules
- firewall-aware troubleshooting
- endpoint validation
- centralized detection engineering

The final monitored Linux endpoint is:

`LAB-UBUNTU-01`

Final address:

`10.10.30.30`

Final zone:

`SERVERNET`

Monitoring destination:

`LAB-SIEM-01 — 10.10.40.10`

Final Wazuh agent:

`ID 002 — LAB-UBUNTU-01`

Status:

`Active`

---

# Phase 1 — Original Linux Monitoring

The original Ubuntu Server operated on:

`192.168.56.30`

within the early flat lab.

The other main systems were:

- Kali — `192.168.56.10`
- Windows — `192.168.56.20`

At this stage, security monitoring was local and manual.

The server was primarily used to learn:

- SSH
- authentication
- Linux logs
- fail2ban
- Nmap
- Hydra
- attacker/defender interaction

There was no centralized SIEM.

---

# Early SSH Visibility

SSH became one of the first monitored Linux services.

Kali interacted with the Ubuntu SSH service from:

`192.168.56.10`

The Ubuntu system recorded authentication activity locally.

This allowed the project to observe:

- successful SSH connections
- failed login attempts
- repeated authentication failures
- source addresses
- service behaviour

This was the earliest practical Linux security telemetry in the lab.

---

# Nmap and Service Validation

Kali used Nmap to examine Ubuntu.

An early result showed:

`22/tcp open`

This confirmed that:

- the server was reachable
- SSH was exposed
- the service was listening

After defensive controls were applied, the same service could appear differently from the attacker side.

This introduced the concept of validating monitoring and controls from both perspectives.

---

# Hydra Testing

Hydra was used against the SSH service in a controlled lab scenario.

The objective was to generate repeated authentication failures.

The Ubuntu system was then inspected to determine:

- whether the failures were logged
- whether fail2ban detected them
- whether the attacker was blocked
- whether the service behaviour changed

This was an early example of generating telemetry intentionally.

---

# fail2ban Monitoring

fail2ban became the first major automated Linux defensive control used in the project.

It monitored authentication failures and reacted to repeated suspicious activity.

The workflow became:

    Kali
      |
      v
    SSH Authentication Failures
      |
      v
    Ubuntu Authentication Logs
      |
      v
    fail2ban
      |
      v
    Attacker Source Blocked
      |
      v
    Kali Retest

This represented the first Linux attack-detection-response cycle in the lab.

---

# Attacker-Side Validation

After fail2ban acted, Kali retested the target.

An earlier:

`22/tcp open`

state could later appear as:

`filtered`

This provided visible proof that the defensive control changed attacker access.

The lesson was important:

**A security control should be validated from both the defender and attacker perspectives.**

---

# Limits of Local Monitoring

Although fail2ban and local logs were useful, the early approach had limitations.

Investigation required logging directly into Ubuntu and reviewing local information.

There was no central place to correlate Linux activity with:

- Windows
- firewall
- IDS
- identity systems

This became increasingly inadequate as the lab expanded.

---

# Transition to SERVERNET

When OPNsense was introduced, Ubuntu was rebuilt and standardized as:

`LAB-UBUNTU-01`

Final network:

`SERVERNET`

Address:

`10.10.30.30`

Gateway:

`10.10.30.1`

The server was no longer a simple flat-network target.

It became a protected internal enterprise-style Linux asset.

---

# SERVERNET Monitoring Impact

Moving Ubuntu into SERVERNET changed how monitoring had to be approached.

Security now depended on both:

- endpoint visibility
- network policy

A missing event or disconnected agent could be caused by:

- Linux service state
- Wazuh configuration
- OPNsense firewall policy
- routing
- network availability

Monitoring therefore became more systematic.

---

# Wazuh Introduction

Wazuh was introduced as the central monitoring platform.

The SIEM was placed in:

`SOCNET`

Address:

`10.10.40.10`

The Ubuntu server was enrolled as:

    ID: 002
    Name: LAB-UBUNTU-01

The Wazuh agent established centralized Linux monitoring.

---

# Linux-to-SIEM Communication

Primary Wazuh communication uses:

`TCP/1514`

Enrollment may use:

`TCP/1515`

The required firewall path was configured between:

`SERVERNET`

and:

`SOCNET`

This allowed Ubuntu telemetry to reach the SIEM without granting unrestricted access to SOCNET.

---

# Service-Level Connectivity Testing

The Linux monitoring phase produced an important troubleshooting lesson.

At one stage, ICMP testing between Ubuntu and the SIEM did not succeed.

However:

`TCP/1514`

was reachable.

This demonstrated that:

**Ping failure does not prove the required monitoring service is unavailable.**

The correct validation was to test the actual service port.

---

# Wazuh Port Validation

A typical test used:

    nc -zv -w 3 10.10.40.10 1514

Successful connection confirmed that:

- routing worked
- OPNsense allowed the path
- the SIEM service was reachable

This became the preferred troubleshooting approach.

---

# Linux Wazuh Agent Monitoring

Once the agent was connected, the Ubuntu server began providing centralized telemetry.

Important sources included:

- system logs
- authentication logs
- administrative activity
- Linux security events

The monitoring architecture became:

    LAB-UBUNTU-01
          |
          v
      Linux Logs
          |
          v
      Wazuh Agent
          |
          v
    LAB-SIEM-01

---

# auditd Introduction

The Linux monitoring capability was later expanded using:

`auditd`

auditd provides deeper operating-system visibility than basic log collection alone.

It can record security-relevant activity involving:

- account management
- privileged commands
- file access
- system administration
- security configuration

This made it valuable for Linux detection engineering.

---

# auditd to Wazuh

auditd telemetry was ingested into Wazuh.

The monitoring pipeline became:

    Linux Activity
         |
         v
       auditd
         |
         v
    Audit Records
         |
         v
    Wazuh Agent
         |
         v
    LAB-SIEM-01

This provided a better foundation for custom detections.

---

# Account Management Monitoring

One of the first Linux detection-engineering objectives was to monitor local account changes.

User creation and deletion are security-relevant because they may indicate:

- legitimate administration
- attacker persistence
- unauthorized access
- account cleanup
- privilege-related activity

Controlled tests were therefore created.

---

# User Creation Detection

A native Linux user-add operation was generated on:

`LAB-UBUNTU-01`

Wazuh successfully detected the activity using:

`Rule 100200`

Purpose:

Detect native Linux user creation.

Status:

**PASS**

---

# User Creation Detection Path

The validated flow was:

    Linux User Creation
          |
          v
    Native Linux Event
          |
          v
       auditd / Logs
          |
          v
      Wazuh Agent
          |
          v
     LAB-SIEM-01
          |
          v
      Rule 100200

This proved that Linux account creation could be detected centrally.

---

# User Deletion Detection

A native user-deletion event was also generated.

Wazuh rule:

`Rule 100201`

Purpose:

Detect native Linux user deletion.

Status:

**PASS**

---

# User Deletion Detection Path

The flow was:

    Linux User Deletion
          |
          v
    Native Linux Event
          |
          v
       auditd / Logs
          |
          v
      Wazuh Agent
          |
          v
     LAB-SIEM-01
          |
          v
      Rule 100201

This completed a simple but important Linux account-management detection pair.

---

# Why Rules 100200 and 100201 Matter

These detections were significant because they demonstrated that the lab had moved beyond:

- installing Wazuh
- collecting logs
- viewing dashboards

The environment was now actively creating and validating detection logic around Linux activity.

---

# Monitoring vs Protection

Linux monitoring and Linux protection are separate concepts.

Examples:

## Protection

fail2ban blocks repeated authentication failures.

## Monitoring

Wazuh records and detects security-relevant activity.

The mature architecture uses both approaches where appropriate.

A security event does not always need to be blocked automatically.

Some events are better:

- recorded
- correlated
- investigated

---

# fail2ban vs Wazuh Evolution

The project therefore evolved from:

    fail2ban
      |
      v
    Local Automated Protection

to:

    auditd / Linux Logs
         |
         v
       Wazuh
         |
         v
    Centralized Detection

The earlier fail2ban work remains an important historical stage rather than something replaced or forgotten.

---

# OPNsense Firewall Context

The Linux server exists behind OPNsense.

This means monitoring problems can involve both:

- host state
- network policy

Final segmentation confirms the server can reach the SIEM while remaining isolated from unauthorized destinations.

---

# SERVERNET to SOCNET Validation

Test:

    nc -zv -w 3 10.10.40.10 1514

Expected:

`ALLOW`

Result:

Successful connection.

Status:

**PASS**

This confirms the endpoint-monitoring path.

---

# SERVERNET to CORPNET Validation

Test:

    nc -zv -w 3 10.10.20.20 445

Expected:

`BLOCK`

Result:

Timed out.

Status:

**PASS**

This shows that monitoring access to SOCNET did not create broad east-west trust.

---

# SERVERNET to DMZ Validation

Test:

    nc -zv -w 3 10.10.60.10 22

Expected:

`BLOCK`

Result:

Timed out.

Status:

**PASS**

This confirms that the internal Linux server cannot freely administer the DMZ.

---

# Agent Connectivity Troubleshooting

At one stage, the Wazuh manager appeared to show:

`LAB-UBUNTU-01`

as disconnected.

This required careful investigation.

The troubleshooting process examined:

- network connectivity
- agent logs
- timestamps
- manager status
- current service state

---

# Historical Log Lesson

Older Wazuh log entries contained connection errors.

However, more recent logs showed successful communication.

The manager later reported the agent as:

`Active`

The lesson was:

**Do not diagnose current system health using old error messages without checking timestamps and present state.**

---

# Current State vs Historical Errors

A professional monitoring investigation should ask:

- When did the error occur?
- Is the service currently active?
- Is the network path currently reachable?
- Does the manager currently see the agent?
- Are recent logs healthy?

This prevents unnecessary configuration changes.

---

# Reverse DNS Warning Lesson

During some `nc` connectivity checks, reverse-DNS-related warning messages could appear.

These warnings did not necessarily indicate connectivity failure.

The important result was whether the target TCP port connected successfully.

This reinforced another troubleshooting principle:

**Separate cosmetic diagnostic warnings from actual service failure.**

---

# Detection Validation Philosophy

Linux detections were not considered complete simply because a rule file existed.

The workflow required:

1. Generate real activity.
2. Confirm the source event.
3. Confirm Wazuh ingestion.
4. Confirm rule matching.
5. Confirm the resulting alert.
6. Record the outcome.

This became the standard detection-engineering model for the rest of the lab.

---

# Monitoring Architecture

The final Linux monitoring architecture is:

    Linux User / Process Activity
              |
              v
        LAB-UBUNTU-01
              |
              +--> Authentication Logs
              |
              +--> System Logs
              |
              +--> auditd
              |
              v
         Wazuh Agent
              |
              | TCP/1514
              v
         LAB-FW-01
           OPNsense
              |
              v
         LAB-SIEM-01
              |
              v
        Wazuh Detection

---

# Linux Detection Use Cases

The existing monitoring architecture can support future detections involving:

- user creation
- user deletion
- repeated SSH failures
- suspicious sudo activity
- privilege escalation
- persistence
- service changes
- file modifications
- unusual administrative commands
- attacker-created accounts
- unauthorized configuration changes

The infrastructure is already available to develop these detections as specific Purple Team scenarios are introduced.

---

# Linux Monitoring and Purple Team Operations

A future Purple Team scenario can follow:

    Kali / Simulated Attacker
             |
             v
       Linux Activity
             |
             v
        auditd / Logs
             |
             v
        Wazuh Agent
             |
             v
        LAB-SIEM-01
             |
             v
         Detection
             |
             v
       SOC Investigation
             |
             v
      Detection Improvement

This continues the same attacker/defender methodology that began with fail2ban in the earliest lab.

---

# Noise Management

auditd and Linux logs can produce significant telemetry.

The objective is not to create alerts for every administrative action.

Detection logic should prioritize activity that is useful for:

- SOC triage
- threat hunting
- investigation
- security validation

Rules should be tuned based on observed behaviour rather than created only to increase alert volume.

---

# Linux Endpoint vs Network Monitoring

The final environment monitors Linux activity at multiple layers.

## Endpoint Layer

- authentication logs
- auditd
- Wazuh Agent

## Network Layer

- OPNsense
- firewall logs
- Suricata where relevant

This provides stronger investigation capability than relying only on the endpoint.

---

# Example Correlation Scenario

A future SSH attack could potentially generate:

    SSH Connection Attempt
            |
            v
      OPNsense Evidence
            |
            v
      Linux SSH Logs
            |
            v
         auditd
            |
            v
      fail2ban Action
            |
            v
          Wazuh
            |
            v
      SOC Investigation

This represents a mature version of the same exercise performed in the original flat lab.

---

# Wazuh Agent Final State

Final Linux agent:

    ID: 002
    Name: LAB-UBUNTU-01
    Status: Active

The agent remained active during final six-zone validation.

---

# Final Monitoring Validation

| Validation | Result |
|---|---|
| Early SSH logging | PASS |
| Hydra-generated authentication failures | PASS |
| fail2ban response | PASS |
| Attacker-side retest | PASS |
| Wazuh agent installed | PASS |
| SERVERNET → SOCNET TCP/1514 | PASS |
| Linux logs ingested | PASS |
| auditd ingested | PASS |
| User-add Rule 100200 | PASS |
| User-delete Rule 100201 | PASS |
| Agent final state Active | PASS |
| SERVERNET → CORPNET SMB blocked | PASS |
| SERVERNET → DMZ SSH blocked | PASS |

---

# Troubleshooting Lessons

Linux monitoring produced several important lessons.

## Test the Actual Service

Do not rely only on ping.

## Check Timestamps

Historical errors do not prove a current outage.

## Separate Network and Agent Problems

Verify TCP/1514 before changing Wazuh configuration.

## Validate Raw Events First

Do not build a detection before confirming telemetry exists.

## Retest After Changes

A configuration is not complete until the behaviour is proven.

---

# Historical vs Current State

This document intentionally preserves the entire Linux monitoring journey.

Historical stages include:

- Ubuntu `192.168.56.30`
- local SSH logging
- Nmap validation
- Hydra authentication testing
- fail2ban
- attacker-side retesting
- no SIEM

Later stages include:

- SERVERNET migration
- Wazuh Agent deployment
- centralized monitoring
- auditd
- custom Linux detections
- firewall-aware troubleshooting
- final six-zone validation

The authoritative current state is:

    Host: LAB-UBUNTU-01
    IP: 10.10.30.30
    Zone: SERVERNET
    Wazuh Agent: Active
    Agent ID: 002
    auditd: Integrated
    Rule 100200: Validated
    Rule 100201: Validated
    SIEM Destination: 10.10.40.10

---

# Evolution Summary

Linux endpoint monitoring evolved through:

    Local SSH Logs
          |
          v
      Nmap Testing
          |
          v
      Hydra Testing
          |
          v
        fail2ban
          |
          v
    Attacker Retesting
          |
          v
    SERVERNET Migration
          |
          v
      Wazuh Agent
          |
          v
    Centralized Logging
          |
          v
        auditd
          |
          v
    Custom Linux Rules
          |
          v
    Connectivity Troubleshooting
          |
          v
    Final SOC-Ready Linux Monitoring

---

# Final Status

The Linux endpoint security-monitoring baseline is complete.

`LAB-UBUNTU-01` now provides:

- centralized Linux telemetry
- authentication visibility
- auditd data
- validated account-change detections
- controlled SIEM communication
- firewall-enforced segmentation
- a stable platform for Linux Purple Team exercises

The monitoring environment has evolved from local fail2ban validation into centralized SOC detection.

Future work should focus on realistic Linux attack scenarios, alert triage, threat hunting, privilege-escalation detection, persistence detection, and further rule development based on actual telemetry rather than adding monitoring components without a clear operational need.
