# OPNsense Firewall Rules

## 1. Overview

This document records the firewall rules implemented on LAB-FW-01, the OPNsense firewall protecting and routing my cybersecurity homelab.

The firewall policy is designed around:

- Network segmentation
- Least privilege
- Default-deny security
- Controlled administrative access
- Controlled offensive-security testing
- Firewall logging
- Separation of attacker, endpoint, and server networks

The current security zones are:

| Zone | Network | Firewall Interface | Purpose |
|---|---|---|---|
| WAN | `10.0.2.0/24` | `em0` | Internet uplink |
| CORPNET | `10.10.20.0/24` | `em1` | Trusted endpoint / management |
| REDNET | `10.10.10.0/24` | `em2` | Offensive-security network |
| SERVERNET | `10.10.30.0/24` | `em3` | Protected server network |

---

## 2. Key Hosts

Important hosts referenced by the firewall policy include:

| Host | Address | Zone | Role |
|---|---|---|---|
| OPNsense | `10.10.20.1` | CORPNET | Firewall / Router |
| Windows 11 | `10.10.20.20` | CORPNET | Management workstation |
| Kali Linux | `10.10.10.10` | REDNET | Offensive-security workstation |
| Ubuntu Server | `10.10.30.30` | SERVERNET | Protected Linux server |

---

## 3. Firewall Aliases

Aliases were created to improve readability and simplify administration.

### ADMIN_WORKSTATION

Type:

`Host(s)`

Address:

`10.10.20.20`

Purpose:

Identifies the trusted Windows management workstation.

### KALI_ATTACKER

Type:

`Host(s)`

Address:

`10.10.10.10`

Purpose:

Identifies the Kali Linux offensive-security workstation.

### UBUNTU_SERVER

Type:

`Host(s)`

Address:

`10.10.30.30`

Purpose:

Identifies the Ubuntu Server target.

### LAB_INTERNAL_NETS

Type:

`Network(s)`

Networks:

`10.10.10.0/24`

`10.10.20.0/24`

`10.10.30.0/24`

### PRIVATE_NETS

Type:

`Network(s)`

Networks:

`10.0.0.0/8`

`172.16.0.0/12`

`192.168.0.0/16`

### SERVER_ADMIN_PORTS

Type:

`Port(s)`

Ports:

`22`

`443`

Purpose:

Approved SSH and HTTPS server administration.

### DNS_PORT

Port:

`53`

### NTP_PORT

Port:

`123`

---

## 4. Firewall Rule Processing

Rules are applied to traffic entering each security-zone interface.

The policy uses ordered rules so that specific permissions and restrictions are evaluated before broader Internet-access rules.

Important security decisions are logged.

The general model is:

Specific Allow  
↓  
Specific Block  
↓  
Private Network Restrictions  
↓  
Internet Allow  
↓  
Default Deny

---

## 5. CORPNET Policy

CORPNET contains trusted endpoint and management systems.

Network:

`10.10.20.0/24`

Primary management workstation:

`10.10.20.20`

The CORPNET rule order is:

1. `ALLOW - Admin Workstation to OPNsense HTTPS`
2. `ALLOW - CORPNET to OPNsense DNS`
3. `ALLOW - CORPNET to OPNsense NTP`
4. `BLOCK - Unauthorized CORPNET Access to Firewall`
5. `ALLOW - Admin Workstation to SERVERNET Admin Services`
6. `BLOCK - CORPNET to Unauthorized Private Networks`
7. `ALLOW - CORPNET to Internet`

Legacy default LAN rules remain temporarily during migration and will be removed after the custom policy is fully validated.

---

## 6. OPNsense Management Rule

Rule:

`ALLOW - Admin Workstation to OPNsense HTTPS`

Source:

`ADMIN_WORKSTATION`

Destination:

`This Firewall`

Protocol:

`TCP`

Destination Port:

`443`

Purpose:

Restricts firewall Web GUI administration to the trusted Windows workstation.

Connectivity was tested using:

`Test-NetConnection 10.10.20.1 -Port 443`

Result:

`TcpTestSucceeded : True`

This confirms that the explicit firewall-management rule works correctly.

---

## 7. CORPNET DNS and NTP

CORPNET systems are explicitly permitted to use OPNsense infrastructure services.

DNS rule:

`ALLOW - CORPNET to OPNsense DNS`

Protocol:

`TCP/UDP`

Destination:

`This Firewall`

Port:

`53`

NTP rule:

`ALLOW - CORPNET to OPNsense NTP`

Protocol:

`UDP`

Destination:

`This Firewall`

Port:

`123`

DNS functionality was tested from Windows against the OPNsense firewall.

---

## 8. CORPNET Firewall Protection

Rule:

`BLOCK - Unauthorized CORPNET Access to Firewall`

Source:

`CORPNET net`

Destination:

`This Firewall`

Protocol:

`Any`

Purpose:

Blocks CORPNET clients from reaching firewall services that were not explicitly permitted by earlier rules.

This limits the firewall attack surface even inside the trusted network.

---

## 9. CORPNET to SERVERNET Administration

Rule:

`ALLOW - Admin Workstation to SERVERNET Admin Services`

Source:

`ADMIN_WORKSTATION`

Destination:

`SERVERNET net`

Approved ports:

`22/TCP`

`443/TCP`

Purpose:

Allows the trusted Windows management workstation to administer protected servers using approved management protocols.

Other CORPNET-to-SERVERNET traffic is not automatically trusted.

---

## 10. CORPNET Private-Network Protection

Rule:

`BLOCK - CORPNET to Unauthorized Private Networks`

Source:

`CORPNET net`

Destination:

`PRIVATE_NETS`

Purpose:

Prevents CORPNET from reaching private networks unless a specific earlier firewall rule permits the traffic.

This includes protection against unnecessary communication with:

- REDNET
- SERVERNET services not explicitly approved
- Other current or future private laboratory networks

---

## 11. CORPNET Internet Access

Rule:

`ALLOW - CORPNET to Internet`

Source:

`CORPNET net`

Destination:

`Any`

The rule is placed after private-network restrictions.

Therefore, it provides outbound access without functioning as an unrestricted inter-zone allow rule.

---

## 12. REDNET Policy

REDNET contains offensive-security systems.

Network:

`10.10.10.0/24`

Planned Kali workstation:

`10.10.10.10`

The implemented REDNET rule order is:

1. `ALLOW - REDNET to OPNsense DNS`
2. `ALLOW - REDNET to OPNsense NTP`
3. `ALLOW - Kali to OPNsense ICMP`
4. `BLOCK - REDNET to OPNsense Management`
5. `BLOCK - REDNET to CORPNET`
6. `ALLOW - Kali to Ubuntu Lab Testing`
7. `BLOCK - REDNET to Unauthorized Private Networks`
8. `ALLOW - REDNET to Internet`

---

## 13. REDNET DNS

Rule:

`ALLOW - REDNET to OPNsense DNS`

Protocol:

`TCP/UDP`

Source:

`REDNET net`

Destination:

`This Firewall`

Port:

`53`

Purpose:

Allows Kali and future REDNET systems to use the firewall's approved DNS service.

---

## 14. REDNET NTP

Rule:

`ALLOW - REDNET to OPNsense NTP`

Protocol:

`UDP`

Source:

`REDNET net`

Destination:

`This Firewall`

Port:

`123`

Purpose:

Provides controlled time synchronization without providing unrestricted firewall access.

---

## 15. REDNET Gateway ICMP

Rule:

`ALLOW - Kali to OPNsense ICMP`

Source:

`KALI_ATTACKER`

Destination:

`This Firewall`

Protocol:

`ICMP`

Purpose:

Allows controlled troubleshooting and reachability testing between Kali and the REDNET gateway.

This does not grant Kali access to OPNsense administrative services.

---

## 16. REDNET Firewall Management Protection

Rule:

`BLOCK - REDNET to OPNsense Management`

Source:

`REDNET net`

Destination:

`This Firewall`

Protocol:

`Any`

Purpose:

Prevents attacker-network systems from accessing OPNsense management and unauthorized firewall services.

Explicit DNS, NTP, and ICMP rules are evaluated before this block.

---

## 17. REDNET to CORPNET Protection

Rule:

`BLOCK - REDNET to CORPNET`

Source:

`REDNET net`

Destination:

`CORPNET net`

Protocol:

`Any`

Purpose:

Prevents Kali and other attacker systems from receiving unrestricted access to trusted corporate and management endpoints.

This is one of the primary segmentation controls in the laboratory.

---

## 18. Controlled Kali Attack Path

Rule:

`ALLOW - Kali to Ubuntu Lab Testing`

Source:

`KALI_ATTACKER`

Destination:

`UBUNTU_SERVER`

Protocol:

`Any`

Purpose:

Creates an intentional offensive-security pathway between Kali Linux and Ubuntu Server.

Attack path:

Kali Linux  
↓  
REDNET  
↓  
OPNsense  
↓  
SERVERNET  
↓  
Ubuntu Server

This path may be used for authorized exercises such as:

- Host discovery
- Port scanning
- Service enumeration
- Vulnerability assessment
- SSH security testing
- Web application security testing
- Controlled exploitation
- Privilege escalation exercises

The rule is logged to provide defensive visibility into offensive activity.

---

## 19. REDNET Private Network Restrictions

Rule:

`BLOCK - REDNET to Unauthorized Private Networks`

Source:

`REDNET net`

Destination:

`PRIVATE_NETS`

Purpose:

Prevents attacker systems from accessing additional private networks unless an explicit earlier rule authorizes the connection.

The Kali-to-Ubuntu attack rule is intentionally placed before this block.

---

## 20. REDNET Internet Access

Rule:

`ALLOW - REDNET to Internet`

Source:

`REDNET net`

Destination:

`Any`

Purpose:

Provides Internet access for:

- Kali Linux updates
- Package installation
- Security tools
- Repositories
- Documentation
- Authorized training resources

The rule appears after internal-network restrictions so it cannot be used to bypass segmentation.

---

## 21. SERVERNET Policy

SERVERNET contains protected Linux servers and future application targets.

Network:

`10.10.30.0/24`

Ubuntu Server:

`10.10.30.30`

The implemented SERVERNET rule order is:

1. `ALLOW - SERVERNET to OPNsense DNS`
2. `ALLOW - SERVERNET to OPNsense NTP`
3. `ALLOW - Ubuntu to OPNsense ICMP`
4. `BLOCK - SERVERNET to OPNsense Management`
5. `BLOCK - SERVERNET to CORPNET`
6. `BLOCK - SERVERNET to REDNET`
7. `BLOCK - SERVERNET to Unauthorized Private Networks`
8. `ALLOW - SERVERNET to Internet`

---

## 22. SERVERNET Infrastructure Services

SERVERNET systems are explicitly permitted to use OPNsense DNS and NTP.

DNS:

`TCP/UDP 53`

NTP:

`UDP 123`

These permissions provide required infrastructure services without granting unrestricted firewall access.

---

## 23. Ubuntu Gateway ICMP

Rule:

`ALLOW - Ubuntu to OPNsense ICMP`

Source:

`UBUNTU_SERVER`

Destination:

`This Firewall`

Protocol:

`ICMP`

Purpose:

Allows troubleshooting between Ubuntu Server and its OPNsense gateway.

---

## 24. SERVERNET Firewall Protection

Rule:

`BLOCK - SERVERNET to OPNsense Management`

Source:

`SERVERNET net`

Destination:

`This Firewall`

Protocol:

`Any`

Purpose:

Prevents protected servers from accessing unnecessary firewall-management services.

---

## 25. SERVERNET Lateral-Movement Controls

Rules:

`BLOCK - SERVERNET to CORPNET`

and:

`BLOCK - SERVERNET to REDNET`

prevent servers from initiating unrestricted connections toward the trusted endpoint network or attacker network.

This is especially important if a server becomes compromised.

A compromised server should not automatically become a pivot point into other security zones.

---

## 26. Stateful Return Traffic

OPNsense performs stateful firewalling.

For example, when Kali initiates an explicitly permitted connection to Ubuntu:

`10.10.10.10 → 10.10.30.30`

return traffic associated with that established connection can return through the firewall state.

Ubuntu therefore does not require a broad rule allowing it to initiate connections toward REDNET.

The same principle applies to management connections initiated from CORPNET.

---

## 27. SERVERNET Internet Access

Rule:

`ALLOW - SERVERNET to Internet`

Purpose:

Provides outbound connectivity required for activities such as:

- Ubuntu updates
- Package installation
- Repository access
- Approved downloads

Private-network restrictions are evaluated before this rule.

---

## 28. Logging Strategy

Logging is enabled for important policy decisions.

Examples include:

- Firewall management
- Unauthorized firewall access
- REDNET-to-CORPNET blocks
- Kali-to-Ubuntu testing
- SERVERNET segmentation
- Private-network blocks
- Internet access

These logs will later provide data for:

- Firewall analysis
- Wazuh
- SIEM monitoring
- Detection engineering
- Incident response
- Attack timeline reconstruction

---

## 29. Current Migration State

The firewall rules have been built before fully migrating the laboratory hosts.

Current status:

**CORPNET Policy:** 🟢 Configured

**REDNET Policy:** 🟢 Configured

**SERVERNET Policy:** 🟢 Configured

**Windows CORPNET Interface:** 🟢 Connected for management

**Kali REDNET Migration:** 🟡 Pending

**Ubuntu SERVERNET Migration:** 🟡 Pending

**Removal of direct VM NAT:** 🟡 Pending

**Removal of original LAB-NET:** 🟡 Pending

**Legacy default LAN rules:** 🟡 Temporarily retained

The next phase will test the policies with actual hosts before removing migration safety mechanisms.

---

## 30. Next Validation Phase

The next phase of the firewall project will include:

1. Migrate Kali Linux to REDNET.
2. Assign Kali `10.10.10.10/24`.
3. Set OPNsense `10.10.10.1` as Kali's gateway.
4. Verify DNS through OPNsense.
5. Verify Internet routing through OPNsense.
6. Confirm Kali cannot access CORPNET.
7. Confirm Kali cannot access the OPNsense Web GUI.
8. Migrate Ubuntu Server to SERVERNET.
9. Assign Ubuntu `10.10.30.30/24`.
10. Set OPNsense `10.10.30.1` as Ubuntu's gateway.
11. Verify Kali can reach Ubuntu through the authorized attack path.
12. Verify Ubuntu cannot initiate unauthorized connections toward REDNET or CORPNET.
13. Review OPNsense firewall logs.
14. Remove direct NAT adapters after validation.
15. Remove the original flat LAB-NET after migration.
16. Disable legacy broad LAN allow rules after CORPNET validation.

The firewall will therefore become the mandatory routing and security-control point for the laboratory.

---

## Security Disclaimer

The firewall rules documented here are used exclusively in a controlled cybersecurity laboratory.

The REDNET offensive-security network is restricted to systems that I own or have explicit authorization to test.

No unauthorized external systems, networks, accounts, applications, or organizations are targeted.
