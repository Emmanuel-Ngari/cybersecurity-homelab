# OPNsense Security Zones

## 1. Overview

This document describes the security-zone architecture used in my cybersecurity homelab.

The purpose of the design is to separate systems according to trust level, operational role, and security function.

Instead of placing every virtual machine on one flat network, OPNsense provides routing and firewall enforcement between isolated network segments.

This creates a more realistic environment for offensive security, defensive security, network monitoring, incident response, and enterprise-style access control.

---

## 2. Security-Zone Architecture

The current design contains four primary network zones:

- WAN
- CORPNET
- REDNET
- SERVERNET

The architecture is:

Internet  
↓  
VirtualBox NAT  
↓  
OPNsense Firewall  
↓  
Security Zones  
↓  
Controlled Inter-Zone Communication

---

## 3. WAN Zone

**Interface:** `em0`

**Network:** VirtualBox NAT

**IPv4 Address:** `10.0.2.15/24`

**Gateway:** `10.0.2.2`

### Purpose

WAN represents the untrusted external network.

It provides Internet connectivity to OPNsense.

Internal laboratory systems will eventually access the Internet through OPNsense rather than through individual VirtualBox NAT adapters.

### Security Principle

Traffic originating from WAN is considered untrusted.

Inbound traffic from WAN should be blocked unless explicitly permitted.

---

## 4. CORPNET Zone

**Interface:** `em1`

**IPv4 Address:** `10.10.20.1/24`

**Network:** `10.10.20.0/24`

### Purpose

CORPNET represents the trusted corporate and management network.

It is intended to contain:

- Windows endpoints
- Administrative workstations
- Management systems
- Future domain-joined clients

The current management workstation is:

`10.10.20.20`

### Security Role

CORPNET is trusted more than REDNET but is not allowed unrestricted access to all internal networks.

Access is controlled using explicit firewall policy.

---

## 5. REDNET Zone

**Interface:** `em2`

**IPv4 Address:** `10.10.10.1/24`

**Network:** `10.10.10.0/24`

### Purpose

REDNET represents the offensive-security and attacker network.

It is intended to contain:

- Kali Linux
- Penetration-testing systems
- Red Team tools
- Security-testing infrastructure

Future Kali Linux address:

`10.10.10.10`

### Security Role

REDNET is treated as an untrusted internal network.

Systems on REDNET must not receive unrestricted access to trusted networks or firewall management interfaces.

---

## 6. SERVERNET Zone

**Interface:** `em3`

**IPv4 Address:** `10.10.30.1/24`

**Network:** `10.10.30.0/24`

### Purpose

SERVERNET represents the protected server environment.

It is intended to contain:

- Ubuntu Server
- Linux services
- Web applications
- Internal services
- Future intentionally vulnerable systems

Future Ubuntu Server address:

`10.10.30.30`

### Security Role

SERVERNET hosts systems that may be accessed by trusted administrators and controlled offensive-security systems.

Access is limited according to service requirements.

---

## 7. Zone Trust Model

The zones have different trust levels.

| Zone | Trust Level | Primary Role |
|---|---|---|
| WAN | Untrusted | External network |
| REDNET | Untrusted Internal | Offensive security |
| CORPNET | Trusted Internal | User and management systems |
| SERVERNET | Protected Internal | Servers and services |

Trust does not mean unrestricted access.

Even trusted systems are subject to firewall policy.

---

## 8. Default-Deny Philosophy

The security architecture follows a default-deny approach.

This means:

- Traffic should be blocked unless there is a valid reason to allow it.
- Access should be limited to required protocols.
- Administrative services should be restricted to trusted systems.
- Offensive systems should not automatically receive access to trusted networks.
- Server access should be based on explicitly approved services.

This reduces unnecessary exposure and improves visibility.

---

## 9. CORPNET Access Policy

CORPNET is the primary trusted network.

Current intended access includes:

### Allowed

- Windows management workstation → OPNsense HTTPS
- CORPNET → OPNsense DNS
- CORPNET → OPNsense NTP
- Management workstation → SERVERNET SSH
- Management workstation → SERVERNET HTTPS
- CORPNET → Internet

### Restricted

- CORPNET → REDNET
- CORPNET → unauthorized private networks
- CORPNET → unauthorized firewall services

---

## 10. REDNET Access Policy

REDNET is the attacker network.

Current intended policy includes:

### Allowed

- REDNET → Internet for updates and tool installation
- REDNET → DNS
- REDNET → NTP
- REDNET → SERVERNET for controlled security testing

### Restricted

- REDNET → CORPNET
- REDNET → OPNsense management interfaces
- REDNET → unauthorized internal services

This allows Kali Linux to perform authorized testing while preventing unrestricted access to the trusted management network.

---

## 11. SERVERNET Access Policy

SERVERNET contains protected servers.

Current intended policy includes:

### Allowed

- SERVERNET → DNS
- SERVERNET → NTP
- SERVERNET → Internet for required updates
- Trusted management → approved server administration services

### Restricted

- SERVERNET → CORPNET unsolicited connections
- SERVERNET → REDNET unsolicited connections
- SERVERNET → firewall management interfaces

SERVERNET should not be able to initiate unnecessary connections into trusted user networks.

---

## 12. Firewall Management Security

OPNsense management should only be performed from trusted systems.

The current administrative workstation is:

`10.10.20.20`

An explicit rule allows:

`10.10.20.20 → OPNsense → TCP/443`

REDNET systems should not be allowed to manage the firewall.

This prevents the offensive-security environment from becoming a trusted administrative zone.

---

## 13. Inter-Zone Routing

OPNsense performs routing between the internal networks.

Connected routes include:

`10.10.10.0/24 → REDNET`

`10.10.20.0/24 → CORPNET`

`10.10.30.0/24 → SERVERNET`

Because OPNsense sits between these networks, firewall policy can control which traffic is permitted between zones.

---

## 14. Internet Access Architecture

The final intended Internet path is:

Internal System  
↓  
Security Zone  
↓  
OPNsense  
↓  
WAN  
↓  
VirtualBox NAT  
↓  
Internet

Examples:

Kali  
↓  
REDNET  
↓  
OPNsense  
↓  
Internet

Windows  
↓  
CORPNET  
↓  
OPNsense  
↓  
Internet

Ubuntu  
↓  
SERVERNET  
↓  
OPNsense  
↓  
Internet

Direct VirtualBox NAT adapters on internal systems will eventually be removed after successful migration.

---

## 15. Security Benefits of Segmentation

Network segmentation provides several security benefits.

### Reduced Attack Surface

Systems only communicate with networks and services that are necessary.

### Controlled Offensive Testing

Kali Linux can perform approved attacks without receiving unrestricted access to the entire environment.

### Improved Monitoring

Traffic crossing network boundaries can be logged by OPNsense.

### Realistic Enterprise Architecture

The design resembles real environments where users, servers, security tools, and untrusted systems operate on separate networks.

### Better Incident Analysis

Security events can be associated with a specific zone and trust level.

---

## 16. Planned Additional Zones

As the lab develops, additional networks may be introduced.

### SOC-NET

Possible network:

`10.10.40.0/24`

Purpose:

- SIEM
- Wazuh
- Monitoring infrastructure
- Security dashboards
- Log collectors

### AD-NET

Possible network:

`10.10.50.0/24`

Purpose:

- Windows Server
- Active Directory
- DNS
- Group Policy
- Identity services

### DMZ

Possible network:

`10.10.60.0/24`

Purpose:

- Public-facing services
- Vulnerable web applications
- Web servers
- Security-testing targets

Additional zones will be introduced only when the lab resources and use cases require them.

---

## 17. Planned Monitoring

OPNsense will eventually provide telemetry for:

- Allowed connections
- Blocked connections
- Inter-zone traffic
- Internet access
- Suspicious network activity
- IDS alerts
- IPS alerts

Firewall logs may later be forwarded to:

- Wazuh
- SIEM infrastructure
- Central log collectors

This will allow offensive activity to be observed from the defensive side.

---

## 18. Planned IDS and IPS

The firewall architecture is designed to support Suricata.

Suricata may initially operate in:

`IDS Mode`

to detect suspicious traffic without automatically blocking it.

After testing and tuning, selected policies may move to:

`IPS Mode`

This will allow the lab to demonstrate:

Attack  
↓  
Network Detection  
↓  
Alert  
↓  
Analysis  
↓  
Blocking  
↓  
Investigation

---

## 19. Current Zone Status

Current firewall zones are:

| Zone | Interface | Network | Status |
|---|---|---|---|
| WAN | `em0` | `10.0.2.0/24` | 🟢 Operational |
| CORPNET | `em1` | `10.10.20.0/24` | 🟢 Operational |
| REDNET | `em2` | `10.10.10.0/24` | 🟢 Configured |
| SERVERNET | `em3` | `10.10.30.0/24` | 🟢 Configured |

Current management workstation:

`Windows 11 → 10.10.20.20`

Future offensive workstation:

`Kali Linux → 10.10.10.10`

Future Linux server:

`Ubuntu Server → 10.10.30.30`

The security zones are configured and ready for controlled VM migration.

---

## 20. Future Development

The next stages of the segmented architecture include:

- Complete CORPNET policy verification
- Build REDNET firewall rules
- Build SERVERNET firewall rules
- Migrate Kali Linux to REDNET
- Migrate Ubuntu Server to SERVERNET
- Fully migrate Windows to CORPNET
- Remove direct VirtualBox NAT from internal VMs
- Test firewall-routed Internet connectivity
- Test inter-zone access controls
- Test blocked traffic
- Enable detailed firewall logging
- Deploy Suricata
- Deploy SIEM infrastructure
- Integrate endpoint logging
- Create attack-and-detection exercises
- Add Active Directory
- Add vulnerable application targets
- Create SOC and incident-response workflows

The long-term objective is a professional offensive and defensive cybersecurity environment where attack traffic, defensive telemetry, firewall enforcement, detection, and incident response can all be practiced together.

---

## Security Disclaimer

This segmented cybersecurity environment is built exclusively for authorized education, experimentation, ethical hacking, defensive security, and incident-response training.

All offensive-security activity is restricted to systems that I own or have explicit authorization to test.

No unauthorized systems, networks, services, accounts, or organizations are targeted.
