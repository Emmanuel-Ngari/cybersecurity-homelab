# VirtualBox Networking Architecture

## 1. Overview

This document describes the VirtualBox networking architecture used in my cybersecurity homelab.

The environment originally used a simple two-adapter design:

- VirtualBox NAT for Internet access
- A shared internal laboratory network for VM-to-VM communication

As the homelab developed, this flat architecture was replaced with a segmented design using OPNsense as the central firewall, router, NAT gateway, DNS service, and security-policy enforcement point.

The current architecture uses separate VirtualBox Internal Networks for:

- CORPNET
- REDNET
- SERVERNET

Only the OPNsense firewall has direct access to VirtualBox NAT.

All internal systems must reach external networks through OPNsense.

---

## 2. Virtualization Platform

The laboratory is hosted using:

`Oracle VirtualBox`

VirtualBox provides:

- Virtual machines
- Virtual network adapters
- NAT connectivity
- Internal networks
- Network isolation
- Snapshots
- Virtual storage
- Controlled laboratory environments

VirtualBox networking is used to simulate separate enterprise-style security zones.

---

## 3. Original Networking Design

The original laboratory contained:

- Kali Linux
- Windows 11 Enterprise
- Ubuntu Server

Each VM had:

Adapter 1:

`NAT`

Purpose:

Direct Internet access through VirtualBox.

Adapter 2:

`LAB-NET`

Purpose:

Communication between laboratory systems.

The original LAB-NET network was:

`192.168.56.0/24`

Addresses included:

Kali:

`192.168.56.10`

Windows:

`192.168.56.20`

Ubuntu:

`192.168.56.30`

---

## 4. Limitations of the Original Design

The flat architecture was useful for learning basic networking but had several limitations.

### No Central Firewall

Traffic between Kali, Windows, and Ubuntu could travel directly across LAB-NET.

### Limited Segmentation

Offensive, defensive, and server systems existed on the same network.

### Direct Internet Bypass

Each VM had its own VirtualBox NAT adapter.

This meant Internet traffic did not need to pass through a security gateway.

### Limited Monitoring

There was no central point where inter-zone traffic could be inspected or logged.

### Reduced Enterprise Realism

Real environments commonly separate:

- Users
- Servers
- Security systems
- Management networks
- Untrusted systems

The original topology did not provide this separation.

---

## 5. Redesigned Architecture

The networking design was upgraded by deploying:

`LAB-FW-01`

running:

`OPNsense`

OPNsense now acts as the central:

- Firewall
- Router
- NAT gateway
- DNS service
- NTP service
- Security-policy enforcement point
- Logging platform

The architecture changed from:

VM → VirtualBox NAT → Internet

to:

VM → Security Zone → OPNsense → VirtualBox NAT → Internet

---

## 6. Final VirtualBox Networks

The current design uses the following networks:

| VirtualBox Network | Type | Purpose |
|---|---|---|
| NAT | VirtualBox NAT | OPNsense WAN Internet access |
| `CORP-NET` | Internal Network | Trusted endpoint and management systems |
| `RED-NET` | Internal Network | Offensive-security systems |
| `SERVER-NET` | Internal Network | Protected server systems |

The former:

`LAB-NET`

network has been retired from the active architecture.

---

## 7. High-Level VirtualBox Topology

The current topology is:

Internet
|
v
VirtualBox NAT
|
v
LAB-FW-01
OPNsense
|
+----------------------+----------------------+----------------------+
|                      |                      |
v                      v                      v
RED-NET               CORP-NET               SERVER-NET
10.10.10.0/24         10.10.20.0/24          10.10.30.0/24
|                      |                      |
v                      v                      v
LAB-KALI-01           LAB-WIN-01             LAB-UBUNTU-01
10.10.10.10           10.10.20.20            10.10.30.30

All inter-zone traffic passes through OPNsense.

---

## 8. OPNsense VirtualBox Adapters

LAB-FW-01 uses four VirtualBox network adapters.

### Adapter 1

Attached To:

`NAT`

OPNsense Interface:

`em0`

Role:

`WAN`

Address:

`10.0.2.15/24`

Gateway:

`10.0.2.2`

Purpose:

Internet connectivity.

---

### Adapter 2

Attached To:

`Internal Network`

Network Name:

`CORP-NET`

OPNsense Interface:

`em1`

Role:

`CORPNET`

Address:

`10.10.20.1/24`

Purpose:

Trusted management and endpoint network.

---

### Adapter 3

Attached To:

`Internal Network`

Network Name:

`RED-NET`

OPNsense Interface:

`em2`

Role:

`REDNET`

Address:

`10.10.10.1/24`

Purpose:

Offensive-security network.

---

### Adapter 4

Attached To:

`Internal Network`

Network Name:

`SERVER-NET`

OPNsense Interface:

`em3`

Role:

`SERVERNET`

Address:

`10.10.30.1/24`

Purpose:

Protected server network.

---

## 9. OPNsense Interface Mapping

The final firewall mapping is:

| VirtualBox Adapter | OPNsense Device | Zone | Address |
|---|---|---|---|
| Adapter 1 | `em0` | WAN | `10.0.2.15/24` |
| Adapter 2 | `em1` | CORPNET | `10.10.20.1/24` |
| Adapter 3 | `em2` | REDNET | `10.10.10.1/24` |
| Adapter 4 | `em3` | SERVERNET | `10.10.30.1/24` |

The interface assignments were verified using MAC addresses before final configuration.

---

## 10. OPNsense Interface MAC Verification

The firewall interfaces were identified using their MAC addresses.

| Interface | MAC Address | Role |
|---|---|---|
| `em0` | `08:00:27:3B:19:9B` | WAN |
| `em1` | `08:00:27:04:74:7C` | CORPNET |
| `em2` | `08:00:27:6C:10:29` | REDNET |
| `em3` | `08:00:27:93:06:8C` | SERVERNET |

This reduced the risk of assigning the wrong VirtualBox adapter to a security zone.

---

## 11. Kali Linux Networking

Kali Linux is assigned to:

`RED-NET`

Final IPv4 configuration:

`10.10.10.10/24`

Default Gateway:

`10.10.10.1`

DNS:

`10.10.10.1`

Primary role:

`Offensive Security Workstation`

Kali's original direct NAT adapter is disabled.

Its active network path is:

Kali  
↓  
RED-NET  
↓  
OPNsense  
↓  
WAN  
↓  
Internet

---

## 12. Kali Interface Renumbering

During the final Kali migration, the original NAT adapter was disabled.

After this occurred, the remaining RED-NET interface was renumbered by the guest operating system.

The interface that had previously been used as:

`eth1`

became:

`eth0`

The NetworkManager profile was therefore rebound to the active RED-NET interface.

The final active Kali interface is:

`eth0`

IPv4:

`10.10.10.10/24`

Default Gateway:

`10.10.10.1`

---

## 13. Kali NetworkManager Configuration

The RED-NET NetworkManager profile is configured with:

IPv4 Address:

`10.10.10.10/24`

Gateway:

`10.10.10.1`

DNS:

`10.10.10.1`

Default Route:

Enabled

Interface:

`eth0`

The profile was also configured for automatic connection after reboot.

This ensures Kali reconnects to REDNET automatically.

---

## 14. Windows Networking

Windows 11 Enterprise is assigned to:

`CORP-NET`

Final IPv4 Address:

`10.10.20.20/24`

Default Gateway:

`10.10.20.1`

DNS:

`10.10.20.1`

Active Interface Name:

`CORP-NET`

Primary Role:

`Management / Defensive Endpoint`

Windows currently has only CORP-NET active for laboratory networking.

---

## 15. Windows Direct NAT Removal

Windows originally had a direct VirtualBox NAT adapter.

During final migration, CORP-NET was configured as the preferred route through OPNsense.

After OPNsense routing and Internet access were verified, the direct NAT adapter was disabled.

The old LAB-NET adapter was also disabled.

The final Windows default route is:

`0.0.0.0/0 → 10.10.20.1`

through:

`CORP-NET`

There is no active default route through:

`10.0.2.2`

---

## 16. Ubuntu Server Networking

Ubuntu Server is assigned to:

`SERVER-NET`

Final IPv4 Address:

`10.10.30.30/24`

Default Gateway:

`10.10.30.1`

DNS:

`10.10.30.1`

Active Interface:

`enp0s8`

Primary Role:

`Protected Linux Server`

Ubuntu's original direct VirtualBox NAT adapter is disabled.

---

## 17. Ubuntu Interface Persistence

Before disabling Ubuntu's original NAT adapter, the SERVERNET interface was associated with its MAC address.

SERVERNET MAC:

`08:00:27:07:1D:91`

Netplan uses:

`match:`

with the interface MAC address.

It also uses:

`set-name: enp0s8`

This prevents unexpected interface naming changes from breaking the SERVERNET configuration.

---

## 18. Ubuntu Final Routing

Ubuntu's final default route is:

`default via 10.10.30.1 dev enp0s8`

Additional internal routes include:

`10.10.10.0/24 via 10.10.30.1`

`10.10.20.0/24 via 10.10.30.1`

The directly connected network is:

`10.10.30.0/24`

There is no active default route through the original VirtualBox NAT gateway.

---

## 19. Direct NAT Bypass Status

The final architecture intentionally prevents internal VMs from bypassing OPNsense.

| System | Direct VirtualBox NAT |
|---|---|
| OPNsense | Enabled |
| Kali Linux | Disabled |
| Windows 11 | Disabled |
| Ubuntu Server | Disabled |

Only:

`LAB-FW-01`

has direct VirtualBox NAT connectivity.

This makes OPNsense the mandatory security gateway.

---

## 20. Final Internet Paths

### Kali

`10.10.10.10`

↓

`10.10.10.1`

↓

`OPNsense`

↓

`WAN`

↓

`VirtualBox NAT`

↓

`Internet`

---

### Windows

`10.10.20.20`

↓

`10.10.20.1`

↓

`OPNsense`

↓

`WAN`

↓

`VirtualBox NAT`

↓

`Internet`

---

### Ubuntu

`10.10.30.30`

↓

`10.10.30.1`

↓

`OPNsense`

↓

`WAN`

↓

`VirtualBox NAT`

↓

`Internet`

---

## 21. Outbound NAT

OPNsense uses:

`Automatic Source NAT rule generation`

Automatically generated rules include:

- REDNET
- CORPNET
- SERVERNET

Traffic from the internal networks is translated to the OPNsense WAN address before leaving the firewall.

This allows Internet access without giving the internal VMs their own direct NAT adapters.

---

## 22. Routing Between Security Zones

The internal networks are directly connected to OPNsense.

Routes include:

`10.10.10.0/24 → REDNET`

`10.10.20.0/24 → CORPNET`

`10.10.30.0/24 → SERVERNET`

Traffic between these networks must therefore pass through OPNsense.

This allows firewall rules to control inter-zone communication.

---

## 23. REDNET Routing

Kali uses:

`10.10.10.1`

as its default gateway.

Traffic to:

- CORPNET
- SERVERNET
- Internet

therefore reaches OPNsense first.

Firewall policy decides whether the traffic is permitted.

---

## 24. CORPNET Routing

Windows uses:

`10.10.20.1`

as its sole IPv4 default gateway.

Traffic destined for:

- SERVERNET
- REDNET
- Internet

is routed through OPNsense.

This allows Windows administrative access to be controlled centrally.

---

## 25. SERVERNET Routing

Ubuntu uses:

`10.10.30.1`

as its default gateway.

Traffic attempting to leave SERVERNET therefore crosses OPNsense.

This is important because SERVERNET is intentionally prevented from initiating unrestricted connections toward CORPNET and REDNET.

---

## 26. Security-Zone Isolation

VirtualBox Internal Networks provide Layer 2 separation.

The three internal networks are:

`RED-NET`

`CORP-NET`

`SERVER-NET`

A VM connected only to RED-NET cannot directly communicate at Layer 2 with a VM connected only to CORP-NET.

Any Layer 3 communication between them must be routed through OPNsense.

This provides both:

- VirtualBox network isolation
- OPNsense firewall enforcement

---

## 27. REDNET Security Role

RED-NET contains systems used for offensive-security testing.

Current system:

`LAB-KALI-01`

Network:

`10.10.10.0/24`

The network is treated as an untrusted internal security zone.

REDNET cannot directly access CORPNET.

Firewall management access from REDNET is blocked.

Controlled access to approved SERVERNET targets is permitted.

---

## 28. CORPNET Security Role

CORP-NET represents the trusted endpoint and management environment.

Current system:

`LAB-WIN-01`

Network:

`10.10.20.0/24`

Windows uses CORPNET to:

- Manage OPNsense
- Reach approved SERVERNET administration services
- Access the Internet
- Perform defensive-security exercises

CORPNET does not receive unrestricted access to REDNET.

---

## 29. SERVERNET Security Role

SERVER-NET represents the protected server environment.

Current system:

`LAB-UBUNTU-01`

Network:

`10.10.30.0/24`

SERVERNET is designed for:

- Linux servers
- Internal services
- Web applications
- Security targets
- Protected workloads

Systems on SERVERNET cannot initiate unrestricted communication toward CORPNET or REDNET.

---

## 30. Validated Network Paths

The following network paths were validated.

### Windows to OPNsense

`10.10.20.20 → 10.10.20.1:443`

Result:

`ALLOW`

---

### Windows to Ubuntu

`10.10.20.20 → 10.10.30.30:22`

Result:

`ALLOW`

---

### Windows to Kali

`10.10.20.20 → 10.10.10.10`

Result:

`BLOCK`

---

### Kali to Ubuntu

`10.10.10.10 → 10.10.30.30`

Result:

`ALLOW`

---

### Kali to Windows

`10.10.10.10 → 10.10.20.20`

Result:

`BLOCK`

---

### Kali to OPNsense HTTPS

`10.10.10.10 → 10.10.10.1:443`

Result:

`BLOCK`

---

### Ubuntu to Windows

`10.10.30.30 → 10.10.20.20`

Result:

`BLOCK`

---

### Ubuntu to Kali

`10.10.30.30 → 10.10.10.10`

Result:

`BLOCK`

---

## 31. Internet Connectivity Validation

Internet access was tested from all three internal systems after disabling their direct NAT adapters.

### Kali

Tests included:

`ping 1.1.1.1`

`nslookup opnsense.org 10.10.10.1`

`curl -I https://example.com`

Result:

`PASS`

---

### Windows

Tests included:

`ping 1.1.1.1`

`Resolve-DnsName opnsense.org -Server 10.10.20.1`

`curl.exe -I https://example.com`

Result:

`PASS`

---

### Ubuntu

Tests included:

`ping 1.1.1.1`

`nslookup opnsense.org 10.10.30.1`

`curl -I https://example.com`

Result:

`PASS`

---

## 32. DNS Architecture

Each internal system uses its local OPNsense zone address as DNS.

Kali:

`10.10.10.1`

Windows:

`10.10.20.1`

Ubuntu:

`10.10.30.1`

This provides centrally controlled DNS access through the firewall.

---

## 33. Firewall Management Network

OPNsense management is performed from CORPNET.

Management workstation:

`10.10.20.20`

Firewall management address:

`10.10.20.1`

Protocol:

`HTTPS`

Port:

`443`

The REDNET attacker workstation cannot access the OPNsense Web GUI.

---

## 34. Legacy LAB-NET

The old network:

`LAB-NET`

with subnet:

`192.168.56.0/24`

is no longer part of the active architecture.

Its purpose was originally to allow direct communication between:

- Kali
- Windows
- Ubuntu

It was retired after the segmented OPNsense architecture became operational.

The previous addresses remain useful as historical documentation of the lab's development.

---

## 35. Migration Strategy

The migration was performed gradually to reduce the risk of losing connectivity.

The process included:

1. Deploy OPNsense.
2. Configure four firewall interfaces.
3. Create security zones.
4. Configure firewall aliases.
5. Build CORPNET firewall policy.
6. Build REDNET firewall policy.
7. Build SERVERNET firewall policy.
8. Keep direct NAT adapters temporarily.
9. Move Kali to REDNET.
10. Move Ubuntu to SERVERNET.
11. Configure Windows on CORPNET.
12. Validate inter-zone traffic.
13. Validate OPNsense outbound NAT.
14. Force test traffic through OPNsense.
15. Create pre-cutover snapshots.
16. Change default gateways to OPNsense.
17. Disable direct NAT adapters one VM at a time.
18. Reboot and verify connectivity.
19. Disable old LAB-NET connectivity.
20. Disable OPNsense default LAN allow rules.
21. Validate explicit firewall policies.
22. Create post-cutover snapshots and backups.

This staged migration reduced the risk of losing all management access simultaneously.

---

## 36. Snapshot Strategy

VirtualBox snapshots were used throughout the network migration.

Snapshots were created:

- Before major networking changes
- Before security-zone migration
- Before disabling NAT adapters
- After successful final cutovers
- After firewall policy validation

Examples include:

`Kali - Pre OPNsense REDNET Migration`

`Kali - REDNET Policy Verified Pre Final Cutover`

`Kali - REDNET Final Cutover Verified`

`Ubuntu - Pre OPNsense SERVERNET Migration`

`Ubuntu - SERVERNET Policy Verified Pre Final Cutover`

`Ubuntu - SERVERNET Final Cutover Verified`

`Windows - CORPNET Policy Verified Pre Final Cutover`

`Windows - CORPNET Final Cutover Verified`

These snapshots provide recovery points if future networking changes cause problems.

---

## 37. OPNsense Recovery

OPNsense configuration backups are exported after important configuration milestones.

These backups include firewall and network configuration.

The XML configuration files are stored privately and are not committed to the public GitHub repository.

VirtualBox snapshots provide an additional recovery mechanism.

---

## 38. Current Network Status

| Component | Status |
|---|---|
| VirtualBox NAT | 🟢 OPNsense WAN only |
| OPNsense WAN | 🟢 Operational |
| CORP-NET | 🟢 Operational |
| RED-NET | 🟢 Operational |
| SERVER-NET | 🟢 Operational |
| LAB-NET | 🔴 Retired |
| Kali Direct NAT | 🔴 Disabled |
| Windows Direct NAT | 🔴 Disabled |
| Ubuntu Direct NAT | 🔴 Disabled |
| Inter-Zone Routing | 🟢 Operational |
| Outbound NAT | 🟢 Operational |
| DNS | 🟢 Operational |
| Internet Access | 🟢 Operational |
| Firewall Segmentation | 🟢 Verified |

---

## 39. Future VirtualBox Networks

As the laboratory expands, additional Internal Networks may be introduced.

### SOC-NET

Possible subnet:

`10.10.40.0/24`

Potential systems:

- Wazuh
- SIEM
- Log collection
- Monitoring systems

---

### AD-NET

Possible subnet:

`10.10.50.0/24`

Potential systems:

- Windows Server
- Active Directory
- Domain Controller
- Domain-joined clients

---

### DMZ-NET

Possible subnet:

`10.10.60.0/24`

Potential systems:

- Web servers
- Vulnerable applications
- Public-facing test services

Each network will be connected to OPNsense and protected by explicit firewall policy.

---

## 40. Planned Monitoring Architecture

Future traffic monitoring may include:

Kali Attack Traffic
|
v
RED-NET
|
v
OPNsense
|
+---- Firewall Logs
|
+---- Suricata IDS/IPS
|
v
SERVER-NET Target
|
v
Security Telemetry
|
v
SOC-NET
|
v
Wazuh / SIEM

This will allow attack activity to be generated on REDNET and observed by defensive systems.

---

## 41. Network Security Benefits

The current VirtualBox architecture provides several important benefits.

### Isolation

Offensive-security systems are separated from management systems.

### Centralized Policy

OPNsense controls communication between networks.

### Visibility

Traffic crossing zones can be logged.

### Realistic Routing

Internal systems use dedicated gateways rather than communicating on one flat subnet.

### Controlled Attack Paths

Kali can reach specifically authorized targets without receiving unrestricted access to the entire lab.

### Reduced Bypass Risk

Internal VMs cannot bypass OPNsense using direct NAT adapters.

### Scalability

Additional enterprise-style security zones can be added later.

---

## 42. Final Networking Milestone

The VirtualBox environment has evolved from:

Flat VM Network

↓

Shared LAB-NET

↓

Multiple Direct NAT Connections

↓

OPNsense Deployment

↓

Dedicated Internal Networks

↓

Security-Zone Segmentation

↓

Mandatory Firewall Routing

↓

Explicit Firewall Policy

↓

Validated Offensive and Defensive Network Paths

The networking foundation is now ready for more advanced cybersecurity infrastructure.

---

## 43. Next Development Phase

The next networking and security improvements will include:

- Professional network diagram
- SOC-NET deployment
- Wazuh SIEM
- Centralized logging
- OPNsense log forwarding
- Windows telemetry
- Linux telemetry
- Sysmon
- Suricata IDS
- Suricata IPS experimentation
- Active Directory
- DMZ architecture
- Vulnerable application targets
- Detection engineering
- Incident-response exercises
- Threat hunting
- Attack simulation
- Network packet analysis

The current segmented VirtualBox architecture provides the foundation for these capabilities.

---

## Security Disclaimer

This VirtualBox network architecture is used exclusively for authorized cybersecurity education, ethical hacking, defensive-security training, and incident-response exercises.

All offensive-security traffic is generated only against systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, services, applications, accounts, or organizations are targeted.
