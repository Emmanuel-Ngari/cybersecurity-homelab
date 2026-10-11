# VirtualBox Networking Architecture

## Overview

The Cybersecurity Home Lab is implemented entirely within Oracle VirtualBox using a combination of:

- one upstream NAT interface
- six isolated VirtualBox Internal Networks
- OPNsense as the central Layer 3 firewall and router

This design allows the lab to simulate enterprise network segmentation without requiring physical networking equipment.

All communication between security zones must traverse `LAB-FW-01`, where OPNsense applies firewall policy, routing, logging, and intrusion detection.

The final environment contains six internal security zones:

| Zone | VirtualBox Internal Network | Subnet |
|---|---|---|
| REDNET | `RED-NET` | `10.10.10.0/24` |
| CORPNET | `CORP-NET` | `10.10.20.0/24` |
| SERVERNET | `SERVER-NET` | `10.10.30.0/24` |
| SOCNET | `SOC-NET` | `10.10.40.0/24` |
| ADNET | `AD-NET` | `10.10.50.0/24` |
| DMZ | `DMZ-NET` | `10.10.60.0/24` |

---

## Design Principles

The VirtualBox network design follows several security principles:

1. Each trust zone uses a separate Layer 2 network.
2. Endpoint virtual machines connect only to their assigned security zone.
3. OPNsense is the only system connected to multiple security zones.
4. Inter-zone communication is controlled by firewall policy.
5. Offensive systems are isolated from production-style infrastructure.
6. Internet connectivity is routed through OPNsense.
7. Security monitoring remains centralized in SOCNET.
8. No endpoint requires VirtualBox NAT directly.

This prevents the lab from becoming a flat network where systems can communicate freely.

---

## High-Level VirtualBox Design

The architecture can be represented as:

    Internet
       |
       |
    VirtualBox NAT
       |
       v
    LAB-FW-01
    OPNsense
       |
       +---------------------------------------------+
       |         |          |         |       |     |
       v         v          v         v       v     v
    RED-NET   CORP-NET   SERVER-NET SOC-NET AD-NET DMZ-NET
       |         |          |         |       |     |
       v         v          v         v       v     v
     Kali      Windows    Ubuntu     Wazuh    DC   Nginx

OPNsense provides routing between the internal networks.

---

# LAB-FW-01 VirtualBox Networking

`LAB-FW-01` is the central networking device in the environment.

It is the only virtual machine with interfaces connected to all six security zones.

## Adapter Mapping

| Adapter | VirtualBox Mode | Network | OPNsense Role |
|---|---|---|---|
| Adapter 1 | NAT | VirtualBox NAT | WAN / Internet |
| Adapter 2 | Internal Network | `CORP-NET` | CORPNET |
| Adapter 3 | Internal Network | `RED-NET` | REDNET |
| Adapter 4 | Internal Network | `SERVER-NET` | SERVERNET |
| Adapter 5 | Internal Network | `SOC-NET` | SOCNET |
| Adapter 6 | Internal Network | `AD-NET` | ADNET |
| Adapter 7 | Internal Network | `DMZ-NET` | DMZ |
| Adapter 8 | Disabled | — | Unused |

The additional interfaces beyond the standard VirtualBox GUI-visible adapters were configured using `VBoxManage`.

---

## DMZ Adapter Creation

The seventh network adapter was added to `LAB-FW-01` using VirtualBox command-line management.

The adapter was configured as:

    NIC 7
    Mode: Internal Network
    Network: DMZ-NET
    Adapter Type: Intel PRO/1000 MT Desktop (82540EM)
    Cable Connected: Yes

This provided the dedicated OPNsense interface required for the DMZ.

OPNsense detected this adapter as:

`em6`

The interface was assigned as:

`DMZ`

with:

`10.10.60.1/24`

---

# OPNsense Interface Addressing

The firewall provides the default gateway for every internal security zone.

| Security Zone | OPNsense Gateway |
|---|---|
| REDNET | `10.10.10.1` |
| CORPNET | `10.10.20.1` |
| SERVERNET | `10.10.30.1` |
| SOCNET | `10.10.40.1` |
| ADNET | `10.10.50.1` |
| DMZ | `10.10.60.1` |

No upstream gateway is configured on the internal OPNsense interfaces.

The WAN/NAT interface provides external connectivity.

---

# LAB-KALI-01

## Purpose

`LAB-KALI-01` represents the offensive security workstation.

## VirtualBox Network

    Adapter 1
    Mode: Internal Network
    Network: RED-NET

## Addressing

    IP: 10.10.10.10
    Gateway: 10.10.10.1
    Zone: REDNET

The Kali system does not receive unrestricted access to protected internal networks.

All communication must pass through OPNsense.

---

# LAB-WIN-01

## Purpose

`LAB-WIN-01` represents a corporate Windows workstation.

It is joined to the:

`cyberlab.internal`

Active Directory domain.

## VirtualBox Network

    Adapter 1
    Mode: Internal Network
    Network: CORP-NET

## Addressing

    IP: 10.10.20.20
    Gateway: 10.10.20.1
    Zone: CORPNET

The workstation communicates with Active Directory through explicit firewall rules.

It also functions as the approved administrative workstation for DMZ SSH management.

---

# LAB-UBUNTU-01

## Purpose

`LAB-UBUNTU-01` represents an internal Linux server.

## VirtualBox Network

    Adapter 1
    Mode: Internal Network
    Network: SERVER-NET

## Addressing

    IP: 10.10.30.30
    Gateway: 10.10.30.1
    Zone: SERVERNET

The server is isolated from unnecessary workstation and DMZ access.

Required Wazuh communication to SOCNET is explicitly permitted.

---

# LAB-SIEM-01

## Purpose

`LAB-SIEM-01` hosts the centralized Wazuh security monitoring infrastructure.

## VirtualBox Network

    Adapter 1
    Mode: Internal Network
    Network: SOC-NET

## Addressing

    IP: 10.10.40.10
    Gateway: 10.10.40.1
    Zone: SOCNET

The SIEM receives security telemetry from multiple zones through OPNsense.

Monitored endpoints include:

- `LAB-WIN-01`
- `LAB-UBUNTU-01`
- `LAB-DC-01`
- `LAB-DMZ-01`

SOCNET also receives Suricata telemetry generated by OPNsense.

---

# LAB-DC-01

## Purpose

`LAB-DC-01` provides enterprise identity services.

Services include:

- Active Directory Domain Services
- DNS
- Kerberos
- LDAP
- Group Policy

## VirtualBox Network

    Adapter 1
    Mode: Internal Network
    Network: AD-NET

## Addressing

    IP: 10.10.50.10
    Gateway: 10.10.50.1
    Zone: ADNET

Identity infrastructure is intentionally separated from the corporate workstation network.

CORPNET reaches ADNET only through required domain protocols.

---

# LAB-DMZ-01

## Purpose

`LAB-DMZ-01` represents an externally exposed Linux web server.

It hosts Nginx and is monitored by Wazuh.

## VirtualBox Network

    Adapter 1
    Mode: Internal Network
    Network: DMZ-NET

All other adapters are disabled.

## Addressing

    IP: 10.10.60.10
    Gateway: 10.10.60.1
    DNS: 10.10.60.1
    Zone: DMZ

The host is deliberately prevented from having direct connections to internal networks.

All traffic passes through OPNsense.

---

# Why Internal Networks Are Used

VirtualBox Internal Networks were selected instead of Host-Only networking for the final architecture because they better simulate isolated enterprise security zones.

Internal Networks provide:

- Layer 2 isolation
- no automatic host connectivity
- no automatic DHCP unless explicitly provided
- controlled routing through OPNsense
- realistic firewall enforcement

A system in one Internal Network cannot directly communicate with another Internal Network without a router.

In this lab, that router is:

`LAB-FW-01`

This creates a realistic enforcement point for security policy.

---

# Internet Connectivity

The lab uses VirtualBox NAT only on the OPNsense WAN interface.

The endpoint machines do not require their own NAT adapters.

Traffic flow is:

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

This allows OPNsense to control and observe outbound connectivity.

---

# DNS Architecture

DNS requirements differ between zones.

## CORPNET

The Windows workstation relies on Active Directory DNS through `LAB-DC-01`.

This is necessary for:

- domain discovery
- Active Directory authentication
- Kerberos
- Group Policy
- service discovery

## DMZ

`LAB-DMZ-01` uses:

`10.10.60.1`

as its configured DNS resolver.

OPNsense provides approved DNS forwarding for the DMZ.

---

# Network Segmentation

Because each VirtualBox Internal Network is isolated, OPNsense becomes the enforcement point for east-west communication.

Examples include:

    REDNET → DMZ
    CORPNET → ADNET
    SERVERNET → SOCNET
    ADNET → SOCNET
    DMZ → SOCNET

Traffic cannot bypass OPNsense by communicating directly at Layer 2.

---

# Validated REDNET Controls

`LAB-KALI-01` was used to validate the offensive network boundary.

Validated results:

    REDNET → DMZ HTTP/80       ALLOWED
    REDNET → DMZ SSH/22        BLOCKED
    REDNET → CORPNET SMB/445   BLOCKED
    REDNET → ADNET LDAP/389    BLOCKED

This confirms that the Kali attack workstation is isolated from sensitive infrastructure while retaining the intended web-testing path to the DMZ.

---

# Validated CORPNET Controls

From `LAB-WIN-01`:

    CORPNET → ADNET DNS/53        ALLOWED
    CORPNET → ADNET Kerberos/88   ALLOWED
    CORPNET → ADNET LDAP/389      ALLOWED
    CORPNET → SOCNET Wazuh/1514   ALLOWED
    CORPNET → DMZ SSH/22          ALLOWED

The DMZ SSH rule is restricted specifically to the approved administrative workstation.

---

# Validated SERVERNET Controls

From `LAB-UBUNTU-01`:

    SERVERNET → SOCNET Wazuh/1514   ALLOWED
    SERVERNET → CORPNET SMB/445     BLOCKED
    SERVERNET → DMZ SSH/22          BLOCKED

This limits unnecessary lateral movement from the internal server environment.

---

# Validated ADNET Controls

From `LAB-DC-01`:

    ADNET → SOCNET Wazuh/1514   ALLOWED
    ADNET → DMZ SSH/22          BLOCKED

The Domain Controller is not used as a jump host or general management workstation.

---

# Validated DMZ Controls

From `LAB-DMZ-01`:

    DMZ → SOCNET Wazuh/1514   ALLOWED
    DMZ → CORPNET SMB/445     BLOCKED
    DMZ → ADNET LDAP/389      BLOCKED
    DMZ → Internet HTTPS      ALLOWED

The architecture therefore prevents the DMZ server from freely initiating connections into protected enterprise networks.

---

# Suricata Placement

Suricata runs on OPNsense.

The REDNET-facing interface is monitored to provide visibility into traffic generated by the offensive workstation.

The monitored OPNsense interface is:

`em2`

This allows attack-simulation traffic from Kali to be inspected before reaching target environments.

---

# Suricata Telemetry Path

The network telemetry path is:

    LAB-KALI-01
         |
         v
    RED-NET
         |
         v
    LAB-FW-01 / Suricata
         |
         v
    EVE JSON
         |
         v
    Remote Syslog UDP/5514
         |
         v
    LAB-SIEM-01
         |
         v
    /var/log/suricata/eve.json
         |
         v
    Wazuh

A controlled REDNET-to-DMZ test successfully triggered:

    Suricata SID 2100498
    GPL ATTACK_RESPONSE id check returned root

The resulting event was subsequently processed by Wazuh rule:

`86601`

---

# DMZ Administrative Architecture

The approved administrative path is:

    LAB-WIN-01
    10.10.20.20
         |
         | TCP/22
         v
    LAB-FW-01
         |
         v
    LAB-DMZ-01
    10.10.60.10

REDNET is not allowed to use SSH to reach the DMZ server.

ADNET is also prevented from using the Domain Controller as a DMZ administration system.

This maintains separation between:

- offensive activity
- identity infrastructure
- system administration

---

# DMZ SSH Hardening

The network restriction is reinforced by host-level SSH controls.

Effective SSH policy on `LAB-DMZ-01`:

    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no

Public-key authentication was verified from `LAB-WIN-01`.

Password-only authentication was separately tested and rejected.

---

# Monitoring Architecture

The network design supports multiple monitoring paths.

## Wazuh Endpoint Telemetry

    LAB-WIN-01
         |
         v
    LAB-SIEM-01

    LAB-UBUNTU-01
         |
         v
    LAB-SIEM-01

    LAB-DC-01
         |
         v
    LAB-SIEM-01

    LAB-DMZ-01
         |
         v
    LAB-SIEM-01

Primary agent communication:

`TCP/1514`

Enrollment:

`TCP/1515`

---

# Application Monitoring

Nginx activity on `LAB-DMZ-01` is collected through:

`/var/log/nginx/access.log`

and:

`/var/log/nginx/error.log`

This creates an additional application-layer telemetry source independent of Suricata network monitoring.

---

# VirtualBox Security Advantages

The final design provides several advantages for cybersecurity training.

## Isolation

Attack traffic remains inside the controlled virtual environment.

## Repeatability

Virtual machines can be returned to known-good states using snapshots.

## Segmentation

Every security zone is independently controlled.

## Visibility

Traffic must cross the firewall, creating opportunities for logging and IDS inspection.

## Flexibility

New scenarios can be introduced without requiring physical hardware.

## Safety

The offensive workstation does not have unrestricted access to the host or protected networks.

---

# Snapshot Strategy

Snapshots were created before major networking and security changes throughout the infrastructure build.

Examples included:

- network segmentation changes
- Suricata deployment
- Active Directory firewall rules
- DMZ deployment
- web exposure
- SSH administration
- endpoint-monitoring configuration

After the infrastructure was fully validated, obsolete build snapshots were removed.

Every VM now retains only:

`Final-Six-Zone-Validated-Baseline`

This provides a clean common rollback point for future SOC and Purple Team exercises.

---

# Final VirtualBox Topology

    Internet
       |
       |
    VirtualBox NAT
       |
       v
    +---------------------+
    |      LAB-FW-01      |
    |      OPNsense       |
    | Firewall / Routing  |
    |    Suricata IDS     |
    +---------------------+
       |      |      |      |      |      |
       |      |      |      |      |      |
       v      v      v      v      v      v

    RED    CORP   SERVER   SOC     AD     DMZ
    NET     NET     NET     NET    NET    NET

     |       |       |       |      |      |
     v       v       v       v      v      v

    KALI   WIN-01  UBUNTU   SIEM    DC    NGINX
    .10     .20      .30     .10    .10    .10

---

# Final Status

The VirtualBox network architecture is complete and validated.

The environment now provides:

- six isolated security zones
- centralized OPNsense routing
- least-privilege firewall enforcement
- controlled Internet access
- Active Directory segmentation
- restricted administrative paths
- isolated offensive infrastructure
- dedicated DMZ
- centralized Wazuh monitoring
- Suricata network visibility
- application-layer web monitoring

Future project work will use this architecture for realistic SOC investigations and Purple Team simulations rather than continuing to modify the underlying network design.
