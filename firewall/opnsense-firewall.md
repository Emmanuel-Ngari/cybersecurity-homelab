# OPNsense Firewall

## Overview

This document records the complete lifecycle of `LAB-FW-01`, the OPNsense firewall used in the Cybersecurity Home Lab.

The project did not begin with OPNsense.

The earliest version of the lab used a simple VirtualBox design with Kali Linux, Windows, and Ubuntu Server communicating primarily over:

`192.168.56.0/24`

That environment was useful for foundational attacker/defender exercises, but it lacked realistic enterprise segmentation.

As the project evolved toward SOC and Purple Team training, OPNsense was introduced to provide:

- centralized routing
- network segmentation
- firewall policy enforcement
- DNS and NTP services
- controlled Internet access
- security logging
- Suricata IDS
- telemetry forwarding to Wazuh
- DMZ isolation
- Active Directory network protection

The final firewall supports six security zones plus an upstream Internet-facing interface.

---

# Phase 1 — Before OPNsense

The original lab used a flat or lightly segmented VirtualBox network.

Primary systems included:

- Kali Linux
- Windows
- Ubuntu Server

Typical addressing included:

- Kali — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu — `192.168.56.30`

The environment supported early exercises such as:

- Nmap scanning
- SSH
- fail2ban
- Hydra testing
- service exposure
- connectivity troubleshooting
- attacker/defender interaction

At this stage, the lab did not yet have a dedicated firewall enforcing trust boundaries between different classes of systems.

The lack of segmentation became a limitation as the environment grew.

---

# Why OPNsense Was Introduced

The lab needed a platform capable of supporting:

- separate attacker and defender networks
- explicit east-west firewall policy
- centralized routing
- enterprise-style trust zones
- SIEM connectivity
- Active Directory isolation
- DMZ deployment
- IDS monitoring
- security-event logging

OPNsense was selected as the central network-security platform.

This marked the beginning of the transition from a basic home lab into a more realistic enterprise-style CyberLab.

---

# LAB-FW-01 Role

Hostname:

`LAB-FW-01`

Platform:

`OPNsense`

Primary responsibilities:

- firewall
- inter-zone router
- network segmentation
- DNS forwarding
- NTP services
- Internet gateway
- firewall logging
- Suricata IDS
- security telemetry source
- controlled trust enforcement

---

# Initial Segmented Architecture

The first OPNsense-based design introduced four internal zones:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

These zones separated the offensive workstation, corporate workstation, Linux server, and security-monitoring infrastructure.

---

# Initial Four-Zone Networks

## REDNET

Network:

`10.10.10.0/24`

Gateway:

`10.10.10.1`

Primary system:

`LAB-KALI-01 — 10.10.10.10`

Purpose:

Offensive security and attack simulation.

---

## CORPNET

Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

Primary system:

`LAB-WIN-01 — 10.10.20.20`

Purpose:

Corporate Windows workstation environment.

---

## SERVERNET

Network:

`10.10.30.0/24`

Gateway:

`10.10.30.1`

Primary system:

`LAB-UBUNTU-01 — 10.10.30.30`

Purpose:

Protected internal Linux server environment.

---

## SOCNET

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

Primary system:

`LAB-SIEM-01 — 10.10.40.10`

Purpose:

Centralized Wazuh security monitoring.

---

# OPNsense Interface Philosophy

Each security zone receives its own OPNsense interface.

This ensures:

- traffic between zones must traverse the firewall
- policy is enforced centrally
- logging can occur at trust boundaries
- network attacks can be inspected
- systems cannot bypass segmentation at Layer 2

The firewall became the single Layer 3 enforcement point for the virtual environment.

---

# Upstream Internet Connectivity

The OPNsense WAN interface uses VirtualBox NAT.

This provides controlled Internet access for internal systems without giving each virtual machine its own direct NAT adapter.

Traffic flow:

    Endpoint
        |
        v
    Internal Network
        |
        v
    LAB-FW-01
        |
        v
    VirtualBox NAT
        |
        v
    Internet

This architecture allows OPNsense to remain the authoritative network gateway.

---

# Initial Adapter Layout

As the lab developed, `LAB-FW-01` used multiple VirtualBox network adapters.

The final mapping became:

| Adapter | VirtualBox Mode | Network | OPNsense Function |
|---|---|---|---|
| Adapter 1 | NAT | VirtualBox NAT | WAN |
| Adapter 2 | Internal Network | `CORP-NET` | CORPNET |
| Adapter 3 | Internal Network | `RED-NET` | REDNET |
| Adapter 4 | Internal Network | `SERVER-NET` | SERVERNET |
| Adapter 5 | Internal Network | `SOC-NET` | SOCNET |
| Adapter 6 | Internal Network | `AD-NET` | ADNET |
| Adapter 7 | Internal Network | `DMZ-NET` | DMZ |
| Adapter 8 | Disabled | — | Unused |

---

# First Firewall Policy Development

Once OPNsense was operational, the firewall rule base was developed zone by zone.

The initial objective was to prevent the lab from becoming a flat routed network where every segment could communicate freely.

The early rules focused on:

- CORPNET administration
- REDNET containment
- SERVERNET isolation
- DNS
- NTP
- Internet access
- controlled attacker-to-server testing

---

# CORPNET Early Role

CORPNET became the primary administrative and enterprise-workstation network.

Early allowed communication included:

- OPNsense management
- DNS
- selected access to SERVERNET
- Internet access

The workstation was intentionally prevented from receiving unrestricted access to REDNET.

---

# REDNET Early Role

REDNET became the attacker network.

Kali retained:

- DNS
- Internet access
- selected testing paths

while losing broad access to enterprise-style systems.

This created a controlled attacker environment rather than an unrestricted internal host.

---

# SERVERNET Early Role

SERVERNET contained the Linux server.

The server retained:

- DNS
- NTP
- Internet access
- approved management paths

while unsolicited access toward workstation and attacker zones was restricted.

---

# SOCNET Introduction

SOCNET was introduced when Wazuh became a central component of the lab.

The SIEM was assigned:

`10.10.40.10`

and became the central destination for endpoint security telemetry.

This required OPNsense to support narrowly scoped monitoring paths rather than broad inter-zone permissions.

---

# Wazuh Communication Through OPNsense

Primary Wazuh agent communication:

`TCP/1514`

Enrollment:

`TCP/1515`

Monitored zones were granted access to the SIEM using explicit rules.

This included:

- CORPNET
- SERVERNET
- later ADNET
- later DMZ

---

# OPNsense as a Telemetry Source

As the project matured, OPNsense itself became part of the security-monitoring pipeline.

Firewall events were forwarded to Wazuh.

This allowed the SOC to investigate:

- blocked connections
- segmentation violations
- unauthorized access attempts
- suspicious inter-zone traffic

A firewall correlation detection was successfully validated using:

`Wazuh Rule 87702`

This was an important transition.

OPNsense was no longer used only to enforce traffic.

It also became a security telemetry source.

---

# Suricata Introduction

Suricata IDS was later installed on OPNsense.

The objective was to add network-level detection in addition to normal firewall enforcement.

Primary monitored interface:

`em2`

This corresponds to REDNET traffic.

This placement was selected because REDNET generates controlled attacker activity.

---

# Suricata Capabilities

Suricata provides:

- network intrusion detection
- signature-based alerts
- protocol visibility
- EVE JSON events
- network-layer attack telemetry

The lab used Suricata together with ET Open rules.

---

# Early Suricata Validation

A controlled test generated an alert matching:

`GPL ATTACK_RESPONSE id check returned root`

Signature ID:

`2100498`

Category:

`Potentially Bad Traffic`

Severity:

`2`

This confirmed that Suricata could inspect lab traffic and produce meaningful IDS events.

---

# Suricata Remote Logging

Suricata telemetry was integrated into SOCNET.

Final event path:

    LAB-FW-01
    Suricata
        |
        v
    Remote Syslog
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
    Wazuh

The SIEM uses a dedicated rsyslog listener on:

`UDP/5514`

for Suricata telemetry.

---

# Suricata Wazuh Validation

The IDS pipeline was validated end to end.

A Suricata alert ultimately generated:

`Wazuh Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

This confirmed that OPNsense IDS telemetry reached the SIEM successfully.

---

# Phase 2 — ADNET Expansion

After the initial four-zone environment and security-monitoring stack were working, Active Directory was introduced.

A new network was created:

`ADNET — 10.10.50.0/24`

Gateway:

`10.10.50.1`

Primary system:

`LAB-DC-01 — 10.10.50.10`

Domain:

`cyberlab.internal`

---

# Why ADNET Was Separated

The Domain Controller was not placed directly inside CORPNET.

A dedicated identity zone provided better separation between:

- user endpoints
- authentication infrastructure
- privileged directory services

This improved realism and reduced unnecessary trust.

---

# Active Directory Firewall Requirements

CORPNET needed access to multiple AD services.

The firewall was expanded to permit required traffic to `LAB-DC-01`.

Important TCP services included:

- `53` DNS
- `88` Kerberos
- `135` RPC Endpoint Mapper
- `389` LDAP
- `445` SMB
- `464` Kerberos password operations
- `3268` Global Catalog
- `49152:65535` Dynamic RPC

Important UDP services included:

- `53` DNS
- `88` Kerberos
- `123` NTP
- `389` LDAP
- `464` Kerberos password operations

This allowed:

- domain join
- authentication
- Group Policy
- LDAP
- SYSVOL access
- Kerberos
- RPC-based Windows domain functions

---

# ADNET Monitoring

The Domain Controller was also integrated into Wazuh.

Required path:

`ADNET → SOCNET`

Primary port:

`TCP/1514`

Enrollment:

`TCP/1515`

The Domain Controller remained isolated from unnecessary access to other zones.

---

# Active Directory Validation

Final connectivity validation confirmed:

- CORPNET → ADNET DNS/53 allowed
- CORPNET → ADNET Kerberos/88 allowed
- CORPNET → ADNET LDAP/389 allowed
- ADNET → SOCNET Wazuh/1514 allowed
- REDNET → ADNET LDAP/389 blocked
- ADNET → DMZ SSH/22 blocked

The Active Directory firewall design therefore supported required domain functionality while preserving segmentation.

---

# Phase 3 — DMZ Expansion

The DMZ was the final major network added to the firewall.

Network:

`10.10.60.0/24`

Gateway:

`10.10.60.1`

Primary server:

`LAB-DMZ-01 — 10.10.60.10`

Primary service:

`Nginx HTTP`

---

# Adding the DMZ Adapter

The firewall required a seventh VirtualBox NIC.

The adapter was added while the VM was powered off using:

`VBoxManage`

The final adapter was configured as:

    NIC 7
    Mode: Internal Network
    Network: DMZ-NET
    Adapter Type: Intel PRO/1000 MT Desktop (82540EM)
    Cable Connected: Yes

OPNsense detected the adapter as:

`em6`

It was assigned as:

`DMZ`

---

# DMZ Interface Configuration

Interface:

`DMZ`

Device:

`em6`

IPv4 configuration:

`Static IPv4`

Address:

`10.10.60.1/24`

IPv6:

`None`

Upstream gateway:

`None`

This interface became the default gateway for:

`LAB-DMZ-01`

---

# DMZ Host Configuration

The DMZ server was configured with:

    Hostname: lab-dmz-01
    IP: 10.10.60.10
    Prefix: /24
    Gateway: 10.10.60.1
    DNS: 10.10.60.1

This confirmed that the server depended on OPNsense for both routing and DNS forwarding.

---

# DMZ Firewall Aliases

Several aliases supported the DMZ design.

## `LAB_DMZ_01`

Address:

`10.10.60.10`

---

## `HTTP_HTTPS_PORTS`

Ports:

- `80`
- `443`

---

## Existing Reused Aliases

The DMZ also reused:

- `WAZUH_SERVER`
- `WAZUH_AGENT_PORTS`
- `DNS_PORT`
- `NTP_PORT`
- `PRIVATE_NETS`
- `ADMIN_WORKSTATION`

---

# Final DMZ Rule Order

The DMZ interface uses the following important rule order:

    1. ALLOW - DMZ to OPNsense DNS
    2. ALLOW - DMZ to OPNsense NTP
    3. ALLOW - DMZ to Wazuh Agent Services
    4. BLOCK - DMZ to Unauthorized Private Networks
    5. ALLOW - DMZ HTTP HTTPS Outbound

This order is intentional.

The Wazuh exception must occur before the broad private-network block because SOCNET is itself part of the protected internal addressing space.

---

# DMZ DNS Rule

Rule:

`ALLOW - DMZ to OPNsense DNS`

Purpose:

Permit the DMZ server to use OPNsense for DNS resolution.

Protocol:

`TCP/UDP`

Port:

`53`

---

# DMZ NTP Rule

Rule:

`ALLOW - DMZ to OPNsense NTP`

Purpose:

Permit time synchronization.

Protocol:

`UDP`

Port:

`123`

---

# DMZ Wazuh Rule

Rule:

`ALLOW - DMZ to Wazuh Agent Services`

Destination:

`WAZUH_SERVER`

Ports:

- `1514`
- `1515`

Protocol:

`TCP`

Purpose:

Permit:

- Wazuh telemetry
- agent enrollment

Connectivity to both ports was tested successfully before installing the DMZ agent.

---

# DMZ Private-Network Block

Rule:

`BLOCK - DMZ to Unauthorized Private Networks`

Action:

`Block`

Source:

`DMZ network`

Destination:

`PRIVATE_NETS`

Logging:

`Enabled`

Purpose:

Prevent a compromised DMZ server from freely reaching internal enterprise networks.

Validated examples:

- DMZ → CORPNET SMB/445 blocked
- DMZ → ADNET LDAP/389 blocked

---

# DMZ HTTP/HTTPS Outbound

Rule:

`ALLOW - DMZ HTTP HTTPS Outbound`

Ports:

- `80`
- `443`

Protocol:

`TCP`

Destination:

`Any`

Purpose:

Permit:

- software updates
- package downloads
- approved HTTPS connectivity

---

# DMZ Default-Deny Validation

An outbound connection attempt using an unapproved port was blocked by:

`Default deny / state violation rule`

This confirmed that the DMZ did not have unrestricted outbound connectivity.

---

# REDNET to DMZ Web Exposure

The Nginx server needed one controlled attack path from REDNET.

A specific rule was introduced:

`ALLOW - REDNET to LAB-DMZ-01 HTTP`

Source:

`REDNET network`

Destination:

`LAB_DMZ_01`

Protocol:

`TCP`

Port:

`80`

Logging:

`Enabled`

This provided an intentional web-security testing path.

---

# OPNsense Rule-Order Troubleshooting

During deployment, the HTTP allow rule was correctly created but the connection still timed out.

OPNsense Live View showed the request matching:

`BLOCK - REDNET to Unauthorized Private`

The specific allow rule was placed above the broad block, but the behaviour still did not change.

The final cause was that the OPNsense configuration had not yet been applied.

The red:

`Apply`

button was still visible.

After applying the pending changes, the request succeeded immediately.

This created an important operational lesson:

**A saved OPNsense rule is not necessarily active until the pending configuration is applied.**

---

# REDNET DMZ Validation

Final behaviour:

    REDNET → DMZ HTTP/80    ALLOW
    REDNET → DMZ SSH/22     BLOCK

This provides a realistic attacker-to-web-server path without providing administrative access.

---

# Restricted CORPNET to DMZ Administration

A management path was later required to simplify administration of the Linux DMZ server.

Initial test from:

`LAB-WIN-01`

to:

`LAB-DMZ-01:22`

timed out.

This confirmed that the firewall was correctly blocking unapproved CORPNET-to-DMZ SSH.

---

# Approved Administrative Rule

Rule:

`ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH`

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

Logging:

`Enabled`

This rule was positioned above:

`BLOCK - CORPNET to Unauthorized Private Networks`

---

# Why the Domain Controller Was Not Used for Administration

Allowing:

`LAB-DC-01 → LAB-DMZ-01 SSH`

was considered but rejected.

The Domain Controller is high-value identity infrastructure and should not be used as:

- a jump host
- a general management workstation
- a convenience administration system

The approved management source remains:

`LAB-WIN-01`

Final validation confirmed:

`ADNET → DMZ SSH/22`

was blocked.

---

# OPNsense Rule Order

Rule order became one of the most important operational concepts in the lab.

The design pattern used throughout is:

    Specific Allow
    Specific Allow
    Broad Block
    Approved Internet Access
    Default Deny

Examples:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

and:

    ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH
    BLOCK - CORPNET to Unauthorized Private Networks

This preserves least privilege while still allowing legitimate exceptions.

---

# OPNsense Logging Strategy

Logging is enabled where it provides useful security evidence.

Important logged events include:

- REDNET access violations
- DMZ private-network violations
- restricted SSH traffic
- firewall deny events
- selected security-boundary rules

Not every normal allow rule is logged.

This reduces unnecessary event volume while retaining high-value evidence.

---

# OPNsense Live View

Live View became an important troubleshooting tool throughout the project.

It was used to determine:

- which rule matched a packet
- whether a rule had actually been applied
- whether traffic reached the firewall
- whether a broad block was overriding a narrow allow
- whether a security-zone boundary behaved correctly

Example:

    Source: 10.10.10.10
    Destination: 10.10.60.10
    Port: 22
    Action: block
    Interface: em2

This confirmed REDNET SSH attempts to the DMZ were being denied.

---

# OPNsense and Wazuh Correlation

Firewall logs were forwarded to Wazuh.

A custom or correlation detection was verified using:

`Rule 87702`

This allowed OPNsense events to be analyzed alongside:

- Windows telemetry
- Linux telemetry
- Active Directory telemetry
- Suricata IDS events
- Nginx logs

---

# Suricata REDNET-to-DMZ Validation

A controlled HTTP test was used to confirm that Suricata could inspect DMZ traffic involving REDNET.

The DMZ temporarily returned:

`uid=0(root) gid=0(root) groups=0(root)`

Kali requested the page.

Suricata generated:

- Event type: `alert`
- Interface: `em2`
- Source IP: `10.10.60.10`
- Destination IP: `10.10.10.10`
- Signature ID: `2100498`
- Signature: `GPL ATTACK_RESPONSE id check returned root`
- Direction: `to_client`

This confirmed:

**Suricata had visibility into the REDNET ↔ DMZ traffic path.**

---

# Suricata-to-Wazuh Validation

The event was forwarded to the SIEM.

Wazuh generated:

`Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

This validated:

    Network Traffic
        |
        v
    OPNsense
        |
        v
    Suricata
        |
        v
    EVE JSON
        |
        v
    LAB-SIEM-01
        |
        v
    Wazuh Alert

---

# Final OPNsense Interfaces

The final firewall provides gateways for:

| Zone | Gateway |
|---|---|
| REDNET | `10.10.10.1` |
| CORPNET | `10.10.20.1` |
| SERVERNET | `10.10.30.1` |
| SOCNET | `10.10.40.1` |
| ADNET | `10.10.50.1` |
| DMZ | `10.10.60.1` |

WAN connectivity is provided through VirtualBox NAT.

---

# Final Zone Behaviour

## REDNET

Allowed:

- DNS
- approved Internet access
- DMZ HTTP/80

Blocked:

- DMZ SSH/22
- CORPNET SMB/445
- ADNET LDAP/389
- firewall-management access

---

## CORPNET

Allowed:

- OPNsense administration where approved
- Active Directory services
- Wazuh telemetry
- restricted SSH administration of `LAB-DMZ-01`
- approved Internet access

Restricted:

- unnecessary access to REDNET and protected networks

---

## SERVERNET

Allowed:

- DNS
- NTP
- Wazuh telemetry
- approved Internet access

Blocked or restricted:

- CORPNET SMB
- DMZ SSH
- unnecessary REDNET access

---

## SOCNET

Receives:

- Windows telemetry
- Linux telemetry
- Domain Controller telemetry
- DMZ telemetry
- firewall logs
- Suricata events

SOCNET is not intended to function as a general transit network.

---

## ADNET

Allowed:

- required CORPNET domain services
- Wazuh telemetry

Blocked or restricted:

- REDNET LDAP
- DMZ SSH
- unnecessary non-domain access

---

## DMZ

Allowed:

- OPNsense DNS
- OPNsense NTP
- Wazuh communication
- HTTP/HTTPS outbound
- inbound HTTP from REDNET
- inbound SSH from `ADMIN_WORKSTATION`

Blocked:

- unauthorized private-network access
- REDNET SSH
- ADNET SSH
- CORPNET SMB initiated from DMZ
- ADNET LDAP initiated from DMZ
- unapproved outbound ports

---

# Final Six-Zone Validation

The final firewall configuration was tested using real service connections.

Validated results:

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

# Change Management

OPNsense changes were treated as significant infrastructure changes.

The standard workflow became:

    Snapshot
        |
        v
    Configure
        |
        v
    Review
        |
        v
    Save
        |
        v
    Apply
        |
        v
    Generate Test Traffic
        |
        v
    Review Live View
        |
        v
    Verify SIEM / Endpoint Behaviour
        |
        v
    Document

Snapshots were taken before important phases such as:

- Suricata integration
- ADNET introduction
- Active Directory firewall rules
- DMZ build
- web exposure
- CORPNET-to-DMZ SSH administration

---

# Snapshot Cleanup

The project accumulated many temporary snapshots during development.

Once the environment passed final six-zone validation, those temporary checkpoints were deleted and merged.

Every virtual machine now retains only:

`Final-Six-Zone-Validated-Baseline`

This provides a clean known-good rollback point.

---

# Historical vs Current OPNsense State

This repository intentionally records the full evolution of the firewall.

Earlier stages may show:

- the original `192.168.56.0/24` network
- the initial four-zone architecture
- earlier aliases
- temporary firewall exceptions
- pre-Active Directory policy
- pre-DMZ policy

These represent valid historical stages.

The current authoritative OPNsense architecture is the six-zone design documented in the final sections of this file.

---

# Final Architecture

The final OPNsense design can be summarized as:

    Internet
       |
       v
    VirtualBox NAT
       |
       v
    +----------------------+
    |      LAB-FW-01       |
    |       OPNsense       |
    | Firewall / Routing   |
    |    Suricata IDS      |
    +----------------------+
      |     |      |      |      |      |
      v     v      v      v      v      v
    RED   CORP   SERVER   SOC    AD     DMZ
    NET    NET     NET     NET    NET    NET

OPNsense is the central security boundary for all inter-zone communication.

---

# Final Status

The OPNsense infrastructure is complete and validated.

It now provides:

- six-zone routing
- least-privilege firewall policy
- attacker-network isolation
- corporate-network controls
- server-network protection
- SOC connectivity
- Active Directory isolation
- DMZ segmentation
- restricted administrative access
- DNS and NTP services
- controlled Internet access
- firewall logging
- Wazuh firewall telemetry
- Suricata IDS
- Suricata-to-Wazuh integration
- validated default-deny behaviour

The firewall is now considered stable infrastructure for realistic SOC and Purple Team exercises.
