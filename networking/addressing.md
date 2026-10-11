# Network Addressing Plan

## Overview

The Cybersecurity Home Lab uses a segmented six-zone network architecture designed to simulate a small enterprise environment.

OPNsense (`LAB-FW-01`) operates as the central firewall and Layer 3 routing point between all security zones. Inter-zone communication is controlled through explicit firewall rules following least-privilege principles.

The environment separates offensive systems, corporate endpoints, servers, security monitoring infrastructure, Active Directory services, and externally exposed services into dedicated networks.

---

## Network Zones

| Security Zone | Network | Gateway | Primary Purpose |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `10.10.10.1` | Offensive security and attack simulation |
| CORPNET | `10.10.20.0/24` | `10.10.20.1` | Corporate workstation environment |
| SERVERNET | `10.10.30.0/24` | `10.10.30.1` | Internal Linux server environment |
| SOCNET | `10.10.40.0/24` | `10.10.40.1` | Security monitoring and SIEM infrastructure |
| ADNET | `10.10.50.0/24` | `10.10.50.1` | Active Directory and identity infrastructure |
| DMZ | `10.10.60.0/24` | `10.10.60.1` | Externally exposed services |

All gateways are provided by dedicated OPNsense interfaces on `LAB-FW-01`.

---

## Host Addressing

| Hostname | IP Address | Zone | Operating System / Platform | Primary Role |
|---|---|---|---|---|
| `LAB-FW-01` | Multiple `.1` interfaces | All Zones | OPNsense | Firewall, routing, segmentation and IDS |
| `LAB-KALI-01` | `10.10.10.10` | REDNET | Kali Linux | Offensive security workstation |
| `LAB-WIN-01` | `10.10.20.20` | CORPNET | Windows 11 Enterprise | Domain workstation / monitored endpoint |
| `LAB-UBUNTU-01` | `10.10.30.30` | SERVERNET | Ubuntu Server | Internal Linux server |
| `LAB-SIEM-01` | `10.10.40.10` | SOCNET | Ubuntu Server | Wazuh SIEM infrastructure |
| `LAB-DC-01` | `10.10.50.10` | ADNET | Windows Server 2025 | Active Directory Domain Controller and DNS |
| `LAB-DMZ-01` | `10.10.60.10` | DMZ | Ubuntu Server | Nginx DMZ web server |

---

## REDNET — Offensive Security Zone

**Network:** `10.10.10.0/24`  
**Gateway:** `10.10.10.1`

### Host

- `LAB-KALI-01` — `10.10.10.10`

### Purpose

REDNET represents an untrusted or adversarial network used for controlled attack simulation.

Kali Linux is deliberately isolated from sensitive internal systems. Access from REDNET to protected networks is denied unless an explicit firewall rule exists for a controlled security test.

### Validated Behaviour

- REDNET → DMZ HTTP/80: **Allowed**
- REDNET → DMZ SSH/22: **Blocked**
- REDNET → CORPNET SMB/445: **Blocked**
- REDNET → ADNET LDAP/389: **Blocked**

HTTP traffic between REDNET and the DMZ is inspected by Suricata on OPNsense.

---

## CORPNET — Corporate Network

**Network:** `10.10.20.0/24`  
**Gateway:** `10.10.20.1`

### Host

- `LAB-WIN-01` — `10.10.20.20`

### Purpose

CORPNET represents the enterprise user workstation environment.

`LAB-WIN-01` is joined to the `cyberlab.internal` Active Directory domain and receives security configuration through Group Policy.

The workstation also functions as the approved administrative workstation for restricted SSH management of the DMZ server.

### Validated Services

CORPNET → ADNET:

- DNS TCP/53: **Allowed**
- Kerberos TCP/88: **Allowed**
- LDAP TCP/389: **Allowed**

CORPNET → SOCNET:

- Wazuh TCP/1514: **Allowed**

CORPNET → DMZ:

- SSH TCP/22 from `LAB-WIN-01` to `LAB-DMZ-01`: **Allowed**

DMZ SSH administration is restricted through an OPNsense host alias representing the approved administrative workstation.

---

## SERVERNET — Internal Server Zone

**Network:** `10.10.30.0/24`  
**Gateway:** `10.10.30.1`

### Host

- `LAB-UBUNTU-01` — `10.10.30.30`

### Purpose

SERVERNET represents protected internal server infrastructure.

The zone is isolated from unnecessary workstation and DMZ access while maintaining required communication with the SOC monitoring infrastructure.

### Validated Behaviour

- SERVERNET → SOCNET Wazuh TCP/1514: **Allowed**
- SERVERNET → CORPNET SMB/445: **Blocked**
- SERVERNET → DMZ SSH/22: **Blocked**

`LAB-UBUNTU-01` is monitored by the Wazuh SIEM.

---

## SOCNET — Security Operations Zone

**Network:** `10.10.40.0/24`  
**Gateway:** `10.10.40.1`

### Host

- `LAB-SIEM-01` — `10.10.40.10`

### Purpose

SOCNET contains the centralized security monitoring infrastructure.

`LAB-SIEM-01` hosts:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat
- Centralized Suricata event collection

Wazuh agents from monitored systems communicate with the SIEM over approved firewall paths.

### Monitored Endpoints

| Agent | IP Address | Status During Final Validation |
|---|---|---|
| `LAB-WIN-01` | `10.10.20.20` | Active |
| `LAB-UBUNTU-01` | `10.10.30.30` | Active |
| `LAB-DC-01` | `10.10.50.10` | Active |
| `LAB-DMZ-01` | `10.10.60.10` | Active |

---

## ADNET — Identity and Directory Services Zone

**Network:** `10.10.50.0/24`  
**Gateway:** `10.10.50.1`

### Host

- `LAB-DC-01` — `10.10.50.10`

### Purpose

ADNET isolates critical identity infrastructure from general workstation, server, offensive and DMZ networks.

`LAB-DC-01` provides:

- Active Directory Domain Services
- DNS
- Kerberos authentication
- LDAP directory services
- Group Policy
- Centralized identity management

### Domain

`cyberlab.internal`

### Validated Behaviour

- CORPNET → AD DNS: **Allowed**
- CORPNET → AD Kerberos: **Allowed**
- CORPNET → AD LDAP: **Allowed**
- ADNET → SOCNET Wazuh TCP/1514: **Allowed**
- ADNET → DMZ SSH/22: **Blocked**
- REDNET → ADNET LDAP/389: **Blocked**

Final targeted `dcdiag` validation completed without errors for:

- Advertising
- SYSVOL
- NetLogons
- Services

---

## DMZ — Demilitarized Zone

**Network:** `10.10.60.0/24`  
**Gateway:** `10.10.60.1`

### Host

- `LAB-DMZ-01` — `10.10.60.10`

### Purpose

The DMZ represents a network segment containing services that may need to interact with less-trusted networks while remaining isolated from protected enterprise infrastructure.

`LAB-DMZ-01` currently hosts an Nginx web service.

### Inbound Access

REDNET:

- HTTP TCP/80: **Allowed**
- SSH TCP/22: **Blocked**

CORPNET:

- SSH TCP/22 from `LAB-WIN-01` only: **Allowed**

ADNET:

- SSH TCP/22: **Blocked**

### Outbound Access

DMZ → SOCNET:

- Wazuh TCP/1514: **Allowed**
- Wazuh enrollment TCP/1515: **Allowed where required**

DMZ → internal protected networks:

- Unauthorized private-network access: **Blocked**

DMZ → Internet:

- HTTP/HTTPS outbound access: **Allowed**
- Other unauthorized outbound traffic: **Denied**

---

## DMZ Administrative Access

Remote administration of `LAB-DMZ-01` is intentionally restricted.

Approved path:

`LAB-WIN-01 (10.10.20.20) → LAB-DMZ-01 (10.10.60.10) → TCP/22`

SSH security controls include:

- Public-key authentication enabled
- Password authentication disabled
- Keyboard-interactive authentication disabled
- Direct root SSH login disabled
- OPNsense restricts the network path to the approved administrative workstation

The Domain Controller is deliberately not used as a general-purpose management or jump host.

---

## Security Monitoring Paths

### Endpoint Telemetry

`LAB-WIN-01 → LAB-SIEM-01`

`LAB-UBUNTU-01 → LAB-SIEM-01`

`LAB-DC-01 → LAB-SIEM-01`

`LAB-DMZ-01 → LAB-SIEM-01`

### Network IDS Telemetry

Suricata operates on OPNsense and provides IDS visibility into monitored REDNET traffic.

Validated detection path:

`LAB-KALI-01 → OPNsense / Suricata → LAB-DMZ-01`

A controlled HTTP response containing:

`uid=0(root) gid=0(root) groups=0(root)`

triggered Suricata signature:

- SID: `2100498`
- Signature: `GPL ATTACK_RESPONSE id check returned root`

The resulting Suricata event was forwarded to Wazuh and generated:

- Wazuh Rule ID: `86601`

---

## DMZ Web Monitoring

Nginx access and error logs from `LAB-DMZ-01` are collected by its Wazuh agent.

Monitored files:

- `/var/log/nginx/access.log`
- `/var/log/nginx/error.log`

A controlled REDNET request to the DMZ produced a Wazuh web-server alert:

- Wazuh Rule ID: `31101`
- Description: `Web server 400 error code.`
- Source: `10.10.10.10`
- Agent: `LAB-DMZ-01`
- Decoder: `web-accesslog`

This validates application-layer visibility in addition to network-layer Suricata monitoring.

---

## Segmentation Philosophy

The architecture follows several core security principles:

1. **Network segmentation** — systems with different trust levels are placed into separate zones.
2. **Least privilege** — inter-zone traffic is permitted only when required.
3. **Default-deny behaviour** — unauthorized communication between protected zones is blocked.
4. **Centralized monitoring** — security telemetry is forwarded to SOCNET.
5. **Attack visibility** — REDNET provides a controlled adversarial environment for testing defensive controls.
6. **Identity isolation** — Active Directory services are separated from general-purpose systems.
7. **DMZ isolation** — externally exposed services cannot freely communicate with protected internal networks.
8. **Restricted administration** — management access is limited to approved systems and protocols.

---

## Final Validation Status

The six-zone network was validated after completion of the firewall, Active Directory, SIEM, IDS, endpoint-monitoring and DMZ phases.

Validated zones:

- REDNET — `10.10.10.0/24`
- CORPNET — `10.10.20.0/24`
- SERVERNET — `10.10.30.0/24`
- SOCNET — `10.10.40.0/24`
- ADNET — `10.10.50.0/24`
- DMZ — `10.10.60.0/24`

All lab virtual machines were subsequently placed at the common known-good VirtualBox snapshot:

`Final-Six-Zone-Validated-Baseline`

This snapshot represents the validated infrastructure baseline prior to beginning operational SOC and Purple Team simulations.
