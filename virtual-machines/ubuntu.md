# Ubuntu Server — LAB-UBUNTU-01

## 1. Overview

Ubuntu Server is one of the core systems in my cybersecurity homelab.

The server is used to develop practical Linux administration, networking, system security, logging, SSH, hardening, monitoring, and defensive security skills.

It also provides a Linux server target for authorized security testing from Kali Linux.

---

## 2. System Role

**Hostname:** `lab-ubuntu-01`

**Operating System:** Ubuntu Server 26.04.1 LTS

**Primary Role:** Linux Server / Defensive Security

**LAB-NET IP Address:** `192.168.56.30/24`

Ubuntu acts as the Linux server within the isolated cybersecurity laboratory.

---

## 3. Network Interfaces

Ubuntu Server uses two network interfaces.

| Interface | Network | IP Address | Purpose |
|---|---|---|---|
| `enp0s3` | NAT | `10.0.2.15/24` | Internet connectivity |
| `enp0s8` | LAB-NET | `192.168.56.30/24` | Isolated laboratory communication |

The two-interface design separates Internet access from laboratory communication.

---

## 4. NAT Interface

The NAT interface is:

`enp0s3`

The interface provides Internet connectivity to the Ubuntu Server.

**IPv4 Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

The NAT connection can be used for:

- System updates
- Package installation
- Downloading legitimate security tools
- Installing required dependencies
- Accessing learning resources

---

## 5. LAB-NET Interface

The isolated laboratory interface is:

`enp0s8`

**IPv4 Address:** `192.168.56.30/24`

**Network:** `192.168.56.0/24`

**Default Gateway:** None

The interface provides communication with the other laboratory systems.

### Laboratory Hosts

| System | LAB-NET Address | Role |
|---|---|---|
| Kali Linux | `192.168.56.10` | Offensive Security |
| Windows 11 | `192.168.56.20` | Windows Endpoint |
| Ubuntu Server | `192.168.56.30` | Linux Server |

---

## 6. Initial Network Configuration

The Ubuntu LAB-NET interface initially did not have the required IPv4 address.

The interface was manually assigned:

`192.168.56.30/24`

using the Linux `ip` command.

The resulting routing information included:

`192.168.56.0/24 dev enp0s8`

This confirmed that Ubuntu had a connected route to the isolated LAB-NET network.

The manual configuration was initially temporary and will later be converted into a persistent Netplan configuration.

---

## 7. Network Routing

Ubuntu uses separate network paths for Internet and laboratory communication.

### Default Route

The default route uses the NAT interface:

`default via 10.0.2.2`

### LAB-NET Route

The isolated network uses:

`192.168.56.0/24 dev enp0s8`

This design prevents the LAB-NET interface from becoming the default Internet route.

It also provides a clear separation between external connectivity and internal laboratory traffic.

---

## 8. Connectivity Testing

Connectivity was tested after configuring the LAB-NET interface.

### Ubuntu → Kali

Source:

`192.168.56.30`

Destination:

`192.168.56.10`

**Result:** Successful

### Ubuntu → Windows

Source:

`192.168.56.30`

Destination:

`192.168.56.20`

**Result:** Successful

### Ubuntu → NAT Gateway

Source:

`10.0.2.15`

Destination:

`10.0.2.2`

**Result:** Successful

These tests confirmed that Ubuntu could communicate with the laboratory systems and reach the NAT gateway.

---

## 9. SSH Service

SSH is enabled on the Ubuntu Server.

**Service:** OpenSSH Server

**Port:** `22/TCP`

SSH provides remote administration and will also be useful for cybersecurity exercises.

Planned SSH security activities include:

- SSH configuration
- Authentication analysis
- Failed login investigation
- SSH hardening
- Key-based authentication
- Log analysis
- Brute-force detection
- Fail2ban configuration

SSH will only be exposed within the controlled laboratory environment unless intentionally configured otherwise.

---

## 10. Linux Security Role

Ubuntu Server will be used to develop Linux defensive security skills.

Planned activities include:

### System Hardening

- User account security
- Least privilege
- File permissions
- SSH hardening
- Service management
- Package management
- System updates

### Monitoring

- Authentication logs
- System logs
- Process monitoring
- Network connections
- Running services
- Resource usage

### Security Tools

Future exercises may include:

- Fail2ban
- UFW
- auditd
- Linux auditing
- Log analysis tools
- Network monitoring tools

---

## 11. Offensive Security Testing

Kali Linux will be used for authorized security testing against the Ubuntu Server.

Planned exercises include:

- Host discovery
- Port scanning
- Service enumeration
- SSH enumeration
- Vulnerability assessment
- Authentication security testing
- Controlled exploitation
- Privilege escalation
- Post-exploitation analysis

All offensive security activities will remain inside the controlled homelab.

---

## 12. Defensive Security Exercises

Ubuntu will also serve as a defensive security training platform.

Planned exercises include:

- Monitoring authentication attempts
- Investigating SSH logs
- Detecting suspicious processes
- Monitoring network connections
- Hardening unnecessary services
- Configuring firewall rules
- Detecting brute-force activity
- Investigating suspicious user activity
- Reviewing system logs
- Performing incident-response exercises

The objective is to understand how Linux systems can be protected, monitored, investigated, and recovered.

---

## 13. Useful Linux Commands

The following commands are useful for Ubuntu administration and security investigations.

### Display System Information

`uname -a`

### Display IP Addresses

`ip addr`

### Display Routing Table

`ip route`

### Display Network Interfaces

`ip link`

### Test Connectivity to Kali

`ping -c 4 192.168.56.10`

### Test Connectivity to Windows

`ping -c 4 192.168.56.20`

### Test NAT Gateway

`ping -c 4 10.0.2.2`

### Display Listening Services

`sudo ss -tulnp`

### Check SSH Service

`sudo systemctl status ssh`

### Display Running Services

`systemctl --type=service --state=running`

### Display Processes

`ps aux`

### Monitor Processes

`top`

### Check Disk Usage

`df -h`

### Check Memory Usage

`free -h`

### View System Logs

`journalctl`

### View Recent Authentication Activity

`last`

These commands will form part of the Linux administration and security investigation workflow.

---

## 14. Evidence Collection

Evidence from Ubuntu security exercises will be documented using:

- Terminal output
- Linux command output
- Network configuration
- Routing tables
- SSH configuration
- System logs
- Authentication logs
- Service information
- Firewall configuration
- Security testing results
- Remediation results
- Screenshots

Screenshots and command output will be reviewed before being uploaded to GitHub.

Sensitive information will not be committed.

---

## 15. Security Considerations

Ubuntu has two network interfaces.

The NAT interface provides Internet connectivity while LAB-NET provides isolated communication with the cybersecurity laboratory.

Security configuration must therefore be carefully controlled.

The following information must never be committed to the repository:

- Passwords
- Private keys
- SSH keys
- API keys
- Authentication tokens
- Recovery codes
- Sensitive personal information
- Secrets stored in configuration files

Only authorized laboratory systems will be targeted during security testing.

---

## 16. Future Improvements

Planned improvements for the Ubuntu environment include:

- Persistent Netplan configuration
- SSH hardening
- UFW firewall configuration
- Fail2ban
- Linux auditing
- Centralized logging
- SIEM integration
- Security monitoring
- File integrity monitoring
- Vulnerability management
- Detection engineering
- Incident response exercises
- Linux privilege escalation labs
- Server hardening exercises

---

## 17. Lessons Learned

Configuring Ubuntu Server provided practical experience with:

- Linux networking
- Network interfaces
- Static IPv4 addressing
- Routing
- VirtualBox networking
- SSH
- Linux services
- System administration
- Network troubleshooting
- Linux security

An important lesson was that Linux networking can be configured temporarily using commands such as `ip`, but persistent configuration should be managed through the operating system's network configuration system.

The next stage is to convert the temporary LAB-NET address into a persistent Netplan configuration.

---

## 18. Current Status

**Status:** 🟡 Operational — Network Configuration Being Finalized

Ubuntu Server is currently connected to:

- NAT for Internet connectivity
- LAB-NET for isolated cybersecurity activities

Current LAB-NET address:

`192.168.56.30/24`

Connectivity to Kali Linux, Windows 11, and the NAT gateway has been successfully tested.

The LAB-NET address is currently configured manually and will be made persistent using Netplan.

---

## 19. Planned Blue Team Integration

Ubuntu will become an important Linux telemetry source as the homelab develops.

Future defensive architecture may include:

Ubuntu Server  
↓  
Authentication Logs  
↓  
System Logs  
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

This will provide practical experience with Linux monitoring, threat detection, SOC operations, and incident response.

---

## 20. Security Disclaimer

Ubuntu Server is used exclusively for authorized cybersecurity training and experimentation within my controlled laboratory environment.

All offensive security testing is performed only against systems that I own or have explicit authorization to test.

No unauthorized systems, networks, accounts, or organizations are targeted.
