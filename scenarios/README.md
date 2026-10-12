# Purple Team Scenarios

## Overview

This directory contains realistic SOC and Purple Team exercises performed against the completed Cybersecurity Home Lab.

The infrastructure baseline is already complete.

Future work in this directory should focus on:

- attack simulation
- telemetry generation
- detection validation
- SOC investigation
- detection-gap analysis
- rule improvement
- retesting

The goal is no longer to keep adding infrastructure.

The goal is to use the existing environment as a realistic security operations platform.

---

# Lab Context

The current environment includes:

- REDNET — attacker network
- CORPNET — corporate workstation network
- SERVERNET — internal Linux server network
- SOCNET — security operations network
- ADNET — Active Directory network
- DMZ — public-facing server network

Core systems include:

- LAB-KALI-01
- LAB-WIN-01
- LAB-UBUNTU-01
- LAB-SIEM-01
- LAB-DC-01
- LAB-DMZ-01
- LAB-FW-01

---

# Scenario Philosophy

Each scenario should follow a repeatable workflow:

    Define Objective
          |
          v
    Select MITRE ATT&CK Technique
          |
          v
    Establish Baseline
          |
          v
    Execute Controlled Activity
          |
          v
    Collect Telemetry
          |
          v
    Validate Detection
          |
          v
    Investigate as SOC Analyst
          |
          v
    Identify Detection Gaps
          |
          v
    Improve Detection
          |
          v
    Retest
          |
          v
    Document Final Result

---

# Scenario Requirements

Every scenario should answer:

1. What behaviour is being simulated?
2. What system performs the activity?
3. What system is targeted?
4. Which MITRE ATT&CK technique applies?
5. What telemetry should be generated?
6. Which security controls should detect it?
7. What was actually detected?
8. What evidence supports the finding?
9. Was there a detection gap?
10. What improvement was made?
11. Did the retest succeed?

---

# Documentation Standard

Each scenario should include:

- title
- scenario ID
- objective
- attack hypothesis
- environment
- source system
- target system
- MITRE ATT&CK mapping
- prerequisites
- baseline state
- attack execution
- expected telemetry
- observed telemetry
- Wazuh or IDS detections
- SOC investigation
- timeline
- findings
- detection gap
- remediation or tuning
- retest
- final result
- lessons learned
- evidence references

---

# Safety and Scope

All activity must remain inside the authorized home lab.

Do not target:

- public systems
- third-party infrastructure
- external organizations
- systems without explicit authorization

Offensive activity should be controlled and designed to produce useful security telemetry.

---

# Detection Sources

Scenarios may use telemetry from:

## Windows

- Security logs
- Sysmon
- PowerShell
- Defender
- Windows Firewall
- Wazuh FIM

## Linux

- authentication logs
- system logs
- auditd
- Wazuh Agent

## Active Directory

- Windows Security logs
- directory service events
- Sysmon
- PowerShell
- Group Policy-related events

## Network

- OPNsense firewall logs
- Suricata
- rsyslog

## Applications

- Nginx access logs
- Nginx error logs

---

# Scenario Categories

Future scenarios may include:

- reconnaissance
- credential attacks
- suspicious PowerShell
- account manipulation
- privilege escalation
- persistence
- lateral movement
- Active Directory reconnaissance
- Active Directory abuse
- web attacks
- suspicious outbound traffic
- firewall-policy violations
- Linux account changes
- scheduled tasks
- command-and-control simulation
- incident-response exercises

---

# Evidence Strategy

Evidence should be added only when it clearly supports a documented finding.

Each evidence item should show:

    Action
      +
    Telemetry
      +
    Detection
      +
    Analyst Conclusion

Avoid collecting screenshots without context.

---

# Scenario Naming

Recommended directory format:

    SCN-001-<short-name>/
    SCN-002-<short-name>/
    SCN-003-<short-name>/

Example:

    scenarios/
    ├── README.md
    ├── scenario-template.md
    ├── SCN-001-powershell-recon/
    ├── SCN-002-linux-account-persistence/
    └── SCN-003-dmz-web-recon/

---

# Maturity Goal

The purpose of this directory is to demonstrate progression from:

`Building Security Infrastructure`

to:

`Operating and Testing Security Infrastructure`

A strong scenario should show both offensive and defensive understanding.

The most valuable outcome is not simply:

`Attack succeeded`

or:

`Alert fired`

The preferred outcome is:

`Activity was executed, telemetry was generated, detection was evaluated, analyst investigation was performed, gaps were identified, controls were improved, and the scenario was retested.`

---

# Current Status

The lab infrastructure baseline is complete.

All new operational security work should now be documented in this directory using the scenario template.
