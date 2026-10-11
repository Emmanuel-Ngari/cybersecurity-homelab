# Firewall Policy Matrix

## Overview

This document records the firewall-policy model used throughout the Cybersecurity Home Lab and the final validated inter-zone communication matrix.

The lab evolved from a simple flat network into a six-zone architecture controlled by OPNsense.

The final firewall design follows a least-privilege and default-deny approach.

Each communication path is evaluated according to:

- source zone
- source host where required
- destination zone
- destination host
- protocol
- destination port
- operational requirement
- security risk
- logging requirement
- final validation result

The purpose of this policy matrix is to provide a clear reference for what communication is intentionally allowed and what communication is intentionally denied.

---

# Historical Context

## Original Flat Lab

The earliest version of the lab used:

`192.168.56.0/24`

with systems including:

- Kali Linux
- Windows
- Ubuntu Server

Because the systems shared the same general internal network, there was limited opportunity to enforce realistic east-west firewall policy.

The early environment was useful for learning:

- Nmap
- SSH
- fail2ban
- Hydra
- Windows/Linux connectivity
- basic attacker/defender interaction

However, it did not provide the network separation required for realistic Purple Team and SOC operations.

---

# Segmented Firewall Architecture

OPNsense was later introduced as the central firewall and Layer 3 routing platform.

The lab evolved through multiple segmentation phases.

The initial segmented environment introduced:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

Later development introduced:

- ADNET
- DMZ

The result is the final six-zone architecture.

---

# Final Security Zones

| Zone | Network | Gateway | Primary System |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `10.10.10.1` | `LAB-KALI-01` |
| CORPNET | `10.10.20.0/24` | `10.10.20.1` | `LAB-WIN-01` |
| SERVERNET | `10.10.30.0/24` | `10.10.30.1` | `LAB-UBUNTU-01` |
| SOCNET | `10.10.40.0/24` | `10.10.40.1` | `LAB-SIEM-01` |
| ADNET | `10.10.50.0/24` | `10.10.50.1` | `LAB-DC-01` |
| DMZ | `10.10.60.0/24` | `10.10.60.1` | `LAB-DMZ-01` |

All inter-zone traffic is routed through:

`LAB-FW-01`

---

# Firewall Design Principles

The firewall policy is based on the following principles.

## Default Deny

Traffic that is not explicitly required should not be permitted between protected zones.

## Least Privilege

Only the minimum required protocols and destinations are allowed.

## Specific Allow Before Broad Deny

Narrow approved rules are positioned before broader private-network blocks.

## Administrative Restriction

Administrative access is limited to approved management hosts.

## Security Monitoring

Telemetry paths to SOCNET are explicitly allowed.

## Attack Isolation

REDNET is treated as untrusted and is prevented from freely accessing protected enterprise infrastructure.

## DMZ Isolation

The DMZ can provide exposed services without receiving unrestricted access to internal networks.

## Logging

Important security boundaries are logged to support investigation and validation.

---

# Important OPNsense Aliases

Aliases were introduced to make firewall rules easier to understand and maintain.

| Alias | Purpose |
|---|---|
| `ADMIN_WORKSTATION` | Approved administration host `10.10.20.20` |
| `LAB_DMZ_01` | DMZ web server `10.10.60.10` |
| `WAZUH_SERVER` | Wazuh SIEM `10.10.40.10` |
| `WAZUH_AGENT_PORTS` | Wazuh TCP ports `1514` and `1515` |
| `DNS_PORT` | DNS service |
| `NTP_PORT` | NTP service |
| `HTTP_HTTPS_PORTS` | TCP ports `80` and `443` |
| `PRIVATE_NETS` | Protected internal/private address ranges |

Aliases improve rule readability and reduce dependence on raw IP addresses inside firewall policy.

---

# REDNET Policy

## Purpose

REDNET contains:

`LAB-KALI-01 — 10.10.10.10`

This zone is used for:

- penetration testing
- adversarial simulation
- network-security testing
- controlled Purple Team activity

REDNET is treated as the least-trusted internal zone.

---

## REDNET → DMZ HTTP

Source:

`REDNET`

Destination:

`LAB_DMZ_01`

Protocol:

`TCP`

Port:

`80`

Action:

`ALLOW`

Purpose:

Controlled web attack simulation.

Validated result:

`PASS`

---

## REDNET → DMZ SSH

Source:

`REDNET`

Destination:

`LAB_DMZ_01`

Protocol:

`TCP`

Port:

`22`

Action:

`BLOCK`

Purpose:

Prevent offensive systems from receiving administrative access to the DMZ server.

Validated result:

`PASS`

---

## REDNET → CORPNET SMB

Source:

`REDNET`

Destination:

`CORPNET`

Protocol:

`TCP`

Port:

`445`

Action:

`BLOCK`

Purpose:

Prevent direct attacker access to corporate workstation SMB services.

Validated result:

`PASS`

---

## REDNET → ADNET LDAP

Source:

`REDNET`

Destination:

`ADNET`

Protocol:

`TCP`

Port:

`389`

Action:

`BLOCK`

Purpose:

Prevent direct LDAP access from the offensive network to identity infrastructure.

Validated result:

`PASS`

---

# CORPNET Policy

## Purpose

CORPNET contains:

`LAB-WIN-01 — 10.10.20.20`

The workstation is:

- domain joined
- centrally monitored
- configured through Group Policy
- approved for restricted DMZ administration

---

# CORPNET → ADNET

The Windows workstation requires Active Directory connectivity.

## DNS

Source:

`CORPNET`

Destination:

`LAB-DC-01`

Protocol:

`TCP`

Port:

`53`

Action:

`ALLOW`

Validated result:

`PASS`

---

## Kerberos

Source:

`CORPNET`

Destination:

`LAB-DC-01`

Protocol:

`TCP`

Port:

`88`

Action:

`ALLOW`

Validated result:

`PASS`

---

## LDAP

Source:

`CORPNET`

Destination:

`LAB-DC-01`

Protocol:

`TCP`

Port:

`389`

Action:

`ALLOW`

Validated result:

`PASS`

---

## Operational Purpose

These services support:

- domain discovery
- authentication
- Kerberos
- LDAP queries
- Group Policy
- Active Directory operation

The lab may contain additional domain-related rules required for full Active Directory operation.

The specific ports above were directly validated during the final connectivity test.

---

# CORPNET → SOCNET

## Wazuh Telemetry

Source:

`CORPNET`

Destination:

`WAZUH_SERVER`

Protocol:

`TCP`

Port:

`1514`

Action:

`ALLOW`

Purpose:

Permit `LAB-WIN-01` to send security telemetry to Wazuh.

Validated result:

`PASS`

---

# CORPNET → DMZ

## Administrative SSH

Source:

`ADMIN_WORKSTATION`

Address:

`10.10.20.20`

Destination:

`LAB_DMZ_01`

Address:

`10.10.60.10`

Protocol:

`TCP`

Port:

`22`

Action:

`ALLOW`

Logging:

`Enabled`

Description:

`ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH`

Purpose:

Provide a controlled administrative path to the DMZ server.

Validated result:

`PASS`

---

## Why the Rule Is Host-Specific

The firewall does not broadly allow:

`CORPNET → DMZ SSH`

Instead, the rule is restricted specifically to:

`LAB-WIN-01`

This reduces the number of systems capable of administering the exposed server.

---

# SERVERNET Policy

## Purpose

SERVERNET contains:

`LAB-UBUNTU-01 — 10.10.30.30`

This network represents protected internal Linux server infrastructure.

---

# SERVERNET → SOCNET

## Wazuh Telemetry

Source:

`SERVERNET`

Destination:

`WAZUH_SERVER`

Protocol:

`TCP`

Port:

`1514`

Action:

`ALLOW`

Validated result:

`PASS`

---

# SERVERNET → CORPNET

## SMB

Source:

`SERVERNET`

Destination:

`CORPNET`

Protocol:

`TCP`

Port:

`445`

Action:

`BLOCK`

Purpose:

Reduce unnecessary lateral movement between the server and corporate workstation zones.

Validated result:

`PASS`

---

# SERVERNET → DMZ

## SSH

Source:

`SERVERNET`

Destination:

`DMZ`

Protocol:

`TCP`

Port:

`22`

Action:

`BLOCK`

Purpose:

Prevent the internal Linux server from becoming an unauthorized DMZ administration source.

Validated result:

`PASS`

---

# SOCNET Policy

## Purpose

SOCNET contains:

`LAB-SIEM-01 — 10.10.40.10`

This zone provides centralized security monitoring.

Services include:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat
- Suricata event ingestion

---

# Endpoint → SOCNET

Approved endpoint telemetry includes:

    LAB-WIN-01 → LAB-SIEM-01 TCP/1514
    LAB-UBUNTU-01 → LAB-SIEM-01 TCP/1514
    LAB-DC-01 → LAB-SIEM-01 TCP/1514
    LAB-DMZ-01 → LAB-SIEM-01 TCP/1514

Agent enrollment may use:

`TCP/1515`

where required.

---

# Suricata → SOCNET

Suricata telemetry from OPNsense is forwarded using:

`UDP/5514`

Destination:

`LAB-SIEM-01`

The SIEM receives the events using rsyslog and writes them to:

`/var/log/suricata/eve.json`

Wazuh then processes the JSON events.

---

# ADNET Policy

## Purpose

ADNET contains:

`LAB-DC-01 — 10.10.50.10`

The Domain Controller provides:

- Active Directory Domain Services
- DNS
- Kerberos
- LDAP
- Group Policy

ADNET is treated as high-value identity infrastructure.

---

# ADNET → SOCNET

## Wazuh Telemetry

Source:

`ADNET`

Destination:

`WAZUH_SERVER`

Protocol:

`TCP`

Port:

`1514`

Action:

`ALLOW`

Validated result:

`PASS`

---

# ADNET → DMZ

## SSH

Source:

`ADNET`

Destination:

`LAB_DMZ_01`

Protocol:

`TCP`

Port:

`22`

Action:

`BLOCK`

Purpose:

Prevent the Domain Controller from being used as a general-purpose jump host.

Validated result:

`PASS`

---

# DMZ Policy

## Purpose

The DMZ contains:

`LAB-DMZ-01 — 10.10.60.10`

The server provides:

- Nginx
- Wazuh monitoring
- controlled web attack surface

The DMZ is intentionally positioned between untrusted attack traffic and protected internal infrastructure.

---

# DMZ → OPNsense DNS

Source:

`DMZ network`

Destination:

`This Firewall`

Protocol:

`TCP/UDP`

Port:

`DNS_PORT`

Action:

`ALLOW`

Purpose:

Allow DMZ name resolution through OPNsense.

---

# DMZ → OPNsense NTP

Source:

`DMZ network`

Destination:

`This Firewall`

Protocol:

`UDP`

Port:

`NTP_PORT`

Action:

`ALLOW`

Purpose:

Permit approved time synchronization.

---

# DMZ → Wazuh

Source:

`DMZ network`

Destination:

`WAZUH_SERVER`

Protocol:

`TCP`

Ports:

`WAZUH_AGENT_PORTS`

Equivalent required ports:

    1514
    1515

Action:

`ALLOW`

Purpose:

Permit endpoint telemetry and enrollment communication.

Validated result:

`PASS`

---

# DMZ → Private Networks

Source:

`DMZ network`

Destination:

`PRIVATE_NETS`

Protocol:

`Any`

Action:

`BLOCK`

Logging:

`Enabled`

Description:

`BLOCK - DMZ to Unauthorized Private Networks`

Purpose:

Prevent a compromised DMZ server from freely accessing protected enterprise networks.

---

# DMZ → Internet Web Access

Source:

`DMZ network`

Destination:

`Any`

Protocol:

`TCP`

Ports:

`HTTP_HTTPS_PORTS`

Equivalent ports:

    80
    443

Action:

`ALLOW`

Purpose:

Permit:

- package updates
- software downloads
- required HTTPS communication

Validated HTTPS result:

`PASS`

---

# DMZ → CORPNET

## SMB

Source:

`DMZ`

Destination:

`CORPNET`

Protocol:

`TCP`

Port:

`445`

Action:

`BLOCK`

Validated result:

`PASS`

---

# DMZ → ADNET

## LDAP

Source:

`DMZ`

Destination:

`ADNET`

Protocol:

`TCP`

Port:

`389`

Action:

`BLOCK`

Validated result:

`PASS`

---

# Broad Private-Network Blocks

Each user-facing or less-trusted zone uses broad private-network restrictions where appropriate.

This provides a general deny boundary while allowing specific exceptions above it.

Examples include:

`BLOCK - REDNET to Unauthorized Private Networks`

`BLOCK - CORPNET to Unauthorized Private Networks`

`BLOCK - DMZ to Unauthorized Private Networks`

These rules prevent unplanned communication paths from appearing as the environment expands.

---

# Rule Ordering

OPNsense firewall rules are evaluated in order.

Specific approved traffic must therefore appear before broad block policies.

Example:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

Another example:

    ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH
    BLOCK - CORPNET to Unauthorized Private Networks

Incorrect ordering would cause the broad block to match first.

---

# Firewall Logging Policy

Not every allowed connection is logged.

Logging is prioritized for security-relevant policy boundaries.

Examples include:

- REDNET deny activity
- DMZ private-network violations
- restricted administrative access
- important segmentation tests

This provides investigation value without producing unnecessary firewall noise.

---

# REDNET Validation Matrix

| Source | Destination | Service | Policy | Result |
|---|---|---|---|---|
| REDNET | DMZ | HTTP/80 | Allow | PASS |
| REDNET | DMZ | SSH/22 | Block | PASS |
| REDNET | CORPNET | SMB/445 | Block | PASS |
| REDNET | ADNET | LDAP/389 | Block | PASS |

---

# CORPNET Validation Matrix

| Source | Destination | Service | Policy | Result |
|---|---|---|---|---|
| CORPNET | ADNET | DNS/53 | Allow | PASS |
| CORPNET | ADNET | Kerberos/88 | Allow | PASS |
| CORPNET | ADNET | LDAP/389 | Allow | PASS |
| CORPNET | SOCNET | Wazuh/1514 | Allow | PASS |
| `LAB-WIN-01` | DMZ | SSH/22 | Allow | PASS |

---

# SERVERNET Validation Matrix

| Source | Destination | Service | Policy | Result |
|---|---|---|---|---|
| SERVERNET | SOCNET | Wazuh/1514 | Allow | PASS |
| SERVERNET | CORPNET | SMB/445 | Block | PASS |
| SERVERNET | DMZ | SSH/22 | Block | PASS |

---

# ADNET Validation Matrix

| Source | Destination | Service | Policy | Result |
|---|---|---|---|---|
| ADNET | SOCNET | Wazuh/1514 | Allow | PASS |
| ADNET | DMZ | SSH/22 | Block | PASS |

---

# DMZ Validation Matrix

| Source | Destination | Service | Policy | Result |
|---|---|---|---|---|
| DMZ | SOCNET | Wazuh/1514 | Allow | PASS |
| DMZ | CORPNET | SMB/445 | Block | PASS |
| DMZ | ADNET | LDAP/389 | Block | PASS |
| DMZ | Internet | HTTPS/443 | Allow | PASS |

---

# Consolidated Final Policy Matrix

| Source | Destination | Protocol / Port | Action | Purpose |
|---|---|---|---|---|
| REDNET | `LAB-DMZ-01` | TCP/80 | Allow | Controlled web testing |
| REDNET | `LAB-DMZ-01` | TCP/22 | Block | Prevent attacker administration |
| REDNET | CORPNET | TCP/445 | Block | Prevent SMB access |
| REDNET | ADNET | TCP/389 | Block | Protect LDAP |
| CORPNET | `LAB-DC-01` | TCP/53 | Allow | DNS |
| CORPNET | `LAB-DC-01` | TCP/88 | Allow | Kerberos |
| CORPNET | `LAB-DC-01` | TCP/389 | Allow | LDAP |
| CORPNET | `LAB-SIEM-01` | TCP/1514 | Allow | Wazuh telemetry |
| `LAB-WIN-01` | `LAB-DMZ-01` | TCP/22 | Allow | Approved administration |
| SERVERNET | `LAB-SIEM-01` | TCP/1514 | Allow | Wazuh telemetry |
| SERVERNET | CORPNET | TCP/445 | Block | Limit lateral movement |
| SERVERNET | DMZ | TCP/22 | Block | Prevent unauthorized administration |
| ADNET | `LAB-SIEM-01` | TCP/1514 | Allow | Wazuh telemetry |
| ADNET | DMZ | TCP/22 | Block | Protect Domain Controller role |
| DMZ | OPNsense | TCP/UDP 53 | Allow | DNS |
| DMZ | OPNsense | UDP/123 | Allow | NTP |
| DMZ | `LAB-SIEM-01` | TCP/1514 | Allow | Wazuh telemetry |
| DMZ | `LAB-SIEM-01` | TCP/1515 | Allow | Wazuh enrollment |
| DMZ | PRIVATE_NETS | Any | Block | Internal isolation |
| DMZ | Internet | TCP/80 | Allow | Approved web access |
| DMZ | Internet | TCP/443 | Allow | Updates and HTTPS |
| DMZ | CORPNET | TCP/445 | Block | Protect workstations |
| DMZ | ADNET | TCP/389 | Block | Protect directory services |
| OPNsense Suricata | `LAB-SIEM-01` | UDP/5514 | Allow | IDS telemetry |

---

# Detection and Firewall Correlation

The firewall is not treated purely as a traffic-control device.

Firewall events are also forwarded into the security-monitoring environment.

This enables the SOC to investigate:

- denied connections
- attempted lateral movement
- unauthorized service access
- security-zone violations

Earlier testing successfully validated OPNsense firewall correlation within Wazuh using:

`Rule 87702`

This established the firewall as both:

- an enforcement control
- a telemetry source

---

# Suricata Integration

Suricata operates on OPNsense and monitors REDNET traffic.

This provides network-level visibility in addition to normal firewall enforcement.

A controlled REDNET-to-DMZ test produced:

    Suricata SID 2100498
    GPL ATTACK_RESPONSE id check returned root

The event reached Wazuh and generated:

`Rule 86601`

This demonstrates that firewall policy and IDS monitoring operate together.

---

# Security Impact of the Final Policy

The final firewall architecture prevents several unwanted attack paths.

Examples include:

    Kali → Domain Controller LDAP
    Kali → Windows SMB
    Kali → DMZ SSH
    Ubuntu Server → DMZ SSH
    Domain Controller → DMZ SSH
    DMZ → Windows SMB
    DMZ → Active Directory LDAP

At the same time, legitimate operational requirements remain functional.

Examples include:

    Windows → Active Directory
    Windows → Wazuh
    Ubuntu → Wazuh
    Domain Controller → Wazuh
    DMZ → Wazuh
    Windows Admin → DMZ SSH
    Kali → DMZ HTTP

---

# Defense-in-Depth Relationship

Firewall policy is only one layer of the security architecture.

The environment combines:

    Firewall policy
        |
        v
    Network IDS
        |
        v
    Endpoint monitoring
        |
        v
    Application logging
        |
        v
    Centralized SIEM
        |
        v
    Analyst investigation

A permitted connection can therefore still be monitored and detected if suspicious activity occurs.

---

# Change Management

During the project, snapshots were taken before significant firewall changes.

Examples included:

- ADNET introduction
- Active Directory firewall rules
- Wazuh communication rules
- DMZ creation
- DMZ web exposure
- restricted CORPNET-to-DMZ SSH access

Changes followed the workflow:

    Snapshot
       |
       v
    Configure
       |
       v
    Apply
       |
       v
    Test
       |
       v
    Review Logs
       |
       v
    Validate
       |
       v
    Document

This reduced the risk of losing a known-good network state.

---

# Historical vs Current Policy

The project documentation records the environment from the beginning of the lab through the final architecture.

Earlier documents, screenshots, commits, or build-journal entries may therefore reference:

- `192.168.56.0/24`
- fewer security zones
- temporary firewall rules
- earlier connectivity models

Those configurations represent legitimate historical development stages.

They are not the current authoritative firewall policy.

The current authoritative model is the six-zone matrix documented here.

---

# Final Validation Status

All major security-zone communication paths were tested after the infrastructure build was completed.

Final status:

**PASS**

Validated areas include:

- offensive network isolation
- corporate Active Directory access
- endpoint-to-SIEM communication
- internal server isolation
- Domain Controller protection
- DMZ isolation
- restricted SSH administration
- Internet HTTPS access
- Suricata telemetry
- firewall telemetry
- application monitoring

The final infrastructure state is preserved using:

`Final-Six-Zone-Validated-Baseline`

on every virtual machine.

---

# Final Policy Philosophy

The final firewall architecture is based on a simple operational principle:

**Allow what the environment requires, block what it does not require, and monitor the paths that matter.**

The firewall is therefore not configured merely to make the lab functional.

It is designed to make the lab:

- segmented
- observable
- testable
- defensible
- realistic enough for SOC and Purple Team exercises
