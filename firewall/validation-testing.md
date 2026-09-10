# OPNsense Firewall Validation and Security Testing

## 1. Overview

This document records the validation and security testing performed after migrating my cybersecurity homelab from a flat VirtualBox network architecture to a segmented OPNsense-controlled environment.

The objective of the validation phase was to confirm that:

- OPNsense is the mandatory gateway for internal systems
- Direct VirtualBox NAT bypasses are removed
- Internet access works through OPNsense
- DNS resolution works through OPNsense
- Trusted management access is permitted
- Unauthorized management access is blocked
- Offensive-security traffic is controlled
- Inter-zone segmentation is enforced
- Stateful return traffic functions correctly
- Explicit firewall rules function without the default LAN allow rule

---

## 2. Final Security Zones

The completed architecture contains three protected internal zones and one WAN interface.

| Zone | Network | Gateway | Primary System |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `10.10.10.1` | Kali Linux |
| CORPNET | `10.10.20.0/24` | `10.10.20.1` | Windows 11 |
| SERVERNET | `10.10.30.0/24` | `10.10.30.1` | Ubuntu Server |
| WAN | `10.0.2.0/24` | `10.0.2.2` | OPNsense WAN |

---

## 3. Final Host Addressing

### Kali Linux

Hostname / Role:

`LAB-KALI-01`

Security Zone:

`REDNET`

IPv4 Address:

`10.10.10.10/24`

Default Gateway:

`10.10.10.1`

DNS Server:

`10.10.10.1`

Direct VirtualBox NAT:

`Disabled`

---

### Windows 11

Hostname / Role:

`LAB-WIN-01`

Security Zone:

`CORPNET`

IPv4 Address:

`10.10.20.20/24`

Default Gateway:

`10.10.20.1`

DNS Server:

`10.10.20.1`

Direct VirtualBox NAT:

`Disabled`

Original LAB-NET adapter:

`Disabled`

---

### Ubuntu Server

Hostname / Role:

`LAB-UBUNTU-01`

Security Zone:

`SERVERNET`

IPv4 Address:

`10.10.30.30/24`

Default Gateway:

`10.10.30.1`

DNS Server:

`10.10.30.1`

Direct VirtualBox NAT:

`Disabled`

---

## 4. Mandatory Firewall Routing

After migration, internal systems no longer connect directly to the Internet using their own VirtualBox NAT adapters.

The final Internet path is:

Internal VM  
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

Kali:

`10.10.10.10 → 10.10.10.1 → OPNsense → WAN → Internet`

Windows:

`10.10.20.20 → 10.10.20.1 → OPNsense → WAN → Internet`

Ubuntu:

`10.10.30.30 → 10.10.30.1 → OPNsense → WAN → Internet`

This ensures firewall policy and logging can be applied to internal traffic.

---

## 5. Outbound NAT Validation

OPNsense was configured using:

`Automatic Source NAT rule generation`

The automatically generated outbound NAT rules included:

- CORPNET
- REDNET
- SERVERNET

Internal addresses are translated to the OPNsense WAN address before traffic is sent toward the Internet.

---

## 6. Pre-Cutover NAT Testing

Before disabling the direct NAT adapters, temporary host routes were created to force traffic toward OPNsense.

A test destination of:

`1.1.1.1`

was used.

The objective was to confirm that each security zone could successfully reach the Internet through OPNsense before removing rollback connectivity.

Results:

| System | OPNsense Gateway | Internet Test |
|---|---|---|
| Kali | `10.10.10.1` | PASS |
| Windows | `10.10.20.1` | PASS |
| Ubuntu | `10.10.30.1` | PASS |

This confirmed outbound NAT functionality before final cutover.

---

## 7. Kali REDNET Final Cutover

Kali Linux was migrated completely onto REDNET.

Final address:

`10.10.10.10/24`

Final default route:

`default via 10.10.10.1`

The direct NAT adapter was disabled.

During the cutover, VirtualBox interface numbering changed after the original NAT adapter was disabled.

The REDNET adapter became:

`eth0`

The NetworkManager connection was rebound to the remaining interface using its MAC address and interface name.

After correction, Kali successfully obtained:

`10.10.10.10/24`

and the correct OPNsense default gateway.

---

## 8. Kali Internet Validation

The following tests were performed from Kali.

### Gateway Test

Command:

`ping -c 4 10.10.10.1`

Result:

`PASS`

### Internet ICMP Test

Command:

`ping -c 4 1.1.1.1`

Result:

`PASS`

### DNS Test

Command:

`nslookup opnsense.org 10.10.10.1`

Result:

`PASS`

### HTTPS Internet Test

Command:

`curl -I --connect-timeout 10 https://example.com`

Result:

`PASS`

These tests confirmed that Kali can access the Internet entirely through OPNsense.

---

## 9. Kali Firewall Management Isolation

Kali was intentionally prevented from accessing the OPNsense Web GUI.

Command:

`curl -k --connect-timeout 5 https://10.10.10.1`

Result:

`Connection timed out`

Status:

`BLOCKED AS DESIGNED`

This validated:

`BLOCK - REDNET to OPNsense Management`

Kali can use specifically authorized firewall infrastructure services while remaining unable to manage the firewall.

---

## 10. Kali to CORPNET Isolation

Kali attempted to reach the Windows management workstation.

Command:

`ping -c 4 10.10.20.20`

Result:

`100% packet loss`

Status:

`BLOCKED AS DESIGNED`

This validates REDNET-to-CORPNET segmentation.

---

## 11. Kali to Ubuntu Offensive Path

Kali was intentionally permitted to reach the Ubuntu Server in SERVERNET.

Command:

`ping -c 4 10.10.30.30`

Result:

`PASS`

This confirms the authorized path:

REDNET  
↓  
OPNsense  
↓  
SERVERNET

---

## 12. Kali SSH Enumeration Test

Kali performed an Nmap TCP connection scan against Ubuntu SSH.

Command:

`nmap -sT -Pn -p 22 10.10.30.30`

Result:

`22/tcp open ssh`

This confirmed that Kali could reach the approved SERVERNET target through OPNsense.

This provides a controlled pathway for future offensive-security exercises.

---

## 13. Ubuntu SERVERNET Final Cutover

Ubuntu Server was fully migrated to SERVERNET.

Final address:

`10.10.30.30/24`

Final gateway:

`10.10.30.1`

Final DNS server:

`10.10.30.1`

The direct VirtualBox NAT adapter was disabled.

The SERVERNET interface was pinned using its MAC address through Netplan to prevent interface renumbering.

---

## 14. Ubuntu Routing Validation

The final Ubuntu routing configuration included:

`default via 10.10.30.1`

and explicit routes for:

`10.10.10.0/24`

and:

`10.10.20.0/24`

through OPNsense.

No direct route through:

`10.0.2.2`

remained after the final cutover.

---

## 15. Ubuntu Internet Validation

Ubuntu successfully completed the following tests.

### Gateway

`ping -c 4 10.10.30.1`

Result:

`PASS`

### Internet

`ping -c 4 1.1.1.1`

Result:

`PASS`

### DNS

`nslookup opnsense.org 10.10.30.1`

Result:

`PASS`

### HTTPS

`curl -I --connect-timeout 10 https://example.com`

Result:

`PASS`

This verified full Internet connectivity through OPNsense.

---

## 16. Ubuntu to REDNET Isolation

Ubuntu attempted to initiate communication toward Kali Linux.

Destination:

`10.10.10.10`

Result:

`BLOCKED`

Status:

`PASS`

This confirms that SERVERNET systems cannot initiate unauthorized connections toward the offensive-security network.

---

## 17. Ubuntu to CORPNET Isolation

Ubuntu attempted to initiate communication toward the Windows management workstation.

Destination:

`10.10.20.20`

Result:

`BLOCKED`

Status:

`PASS`

This helps reduce lateral-movement opportunities if a server becomes compromised.

---

## 18. Windows CORPNET Final Cutover

Windows was migrated completely to CORPNET.

Final interface:

`CORP-NET`

IPv4 Address:

`10.10.20.20`

Default Gateway:

`10.10.20.1`

DNS Server:

`10.10.20.1`

The direct NAT adapter was disabled.

The original LAB-NET adapter was also disabled.

Windows therefore has no direct VirtualBox NAT bypass.

---

## 19. Windows Default Gateway Validation

PowerShell confirmed the final default route:

`0.0.0.0/0 → 10.10.20.1`

through:

`CORP-NET`

No default route through:

`10.0.2.2`

remained.

This confirmed that OPNsense became the sole IPv4 gateway for Windows.

---

## 20. Windows Internet Validation

Windows successfully completed an Internet ICMP test.

Command:

`ping 1.1.1.1`

Result:

`4 packets sent, 4 received, 0% loss`

Status:

`PASS`

HTTPS connectivity was tested using:

`curl.exe -I --connect-timeout 10 https://example.com`

Result:

`HTTP/1.1 200 OK`

Status:

`PASS`

---

## 21. Windows DNS Validation

PowerShell command:

`Resolve-DnsName opnsense.org -Server 10.10.20.1`

Result:

Successful IPv4 and IPv6 DNS records were returned.

Status:

`PASS`

This confirmed OPNsense DNS functionality from CORPNET.

---

## 22. Windows Firewall Management Access

The Windows management workstation tested OPNsense HTTPS.

Command:

`Test-NetConnection 10.10.20.1 -Port 443`

Result:

`TcpTestSucceeded : True`

Status:

`PASS`

This validates the explicit rule:

`ALLOW - Admin Workstation to OPNsense HTTPS`

---

## 23. CORPNET Gateway ICMP

Windows attempted to ping:

`10.10.20.1`

Result:

`100% packet loss`

This is expected under the current policy.

CORPNET does not currently have an explicit ICMP echo permission to the firewall.

HTTPS management, DNS, and NTP are explicitly permitted while other firewall services are restricted.

Therefore:

`Windows → OPNsense ICMP`

is:

`BLOCKED AS DESIGNED`

---

## 24. Windows to Ubuntu Administration

Windows tested SSH connectivity to Ubuntu Server.

Command:

`Test-NetConnection 10.10.30.30 -Port 22`

Final result:

`TcpTestSucceeded : True`

Source:

`10.10.20.20`

Interface:

`CORP-NET`

Destination:

`10.10.30.30`

Port:

`22`

Status:

`PASS`

This validates:

`ALLOW - Admin Workstation to SERVERNET Admin Services`

---

## 25. Windows to REDNET Isolation

Windows attempted to ping Kali Linux.

Command:

`ping 10.10.10.10`

Result:

`100% packet loss`

Status:

`BLOCKED AS DESIGNED`

This confirms that CORPNET does not receive unrestricted access to REDNET.

---

## 26. Default LAN Rule Removal

During the firewall deployment, the original OPNsense LAN allow rules were temporarily retained as a rollback mechanism.

After explicit policies were created and tested, the following rules were disabled:

`Default allow LAN to any rule`

`Default allow LAN IPv6 to any rule`

The environment was then tested again.

Critical functionality remained operational using only the custom explicit rules.

This confirmed that the lab no longer depends on OPNsense's broad default LAN trust policy.

---

## 27. Explicit CORPNET Policy Validation

After disabling the default LAN rules:

| Test | Result |
|---|---|
| Windows → OPNsense HTTPS | PASS |
| Windows → OPNsense DNS | PASS |
| Windows → Internet | PASS |
| Windows → Ubuntu SSH | PASS |
| Windows → REDNET | BLOCK |
| Windows → Unauthorized Firewall Services | BLOCK |

The CORPNET policy is therefore operating using least-privilege firewall rules.

---

## 28. Explicit REDNET Policy Validation

| Test | Result |
|---|---|
| Kali → OPNsense ICMP | PASS |
| Kali → OPNsense DNS | PASS |
| Kali → Internet | PASS |
| Kali → OPNsense HTTPS | BLOCK |
| Kali → CORPNET | BLOCK |
| Kali → Ubuntu | PASS |
| Kali → Ubuntu SSH | PASS |

The REDNET network is functional for offensive-security testing while remaining isolated from trusted administrative systems.

---

## 29. Explicit SERVERNET Policy Validation

| Test | Result |
|---|---|
| Ubuntu → OPNsense Gateway | PASS |
| Ubuntu → OPNsense DNS | PASS |
| Ubuntu → Internet | PASS |
| Ubuntu → REDNET | BLOCK |
| Ubuntu → CORPNET | BLOCK |
| Kali-initiated traffic → Ubuntu | PASS |
| Windows administrative SSH → Ubuntu | PASS |

This confirms that SERVERNET operates as a protected server zone.

---

## 30. Stateful Firewall Validation

The firewall allows established return traffic without creating unnecessary reverse-direction permissions.

Example:

Kali initiates:

`10.10.10.10 → 10.10.30.30:22`

OPNsense permits the connection through the REDNET policy.

Ubuntu's response traffic returns through the existing firewall state.

Ubuntu does not require permission to initiate arbitrary new connections into REDNET.

The same principle applies to Windows administrative connections toward SERVERNET.

---

## 31. Final Segmentation Matrix

| Source | Destination | Result |
|---|---|---|
| Windows / CORPNET | OPNsense HTTPS | ALLOW |
| Windows / CORPNET | OPNsense DNS | ALLOW |
| Windows / CORPNET | Internet | ALLOW |
| Windows / CORPNET | Ubuntu SSH | ALLOW |
| Windows / CORPNET | Kali | BLOCK |
| Kali / REDNET | Internet | ALLOW |
| Kali / REDNET | OPNsense DNS | ALLOW |
| Kali / REDNET | OPNsense HTTPS | BLOCK |
| Kali / REDNET | Windows | BLOCK |
| Kali / REDNET | Ubuntu | ALLOW |
| Ubuntu / SERVERNET | Internet | ALLOW |
| Ubuntu / SERVERNET | OPNsense DNS | ALLOW |
| Ubuntu / SERVERNET | Windows | BLOCK |
| Ubuntu / SERVERNET | Kali | BLOCK |

---

## 32. Final Architecture

The validated architecture is:

Internet  
↓  
VirtualBox NAT  
↓  
OPNsense WAN  
↓  
Firewall / NAT / Routing / Logging  
↓  
Security Zones

REDNET:

`10.10.10.0/24`

Kali:

`10.10.10.10`

CORPNET:

`10.10.20.0/24`

Windows:

`10.10.20.20`

SERVERNET:

`10.10.30.0/24`

Ubuntu:

`10.10.30.30`

All communication between internal security zones is routed through OPNsense.

---

## 33. Recovery Strategy

Recovery points were created throughout the migration process.

VirtualBox snapshots were taken:

- Before major network migrations
- Before final NAT removal
- After successful zone migration
- After firewall policy verification

OPNsense configuration backups were also exported.

Configuration backup files are stored privately and are not committed to the public GitHub repository.

---

## 34. Evidence Collection

Evidence collected during validation included:

- `ip addr`
- `ip route`
- NetworkManager configuration
- Netplan configuration
- PowerShell networking output
- Firewall rule screenshots
- OPNsense interface configuration
- OPNsense Live View logs
- DNS lookup results
- ICMP tests
- TCP connectivity tests
- Nmap results
- HTTP/HTTPS connectivity tests
- Routing-table verification

Screenshots may be added to the repository after reviewing them for sensitive information.

---

## 35. Lessons Learned

Several important lessons were demonstrated during this project.

### Interface Names Can Change

Disabling a VirtualBox adapter can cause guest operating systems to renumber network interfaces.

Interface configuration should therefore be associated with predictable device information such as MAC addresses where appropriate.

### Firewall Rule Names Do Not Control Behavior

A rule description can say "BLOCK" while the actual action is accidentally configured as "Pass."

The rule action itself must always be verified.

### Rule Ordering Matters

Specific allow rules must appear before broader block rules.

Likewise, private-network block rules must appear before general Internet access rules.

### Routing Must Match Firewall Policy

A correct firewall rule does not help if the operating system sends traffic through the wrong interface.

Static routes were required during the staged migration to ensure traffic crossed OPNsense.

### Test Before Removing Rollback Paths

Outbound NAT was validated before direct NAT interfaces were disabled.

This prevented unnecessary outages during migration.

### Default Rules Should Not Be Trusted Indefinitely

The original broad LAN allow policy was retained during migration, then disabled after explicit policies were proven to work.

---

## 36. Security Outcome

The project successfully transformed the original flat homelab into a segmented security environment.

The firewall now provides:

- Centralized routing
- Stateful firewalling
- Network segmentation
- Outbound NAT
- Controlled administrative access
- Controlled offensive-security access
- Inter-zone isolation
- DNS services
- Security logging
- A foundation for IDS/IPS
- A foundation for SIEM monitoring

The environment is now suitable for more advanced offensive and defensive cybersecurity exercises.

---

## 37. Current Status

**OPNsense Firewall:** 🟢 Operational

**CORPNET:** 🟢 Operational

**REDNET:** 🟢 Operational

**SERVERNET:** 🟢 Operational

**Kali Direct NAT Bypass:** 🔴 Disabled

**Windows Direct NAT Bypass:** 🔴 Disabled

**Ubuntu Direct NAT Bypass:** 🔴 Disabled

**Original LAB-NET:** 🔴 No longer used for the migrated systems

**Custom Firewall Policy:** 🟢 Operational

**Default LAN Allow Rules:** 🔴 Disabled

**Outbound NAT:** 🟢 Verified

**Internet Access:** 🟢 Verified

**DNS:** 🟢 Verified

**Controlled Attack Path:** 🟢 Verified

**Segmentation:** 🟢 Verified

---

## 38. Next Development Phase

The next stages of the professional cybersecurity homelab will include:

- Update the main architecture documentation
- Update the repository README
- Create the final network diagram
- Centralized log collection
- Wazuh SIEM deployment
- Windows event forwarding
- Linux log forwarding
- OPNsense firewall log forwarding
- Suricata IDS deployment
- IDS alert analysis
- IPS experimentation
- Vulnerable application targets
- Additional server workloads
- Active Directory
- SOC network
- Detection engineering
- Incident-response exercises
- Attack simulation and telemetry correlation
- Threat hunting
- Security automation

The objective is to create an environment where offensive activity can be generated, observed, detected, investigated, and documented.

---

## Security Disclaimer

This cybersecurity homelab is used exclusively for authorized cybersecurity education, ethical hacking, defensive-security training, and incident-response exercises.

All scanning, enumeration, testing, exploitation, and attack simulation are performed only against systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, services, accounts, or organizations are targeted.
