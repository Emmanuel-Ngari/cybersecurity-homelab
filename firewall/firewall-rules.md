# Firewall Rules

## Overview

This document records the evolution, design, implementation, and final validated state of the firewall rules used in the Cybersecurity Home Lab.

The lab began as a simple flat VirtualBox network and later evolved into a segmented six-zone architecture protected by OPNsense.

The final firewall design supports:

- network segmentation
- least-privilege communication
- Active Directory access
- SIEM telemetry
- Suricata monitoring
- controlled REDNET attack traffic
- DMZ exposure
- restricted administrative access
- logged deny boundaries
- controlled Internet access

The authoritative final firewall is implemented on:

`LAB-FW-01`

using OPNsense.

---

# Historical Evolution

## Early Lab

The original lab used a simple network:

`192.168.56.0/24`

with:

- Kali Linux
- Windows
- Ubuntu Server

This environment supported basic attacker/defender exercises but provided limited segmentation.

There was no dedicated:

- firewall zone architecture
- SOC network
- Active Directory network
- DMZ
- structured east-west policy

As the project evolved toward Purple Team and SOC training, this model was replaced.

---

# Introduction of OPNsense

OPNsense was introduced as the central routing and security platform.

The first segmented design created:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

The architecture was later expanded with:

- ADNET
- DMZ

This produced the final six-zone environment.

---

# Final Security Zones

| Zone | Network | Gateway |
|---|---|---|
| REDNET | `10.10.10.0/24` | `10.10.10.1` |
| CORPNET | `10.10.20.0/24` | `10.10.20.1` |
| SERVERNET | `10.10.30.0/24` | `10.10.30.1` |
| SOCNET | `10.10.40.0/24` | `10.10.40.1` |
| ADNET | `10.10.50.0/24` | `10.10.50.1` |
| DMZ | `10.10.60.0/24` | `10.10.60.1` |

---

# Firewall Design Strategy

The rule base follows several principles.

## Least Privilege

Only required communication paths are allowed.

## Specific Allow Before Broad Deny

Narrow rules are placed above broader private-network blocks.

## Zone Isolation

Each network is treated according to its trust level and role.

## Explicit Administrative Access

Management access is restricted to known source hosts.

## Monitoring First

Required Wazuh and logging flows are explicitly permitted.

## Logged Security Boundaries

Important deny rules are logged for investigation and validation.

---

# Firewall Aliases

Aliases are used to make the policy easier to understand and maintain.

## `ADMIN_WORKSTATION`

Type:

`Host(s)`

Content:

`10.10.20.20`

Represents:

`LAB-WIN-01`

Purpose:

Approved administrative workstation for DMZ SSH management.

---

## `LAB_DMZ_01`

Type:

`Host(s)`

Content:

`10.10.60.10`

Represents:

`LAB-DMZ-01`

Purpose:

Dedicated reference for the DMZ web server.

---

## `WAZUH_SERVER`

Type:

`Host(s)`

Content:

`10.10.40.10`

Represents:

`LAB-SIEM-01`

Purpose:

Central Wazuh destination.

---

## `WAZUH_AGENT_PORTS`

Type:

`Port(s)`

Content:

- `1514`
- `1515`

Purpose:

Wazuh agent communication and enrollment.

---

## `HTTP_HTTPS_PORTS`

Type:

`Port(s)`

Content:

- `80`
- `443`

Purpose:

Approved outbound web traffic.

---

## `DNS_PORT`

Represents:

DNS service traffic.

---

## `NTP_PORT`

Represents:

NTP service traffic.

---

## `PRIVATE_NETS`

Represents protected private/internal networks.

Purpose:

Used as the destination for broad inter-zone deny rules.

---

# REDNET Rules

## Purpose

REDNET contains:

`LAB-KALI-01 — 10.10.10.10`

The zone is intentionally treated as untrusted.

---

## Allow REDNET to DMZ HTTP

Rule description:

`ALLOW - REDNET to LAB-DMZ-01 HTTP`

Configuration:

    Action: Pass
    Interface: REDNET
    Direction: In
    TCP/IP Version: IPv4
    Protocol: TCP
    Source: REDNET network
    Source Port: any
    Destination: LAB_DMZ_01
    Destination Port: HTTP (80)

Purpose:

Permit controlled web attack simulation from Kali to the DMZ.

Validation:

    curl http://10.10.60.10/

Result:

`HTTP 200`

Status:

**PASS**

---

## Block REDNET to Unauthorized Private Networks

Rule description:

`BLOCK - REDNET to Unauthorized Private Networks`

Configuration:

    Action: Block
    Interface: REDNET
    Direction: In
    TCP/IP Version: IPv4
    Protocol: Any
    Source: REDNET network
    Destination: PRIVATE_NETS
    Logging: Enabled

Purpose:

Prevent REDNET from freely accessing protected internal systems.

Validated examples:

    REDNET → DMZ SSH/22        BLOCK
    REDNET → CORPNET SMB/445   BLOCK
    REDNET → ADNET LDAP/389    BLOCK

Status:

**PASS**

---

# REDNET Rule Ordering

The approved HTTP exception must be above the broader private-network block.

Correct order:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

During implementation, traffic continued to be blocked until the allow rule was both correctly positioned and actually applied in OPNsense.

This reinforced the importance of:

- rule order
- Save
- Apply Changes
- retesting with new traffic

---

# CORPNET Rules

## Purpose

CORPNET contains:

`LAB-WIN-01 — 10.10.20.20`

The workstation requires access to:

- Active Directory
- Wazuh
- restricted DMZ administration

---

# CORPNET to ADNET

The workstation requires multiple domain services.

The final environment includes explicit firewall allowances for required Active Directory communication.

Directly validated services include:

    DNS TCP/53
    Kerberos TCP/88
    LDAP TCP/389

These support:

- domain authentication
- DNS
- Kerberos
- LDAP
- Group Policy
- domain membership

---

## CORPNET → AD DNS

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

Validated:

`PASS`

---

## CORPNET → AD Kerberos

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

Validated:

`PASS`

---

## CORPNET → AD LDAP

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

Validated:

`PASS`

---

# CORPNET to SOCNET

## Wazuh Agent Communication

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

Permit Windows endpoint telemetry to reach Wazuh.

Validated:

`PASS`

---

# CORPNET to DMZ

## Restricted Administrative SSH

Rule description:

`ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH`

Configuration:

    Action: Pass
    Interface: CORPNET
    Direction: In
    TCP/IP Version: IPv4
    Protocol: TCP
    Source: ADMIN_WORKSTATION
    Source Port: any
    Destination: LAB_DMZ_01
    Destination Port: SSH (22)
    Logging: Enabled
    Gateway: None

Purpose:

Permit only the approved administrative workstation to manage the DMZ server.

Validated:

`PASS`

---

## Broad CORPNET Block

The SSH exception is positioned above the broad CORPNET private-network block.

Correct order:

    ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH
    BLOCK - CORPNET to Unauthorized Private Networks

This ensures the narrow exception works without weakening general segmentation.

---

# SERVERNET Rules

## Purpose

SERVERNET contains:

`LAB-UBUNTU-01 — 10.10.30.30`

The server requires SIEM connectivity but should not have unrestricted access to other zones.

---

## SERVERNET → SOCNET Wazuh

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

Validated:

`PASS`

---

## SERVERNET → CORPNET SMB

Source:

`SERVERNET`

Destination:

`CORPNET`

Protocol:

`TCP`

Port:

`445`

Expected:

`BLOCK`

Validated:

`PASS`

---

## SERVERNET → DMZ SSH

Source:

`SERVERNET`

Destination:

`LAB_DMZ_01`

Protocol:

`TCP`

Port:

`22`

Expected:

`BLOCK`

Validated:

`PASS`

---

# SOCNET Rules

## Purpose

SOCNET contains:

`LAB-SIEM-01 — 10.10.40.10`

SOCNET is the monitoring and security-analysis zone.

It receives:

- Wazuh endpoint telemetry
- OPNsense firewall logs
- Suricata events

---

# Wazuh Communication

Monitored systems connect to:

`LAB-SIEM-01`

using:

`TCP/1514`

Agent enrollment may use:

`TCP/1515`

Approved monitored systems include:

- `LAB-WIN-01`
- `LAB-UBUNTU-01`
- `LAB-DC-01`
- `LAB-DMZ-01`

During final validation all four agents were Active.

---

# Suricata Remote Logging

Suricata telemetry from OPNsense is forwarded to:

`LAB-SIEM-01`

using:

`UDP/5514`

The SIEM writes the events to:

`/var/log/suricata/eve.json`

Wazuh then ingests the file as JSON.

---

# ADNET Rules

## Purpose

ADNET contains:

`LAB-DC-01 — 10.10.50.10`

This is critical identity infrastructure.

---

## ADNET → SOCNET Wazuh

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

Validated:

`PASS`

---

## ADNET → DMZ SSH

Source:

`ADNET`

Destination:

`LAB_DMZ_01`

Protocol:

`TCP`

Port:

`22`

Expected:

`BLOCK`

Validated:

`PASS`

Purpose:

Prevent the Domain Controller from being used as a general-purpose DMZ management host.

---

# DMZ Rules

## Purpose

The DMZ contains:

`LAB-DMZ-01 — 10.10.60.10`

The server runs:

- Ubuntu Server
- Nginx
- Wazuh Agent

The DMZ is designed to expose selected services while remaining isolated from protected enterprise networks.

---

# DMZ to OPNsense DNS

Rule description:

`ALLOW - DMZ to OPNsense DNS`

Configuration:

    Action: Pass
    Interface: DMZ
    Direction: In
    TCP/IP Version: IPv4
    Protocol: TCP/UDP
    Source: DMZ network
    Destination: This Firewall
    Destination Port: DNS_PORT
    Logging: Disabled

Purpose:

Allow DNS resolution through OPNsense.

---

# DMZ to OPNsense NTP

Rule description:

`ALLOW - DMZ to OPNsense NTP`

Configuration:

    Action: Pass
    Interface: DMZ
    Direction: In
    TCP/IP Version: IPv4
    Protocol: UDP
    Source: DMZ network
    Destination: This Firewall
    Destination Port: NTP_PORT
    Logging: Disabled

Purpose:

Allow approved time synchronization.

---

# DMZ to Wazuh

Rule description:

`ALLOW - DMZ to Wazuh Agent Services`

Configuration:

    Action: Pass
    Interface: DMZ
    Direction: In
    TCP/IP Version: IPv4
    Protocol: TCP
    Source: DMZ network
    Destination: WAZUH_SERVER
    Destination Port: WAZUH_AGENT_PORTS
    Logging: Disabled

Ports:

    1514
    1515

Purpose:

Permit Wazuh telemetry and enrollment.

Validated:

`PASS`

---

# DMZ Private-Network Block

Rule description:

`BLOCK - DMZ to Unauthorized Private Networks`

Configuration:

    Action: Block
    Interface: DMZ
    Direction: In
    TCP/IP Version: IPv4
    Protocol: Any
    Source: DMZ network
    Destination: PRIVATE_NETS
    Logging: Enabled

Purpose:

Prevent a compromised DMZ server from freely reaching protected networks.

Validated examples:

    DMZ → CORPNET SMB/445   BLOCK
    DMZ → ADNET LDAP/389    BLOCK

Status:

**PASS**

---

# DMZ HTTP and HTTPS Outbound

Rule description:

`ALLOW - DMZ HTTP HTTPS Outbound`

Configuration:

    Action: Pass
    Interface: DMZ
    Direction: In
    TCP/IP Version: IPv4
    Protocol: TCP
    Source: DMZ network
    Destination: Any
    Destination Port: HTTP_HTTPS_PORTS
    Logging: Disabled

Ports:

    80
    443

Purpose:

Permit:

- package management
- security updates
- approved HTTPS communication

Validated:

`PASS`

A test to:

`https://packages.wazuh.com`

returned:

`HTTP/2 403`

The 403 response still confirmed successful DNS, routing, TCP/443, TLS, and remote-server reachability.

---

# DMZ Rule Order

The final DMZ policy follows this general order:

    ALLOW - DMZ to OPNsense DNS
    ALLOW - DMZ to OPNsense NTP
    ALLOW - DMZ to Wazuh Agent Services
    BLOCK - DMZ to Unauthorized Private Networks
    ALLOW - DMZ HTTP HTTPS Outbound

This ordering is intentional.

The Wazuh exception must occur before the broad private-network block because SOCNET is itself a private network.

---

# Firewall Logging

Logging is enabled primarily where it provides security value.

Important logged rules include:

- REDNET private-network blocks
- DMZ private-network blocks
- restricted SSH administration
- significant security-zone boundary violations

This reduces unnecessary log volume while preserving useful investigation data.

---

# OPNsense Live View Validation

OPNsense Live View was used extensively during troubleshooting and validation.

Examples included confirming:

    10.10.10.10 → 10.10.60.10:22

was blocked by:

`BLOCK - REDNET to Unauthorized Private`

and confirming:

    10.10.60.10 → 10.10.20.20

was blocked by:

`BLOCK - DMZ to Unauthorized Private Networks`

Live View proved particularly useful when:

- service state looked healthy
- routing appeared correct
- connectivity still failed

The firewall logs identified which rule actually processed the packet.

---

# Default Deny Behaviour

Traffic not matching an approved allow rule is denied.

An example was a DMZ outbound TCP connection using an unapproved port.

The traffic was blocked by:

`Default deny / state violation rule`

This demonstrated that the DMZ did not receive unrestricted outbound access.

---

# Firewall Telemetry to Wazuh

OPNsense firewall events were integrated into Wazuh during an earlier project phase.

A custom firewall-correlation detection was validated using:

`Wazuh Rule 87702`

This established OPNsense as both:

- an enforcement system
- a telemetry source

---

# Suricata on OPNsense

Suricata was later introduced as an additional network-security layer.

Primary monitored interface:

`em2`

This interface corresponds to REDNET traffic.

Suricata provides:

- IDS signatures
- EVE JSON
- network detection telemetry
- attack visibility

---

# Suricata Validation

A controlled REDNET-to-DMZ HTTP test produced the response:

    uid=0(root) gid=0(root) groups=0(root)

Suricata detected:

    Signature ID: 2100498
    Signature: GPL ATTACK_RESPONSE id check returned root
    Category: Potentially Bad Traffic
    Severity: 2

The event showed:

    Source: 10.10.60.10:80
    Destination: 10.10.10.10
    Direction: to_client

This proved that Suricata could inspect the REDNET ↔ DMZ traffic path.

---

# Suricata to Wazuh

The IDS event was forwarded to the SIEM.

The telemetry path is:

    LAB-FW-01
    Suricata
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

The controlled event generated:

`Wazuh Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

---

# Active Directory Firewall Development

The Active Directory environment was introduced after the initial segmented network had already been established.

Adding ADNET required new firewall decisions.

CORPNET needed access to required domain services while REDNET and unrelated zones remained restricted.

This phase introduced and validated:

- DNS
- Kerberos
- LDAP
- Wazuh monitoring
- controlled AD isolation

The final design keeps the Domain Controller in its own security zone rather than sharing CORPNET.

---

# DMZ Firewall Development

The DMZ was introduced after the Active Directory and core monitoring architecture had already been completed.

The firewall was extended with:

- a new DMZ interface
- DMZ-specific aliases
- DNS and NTP access
- Wazuh access
- broad internal deny policy
- approved outbound web access
- REDNET HTTP exposure
- restricted CORPNET SSH administration

This represented the final major network expansion.

---

# Firewall Rule Change Workflow

Throughout the project, significant firewall changes followed a controlled workflow.

    1. Take VirtualBox snapshot
    2. Make one planned firewall change
    3. Save rule
    4. Confirm rule order
    5. Apply Changes
    6. Generate controlled test traffic
    7. Review OPNsense logs
    8. Verify endpoint/service behaviour
    9. Confirm Wazuh visibility where applicable
    10. Document result

This approach reduced troubleshooting complexity and protected the working environment.

---

# Important Troubleshooting Lesson: Apply Changes

One of the most important OPNsense lessons occurred during REDNET-to-DMZ HTTP configuration.

The HTTP rule had:

- correct source
- correct destination
- correct port
- correct rule order

but traffic continued to be blocked.

The cause was simple:

the OPNsense Apply button had not been clicked.

After applying the configuration, traffic succeeded immediately.

The resulting operational rule became:

**Always verify that pending firewall changes have actually been applied.**

---

# Final Validation Matrix

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

# Final Firewall Architecture

The completed rule model can be summarized as:

    REDNET
      |
      | Controlled attack paths
      v

    +----------------------+
    |      LAB-FW-01       |
    |       OPNsense       |
    | Firewall / Routing   |
    |    Suricata IDS      |
    +----------------------+

      |      |      |      |      |
      v      v      v      v      v

    CORP   SERVER   SOC    AD     DMZ

Firewall policy controls every inter-zone path.

---

# Historical and Final-State Documentation

This project repository intentionally preserves the full evolution of the lab.

Earlier configurations may reference:

- `192.168.56.0/24`
- simple NAT networking
- fewer security zones
- temporary allow rules
- earlier endpoint-only scenarios

These represent legitimate historical stages.

They should not be interpreted as the current firewall state.

The final authoritative policy is the six-zone architecture documented in this file and the accompanying policy matrix.

---

# Snapshot Baseline

Throughout the build, snapshots were used before major firewall changes.

Examples included:

- Suricata integration
- ADNET deployment
- CORPNET-to-AD firewall rules
- DMZ creation
- web exposure
- DMZ monitoring
- restricted SSH administration

Once the environment passed final six-zone validation, obsolete snapshots were removed.

Every VM now retains only:

`Final-Six-Zone-Validated-Baseline`

---

# Final Status

The firewall phase is complete.

The final OPNsense configuration provides:

- six-zone segmentation
- explicit least-privilege rules
- broad private-network protection
- controlled Active Directory access
- endpoint-to-SIEM communication
- restricted DMZ administration
- controlled REDNET attack paths
- DMZ Internet access
- firewall logging
- Suricata IDS
- centralized security telemetry

The firewall is now treated as stable infrastructure for realistic SOC and Purple Team operations rather than an unfinished build component.
