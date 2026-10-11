# LAB-UBUNTU-01 — Internal Linux Server

## Overview

This document records the complete lifecycle of the Ubuntu Server system used throughout the Cybersecurity Home Lab, from the original flat-network environment through its final role as `LAB-UBUNTU-01` in the segmented six-zone architecture.

Ubuntu Server has been part of the project since the earliest practical security exercises.

Its role evolved from a simple Linux target used for SSH, Nmap, Hydra, and fail2ban testing into a monitored internal server integrated with:

- OPNsense
- SERVERNET segmentation
- Wazuh
- auditd
- custom Linux detections
- centralized SOC monitoring

Final host:

`LAB-UBUNTU-01`

Final address:

`10.10.30.30`

Final zone:

`SERVERNET`

Primary role:

Protected internal Linux server

---

# Phase 1 — Original Ubuntu Lab Server

The earliest version of the lab used Ubuntu Server on:

`192.168.56.0/24`

Original Ubuntu address:

`192.168.56.30`

The other major systems included:

- Kali — `192.168.56.10`
- Windows — `192.168.56.20`

At this stage, Ubuntu served as the primary Linux target system.

It was used to learn and validate:

- Linux administration
- SSH
- service exposure
- Nmap enumeration
- authentication testing
- Hydra
- fail2ban
- attacker/defender interaction

This was one of the earliest practical security components of the lab.

---

# Early SSH Configuration

SSH was enabled on the Ubuntu server so it could be administered remotely and used as a target for controlled security testing.

Kali was used to:

- identify TCP/22
- connect to the SSH service
- enumerate the host
- generate authentication failures

This provided a practical environment for understanding the relationship between:

- network services
- authentication
- logging
- defensive controls

---

# Nmap Testing

Kali used Nmap against the Ubuntu server.

An early observation showed:

`22/tcp open`

This confirmed that:

- the host was reachable
- SSH was listening
- network communication was functioning

Nmap became one of the first tools used to validate changes in the server’s security posture.

---

# Hydra Authentication Testing

Hydra was used for controlled SSH authentication testing against Ubuntu.

The objective was not simply password guessing.

The exercise was used to understand:

- repeated authentication failures
- SSH logging
- automated defense
- attacker IP blocking
- how service state appears from the attacker side

This became an early Purple Team-style workflow.

---

# fail2ban Deployment

fail2ban was configured on Ubuntu to protect SSH from repeated failed authentication attempts.

Kali then generated controlled repeated failures.

The defensive response was observed.

After fail2ban acted, Nmap or subsequent connection attempts showed the SSH port as:

`filtered`

rather than openly reachable.

This was an important early lesson.

The security workflow became:

    Kali Attack
        |
        v
    SSH Authentication Failures
        |
        v
    Ubuntu Logs
        |
        v
    fail2ban
        |
        v
    Source IP Blocked
        |
        v
    Kali Retest

This was one of the earliest complete attack-detect-block-verify cycles in the project.

---

# Importance of the Early Ubuntu Phase

The original Ubuntu server helped establish several principles that continued throughout the entire lab:

- test security controls from the attacker perspective
- validate actual network behaviour
- observe logs
- compare before and after states
- do not assume a configuration works because it was saved
- retest after defensive changes

These same ideas later guided:

- OPNsense validation
- Wazuh detection engineering
- Suricata testing
- DMZ validation
- Active Directory segmentation

---

# Transition Away from the Flat Network

The original `192.168.56.0/24` lab was useful for foundational learning, but it lacked realistic security boundaries.

Ubuntu could communicate too freely with other systems.

As the lab matured, the server needed to be separated from:

- attacker infrastructure
- corporate workstations
- SOC infrastructure
- identity services

This led to the introduction of SERVERNET.

---

# Rebuild into LAB-UBUNTU-01

The Linux server was rebuilt and standardized as:

`LAB-UBUNTU-01`

Its final network became:

`SERVERNET`

Network:

`10.10.30.0/24`

Gateway:

`10.10.30.1`

Final IP:

`10.10.30.30`

This transformed the server from a simple attack target into a protected internal enterprise-style asset.

---

# SERVERNET Role

SERVERNET represents an internal protected server network.

`LAB-UBUNTU-01` is not intended to behave like a publicly exposed system.

Its role is to represent an internal Linux server that:

- provides services
- generates security telemetry
- receives approved administration
- is protected from unnecessary lateral movement

---

# OPNsense Integration

Once the server moved into SERVERNET, all inter-zone traffic passed through:

`LAB-FW-01`

This provided:

- routing
- firewall enforcement
- logging
- segmentation

The server no longer relied on unrestricted direct communication with other VMs.

---

# Early SERVERNET Policy

Initial SERVERNET behaviour included approved access for:

- DNS
- NTP
- Internet updates
- controlled administration
- later Wazuh telemetry

Unnecessary traffic toward:

- REDNET
- CORPNET

was restricted.

This improved isolation compared with the original flat lab.

---

# CORPNET Administration

During the initial segmented architecture, the Windows workstation was allowed to access selected Ubuntu services.

Approved administrative access included:

- SSH TCP/22
- HTTPS TCP/443 where required

This provided a realistic workstation-to-server management path.

---

# REDNET Testing

During earlier OPNsense stages, Kali retained controlled access to Ubuntu for security testing.

This preserved the original offensive exercises while placing them behind explicit firewall policy.

Examples included:

- Nmap
- SSH testing
- Hydra
- fail2ban validation

The important difference was that this access now existed because of intentional firewall policy rather than flat-network trust.

---

# Wazuh Integration

`LAB-UBUNTU-01` was later integrated with Wazuh.

The server sends telemetry to:

`LAB-SIEM-01`

Address:

`10.10.40.10`

Primary agent communication:

`TCP/1514`

Enrollment may use:

`TCP/1515`

Final agent identity:

`ID 002 — LAB-UBUNTU-01`

Final state:

`Active`

---

# Linux Monitoring

The Linux monitoring phase expanded beyond basic host logs.

The final monitoring design included:

- Wazuh agent telemetry
- authentication logs
- system events
- auditd
- custom detection rules

This transformed Ubuntu from a basic target into a useful Blue Team data source.

---

# auditd Integration

`auditd` was introduced to provide deeper Linux security visibility.

Audit logs were ingested into Wazuh.

This allowed the SIEM to observe security-relevant Linux events that standard logs alone might not capture as clearly.

The audit subsystem became especially useful for:

- account changes
- privileged actions
- system administration
- detection engineering

---

# User-Creation Detection

A controlled Linux user-creation event was used to validate detection logic.

Wazuh generated:

`Rule 100200`

This represented a custom detection for a native Linux user-add event.

The objective was to prove that account creation on the server could be:

- generated
- logged
- ingested
- decoded
- detected

Status:

**PASS**

---

# User-Deletion Detection

A controlled user-deletion event was also generated.

Wazuh generated:

`Rule 100201`

This represented a custom detection for a native Linux user-delete event.

Status:

**PASS**

Together, the two detections demonstrated:

    Linux Administrative Action
        |
        v
    auditd / System Logging
        |
        v
    Wazuh Agent
        |
        v
    LAB-SIEM-01
        |
        v
    Custom Detection

---

# Detection Engineering Role

The Ubuntu server became an important platform for Linux detection engineering.

Rather than only installing an agent and collecting logs, the lab used Ubuntu to create and validate custom security rules.

This represented a major development from the original fail2ban-only phase.

The project had evolved from:

**host protection**

to:

**centralized detection engineering**

---

# SERVERNET to SOCNET Validation

Final test:

    nc -zv -w 3 10.10.40.10 1514

Expected:

`ALLOW`

Result:

Connection succeeded.

Status:

**PASS**

This confirmed the Linux server could send Wazuh telemetry to SOCNET.

---

# SERVERNET to CORPNET Validation

Final test:

    nc -zv -w 3 10.10.20.20 445

Expected:

`BLOCK`

Result:

Connection timed out.

Status:

**PASS**

This confirmed that the Linux server could not freely access Windows SMB services in CORPNET.

---

# SERVERNET to DMZ Validation

Final test:

    nc -zv -w 3 10.10.60.10 22

Expected:

`BLOCK`

Result:

Connection timed out.

Status:

**PASS**

This confirmed that the internal server was not permitted to use SSH to administer the DMZ.

---

# Service Testing vs Ping

One important troubleshooting lesson occurred when ICMP behaviour did not match actual service connectivity.

At one point, Ubuntu could not successfully ping the SIEM.

However:

`TCP/1514`

was reachable.

This demonstrated that:

**ICMP failure does not automatically mean the required service path is broken.**

From that point forward, the project increasingly tested actual application ports instead of relying only on ping.

---

# Wazuh Agent Connectivity Troubleshooting

At one stage, `LAB-UBUNTU-01` appeared disconnected in Wazuh.

The investigation compared:

- agent logs
- timestamps
- network connectivity
- manager state

Recent logs showed the agent had connected successfully.

The manager later reported the agent as:

`Active`

This reinforced an important monitoring lesson:

**Historical errors should not automatically be treated as current failures.**

Always correlate:

- timestamps
- current service status
- current manager state
- current network connectivity

---

# Final Wazuh State

The final Wazuh agent state for Ubuntu is:

    ID: 002
    Name: LAB-UBUNTU-01
    Status: Active

This was confirmed during the final lab-wide validation.

---

# Server Shutdown Troubleshooting

During the final shutdown process, Ubuntu initially refused to power off cleanly because the shutdown was inhibited by:

`unattended-upgrades`

A process inspection showed:

`unattended-upgrade-shutdown --wait-for-signal`

No active package-management process such as:

- apt
- apt-get
- dpkg

was performing an installation at that time.

The service was therefore safely stopped before shutdown.

The resolution used:

    systemctl stop unattended-upgrades.service

followed by:

    poweroff

This allowed the server to shut down normally.

---

# Why the Shutdown Check Mattered

The service was not simply terminated blindly.

The process state was first reviewed to ensure package operations were not actively modifying the system.

This preserved the project’s change-management philosophy:

- inspect
- understand
- act
- verify

rather than forcing a shutdown without determining the cause.

---

# VirtualBox Soft Lockup Messages

During some shutdown or high-load situations, Ubuntu displayed soft-lockup-style messages.

The VM remained responsive.

The likely cause was VirtualBox host CPU contention rather than evidence of persistent guest OS corruption.

Because the system remained functional and validation passed, unnecessary kernel changes were avoided.

This followed another important project principle:

**Do not make major system changes solely because of an isolated symptom when the evidence does not support it.**

---

# Storage and System Health

The Ubuntu server remained operational during final validation.

No critical server-side failure prevented:

- Wazuh telemetry
- auditd monitoring
- networking
- firewall validation
- clean final snapshot creation

The host was therefore considered stable.

---

# Ubuntu Role in Purple Team Operations

The final Ubuntu server can support future scenarios such as:

- suspicious user creation
- privilege escalation
- unauthorized account deletion
- SSH attacks
- persistence
- service changes
- file tampering
- privilege abuse
- suspicious process execution
- detection engineering

Its current purpose is no longer to serve only as a basic SSH target.

It now functions as a protected Linux enterprise asset with centralized monitoring.

---

# Defense-in-Depth Model

The final Ubuntu security architecture can be represented as:

    User / Process Activity
          |
          v
    LAB-UBUNTU-01
          |
          +--> Linux Logs
          |
          +--> auditd
          |
          +--> Wazuh Agent
          |
          v
    LAB-SIEM-01

Network communication is additionally controlled by:

`LAB-FW-01`

This provides both:

- endpoint visibility
- network segmentation

---

# Final Validation Summary

| Validation | Result |
|---|---|
| SERVERNET → SOCNET Wazuh/1514 | PASS |
| SERVERNET → CORPNET SMB/445 blocked | PASS |
| SERVERNET → DMZ SSH/22 blocked | PASS |
| Wazuh agent active | PASS |
| auditd ingestion | PASS |
| Native user-add detection Rule 100200 | PASS |
| Native user-delete detection Rule 100201 | PASS |
| Controlled server shutdown | PASS |

---

# Snapshot History

Snapshots were used before major Linux and networking changes throughout the project.

These protected the server during:

- network rebuild
- OPNsense migration
- Wazuh deployment
- auditd integration
- detection engineering
- firewall changes
- final validation

After the full lab was validated, obsolete development snapshots were removed.

The remaining snapshot is:

`Final-Six-Zone-Validated-Baseline`

Snapshot UUID:

`23d97e1a-a95e-4dd9-8912-f48e312ccf74`

---

# Historical vs Current Ubuntu State

This document intentionally preserves the server’s entire lifecycle.

Historical state includes:

    IP: 192.168.56.30
    Role: Basic Ubuntu attack target
    Services: SSH
    Defensive control: fail2ban
    Testing: Nmap / Hydra / SSH

Current authoritative state is:

    Hostname: LAB-UBUNTU-01
    IP: 10.10.30.30
    Zone: SERVERNET
    Gateway: 10.10.30.1
    Wazuh Agent: Active
    auditd: Integrated
    Role: Protected Internal Linux Server

The earlier configuration remains documented because it represents the original stage of the project rather than an error.

---

# Evolution Summary

The Ubuntu server evolved through several security maturity stages:

    Basic Linux Server
        |
        v
    SSH Target
        |
        v
    Hydra / fail2ban Testing
        |
        v
    Segmented SERVERNET Asset
        |
        v
    Wazuh-Monitored Endpoint
        |
        v
    auditd Integration
        |
        v
    Custom Linux Detection Engineering
        |
        v
    Final Protected Enterprise Server

This progression reflects the wider evolution of the entire Cybersecurity Home Lab.

---

# Final Status

The Ubuntu infrastructure phase is complete.

`LAB-UBUNTU-01` is now:

- correctly placed in SERVERNET
- protected by OPNsense segmentation
- connected to Wazuh
- monitored with auditd
- capable of generating custom Linux detections
- blocked from unauthorized CORPNET and DMZ access
- validated as part of the final six-zone environment
- ready for realistic SOC and Purple Team scenarios

Future activity should focus on controlled attack simulation, detection, investigation, and response rather than further baseline infrastructure changes.
