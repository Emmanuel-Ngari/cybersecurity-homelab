# OPNsense Firewall Policy Matrix

## 1. Overview

This document defines the intended firewall policy between the security zones in my cybersecurity homelab.

The purpose of the policy matrix is to clearly document:

- Which zones may communicate
- Which services are allowed
- Which traffic is blocked
- Which systems may perform administration
- Which systems may perform offensive-security testing
- Which traffic should be logged
- How least privilege is enforced

The firewall policy is designed before full VM migration so that inter-zone communication is intentional rather than unrestricted.

---

## 2. Security Zones

The current OPNsense security zones are:

| Zone | Interface | Network | Primary Role |
|---|---|---|---|
| WAN | `em0` | `10.0.2.0/24` | External / Internet |
| CORPNET | `em1` | `10.10.20.0/24` | Trusted endpoints / management |
| REDNET | `em2` | `10.10.10.0/24` | Offensive security |
| SERVERNET | `em3` | `10.10.30.0/24` | Protected servers |

---

## 3. Key Systems

Current and planned systems include:

| System | Zone | IPv4 Address | Role |
|---|---|---|---|
| OPNsense | CORPNET Gateway | `10.10.20.1` | Firewall management |
| Windows 11 | CORPNET | `10.10.20.20` | Management / endpoint |
| Kali Linux | REDNET | `10.10.10.10` | Offensive security |
| Ubuntu Server | SERVERNET | `10.10.30.30` | Linux server |

---

## 4. Security Philosophy

The firewall policy follows these principles:

- Default deny
- Least privilege
- Explicit access
- Network segmentation
- Dedicated management access
- Controlled offensive-security traffic
- Logging of important security decisions
- Separation of trusted and untrusted systems

Traffic should only be allowed when there is a specific operational or training requirement.

---

## 5. Trust Levels

The zones are assigned different trust levels.

### WAN

Trust Level:

`Untrusted`

Represents external networks and Internet connectivity.

### REDNET

Trust Level:

`Untrusted Internal`

Contains offensive-security systems such as Kali Linux.

### CORPNET

Trust Level:

`Trusted Internal`

Contains administrative and endpoint systems.

### SERVERNET

Trust Level:

`Protected Internal`

Contains servers and services that should only be accessed through approved paths.

---

## 6. High-Level Policy Matrix

| Source | Destination | Policy |
|---|---|---|
| WAN | CORPNET | Block |
| WAN | REDNET | Block |
| WAN | SERVERNET | Block |
| CORPNET | WAN / Internet | Allow |
| CORPNET | REDNET | Block |
| CORPNET | SERVERNET | Limited Allow |
| CORPNET | OPNsense | Limited Allow |
| REDNET | WAN / Internet | Controlled Allow |
| REDNET | CORPNET | Block |
| REDNET | SERVERNET | Controlled Allow |
| REDNET | OPNsense Management | Block |
| SERVERNET | WAN / Internet | Controlled Allow |
| SERVERNET | CORPNET | Block |
| SERVERNET | REDNET | Block |
| SERVERNET | OPNsense Services | Limited Allow |

---

## 7. CORPNET to OPNsense Policy

CORPNET contains the trusted management workstation.

The Windows management workstation is:

`10.10.20.20`

### Allowed

Windows management workstation:

`10.10.20.20`

may access:

`OPNsense HTTPS TCP/443`

Purpose:

- Firewall administration
- Policy management
- Monitoring
- Configuration

CORPNET systems may also access approved firewall infrastructure services such as:

- DNS
- NTP

---

## 8. Unauthorized Firewall Access

Traffic from CORPNET to other OPNsense services should be blocked unless explicitly permitted.

Examples of traffic that should not automatically be allowed include:

- Unapproved management services
- Unnecessary administrative ports
- Unused services
- Future management interfaces

The rule:

`BLOCK - Unauthorized CORPNET Access to Firewall`

provides this protection after explicit service rules are evaluated.

---

## 9. CORPNET to SERVERNET Policy

The management workstation requires limited access to protected servers.

Current approved administration ports are:

`22/TCP`

for SSH

and:

`443/TCP`

for HTTPS.

The approved source is:

`ADMIN_WORKSTATION`

The approved destination is:

`SERVERNET`

This provides controlled server administration without allowing unrestricted access to all server services.

---

## 10. CORPNET to REDNET Policy

Normal communication from CORPNET to REDNET should be blocked.

Purpose:

- Prevent unnecessary trust between management and attacker networks
- Reduce exposure to offensive-security systems
- Maintain realistic segmentation
- Prevent accidental interaction between trusted endpoints and attack infrastructure

Any future exception must be explicitly documented and justified.

---

## 11. CORPNET to Internet Policy

CORPNET requires Internet access for activities such as:

- Windows updates
- Security tool downloads
- Software installation
- Browser access
- Documentation
- Training resources

Traffic toward unauthorized private networks is evaluated before the Internet allow rule.

This prevents the final Internet-access rule from becoming an unrestricted inter-zone allow rule.

---

## 12. REDNET to OPNsense Policy

REDNET contains offensive-security systems.

Kali Linux should not be permitted unrestricted access to OPNsense management.

### Blocked

REDNET should not access:

- OPNsense Web GUI
- OPNsense SSH management
- Administrative interfaces
- Unapproved firewall services

### Limited Infrastructure Access

REDNET may later be permitted to use specific infrastructure services such as:

- DNS
- NTP

These services should be allowed explicitly rather than through a broad firewall-management rule.

---

## 13. REDNET to CORPNET Policy

REDNET to CORPNET is blocked by default.

This is one of the most important segmentation rules in the laboratory.

Kali Linux should not automatically be able to:

- Scan Windows endpoints
- Enumerate CORPNET
- Access administrative systems
- Reach management services
- Attack trusted endpoints

If a Windows attack simulation is intentionally required in the future, a temporary and tightly scoped firewall rule can be created for that exercise.

The rule should then be disabled or removed after the exercise.

---

## 14. REDNET to SERVERNET Policy

REDNET requires controlled access to SERVERNET for authorized security testing.

This is the main attack path in the initial laboratory design.

Example flow:

Kali Linux  
↓  
REDNET  
↓  
OPNsense  
↓  
SERVERNET  
↓  
Ubuntu Server

Approved activities may include:

- Host discovery
- Port scanning
- Service enumeration
- Vulnerability assessment
- SSH testing
- Web application testing
- Controlled exploitation
- Privilege-escalation exercises

This access is intentional because SERVERNET is the initial controlled target environment.

---

## 15. REDNET to Internet Policy

REDNET requires Internet access for:

- Kali updates
- Package installation
- Security tool installation
- Repository access
- Documentation
- Authorized cybersecurity resources

Internet access should pass through OPNsense rather than using a direct VirtualBox NAT adapter.

This ensures that REDNET traffic can be logged and controlled.

---

## 16. SERVERNET Policy

SERVERNET contains protected server systems.

Ubuntu Server should not be treated as a trusted workstation.

### Allowed Outbound Activities

SERVERNET may require controlled access for:

- Operating system updates
- Package installation
- DNS
- NTP
- Approved repositories

### Blocked Internal Activities

SERVERNET should not initiate unrestricted connections to:

- CORPNET
- REDNET
- Firewall management services

This reduces lateral movement opportunities if a server is compromised.

---

## 17. Stateful Firewall Behavior

OPNsense is a stateful firewall.

When an approved connection is initiated, return traffic associated with the established connection can be permitted automatically through the firewall state table.

For example:

Windows:

`10.10.20.20`

initiates SSH to Ubuntu:

`10.10.30.30:22`

If the CORPNET rule permits the connection, Ubuntu's reply traffic can return through the existing firewall state.

A separate broad SERVERNET-to-CORPNET allow rule is not required for that reply traffic.

This supports tighter segmentation.

---

## 18. Current CORPNET Rules

The current custom CORPNET policy includes:

1. `ALLOW - Admin Workstation to OPNsense HTTPS`
2. `ALLOW - CORPNET to OPNsense DNS`
3. `ALLOW - CORPNET to OPNsense NTP`
4. `BLOCK - Unauthorized CORPNET Access to Firewall`
5. `ALLOW - Admin Workstation to SERVERNET Admin Services`
6. `BLOCK - CORPNET to Unauthorized Private Networks`
7. `ALLOW - CORPNET to Internet`

The original default LAN allow rules are temporarily retained during migration and testing.

They will be disabled only after the custom policy has been fully validated.

---

## 19. Planned REDNET Rules

The planned REDNET rule structure is:

1. `ALLOW - REDNET to OPNsense DNS`
2. `ALLOW - REDNET to OPNsense NTP`
3. `BLOCK - REDNET to OPNsense Management`
4. `BLOCK - REDNET to CORPNET`
5. `ALLOW - REDNET to SERVERNET Lab Testing`
6. `BLOCK - REDNET to Unauthorized Private Networks`
7. `ALLOW - REDNET to Internet`

Important firewall decisions should have logging enabled.

This will allow attack traffic and blocked attempts to be reviewed later.

---

## 20. Planned SERVERNET Rules

The planned SERVERNET rule structure is:

1. `ALLOW - SERVERNET to OPNsense DNS`
2. `ALLOW - SERVERNET to OPNsense NTP`
3. `BLOCK - SERVERNET to OPNsense Management`
4. `BLOCK - SERVERNET to CORPNET`
5. `BLOCK - SERVERNET to REDNET`
6. `BLOCK - SERVERNET to Unauthorized Private Networks`
7. `ALLOW - SERVERNET to Internet`

Additional service-specific rules may be added as new server roles are deployed.

---

## 21. Logging Strategy

Logging will be enabled for important firewall rules, especially:

- Management access
- Inter-zone blocks
- REDNET attack traffic
- Unauthorized firewall access
- Server segmentation blocks
- Security testing paths

The logs will later support:

- Firewall analysis
- SOC exercises
- SIEM integration
- Detection engineering
- Incident investigation
- Attack timeline reconstruction

---

## 22. Temporary Lab Exceptions

Cybersecurity exercises may occasionally require temporary access that is normally blocked.

Examples include:

- REDNET → CORPNET Windows attack simulation
- Special server testing ports
- Vulnerability scanning
- Controlled Active Directory attacks
- Temporary administrative access

Temporary rules should:

1. Have a clear description.
2. Be narrowly scoped.
3. Enable logging.
4. Exist only for the required exercise.
5. Be disabled or removed afterward.
6. Be documented.

This prevents temporary testing rules from becoming permanent security weaknesses.

---

## 23. Future Policy Expansion

Future security zones may include:

`SOC-NET → 10.10.40.0/24`

`AD-NET → 10.10.50.0/24`

`DMZ → 10.10.60.0/24`

Each new zone will receive its own firewall policy based on:

- Trust level
- Business role
- Required services
- Security monitoring requirements
- Attack-simulation requirements

No new network should automatically inherit unrestricted access.

---

## 24. IDS/IPS Integration

The firewall policy will later be combined with Suricata IDS/IPS.

This will allow the environment to demonstrate:

Kali Attack  
↓  
Firewall Transit  
↓  
Suricata Inspection  
↓  
Alert Generation  
↓  
Firewall Logging  
↓  
SIEM Collection  
↓  
Blue Team Investigation

Firewall policy determines whether traffic is permitted.

IDS/IPS provides additional detection and prevention capability.

---

## 25. SIEM Integration

OPNsense logs will eventually be forwarded to centralized security monitoring infrastructure.

Planned SIEM platform:

`Wazuh`

This will allow firewall events to be correlated with:

- Windows logs
- Linux logs
- Authentication events
- IDS alerts
- Endpoint telemetry
- Attack activity

This creates a complete offensive-versus-defensive training environment.

---

## 26. Target End-State Policy

The final design is intended to provide:

CORPNET  
→ trusted administration and endpoint activity

REDNET  
→ controlled offensive-security operations

SERVERNET  
→ protected servers and attack targets

WAN  
→ external connectivity

OPNsense  
→ centralized routing, filtering, logging, and security enforcement

The goal is not simply to make every host communicate.

The goal is to control exactly:

- Who can communicate
- With which network
- Using which service
- For what purpose
- With what logging
- Under what security policy

---

## 27. Current Status

**Policy Design Status:** 🟢 Documented

**CORPNET Policy:** 🟢 Initial rules created

**REDNET Policy:** 🟡 Planned

**SERVERNET Policy:** 🟡 Planned

**Kali Migration:** 🟡 Pending

**Ubuntu Migration:** 🟡 Pending

**Full Windows Migration:** 🟡 Pending

The next technical step is to create and test the REDNET and SERVERNET policies before completing VM migration.

---

## Security Disclaimer

This firewall policy is designed exclusively for a controlled cybersecurity homelab.

All offensive-security traffic is generated only against systems that I own or have explicit authorization to test.

Firewall exceptions used during attack simulations are limited to the laboratory and are not intended for unauthorized access to external systems or networks.
