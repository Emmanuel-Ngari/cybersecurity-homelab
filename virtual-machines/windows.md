# Windows 11 Enterprise — Security Endpoint

## 1. Overview

Windows 11 Enterprise is the primary Windows endpoint in my cybersecurity homelab.

The system is used to develop practical skills in Windows administration, endpoint security, network security, PowerShell, firewall configuration, event logging, security monitoring, hardening, and attack detection.

The Windows system also provides a controlled target for authorized security testing from the Kali Linux workstation.

---

## 2. System Role

**Operating System:** Windows 11 Enterprise

**Primary Role:** Windows Endpoint / Defensive Security

**LAB-NET IP Address:** `192.168.56.20/24`

The system serves as an endpoint that can be monitored, secured, tested, and analyzed within the isolated cybersecurity laboratory.

---

## 3. Network Interfaces

Windows 11 uses two network adapters.

| Adapter | Network | IP Address | Purpose |
|---|---|---|---|
| `NAT` | NAT | `10.0.2.15/24` | Internet connectivity |
| `LAB-NET` | LAB-NET | `192.168.56.20/24` | Isolated laboratory communication |

---

## 4. NAT Adapter

The NAT adapter provides Internet connectivity.

**Adapter Name:** `NAT`

**Network:** NAT

**IPv4 Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

**DNS:** `10.0.2.3`

### Purpose

The NAT interface is used for:

- Windows updates
- Downloading required software
- Security tool installation
- Accessing legitimate learning resources
- General Internet connectivity

---

## 5. LAB-NET Adapter

The LAB-NET adapter provides communication with the isolated cybersecurity laboratory.

**Adapter Name:** `LAB-NET`

**IPv4 Address:** `192.168.56.20/24`

**Default Gateway:** None

**DNS:** None

**Network:** `192.168.56.0/24`

The LAB-NET interface is used for communication with:

- Kali Linux — `192.168.56.10`
- Ubuntu Server — `192.168.56.30`

No default gateway is configured on this interface.

---

## 6. Network Configuration

The Windows LAB-NET interface was initially not assigned the required laboratory IPv4 address.

The adapter initially received an APIPA address in the:

`169.254.0.0/16`

range.

A static IPv4 configuration was subsequently applied:

**IP Address:** `192.168.56.20`

**Subnet Mask:** `255.255.255.0`

**Default Gateway:** None

This allowed Windows to communicate correctly with the other laboratory systems.

---

## 7. Windows Network Profile

The Windows network profile was configured as:

**Public**

The Public profile provides a more restrictive Windows Firewall configuration by default and is useful for understanding how Windows applies different security policies to network interfaces.

Firewall configuration was considered during the connectivity troubleshooting process.

---

## 8. Connectivity Testing

Connectivity between Windows and the other laboratory systems was tested after configuring LAB-NET.

### Windows → Kali

Source:

`192.168.56.20`

Destination:

`192.168.56.10`

**Result:** Successful

### Windows → Ubuntu

Source:

`192.168.56.20`

Destination:

`192.168.56.30`

**Result:** Successful

Successful connectivity confirmed that Windows could communicate with the other laboratory systems through LAB-NET.

---

## 9. Connectivity Troubleshooting

During the initial network configuration, communication between Kali Linux and Windows was unsuccessful.

The troubleshooting process involved investigating:

- Windows network adapter configuration
- IPv4 addressing
- APIPA addressing
- Network profile
- Windows Firewall
- ICMP traffic
- Routing
- VirtualBox networking
- LAB-NET configuration

The Windows LAB-NET adapter was eventually assigned the correct static address:

`192.168.56.20/24`

After the network and firewall configuration was corrected, Kali Linux was able to successfully ping Windows.

This provided practical experience with Windows network troubleshooting and the relationship between networking and endpoint security controls.

---

## 10. Windows Security Role

Windows 11 will be used as the primary endpoint for defensive security exercises.

Planned activities include:

### Endpoint Hardening

- Windows security configuration
- Firewall configuration
- User account security
- Least privilege
- Security policy configuration
- System update management

### Monitoring

- Windows Event Viewer
- Windows event logs
- Process monitoring
- Network connection monitoring
- Security event analysis

### PowerShell

PowerShell will be used for:

- System administration
- Network investigation
- Process inspection
- Security configuration
- Automation
- Incident investigation

---

## 11. Offensive Security Testing

Kali Linux will be used to conduct authorized security testing against the Windows endpoint.

Planned exercises include:

- Host discovery
- Port scanning
- Service enumeration
- Vulnerability assessment
- Authentication security testing
- Controlled exploitation
- Privilege escalation
- Post-exploitation analysis

All testing will remain inside the controlled homelab.

---

## 12. Defensive Security Exercises

The Windows system will also be used to practice defensive security.

Planned exercises include:

- Detecting suspicious processes
- Investigating Windows event logs
- Monitoring network connections
- Identifying failed authentication attempts
- Investigating suspicious activity
- Hardening exposed services
- Configuring Windows Firewall
- Validating security controls

The goal is to understand both how attacks occur and how defenders can detect and respond to them.

---

## 13. Useful Windows Commands

The following commands are useful for Windows networking and troubleshooting.

### Display IP Configuration

`ipconfig /all`

### Display Routing Table

`route print`

### Test Connectivity to Kali

`ping 192.168.56.10`

### Test Connectivity to Ubuntu

`ping 192.168.56.30`

### Display Network Connections

`Get-NetTCPConnection`

### Display Network Adapters

`Get-NetAdapter`

### Display IP Configuration

`Get-NetIPConfiguration`

### Display IP Addresses

`Get-NetIPAddress`

### Display Firewall Profiles

`Get-NetFirewallProfile`

### Check Listening Ports

`Get-NetTCPConnection -State Listen`

These commands will become increasingly important as Windows security and PowerShell skills develop.

---

## 14. Evidence Collection

Evidence from Windows security exercises will be documented using:

- PowerShell output
- Command-line output
- Event Viewer screenshots
- Firewall configuration
- Network configuration
- Security event logs
- Process information
- Network connection information
- Security testing results
- Remediation results

Screenshots will be reviewed before being uploaded to GitHub.

Sensitive information will not be committed.

---

## 15. Security Considerations

The Windows endpoint is connected to both the Internet-facing NAT network and the isolated LAB-NET.

Security testing must therefore remain controlled and authorized.

The following information must never be committed to the repository:

- Passwords
- Credentials
- API keys
- Authentication tokens
- Private keys
- Recovery codes
- Sensitive personal information

The Windows endpoint will only be targeted by authorized laboratory activities.

---

## 16. Future Improvements

Planned improvements for the Windows environment include:

- Advanced Windows hardening
- PowerShell security monitoring
- Windows Event Forwarding
- Centralized logging
- SIEM integration
- Microsoft Defender configuration
- Detection engineering
- Active Directory integration
- Group Policy security
- Attack detection
- Incident response exercises
- Endpoint investigation

---

## 17. Lessons Learned

Configuring Windows 11 as a cybersecurity endpoint provided practical experience with:

- Windows networking
- Static IPv4 addressing
- APIPA
- Network profiles
- Windows Firewall
- ICMP
- Routing
- PowerShell
- VirtualBox networking
- Endpoint security

One of the most important lessons was that network connectivity and endpoint security are closely related.

A system may have the correct IP configuration and still fail to communicate because of firewall rules, network profiles, or other security controls.

---

## 18. Current Status

**Status:** 🟢 Operational

Windows 11 is currently connected to:

- NAT for Internet connectivity
- LAB-NET for isolated cybersecurity activities

The Windows endpoint can communicate with the Kali Linux and Ubuntu Server laboratory systems over LAB-NET.

---

## 19. Planned Blue Team Integration

As the homelab develops, Windows will become an important source of security telemetry.

Future architecture may include:

Windows Endpoint  
↓  
Windows Event Logs  
↓  
Log Collection  
↓  
SIEM  
↓  
Detection Rules  
↓  
Security Alert  
↓  
Investigation  
↓  
Incident Response

This will allow practical SOC and defensive security exercises.

---

## 20. Security Disclaimer

Windows 11 is used exclusively for authorized cybersecurity training and experimentation within my controlled laboratory environment.

All offensive security testing is performed only against systems that I own or have explicit authorization to test.

No unauthorized systems, networks, accounts, or organizations are targeted.
