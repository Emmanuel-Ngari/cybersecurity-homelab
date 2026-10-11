# Firewall Rules

## Overview

This document records the complete evolution of the firewall rules used throughout the Cybersecurity Home Lab, from the earliest flat-network environment through the final validated six-zone OPNsense architecture.

The firewall configuration developed in stages as the lab expanded.

The major phases were:

1. Original flat VirtualBox lab
2. Introduction of OPNsense
3. Initial four-zone segmentation
4. CORPNET, REDNET and SERVERNET rule development
5. SOCNET and Wazuh integration
6. Suricata and firewall telemetry integration
7. ADNET and Active Directory firewall policy
8. DMZ deployment
9. Restricted DMZ administration
10. Final six-zone validation

The current authoritative firewall is:

`LAB-FW-01`

running OPNsense.

The final design follows:

- least privilege
- explicit allow rules
- broad private-network deny rules
- default-deny behaviour
- centralized monitoring
- logged security boundaries
- restricted administration
- controlled offensive traffic

---

# Phase 1 — Original Flat Lab

## Original Network

The earliest version of the cybersecurity lab used:

`192.168.56.0/24`

Primary systems included:

- Kali Linux — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu Server — `192.168.56.30`

VirtualBox NAT was also used where Internet access was required.

At this stage there was no dedicated OPNsense firewall separating the systems into trust zones.

The lab supported early security exercises such as:

- Nmap scanning
- SSH administration
- fail2ban
- Hydra testing
- Windows/Linux connectivity
- simple attacker-versus-defender exercises

The limitation was that all primary systems effectively operated within the same internal trust environment.

There was no realistic mechanism for enforcing enterprise-style east-west segmentation.

---

# Phase 2 — Introduction of OPNsense

As the lab evolved toward SOC and Purple Team training, the flat architecture was replaced with OPNsense-controlled segmentation.

`LAB-FW-01` became the central:

- firewall
- router
- network-policy enforcement point
- logging source
- later IDS platform

The first segmented design introduced four primary internal security zones:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

ADNET and DMZ were introduced later.

---

# Initial Four-Zone Addressing

## REDNET

Network:

`10.10.10.0/24`

Gateway:

`10.10.10.1`

Primary host:

`LAB-KALI-01 — 10.10.10.10`

Purpose:

Offensive security and adversarial simulation.

---

## CORPNET

Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

Primary host:

`LAB-WIN-01 — 10.10.20.20`

Purpose:

Corporate Windows workstation environment.

---

## SERVERNET

Network:

`10.10.30.0/24`

Gateway:

`10.10.30.1`

Primary host:

`LAB-UBUNTU-01 — 10.10.30.30`

Purpose:

Protected internal Linux server environment.

---

## SOCNET

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

Primary host:

`LAB-SIEM-01 — 10.10.40.10`

Purpose:

Centralized Wazuh security monitoring.

---

# Initial Firewall Aliases

A structured alias system was introduced early in the OPNsense build.

Important aliases included:

## `ADMIN_WORKSTATION`

Represents the approved Windows administration host.

Final address:

`10.10.20.20`

---

## `KALI_ATTACKER`

Represents the offensive workstation.

Address:

`10.10.10.10`

---

## `UBUNTU_SERVER`

Represents the protected internal Linux server.

Address:

`10.10.30.30`

---

## `PRIVATE_NETS`

Represents protected private/internal networks.

This alias was used as the destination for broad segmentation blocks.

---

## `SERVER_ADMIN_PORTS`

Ports:

- `22`
- `443`

Purpose:

Approved administrative services for the internal Linux server during the early segmented phase.

---

## `DNS_PORT`

Port:

`53`

---

## `NTP_PORT`

Port:

`123`

---

# Initial CORPNET Rules

CORPNET was treated as the trusted corporate workstation zone but was not given unrestricted access to every internal network.

The Windows workstation required:

- firewall administration
- DNS
- Internet connectivity
- access to approved internal server services

---

## CORPNET → OPNsense HTTPS

Purpose:

Allow the administrative workstation to access the OPNsense web interface.

Source:

`ADMIN_WORKSTATION`

Destination:

`This Firewall`

Protocol:

`TCP`

Destination port:

`443`

Action:

`ALLOW`

This provided controlled firewall administration without exposing the management interface to REDNET.

---

## CORPNET → OPNsense DNS

Source:

`CORPNET`

Destination:

`This Firewall`

Protocol:

`TCP/UDP`

Destination port:

`DNS_PORT`

Action:

`ALLOW`

Purpose:

Provide DNS resolution for the corporate workstation before Active Directory DNS was introduced.

---

## CORPNET → SERVERNET Administration

Source:

`CORPNET`

Destination:

`UBUNTU_SERVER`

Protocol:

`TCP`

Destination ports:

`SERVER_ADMIN_PORTS`

Equivalent ports:

- SSH `22`
- HTTPS `443`

Action:

`ALLOW`

Purpose:

Permit the Windows administration workstation to manage or access approved services on the internal Ubuntu server.

---

## CORPNET → REDNET

Traffic from the corporate network to the offensive network was restricted.

A broad private-network policy prevented unnecessary access to Kali.

Final validation during the early segmented phase confirmed:

`LAB-WIN-01 → LAB-KALI-01`

was blocked where no explicit exception existed.

---

## CORPNET Internet Access

CORPNET retained approved outbound Internet access through OPNsense.

This was required for:

- updates
- software downloads
- general connectivity
- security tools

The endpoint did not require a direct VirtualBox NAT adapter once OPNsense routing became authoritative.

---

# Initial REDNET Rules

REDNET was deliberately treated as the least-trusted zone.

The purpose was to allow controlled attacker activity without providing unrestricted access to enterprise-style assets.

---

## REDNET → OPNsense DNS

Source:

`REDNET`

Destination:

`This Firewall`

Protocol:

`TCP/UDP`

Destination port:

`DNS_PORT`

Action:

`ALLOW`

Purpose:

Permit name resolution for Kali.

---

## REDNET → SERVERNET Controlled Testing

Source:

`REDNET`

Destination:

`UBUNTU_SERVER`

Action:

`ALLOW` only for approved testing paths.

This early rule allowed controlled offensive exercises against the Linux server.

Examples included:

- SSH testing
- Nmap enumeration
- fail2ban validation
- Hydra testing

This represented the first deliberate attacker-versus-server path in the segmented design.

---

## REDNET → CORPNET

Traffic from Kali to the corporate Windows workstation was blocked unless specifically required for a controlled test.

This created a meaningful separation between:

- attacker infrastructure
- corporate endpoint infrastructure

---

## REDNET → OPNsense Management

Firewall-management access from REDNET was explicitly blocked.

Purpose:

Prevent the offensive workstation from reaching the OPNsense administrative interface.

This was an important early least-privilege control.

---

## REDNET Internet Access

Kali retained approved outbound Internet access through OPNsense for:

- package updates
- security tools
- research
- controlled external connectivity

---

# Initial SERVERNET Rules

SERVERNET contained the internal Ubuntu server.

The zone was protected from unnecessary access while retaining operational connectivity.

---

## SERVERNET → OPNsense DNS

Source:

`SERVERNET`

Destination:

`This Firewall`

Protocol:

`TCP/UDP`

Destination port:

`DNS_PORT`

Action:

`ALLOW`

---

## SERVERNET → OPNsense NTP

Source:

`SERVERNET`

Destination:

`This Firewall`

Protocol:

`UDP`

Destination port:

`NTP_PORT`

Action:

`ALLOW`

---

## SERVERNET → Internet

Approved outbound Internet access was permitted where required for:

- package management
- security updates
- software installation

---

## SERVERNET → CORPNET

Unsolicited server-to-workstation communication was blocked unless specifically required.

This reduced lateral-movement opportunities.

---

## SERVERNET → REDNET

Unsolicited server-to-attacker-network communication was blocked.

This prevented the protected server from freely initiating sessions into REDNET.

---

# Initial Segmentation Validation

The early OPNsense segmentation phase validated several important paths.

Examples included:

- Windows → OPNsense HTTPS allowed
- Windows → OPNsense DNS allowed
- Windows → Internet allowed
- Windows → Ubuntu SSH allowed
- Windows → Kali blocked
- Kali → OPNsense DNS allowed
- Kali → Internet allowed
- Kali → approved Ubuntu testing services allowed
- Kali → Windows blocked
- REDNET → OPNsense management blocked
- SERVERNET → CORPNET restricted
- SERVERNET → REDNET restricted

This represented the first major transition from a flat home lab into an enterprise-style security environment.

---

# Legacy Default Rules

During the initial OPNsense configuration, default LAN-style rules existed from the original installation.

As the zone-specific policy matured, these broad legacy rules were disabled or superseded.

The objective was to ensure that traffic was controlled by explicit zone policy rather than permissive default LAN rules.

---

# Phase 3 — SOCNET and Wazuh Integration

The next major architecture phase introduced centralized monitoring.

`LAB-SIEM-01`

was deployed in:

`SOCNET — 10.10.40.0/24`

with address:

`10.10.40.10`

---

# Wazuh Firewall Requirements

Wazuh agent communication required:

`TCP/1514`

Agent enrollment required:

`TCP/1515`

Endpoint zones were therefore granted narrow access to the Wazuh server.

---

# Wazuh Server Alias

Alias:

`WAZUH_SERVER`

Address:

`10.10.40.10`

Purpose:

Provide a single readable destination for endpoint telemetry rules.

---

# Wazuh Agent Ports Alias

Alias:

`WAZUH_AGENT_PORTS`

Ports:

- `1514`
- `1515`

Purpose:

Represent Wazuh communication and enrollment services.

---

# CORPNET → SOCNET Wazuh

Source:

`CORPNET`

Destination:

`WAZUH_SERVER`

Protocol:

`TCP`

Destination:

`1514/1515` where required

Action:

`ALLOW`

Purpose:

Permit Windows endpoint telemetry and enrollment.

---

# SERVERNET → SOCNET Wazuh

Source:

`SERVERNET`

Destination:

`WAZUH_SERVER`

Protocol:

`TCP`

Destination:

`1514/1515` where required

Action:

`ALLOW`

Purpose:

Permit Linux endpoint monitoring.

---

# Endpoint Monitoring Principle

SOCNET was not made broadly reachable.

Only the required security-monitoring services were exposed to endpoint zones.

This preserved separation while allowing centralized telemetry.

---

# Phase 4 — OPNsense Firewall Logging to Wazuh

OPNsense itself was later integrated into the monitoring architecture.

Firewall events were forwarded to:

`LAB-SIEM-01`

This allowed denied and suspicious inter-zone traffic to be investigated from Wazuh.

A firewall-correlation rule was successfully validated:

`Wazuh Rule 87702`

This represented an important transition:

the firewall was no longer only an enforcement device.

It also became a telemetry source.

---

# Phase 5 — Suricata IDS Integration

Suricata was installed on OPNsense after the firewall and Wazuh environment were already operational.

Primary monitored interface:

`em2`

This corresponds to REDNET traffic.

---

# Suricata Event Forwarding

Suricata events were forwarded toward SOCNET.

The final dedicated path used:

`UDP/5514`

Destination:

`LAB-SIEM-01 — 10.10.40.10`

The SIEM receives the traffic using rsyslog.

Events are written to:

`/var/log/suricata/eve.json`

Wazuh monitors this file using JSON log collection.

---

# Suricata Validation

A controlled detection produced:

`Signature ID 2100498`

Signature:

`GPL ATTACK_RESPONSE id check returned root`

The event was subsequently processed by Wazuh:

`Rule 86601`

This proved the network detection path from:

`OPNsense → SOCNET → Wazuh`

---

# Phase 6 — ADNET Introduction

Active Directory was introduced after the initial four-zone architecture was already functional.

A fifth protected network was created:

`ADNET — 10.10.50.0/24`

Gateway:

`10.10.50.1`

Domain Controller:

`LAB-DC-01 — 10.10.50.10`

Domain:

`cyberlab.internal`

---

# Active Directory Host Alias

Alias:

`LAB_DC_01`

Address:

`10.10.50.10`

Purpose:

Represent the Domain Controller in firewall policy.

---

# Why Active Directory Required More Than Three Ports

Although final connectivity sampling directly validated:

- DNS `53`
- Kerberos `88`
- LDAP `389`

full Active Directory operation requires additional services.

The firewall rules therefore supported the broader Active Directory service set rather than limiting domain connectivity only to the three sampled validation ports.

---

# CORPNET → LAB-DC-01 TCP Services

The Active Directory TCP rule set included:

- TCP `53` — DNS
- TCP `88` — Kerberos
- TCP `135` — RPC Endpoint Mapper
- TCP `389` — LDAP
- TCP `445` — SMB
- TCP `464` — Kerberos password operations
- TCP `3268` — Global Catalog
- TCP `49152:65535` — Dynamic RPC range

These ports supported:

- domain join
- authentication
- Group Policy
- SYSVOL
- directory queries
- RPC-based domain operations
- Kerberos
- Global Catalog access

---

# CORPNET → LAB-DC-01 UDP Services

The Active Directory UDP rule set included:

- UDP `53` — DNS
- UDP `88` — Kerberos
- UDP `123` — NTP
- UDP `389` — LDAP
- UDP `464` — Kerberos password operations

These rules provided the additional datagram services required by Windows domain operation.

---

# Active Directory Firewall Objective

The design intentionally avoided simply allowing:

`CORPNET → ADNET Any`

Instead, required domain services were explicitly permitted to:

`LAB-DC-01`

This preserved the principle of least privilege while allowing normal domain functionality.

---

# ADNET → SOCNET Wazuh

The Domain Controller required security telemetry access.

Source:

`ADNET`

Destination:

`WAZUH_SERVER`

Protocol:

`TCP`

Ports:

- `1514`
- `1515`

Action:

`ALLOW`

Purpose:

Permit Wazuh monitoring and enrollment.

---

# REDNET → ADNET

REDNET was not granted normal access to Active Directory.

A later final validation explicitly confirmed:

`REDNET → LAB-DC-01 LDAP/389`

was blocked.

This protected the identity infrastructure from direct attacker-network access.

---

# Active Directory Validation

Firewall policy successfully supported:

- domain join
- DNS
- Kerberos
- LDAP
- Group Policy
- SYSVOL
- domain authentication
- Wazuh telemetry

Final targeted connectivity tests confirmed:

- DNS `53` — allowed
- Kerberos `88` — allowed
- LDAP `389` — allowed

Domain Controller health checks also passed.

---

# Phase 7 — DMZ Introduction

The DMZ was the final major network zone introduced.

Network:

`10.10.60.0/24`

Gateway:

`10.10.60.1`

Primary server:

`LAB-DMZ-01 — 10.10.60.10`

The server runs:

- Ubuntu Server
- Nginx
- Wazuh Agent

---

# DMZ Aliases

## `LAB_DMZ_01`

Address:

`10.10.60.10`

Purpose:

Represent the DMZ web server.

---

## `HTTP_HTTPS_PORTS`

Ports:

- `80`
- `443`

Purpose:

Approved outbound web traffic.

---

# Final DMZ Rule Order

The final DMZ interface contained the following important rules in order:

1. `ALLOW - DMZ to OPNsense DNS`
2. `ALLOW - DMZ to OPNsense NTP`
3. `ALLOW - DMZ to Wazuh Agent Services`
4. `BLOCK - DMZ to Unauthorized Private Networks`
5. `ALLOW - DMZ HTTP HTTPS Outbound`

The order is important because Wazuh resides inside a private network and therefore requires a specific exception before the broad private-network block.

---

# DMZ → OPNsense DNS

Rule:

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

Permit DNS resolution through OPNsense.

---

# DMZ → OPNsense NTP

Rule:

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

Permit approved time synchronization.

---

# DMZ → Wazuh Agent Services

Rule:

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

Equivalent ports:

- TCP `1514`
- TCP `1515`

Purpose:

Permit Wazuh monitoring and enrollment.

Connectivity was tested before installing the agent.

Both ports were reachable.

---

# DMZ → Unauthorized Private Networks

Rule:

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

Prevent a compromised DMZ server from freely accessing protected enterprise systems.

Validated examples:

- DMZ → CORPNET SMB/445 blocked
- DMZ → ADNET LDAP/389 blocked

---

# DMZ HTTP/HTTPS Outbound

Rule:

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

- TCP `80`
- TCP `443`

Purpose:

Permit:

- package updates
- Wazuh package downloads
- software installation
- approved HTTPS connectivity

A later HTTPS validation returned:

`HTTP/2 403`

which still proved successful DNS, routing, TCP, TLS, and remote-server connectivity.

---

# Unauthorized DMZ Outbound Traffic

An unapproved outbound TCP connection from the DMZ was observed being blocked by:

`Default deny / state violation rule`

This confirmed that the DMZ had not been given unrestricted outbound Internet access.

---

# Phase 8 — REDNET to DMZ Web Exposure

The DMZ required an intentionally exposed service for controlled web-security testing.

A specific REDNET rule was created.

Rule:

`ALLOW - REDNET to LAB-DMZ-01 HTTP`

Configuration:

    Action: Pass
    Interface: REDNET
    Direction: In
    TCP/IP Version: IPv4
    Protocol: TCP
    Source: REDNET network
    Destination: LAB_DMZ_01
    Destination Port: HTTP (80)
    Logging: Enabled

Purpose:

Permit Kali to access the Nginx service.

---

# REDNET Web Rule Position

The HTTP rule had to appear above:

`BLOCK - REDNET to Unauthorized Private Networks`

Correct order:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

---

# Important OPNsense Apply Lesson

During the REDNET-to-DMZ deployment, the allow rule had been correctly created and positioned.

However:

`curl http://10.10.60.10`

continued timing out.

OPNsense Live View showed the traffic still matching:

`BLOCK - REDNET to Unauthorized Private`

The cause was not the rule definition.

The pending firewall configuration had not yet been applied.

The red:

`Apply`

button was still visible.

After applying the configuration, the same request returned:

`HTTP/1.1 200 OK`

This established an important operational rule:

**Saving a firewall rule is not enough. Confirm that pending OPNsense changes are applied.**

---

# REDNET → DMZ SSH

No SSH exception was created for REDNET.

Test:

    nc -zv -w 3 10.10.60.10 22

Result:

    Connection timed out

Firewall logging showed the traffic blocked by:

`BLOCK - REDNET to Unauthorized Private Networks`

This confirmed that Kali could test the web service without gaining administrative access to the DMZ server.

---

# Phase 9 — Restricted CORPNET to DMZ Administration

Typing long commands directly into the Ubuntu VirtualBox console became inefficient.

A legitimate remote-administration path was therefore required.

Initial test from:

`LAB-WIN-01`

to:

`LAB-DMZ-01:22`

timed out.

This was expected because CORPNET did not yet have an approved SSH path into the DMZ.

---

# Administrative Host Alias

Existing alias:

`ADMIN_WORKSTATION`

Address:

`10.10.20.20`

This alias was reused instead of creating a broad CORPNET permission.

---

# Restricted DMZ SSH Rule

Rule:

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

Permit the approved Windows workstation to administer the DMZ.

---

# CORPNET DMZ Rule Position

The rule was positioned above:

`BLOCK - CORPNET to Unauthorized Private Networks`

Correct order:

    ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH
    BLOCK - CORPNET to Unauthorized Private Networks

After applying the rule, SSH from:

`LAB-WIN-01`

to:

`LAB-DMZ-01`

succeeded.

---

# Why ADNET Was Not Given DMZ SSH

Using:

`LAB-DC-01`

as an SSH jump host was considered but rejected.

The Domain Controller is critical identity infrastructure.

Allowing general-purpose DMZ administration from the DC would:

- increase attack surface
- mix identity and administration roles
- introduce unnecessary trust
- weaken architectural realism

Final validation confirmed:

`ADNET → DMZ SSH/22`

was blocked.

---

# Phase 10 — DMZ SSH Host Hardening

The firewall path was reinforced by host-level SSH controls.

Final effective SSH configuration on `LAB-DMZ-01`:

    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no

Key-only SSH from:

`LAB-WIN-01`

was validated successfully.

Password-only SSH was explicitly tested and rejected.

The result is defense in depth:

    OPNsense source restriction
        +
    SSH key-only authentication

---

# Final REDNET Rules

The authoritative final REDNET behaviour includes:

## Approved

    REDNET → DMZ HTTP/80

Purpose:

Controlled offensive web testing.

## Blocked

    REDNET → DMZ SSH/22
    REDNET → CORPNET SMB/445
    REDNET → ADNET LDAP/389

## Required Infrastructure

REDNET retains required DNS and approved Internet access through OPNsense.

## Management

REDNET is not an approved OPNsense management source.

---

# Final CORPNET Rules

CORPNET provides normal workstation functionality while remaining segmented.

## OPNsense Administration

Approved administrative access from:

`ADMIN_WORKSTATION`

where required.

## Active Directory

CORPNET can reach `LAB-DC-01` using the approved domain-service rules.

TCP services include:

- `53`
- `88`
- `135`
- `389`
- `445`
- `464`
- `3268`
- `49152:65535`

UDP services include:

- `53`
- `88`
- `123`
- `389`
- `464`

## Wazuh

CORPNET → SOCNET Wazuh communication is allowed.

## DMZ Administration

Only:

`ADMIN_WORKSTATION`

may access:

`LAB_DMZ_01`

using SSH TCP/22.

## Broad Private-Network Policy

Unauthorized CORPNET access remains restricted by broad private-network rules.

---

# Final SERVERNET Rules

SERVERNET retains only the communication needed for its role.

## Allowed

    SERVERNET → SOCNET Wazuh/1514
    SERVERNET → approved DNS/NTP
    SERVERNET → approved Internet access

## Restricted

    SERVERNET → CORPNET SMB/445
    SERVERNET → DMZ SSH/22
    SERVERNET → REDNET unsolicited access

---

# Final SOCNET Rules

SOCNET is the central monitoring destination.

Approved telemetry includes:

    CORPNET → Wazuh TCP/1514
    SERVERNET → Wazuh TCP/1514
    ADNET → Wazuh TCP/1514
    DMZ → Wazuh TCP/1514

Enrollment may use:

`TCP/1515`

Suricata telemetry reaches the SIEM using:

`UDP/5514`

SOCNET is not intended to become a general-purpose transit network.

---

# Final ADNET Rules

ADNET contains the Domain Controller and identity infrastructure.

## Allowed

    CORPNET → LAB-DC-01 approved AD TCP services
    CORPNET → LAB-DC-01 approved AD UDP services
    ADNET → SOCNET Wazuh/1514

## Restricted

    REDNET → ADNET LDAP/389
    ADNET → DMZ SSH/22

The Domain Controller remains isolated from unnecessary attacker and management paths.

---

# Final DMZ Rules

Final DMZ rule order:

    1. ALLOW - DMZ to OPNsense DNS
    2. ALLOW - DMZ to OPNsense NTP
    3. ALLOW - DMZ to Wazuh Agent Services
    4. BLOCK - DMZ to Unauthorized Private Networks
    5. ALLOW - DMZ HTTP HTTPS Outbound

Additional inbound policies include:

    REDNET → LAB-DMZ-01 HTTP/80                  ALLOW
    REDNET → LAB-DMZ-01 SSH/22                   BLOCK
    ADMIN_WORKSTATION → LAB-DMZ-01 SSH/22       ALLOW
    ADNET → LAB-DMZ-01 SSH/22                    BLOCK

---

# Firewall Logging Strategy

Not every permitted connection is logged.

Logging is concentrated on security-relevant boundaries.

Examples include:

- REDNET private-network violations
- DMZ private-network violations
- restricted administrative SSH
- attack-path testing
- significant block rules

This provides useful security evidence without creating unnecessary noise.

---

# OPNsense Live View

OPNsense Live View was repeatedly used to determine which rule actually processed traffic.

This was important during:

- REDNET → DMZ HTTP troubleshooting
- REDNET → DMZ SSH validation
- DMZ → CORPNET validation
- inter-zone deny testing

Example blocked flow:

    Source:
    10.10.10.10

    Destination:
    10.10.60.10:22

Result:

Blocked by:

`BLOCK - REDNET to Unauthorized Private Networks`

Another validated blocked flow involved:

    10.10.60.10 → 10.10.20.20

which was blocked by:

`BLOCK - DMZ to Unauthorized Private Networks`

---

# Final Firewall Validation Matrix

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

# Final Rule Philosophy

The final rule architecture follows a simple principle:

**Permit legitimate business and security-monitoring requirements, deny unnecessary inter-zone access, and preserve visibility into important security boundaries.**

The environment therefore avoids broad rules such as:

    REDNET → Any
    CORPNET → Any
    DMZ → PRIVATE_NETS
    ADNET → Any

Instead, access is tied to specific requirements.

---

# Firewall and Detection Relationship

The firewall is part of a larger defense-in-depth architecture.

Traffic may pass through:

    Source Host
        |
        v
    OPNsense Firewall
        |
        v
    Suricata IDS
        |
        v
    Destination Service
        |
        v
    Endpoint / Application Logs
        |
        v
    Wazuh SIEM

This means an allowed connection can still generate security telemetry and detections.

---

# Change Management

Throughout the project, firewall changes followed a consistent workflow:

    Snapshot
        |
        v
    Configure
        |
        v
    Review Rule Order
        |
        v
    Save
        |
        v
    Apply Changes
        |
        v
    Generate Test Traffic
        |
        v
    Review OPNsense Logs
        |
        v
    Verify Wazuh / Endpoint Behaviour
        |
        v
    Document

Important firewall-related snapshots included stages such as:

- OPNsense Suricata integration
- ADNET baseline
- pre-CORPNET Active Directory rules
- pre-ADNET Wazuh rules
- pre-DMZ build
- DMZ firewall baseline
- pre-DMZ web exposure
- DMZ web exposure verified
- pre-CORPNET-to-DMZ SSH
- CORPNET-to-DMZ SSH verified

After the lab reached its final validated state, these temporary build snapshots were removed.

---

# Final Snapshot Baseline

Every virtual machine now retains:

`Final-Six-Zone-Validated-Baseline`

This common snapshot represents the validated infrastructure state before operational SOC and Purple Team exercises begin.

---

# Historical vs Current Rules

This repository intentionally documents the firewall from the beginning of the project through the final state.

Earlier sections therefore describe rules and architectures that were later replaced or expanded.

Examples include:

- the original `192.168.56.0/24` network
- the first four-zone architecture
- early REDNET-to-SERVERNET testing
- workstation access to the original Ubuntu server
- initial DNS and management rules
- later SOCNET monitoring rules
- ADNET introduction
- DMZ introduction

These are preserved because they demonstrate the engineering evolution of the environment.

The authoritative current firewall configuration is the final six-zone policy documented in the later sections of this file.

---

# Final Status

The firewall build is complete.

The environment evolved from a simple flat lab into an OPNsense-controlled architecture providing:

- REDNET attacker isolation
- CORPNET workstation controls
- SERVERNET protection
- SOCNET telemetry collection
- ADNET identity isolation
- DMZ segmentation
- Active Directory service rules
- Wazuh agent communication
- firewall log forwarding
- Suricata IDS telemetry
- controlled REDNET attack paths
- restricted administrative SSH
- controlled Internet access
- default-deny behaviour
- broad private-network protection
- logged security boundaries
- validated end-to-end segmentation

The firewall is now treated as stable infrastructure for realistic SOC and Purple Team operations rather than an unfinished network build.
