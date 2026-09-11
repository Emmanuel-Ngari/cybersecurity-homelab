# Cybersecurity Homelab Changelog

This changelog documents major architecture, networking, security, and capability milestones in the cybersecurity homelab.

---

## 2026-09-11 — Phase 1 Complete: Segmented OPNsense Network & Firewall Foundation

### Milestone

Completed the first major infrastructure phase of the cybersecurity homelab.

The environment has evolved from a flat VirtualBox network into a segmented offensive and defensive security architecture using OPNsense as the central firewall, router, NAT gateway, DNS service, and security-policy enforcement point.

### Current Security Zones

| Zone | Network | Primary System | Purpose |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `LAB-KALI-01` | Offensive Security |
| CORPNET | `10.10.20.0/24` | `LAB-WIN-01` | Trusted Management / Defensive Endpoint |
| SERVERNET | `10.10.30.0/24` | `LAB-UBUNTU-01` | Protected Servers |

### OPNsense Deployment

Deployed `LAB-FW-01` running OPNsense with four VirtualBox network interfaces.

WAN:

`em0 — 10.0.2.15/24`

CORPNET:

`em1 — 10.10.20.1/24`

REDNET:

`em2 — 10.10.10.1/24`

SERVERNET:

`em3 — 10.10.30.1/24`

OPNsense now provides:

- Centralized routing
- Stateful firewalling
- Outbound NAT
- DNS
- NTP
- Network segmentation
- Access-control enforcement
- Security logging

### Mandatory Firewall Routing

Removed direct VirtualBox NAT bypasses from the internal systems.

Kali:

`10.10.10.10 → 10.10.10.1 → OPNsense → WAN → Internet`

Windows:

`10.10.20.20 → 10.10.20.1 → OPNsense → WAN → Internet`

Ubuntu:

`10.10.30.30 → 10.10.30.1 → OPNsense → WAN → Internet`

Direct NAT adapters on Kali, Windows, and Ubuntu are disabled.

Only OPNsense maintains direct VirtualBox NAT connectivity.

### Firewall Policy

Implemented explicit least-privilege firewall policies for:

- CORPNET
- REDNET
- SERVERNET

Created aliases for:

- Administrative workstation
- Kali attacker workstation
- Ubuntu server
- Internal networks
- Private networks
- Administrative server ports
- DNS
- NTP

The original broad OPNsense LAN allow rules were disabled after the custom policies were successfully validated.

### Validated Security Paths

Confirmed the following behavior:

`Windows → OPNsense HTTPS`

Result:

`ALLOW`

`Windows → Ubuntu SSH`

Result:

`ALLOW`

`Windows → Kali`

Result:

`BLOCK`

`Kali → Ubuntu`

Result:

`ALLOW`

Purpose:

Authorized offensive-security testing.

`Kali → Windows`

Result:

`BLOCK`

`Kali → OPNsense Management`

Result:

`BLOCK`

`Ubuntu → Windows`

Result:

`BLOCK`

`Ubuntu → Kali`

Result:

`BLOCK`

### Outbound Connectivity Validation

Successfully validated Internet access through OPNsense from:

- Kali Linux
- Windows 11 Enterprise
- Ubuntu Server

Testing included:

- ICMP connectivity
- DNS resolution
- HTTP/HTTPS connectivity
- Routing-table verification
- TCP connectivity testing

### Offensive Security Path

Established the controlled offensive-security path:

`LAB-KALI-01 → REDNET → OPNsense → SERVERNET → LAB-UBUNTU-01`

Nmap testing confirmed SSH availability on Ubuntu:

`22/tcp open ssh`

This pathway will be used for future authorized offensive-security exercises.

### Administrative Security Path

Established the trusted management path:

`LAB-WIN-01 → CORPNET → OPNsense → SERVERNET → LAB-UBUNTU-01`

Windows PowerShell confirmed successful SSH connectivity to Ubuntu on:

`TCP/22`

### Troubleshooting Milestones

Resolved several networking and firewall issues during the migration, including:

- Incorrect firewall rule actions
- Firewall rule ordering
- Windows route selection
- Kali interface renumbering
- NetworkManager profile rebinding
- Ubuntu Netplan configuration
- Interface persistence
- DNS configuration
- Outbound NAT
- Inter-zone routing
- Stateful firewall behavior

### Recovery and Change Management

Used a structured workflow throughout the deployment:

`Snapshot → Change → Test → Verify → Backup → Document → Continue`

VirtualBox snapshots were created before and after major migrations.

OPNsense configuration backups were exported and retained privately.

Firewall XML configuration backups are not stored in the public repository.

### Documentation Completed

Updated and expanded:

`README.md`

`architecture/lab-architecture.md`

`architecture/virtualbox-networking.md`

`architecture/network-diagram.md`

`architecture/network-diagram.png`

`firewall/opnsense-firewall.md`

`firewall/security-zones.md`

`firewall/policy-matrix.md`

`firewall/firewall-rules.md`

`firewall/validation-testing.md`

### Phase 1 Status

OPNsense:

`Operational`

REDNET:

`Operational`

CORPNET:

`Operational`

SERVERNET:

`Operational`

Outbound NAT:

`Verified`

DNS:

`Verified`

Network Segmentation:

`Verified`

Controlled Offensive Path:

`Verified`

Controlled Administrative Path:

`Verified`

Direct NAT Bypasses:

`Disabled`

Default LAN Allow Rules:

`Disabled`

Legacy LAB-NET:

`Retired`

### Next Phase

Phase 2 will focus on defensive visibility and SOC infrastructure.

Planned development:

`SOC-NET — 10.10.40.0/24`

Wazuh SIEM

Centralized logging

Windows Sysmon

Windows security telemetry

Ubuntu security telemetry

OPNsense log forwarding

Suricata IDS

Detection engineering

Threat hunting

Incident-response exercises

Target defensive workflow:

`Attack → Telemetry → Detection → Alert → Investigation → Response → Remediation → Retest`

---

## Project Methodology

`Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat`
