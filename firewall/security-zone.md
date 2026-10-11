# Security Zone Architecture

## Overview

The Cybersecurity Home Lab evolved from a simple flat internal network into a segmented six-zone enterprise-style architecture.

The final design uses OPNsense as the central firewall, router, segmentation boundary, and network-security enforcement point.

Each security zone represents a different trust level, system function, or security responsibility.

The final zones are:

- REDNET
- CORPNET
- SERVERNET
- SOCNET
- ADNET
- DMZ

All inter-zone traffic must pass through `LAB-FW-01`.

---

# Evolution of the Security-Zone Design

## Initial Lab Design

The early version of the lab used a much simpler VirtualBox network model.

The initial environment included:

- Kali Linux
- Windows
- Ubuntu Server

The systems communicated over:

`192.168.56.0/24`

Example addresses included:

- Kali — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu — `192.168.56.30`

Virtual machines also used VirtualBox NAT where Internet access was required.

This design was useful for learning:

- basic Linux administration
- SSH
- fail2ban
- Nmap
- Hydra
- basic attacker/defender interaction

However, the network was essentially flat.

There was limited separation between offensive systems, endpoints, servers, monitoring infrastructure, and identity services.

---

# Why the Flat Network Was Replaced

As the project matured toward Purple Team and SOC training, the flat design became insufficient.

The major limitations were:

- insufficient trust separation
- unrealistic attacker access
- limited firewall-policy practice
- no dedicated SOC network
- no isolated Active Directory zone
- no DMZ
- limited east-west visibility
- difficult simulation of enterprise lateral movement
- less realistic incident-response scenarios

The lab was therefore rebuilt around a segmented firewall architecture.

---

# Segmented Architecture

OPNsense was introduced as the central security gateway.

The environment was divided into purpose-built networks.

The initial segmented architecture included:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

Later phases introduced:

- ADNET
- DMZ

This produced the final six-zone architecture.

---

# Final Security Zones

| Zone | Network | Gateway | Primary Function |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `10.10.10.1` | Offensive security |
| CORPNET | `10.10.20.0/24` | `10.10.20.1` | Enterprise workstation |
| SERVERNET | `10.10.30.0/24` | `10.10.30.1` | Internal servers |
| SOCNET | `10.10.40.0/24` | `10.10.40.1` | SIEM and security monitoring |
| ADNET | `10.10.50.0/24` | `10.10.50.1` | Identity infrastructure |
| DMZ | `10.10.60.0/24` | `10.10.60.1` | Externally exposed services |

---

# Trust Model

The zones are not treated as equally trusted.

A simplified trust model is:

    REDNET
      |
      | Least trusted
      v

    DMZ
      |
      v

    CORPNET / SERVERNET
      |
      v

    SOCNET / ADNET
      |
      | Highest-value infrastructure
      v

This is not intended to represent a universal enterprise trust hierarchy.

Instead, it reflects the design decisions made for this specific cybersecurity lab.

---

# REDNET

## Network

`10.10.10.0/24`

## Gateway

`10.10.10.1`

## Primary Host

`LAB-KALI-01 — 10.10.10.10`

## Purpose

REDNET is the offensive-security network.

It represents:

- attacker infrastructure
- untrusted systems
- penetration-testing activity
- controlled adversarial simulations

Kali Linux operates from this zone.

---

## Security Philosophy

REDNET should never have unrestricted access to enterprise infrastructure.

Its purpose is to generate controlled attacker activity that defensive systems can observe and investigate.

Allowed traffic is therefore intentionally narrow.

---

## Final Validated REDNET Policy

    REDNET → DMZ HTTP/80       ALLOW
    REDNET → DMZ SSH/22        BLOCK
    REDNET → CORPNET SMB/445   BLOCK
    REDNET → ADNET LDAP/389    BLOCK

This provides an intentional attack path to the DMZ without allowing unrestricted access to protected enterprise systems.

---

# CORPNET

## Network

`10.10.20.0/24`

## Gateway

`10.10.20.1`

## Primary Host

`LAB-WIN-01 — 10.10.20.20`

## Purpose

CORPNET represents the enterprise user workstation environment.

The Windows workstation is joined to:

`cyberlab.internal`

and functions as a monitored domain endpoint.

---

## Required Access

CORPNET requires access to Active Directory services.

Validated examples include:

    CORPNET → ADNET DNS/53
    CORPNET → ADNET Kerberos/88
    CORPNET → ADNET LDAP/389

These communication paths support:

- DNS resolution
- domain authentication
- Kerberos
- LDAP
- Group Policy
- domain services

---

## Monitoring Access

CORPNET is permitted to send Wazuh telemetry to SOCNET:

    CORPNET → SOCNET TCP/1514

---

## Administrative Access

`LAB-WIN-01` also became the approved administrative workstation for DMZ management.

Approved path:

    LAB-WIN-01
    10.10.20.20
         |
         | TCP/22
         v
    LAB-DMZ-01
    10.10.60.10

This rule is restricted to the approved workstation rather than all CORPNET hosts.

---

# SERVERNET

## Network

`10.10.30.0/24`

## Gateway

`10.10.30.1`

## Primary Host

`LAB-UBUNTU-01 — 10.10.30.30`

## Purpose

SERVERNET represents internal Linux server infrastructure.

This zone is treated as a protected internal network rather than a user workstation segment.

---

## Monitoring

The Ubuntu server sends security telemetry to Wazuh:

    SERVERNET → SOCNET TCP/1514

---

## Isolation

Validated blocked traffic includes:

    SERVERNET → CORPNET SMB/445
    SERVERNET → DMZ SSH/22

These controls limit unnecessary east-west communication and reduce potential lateral-movement paths.

---

# SOCNET

## Network

`10.10.40.0/24`

## Gateway

`10.10.40.1`

## Primary Host

`LAB-SIEM-01 — 10.10.40.10`

## Purpose

SOCNET contains the central monitoring infrastructure.

`LAB-SIEM-01` hosts:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat
- centralized Suricata event collection

---

## Security Role

SOCNET receives telemetry from multiple trust zones.

Primary endpoint flows include:

    CORPNET → SOCNET
    SERVERNET → SOCNET
    ADNET → SOCNET
    DMZ → SOCNET

This allows the SIEM to centralize endpoint, network, identity, and application telemetry.

---

## Final Monitored Agents

During final validation:

    LAB-WIN-01       Active
    LAB-UBUNTU-01    Active
    LAB-DC-01        Active
    LAB-DMZ-01       Active

SOCNET therefore acts as the defensive visibility hub of the environment.

---

# ADNET

## Network

`10.10.50.0/24`

## Gateway

`10.10.50.1`

## Primary Host

`LAB-DC-01 — 10.10.50.10`

## Purpose

ADNET contains identity infrastructure.

`LAB-DC-01` provides:

- Active Directory Domain Services
- DNS
- Kerberos
- LDAP
- Group Policy
- identity management

Domain:

`cyberlab.internal`

---

## Why ADNET Was Created

The original lab did not include Active Directory.

As the environment evolved toward enterprise security simulation, identity infrastructure became necessary.

Placing the Domain Controller directly inside CORPNET would have reduced separation between:

- user endpoints
- authentication infrastructure
- privileged services

ADNET was therefore introduced as its own protected zone.

---

## Final Validated ADNET Policy

Required CORPNET domain traffic is permitted.

Examples:

    CORPNET → ADNET DNS/53        ALLOW
    CORPNET → ADNET Kerberos/88   ALLOW
    CORPNET → ADNET LDAP/389      ALLOW

Monitoring:

    ADNET → SOCNET Wazuh/1514     ALLOW

Restricted traffic:

    REDNET → ADNET LDAP/389       BLOCK
    ADNET → DMZ SSH/22            BLOCK

---

# DMZ

## Network

`10.10.60.0/24`

## Gateway

`10.10.60.1`

## Primary Host

`LAB-DMZ-01 — 10.10.60.10`

## Purpose

The DMZ was the final major security zone added to the architecture.

It represents systems that may need to interact with untrusted networks without exposing protected enterprise infrastructure.

`LAB-DMZ-01` hosts:

- Ubuntu Server
- Nginx
- Wazuh Agent

---

# Why the DMZ Was Added

The previous five-zone design lacked a realistic externally facing target.

Without a DMZ:

- REDNET attack simulation would target internal systems directly
- web attack scenarios were less realistic
- Suricata validation lacked a dedicated exposed service
- application-layer monitoring was limited

The DMZ introduced a realistic web-service attack surface while preserving internal segmentation.

---

# DMZ Inbound Policy

## REDNET

Allowed:

    REDNET → DMZ HTTP/80

Blocked:

    REDNET → DMZ SSH/22

This allows offensive web testing while preventing attacker administrative access.

---

## CORPNET

Approved administrative access:

    LAB-WIN-01 → LAB-DMZ-01 SSH/22

The rule is restricted using the OPNsense host alias:

`ADMIN_WORKSTATION`

representing:

`10.10.20.20`

---

## ADNET

Blocked:

    ADNET → DMZ SSH/22

The Domain Controller is intentionally prevented from becoming a general-purpose jump host.

---

# DMZ Outbound Policy

Required monitoring:

    DMZ → SOCNET TCP/1514

Agent enrollment may use:

    DMZ → SOCNET TCP/1515

Protected internal access is blocked.

Validated examples:

    DMZ → CORPNET SMB/445   BLOCK
    DMZ → ADNET LDAP/389    BLOCK

Approved Internet access includes:

- HTTP
- HTTPS

for software updates and package management.

---

# Default-Deny Philosophy

The firewall design follows a default-deny mindset.

The goal is not to permit broad communication and then block a few dangerous ports.

Instead:

1. broad private-network access is restricted
2. required services are explicitly permitted
3. specific allow rules are placed above broader deny rules
4. significant deny boundaries are logged
5. new exceptions are justified and validated

---

# Specific Allow Before Broad Deny

An important design pattern used throughout OPNsense is:

    Specific Allow Rule
    Broad Private-Network Block

Example:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

This ordering ensures approved services work without weakening broader segmentation.

---

# Firewall Aliases

Aliases are used to improve readability and maintainability.

Examples include:

- `ADMIN_WORKSTATION`
- `LAB_DMZ_01`
- `WAZUH_SERVER`
- `WAZUH_AGENT_PORTS`
- `DNS_PORT`
- `NTP_PORT`
- `HTTP_HTTPS_PORTS`
- `PRIVATE_NETS`

Using aliases allows firewall rules to express intent clearly.

Instead of:

    10.10.20.20 → 10.10.60.10 TCP/22

the rule can be understood as:

    ADMIN_WORKSTATION → LAB_DMZ_01 SSH

This is easier to maintain and review.

---

# Monitoring Across Security Zones

Segmentation does not prevent centralized monitoring.

Each monitored host is allowed to communicate with SOCNET using only the required Wazuh services.

Examples:

    LAB-WIN-01 → LAB-SIEM-01
    LAB-UBUNTU-01 → LAB-SIEM-01
    LAB-DC-01 → LAB-SIEM-01
    LAB-DMZ-01 → LAB-SIEM-01

This creates centralized visibility without broadly opening the SOC network.

---

# Suricata and REDNET

Suricata runs on OPNsense.

The REDNET-facing interface:

`em2`

is monitored for attacker-generated traffic.

This placement allows the IDS to inspect traffic involving the Kali offensive workstation.

---

# REDNET-to-DMZ IDS Validation

A controlled test was performed between:

`LAB-KALI-01`

and:

`LAB-DMZ-01`

A temporary HTTP response contained:

    uid=0(root) gid=0(root) groups=0(root)

Suricata generated:

    SID 2100498
    GPL ATTACK_RESPONSE id check returned root

The event showed traffic from:

    10.10.60.10:80

to:

    10.10.10.10

This proved that Suricata had visibility into the REDNET ↔ DMZ communication path.

---

# Suricata to Wazuh

Suricata EVE JSON telemetry is forwarded to SOCNET.

The path is:

    OPNsense Suricata
        |
        v
    UDP/5514
        |
        v
    LAB-SIEM-01
        |
        v
    /var/log/suricata/eve.json
        |
        v
    Wazuh

The controlled IDS test generated:

    Wazuh Rule 86601

confirming the detection pipeline.

---

# DMZ Application Monitoring

Network-layer IDS visibility is complemented by host-level application logging.

Wazuh monitors:

`/var/log/nginx/access.log`

and:

`/var/log/nginx/error.log`

A controlled request from REDNET generated:

    Wazuh Rule 31101
    Web server 400 error code.

This provides defense in depth:

    Firewall
       |
       v
    Suricata
       |
       v
    Nginx
       |
       v
    Wazuh Agent
       |
       v
    SIEM

---

# Administrative Security

Network segmentation alone is not relied upon for administrative protection.

The DMZ SSH configuration was hardened with:

    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no

This creates two independent security layers:

1. OPNsense restricts who can reach TCP/22.
2. SSH restricts how authentication can occur.

---

# Final Security-Zone Validation

The final zone validation produced the following results.

| Source | Destination | Service | Expected | Result |
|---|---|---|---|---|
| REDNET | DMZ | HTTP/80 | Allow | PASS |
| REDNET | DMZ | SSH/22 | Block | PASS |
| REDNET | CORPNET | SMB/445 | Block | PASS |
| REDNET | ADNET | LDAP/389 | Block | PASS |
| CORPNET | ADNET | DNS/53 | Allow | PASS |
| CORPNET | ADNET | Kerberos/88 | Allow | PASS |
| CORPNET | ADNET | LDAP/389 | Allow | PASS |
| CORPNET | SOCNET | Wazuh/1514 | Allow | PASS |
| LAB-WIN-01 | DMZ | SSH/22 | Allow | PASS |
| SERVERNET | SOCNET | Wazuh/1514 | Allow | PASS |
| SERVERNET | CORPNET | SMB/445 | Block | PASS |
| SERVERNET | DMZ | SSH/22 | Block | PASS |
| ADNET | SOCNET | Wazuh/1514 | Allow | PASS |
| ADNET | DMZ | SSH/22 | Block | PASS |
| DMZ | SOCNET | Wazuh/1514 | Allow | PASS |
| DMZ | CORPNET | SMB/445 | Block | PASS |
| DMZ | ADNET | LDAP/389 | Block | PASS |
| DMZ | Internet | HTTPS/443 | Allow | PASS |

---

# Final Security Model

The completed environment can be summarized as:

    REDNET
    Offensive / Untrusted
        |
        | Controlled attack traffic
        v

    LAB-FW-01
    OPNsense
    Firewall + Routing + IDS
        |
        +------------------------------+
        |          |         |         |
        v          v         v         v
    CORPNET    SERVERNET   ADNET      DMZ
        \          |         |         /
         \         |         |        /
          \        |         |       /
           +------ SOCNET -----------+
                   |
                   v
                Wazuh SIEM

---

# Historical vs Current Architecture

The repository contains documentation covering the entire lifecycle of the project.

Historical configurations may include the original:

`192.168.56.0/24`

lab design.

These should be interpreted as earlier project stages rather than current configuration.

The authoritative final architecture is the six-zone model documented here.

---

# Snapshot Baseline

Snapshots were used before major changes to firewall policy, network segmentation, Active Directory, DMZ configuration, monitoring, and hardening.

After final validation, temporary build snapshots were removed.

Every VM now retains:

`Final-Six-Zone-Validated-Baseline`

This represents the stable infrastructure state before realistic SOC and Purple Team scenarios begin.

---

# Final Status

The security-zone architecture is complete and validated.

The lab has evolved from a simple flat training network into a segmented environment with:

- dedicated offensive infrastructure
- enterprise workstation segmentation
- protected internal servers
- centralized SOC monitoring
- isolated identity infrastructure
- monitored DMZ services
- default-deny firewall policy
- explicit least-privilege exceptions
- IDS visibility
- endpoint telemetry
- application telemetry
- restricted administrative access

Future exercises will operate within these established trust boundaries rather than modifying the underlying architecture unnecessarily.
