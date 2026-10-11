# Evidence Index

## Overview

This directory contains validation evidence collected throughout the Cybersecurity Home Lab project.

The purpose of the evidence repository is to demonstrate that the documented architecture and security controls were not only configured, but also tested and observed in operation.

Evidence is organized by security function rather than by build order.

The main categories are:

- networking
- firewall
- Windows
- Linux
- Wazuh
- attack detection

The evidence should be read together with the technical documentation in:

- `architecture/`
- `networking/`
- `firewall/`
- `virtual-machines/`
- `blue-team/`
- `monitoring/`
- `docs/`

---

# Evidence Philosophy

A configuration is not treated as complete simply because it exists.

Where practical, the project follows this validation model:

    Configure
        |
        v
    Generate Activity
        |
        v
    Observe Result
        |
        v
    Verify Expected Behaviour
        |
        v
    Preserve Evidence

This applies to both:

- successful traffic
- blocked traffic

and to both:

- offensive activity
- defensive detection

---

# What the Evidence Proves

The collected evidence is intended to demonstrate practical implementation of:

- network segmentation
- firewall enforcement
- endpoint monitoring
- centralized log collection
- Active Directory services
- Group Policy
- Linux auditing
- Windows auditing
- Sysmon
- PowerShell logging
- Suricata IDS
- Nginx monitoring
- Wazuh detections
- custom detection engineering
- controlled attack validation

---

# Final Lab Context

The authoritative current architecture consists of six internal security zones.

| Zone | Network | Primary System |
|---|---|---|
| REDNET | `10.10.10.0/24` | `LAB-KALI-01` |
| CORPNET | `10.10.20.0/24` | `LAB-WIN-01` |
| SERVERNET | `10.10.30.0/24` | `LAB-UBUNTU-01` |
| SOCNET | `10.10.40.0/24` | `LAB-SIEM-01` |
| ADNET | `10.10.50.0/24` | `LAB-DC-01` |
| DMZ | `10.10.60.0/24` | `LAB-DMZ-01` |

Central firewall:

`LAB-FW-01`

Platform:

`OPNsense`

Historical screenshots from the original:

`192.168.56.0/24`

environment may remain in the repository where they document the early project lifecycle.

They should not be interpreted as the current architecture.

---

# Evidence Directory Structure

## `networking/`

Contains evidence related to:

- IP addressing
- interface configuration
- routing
- host connectivity
- DNS
- service-level connectivity
- final six-zone reachability validation

Important evidence may include tests showing:

- CORPNET → ADNET DNS
- CORPNET → ADNET Kerberos
- CORPNET → ADNET LDAP
- SERVERNET → SOCNET Wazuh
- ADNET → SOCNET Wazuh
- DMZ → SOCNET Wazuh
- DMZ → Internet HTTPS

This category demonstrates that the required network paths operate correctly.

---

# `firewall/`

Contains evidence related to OPNsense policy enforcement.

This should include both allowed and blocked traffic.

Representative validated controls include:

    REDNET → DMZ HTTP/80
    ALLOW

    REDNET → DMZ SSH/22
    BLOCK

    REDNET → CORPNET SMB/445
    BLOCK

    REDNET → ADNET LDAP/389
    BLOCK

    LAB-WIN-01 → DMZ SSH/22
    ALLOW

    SERVERNET → CORPNET SMB/445
    BLOCK

    SERVERNET → DMZ SSH/22
    BLOCK

    ADNET → DMZ SSH/22
    BLOCK

    DMZ → CORPNET SMB/445
    BLOCK

    DMZ → ADNET LDAP/389
    BLOCK

Firewall evidence may include:

- rule configuration
- OPNsense Live View
- successful service tests
- timed-out blocked connections
- logged deny events

---

# Firewall Rule-Order Evidence

The repository may also contain evidence related to the REDNET-to-DMZ HTTP troubleshooting incident.

The rule existed and was correctly ordered but was not active until OPNsense changes were applied.

This demonstrated the operational workflow:

    Save
      |
      v
    Apply
      |
      v
    Generate Fresh Traffic
      |
      v
    Verify

This is an important troubleshooting lesson and should remain documented.

---

# `windows/`

Contains evidence from:

`LAB-WIN-01`

and, where appropriate, Windows-related Domain Controller activity.

Windows evidence may demonstrate:

- Wazuh Agent status
- Sysmon
- Windows Security auditing
- PowerShell logging
- Defender
- Windows Firewall logging
- File Integrity Monitoring
- domain membership
- Group Policy application

---

# Sysmon Evidence

Sysmon evidence demonstrates enhanced endpoint process visibility.

Relevant evidence may show:

- process creation
- command execution
- suspicious utilities
- Wazuh ingestion of Sysmon events

Sysmon is used on:

- `LAB-WIN-01`
- `LAB-DC-01`

---

# PowerShell Evidence

PowerShell monitoring includes:

- Script Block Logging
- Module Logging
- transcription
- process execution context

This evidence demonstrates improved visibility into PowerShell activity beyond simple process creation.

---

# Windows Firewall Evidence

Windows Firewall telemetry includes:

`Event ID 5157`

This evidence is useful for distinguishing host-level firewall enforcement from OPNsense network-level enforcement.

---

# Active Directory Evidence

Windows evidence may also contain Active Directory validation from:

`LAB-DC-01`

Relevant items include:

- domain identity
- Domain Controller discovery
- SYSVOL
- NETLOGON
- Group Policy
- security auditing
- Sysmon

Important observed security events include:

- `4672`
- `4719`
- `5136`

---

# Group Policy Evidence

Evidence may demonstrate application of:

`GPO-Workstation-Security-Baseline`

to:

`CyberLab\Workstations`

Controls include:

- Windows Firewall enabled
- inbound block
- outbound allow
- Logon Success/Failure auditing
- Process Creation auditing
- Special Logon auditing
- command-line process logging
- PowerShell Script Block Logging
- PowerShell Module Logging

---

# `linux/`

Contains evidence from:

`LAB-UBUNTU-01`

and Linux security monitoring.

The Linux monitoring journey began before Wazuh with:

- SSH logs
- Nmap
- Hydra
- fail2ban

and later evolved into:

- Wazuh
- auditd
- centralized Linux detection

---

# Historical Linux Evidence

Earlier evidence may reference:

`192.168.56.30`

This belongs to the original flat lab.

It may show:

- SSH open on port 22
- Hydra authentication testing
- fail2ban
- attacker blocking
- service becoming filtered

These screenshots remain valuable because they demonstrate the early attacker/defender phase of the project.

---

# Current Linux Monitoring Evidence

Current Linux evidence should relate to:

`LAB-UBUNTU-01 — 10.10.30.30`

and may demonstrate:

- Wazuh connectivity
- auditd ingestion
- account-management detections
- segmentation tests

Validated custom detections include:

`Rule 100200`

Native Linux user creation.

`Rule 100201`

Native Linux user deletion.

---

# `wazuh/`

Contains centralized SIEM evidence.

This category should demonstrate the role of:

`LAB-SIEM-01 — 10.10.40.10`

as the central security monitoring platform.

---

# Wazuh Stack Evidence

Relevant evidence may include healthy status for:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Final manager version:

`v4.14.7`

---

# Final Agent Evidence

Final agent state:

    ID 001 — LAB-WIN-01 — Active
    ID 002 — LAB-UBUNTU-01 — Active
    ID 003 — LAB-DC-01 — Active
    ID 004 — LAB-DMZ-01 — Active

Evidence should demonstrate the final monitored endpoint inventory.

---

# Wazuh Detection Evidence

Validated detections include:

| Detection | Rule |
|---|---|
| Linux user creation | `100200` |
| Linux user deletion | `100201` |
| OPNsense firewall correlation | `87702` |
| Suricata alert | `86601` |
| Nginx web error | `31101` |

These represent different telemetry sources and demonstrate multi-source SIEM coverage.

---

# OPNsense Firewall Detection

Wazuh Rule:

`87702`

was validated using OPNsense firewall telemetry.

This demonstrates that traffic blocked at the firewall can still become SOC-visible security evidence.

---

# Nginx Monitoring

`LAB-DMZ-01`

forwards application telemetry through its Wazuh Agent.

Monitored files include:

- `/var/log/nginx/access.log`
- `/var/log/nginx/error.log`

A controlled request to:

`/wazuh-nginx-verify-01`

returned:

`404`

and generated:

`Rule 31101`

This demonstrates application-layer monitoring.

---

# `attack-detection/`

Contains evidence generated from controlled offensive or Purple Team validation activity.

This category should show the relationship between:

    Attack / Test
         |
         v
    Security Telemetry
         |
         v
    Detection
         |
         v
    Analyst Verification

---

# Suricata Evidence

Suricata runs on:

`LAB-FW-01`

Primary monitored interface:

`em2`

Zone:

`REDNET`

ET Open rules are enabled.

---

# Suricata Signature Validation

A known controlled signature was validated:

`SID 2100498`

Signature:

`GPL ATTACK_RESPONSE id check returned root`

Category:

`Potentially Bad Traffic`

Severity:

`2`

---

# Final REDNET-to-DMZ IDS Validation

The final test used:

`LAB-KALI-01 — 10.10.10.10`

and:

`LAB-DMZ-01 — 10.10.60.10`

A temporary HTTP response contained:

    uid=0(root) gid=0(root) groups=0(root)

Suricata generated:

    Source: 10.10.60.10:80
    Destination: 10.10.10.10
    Direction: to_client
    SID: 2100498

Wazuh generated:

`Rule 86601`

This proves the final IDS-to-SIEM pipeline.

---

# Suricata Telemetry Pipeline

The validated architecture is:

    LAB-FW-01
      Suricata
         |
         v
    OPNsense Remote Logging
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

Historical evidence showing the earlier custom decoder troubleshooting should be preserved where clearly identified as an earlier stage.

The current authoritative pipeline uses JSON ingestion and built-in Wazuh Suricata rules.

---

# Evidence Naming Guidance

Evidence files should use descriptive names wherever practical.

Recommended pattern:

    <system>-<control-or-test>-<result>.<extension>

Examples:

    kali-dmz-http-allow.png
    kali-dmz-ssh-block.png
    windows-dc-ldap-pass.png
    ubuntu-siem-1514-pass.png
    dmz-adnet-ldap-block.png
    wazuh-agents-active.png
    wazuh-rule-86601-suricata.png
    wazuh-rule-31101-nginx.png

This makes the evidence understandable without opening every file.

Existing files do not need to be renamed purely for cosmetic reasons if doing so would break repository references.

---

# Evidence Quality Guidelines

Each evidence item should ideally answer:

1. What system generated the evidence?
2. What control or activity was being tested?
3. What result was expected?
4. What result was observed?
5. Why does the evidence matter?

Screenshots should avoid unnecessary unrelated content where practical.

Sensitive information should not be exposed.

---

# Historical Evidence Policy

Historical evidence should not be deleted simply because the architecture changed.

The repository intentionally preserves the project's evolution.

Historical evidence is valuable when it demonstrates:

- early learning
- troubleshooting
- failed approaches
- architectural improvement
- detection development

However, historical evidence must not be presented as the current authoritative configuration.

---

# Current vs Historical Indicators

Examples of historical indicators:

- `192.168.56.x`
- flat network
- no OPNsense
- no SOCNET
- no ADNET
- no DMZ
- custom Suricata wrapper decoder

Examples of current indicators:

- `10.10.10.0/24` through `10.10.60.0/24`
- six security zones
- OPNsense
- Wazuh
- Active Directory
- DMZ
- JSON Suricata ingestion
- four active Wazuh agents

---

# Evidence and Documentation Relationship

The evidence directory should support claims made elsewhere in the repository.

Examples:

`firewall/validation-testing.md`

describes firewall tests.

`evidence/firewall/`

should contain supporting proof.

`monitoring/suricata-wazuh-integration.md`

describes the IDS pipeline.

`evidence/attack-detection/`

and/or:

`evidence/wazuh/`

should contain the corresponding detection proof.

`docs/active-directory.md`

describes AD deployment.

Relevant Windows evidence should support:

- domain health
- GPO
- monitoring
- identity events

---

# Portfolio Value

The evidence repository demonstrates that this project involved more than documentation or theoretical configuration.

It provides practical proof of work involving:

- architecture design
- network engineering
- endpoint monitoring
- firewall implementation
- security testing
- SIEM operations
- detection engineering
- troubleshooting
- Active Directory
- Purple Team validation

The strongest evidence is not simply a screenshot of a configuration page.

The strongest evidence shows:

    Test
      +
    Expected Security Behaviour
      +
    Observed Result
      +
    Detection / Verification

---

# Final Evidence Objectives

The evidence collection should ultimately demonstrate that the final lab can answer questions such as:

- Can the attacker network reach the DMZ web server?
- Is SSH from REDNET blocked?
- Can the corporate workstation use Active Directory services?
- Can unauthorized zones reach protected internal services?
- Can monitored endpoints reach the SIEM?
- Does Wazuh receive Windows telemetry?
- Does Wazuh receive Linux telemetry?
- Does Wazuh receive firewall telemetry?
- Does Suricata detect suspicious network content?
- Does Wazuh receive Suricata alerts?
- Does Wazuh monitor Nginx activity?
- Are Active Directory security events visible?
- Are custom Linux detections working?

The final answer to each of these questions should be supported by evidence in this repository.

---

# Current Status

The infrastructure baseline is complete.

The next evidence added to this repository should increasingly come from realistic SOC and Purple Team scenarios rather than additional baseline configuration.

Future evidence should demonstrate:

- attack execution
- alert generation
- triage
- investigation
- timeline reconstruction
- detection gaps
- detection improvement
- retesting

This will show the transition from:

`Building Security Infrastructure`

to:

`Operating Security Infrastructure`
