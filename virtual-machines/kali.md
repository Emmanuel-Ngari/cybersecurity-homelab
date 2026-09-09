# Kali Linux — Offensive Security Workstation

## 1. Overview

Kali Linux is the primary offensive security workstation in my cybersecurity homelab.

It is used to develop practical skills in reconnaissance, network security testing, vulnerability assessment, penetration testing, and security analysis.

All security testing performed from this system is restricted to my authorized cybersecurity laboratory environment.

---

## 2. System Role

**Operating System:** Kali Linux

**Primary Role:** Offensive Security / Penetration Testing

**LAB-NET IP Address:** `192.168.56.10/24`

Kali Linux acts as the primary attacker and security-testing machine within the laboratory.

---

## 3. Network Interfaces

Kali Linux uses two network interfaces.

| Interface | Network | Purpose |
|---|---|---|
| `eth0` | NAT | Internet connectivity |
| `eth1` | LAB-NET | Isolated laboratory communication |

### NAT Interface

**Interface:** `eth0`

**Purpose:** Internet connectivity

**IP Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

The NAT interface is used for legitimate activities such as system updates and downloading required security tools.

### LAB-NET Interface

**Interface:** `eth1`

**Purpose:** Isolated cybersecurity laboratory communication

**IP Address:** `192.168.56.10/24`

**Network:** `192.168.56.0/24`

No Internet gateway is configured on the LAB-NET interface.

---

## 4. Network Routing

The Kali routing configuration separates Internet traffic from laboratory traffic.

Internet traffic uses the NAT interface.

Example default route:

`0.0.0.0/0 → 10.0.2.2`

Laboratory traffic uses the LAB-NET interface.

Example:

`192.168.56.0/24 → eth1`

This allows Kali to maintain Internet access while communicating with the isolated laboratory systems through LAB-NET.

---

## 5. Laboratory Connectivity

Kali Linux was tested against the other systems on LAB-NET.

### Kali → Windows 11

Source:

`192.168.56.10`

Destination:

`192.168.56.20`

**Result:** Successful

### Kali → Ubuntu Server

Source:

`192.168.56.10`

Destination:

`192.168.56.30`

**Result:** Successful

These tests confirmed that Kali can communicate with the Windows and Ubuntu systems through the isolated laboratory network.

---

## 6. Initial Connectivity Troubleshooting

During the initial laboratory configuration, Kali Linux could not immediately communicate successfully with the Windows 11 LAB-NET interface.

The troubleshooting process involved checking:

- Kali network interfaces
- IPv4 addressing
- Routing
- Windows network adapter configuration
- Windows network profile
- Windows Firewall
- ICMP connectivity
- VirtualBox network configuration

After troubleshooting the configuration and firewall/network settings, Kali was able to successfully communicate with Windows over LAB-NET.

This provided practical experience in diagnosing connectivity problems across different operating systems.

---

## 7. Security Role

Kali Linux will be used as the primary offensive security platform for the laboratory.

The system will be used to practice:

### Reconnaissance

- Network discovery
- Host discovery
- Service identification
- Information gathering

### Enumeration

- Port enumeration
- Service enumeration
- Version identification
- Network service analysis

### Vulnerability Assessment

- Identifying vulnerable services
- Analyzing security weaknesses
- Validating vulnerabilities within the controlled environment

### Penetration Testing

- Controlled exploitation
- Privilege escalation
- Post-exploitation fundamentals
- Security validation
- Documentation and reporting

---

## 8. Planned Security Tools

Security tools will be introduced progressively as they become relevant to each laboratory exercise.

Planned tools include:

- Nmap
- Wireshark
- Burp Suite
- Metasploit Framework
- Gobuster
- Nikto
- Netcat
- John the Ripper
- Hashcat
- tcpdump
- SSH utilities

Tools will only be used against authorized laboratory systems.

---

## 9. Important Commands

The following Linux commands are useful for inspecting and troubleshooting the Kali networking environment.

### Display Network Interfaces

`ip addr`

### Display Routing Table

`ip route`

### Test Windows Connectivity

`ping -c 4 192.168.56.20`

### Test Ubuntu Connectivity

`ping -c 4 192.168.56.30`

### Display Interface Information

`ip addr show eth0`

`ip addr show eth1`

### Display Routing Information

`ip route`

### Check Listening Services

`ss -tuln`

These commands form part of the basic Linux networking and troubleshooting toolkit used in the homelab.

---

## 10. Security Testing Methodology

Security testing from Kali will follow a controlled methodology:

Reconnaissance  
↓  
Discovery  
↓  
Enumeration  
↓  
Vulnerability Assessment  
↓  
Controlled Exploitation  
↓  
Privilege Escalation  
↓  
Post-Exploitation  
↓  
Evidence Collection  
↓  
Reporting  
↓  
Remediation  
↓  
Retesting

This methodology will be applied only to authorized laboratory systems.

---

## 11. Evidence Collection

Evidence from Kali-based exercises will be documented using:

- Terminal output
- Screenshots
- Network scans
- Packet captures
- Tool output
- Findings
- Remediation results
- Lessons learned

Screenshots and command output will be reviewed before being uploaded to GitHub.

Sensitive information must never be committed.

---

## 12. Security Considerations

The Kali system has significant security capabilities and must therefore be used responsibly.

The following information must never be committed to the repository:

- Passwords
- API keys
- Authentication tokens
- Private SSH keys
- Credentials
- Recovery codes
- Sensitive personal information

All offensive security activities will remain within the authorized homelab.

---

## 13. Future Improvements

Planned improvements for the Kali environment include:

- Advanced network reconnaissance
- Vulnerability scanning
- Web application testing
- Exploitation labs
- Privilege escalation exercises
- Active Directory attack simulations
- Password security testing
- Network traffic analysis
- Automated reconnaissance
- Penetration testing reports

---

## 14. Lessons Learned

Configuring Kali Linux provided practical experience with:

- Linux networking
- Network interfaces
- IPv4 addressing
- Routing
- NAT
- Isolated networks
- ICMP troubleshooting
- VirtualBox networking
- Cross-platform connectivity
- Security testing fundamentals

A key lesson was that a cybersecurity workstation is not useful simply because security tools are installed.

The underlying operating system, networking, routing, and communication between systems must first be correctly configured.

---

## 15. Current Status

**Status:** 🟢 Operational

Kali Linux is currently connected to both:

- NAT for Internet connectivity
- LAB-NET for isolated cybersecurity activities

The system can communicate with the Windows 11 and Ubuntu Server laboratory systems over LAB-NET.

---

## 16. Security Disclaimer

Kali Linux is used exclusively for authorized cybersecurity training and experimentation within my controlled laboratory environment.

No unauthorized systems, networks, accounts, or organizations are targeted.
