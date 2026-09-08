# cybersecurity-homelab
A hands-on cybersecurity laboratory built to develop and demonstrate practical skills across network security, Linux, Windows, defensive security, offensive security, and security operations.

This repository documents my journey from building the lab infrastructure to conducting security experiments, troubleshooting failures, testing defenses, and documenting the results.

---

🎯 Objectives

The primary objectives of this homelab are to:

- Build a realistic cybersecurity testing environment
- Develop strong networking fundamentals
- Practice Linux and Windows administration
- Learn offensive security and ethical hacking
- Develop Blue Team and SOC capabilities
- Practice vulnerability assessment and penetration testing
- Perform security monitoring and analysis
- Understand attack and defense techniques
- Document practical cybersecurity experience

---

🧠 Learning Methodology

My cybersecurity learning process:

Learn
↓
Build
↓
Break
↓
Troubleshoot
↓
Fix
↓
Test
↓
Document
↓
Repeat

I use this methodology to turn theoretical knowledge into practical, demonstrable skills.

---

🏗️ Lab Architecture

The lab consists of three virtual machines connected through an isolated laboratory network.

                         INTERNET
                            │
                            │
                           NAT
                            │
             ┌──────────────┼──────────────┐
             │              │              │
         Kali Linux     Windows 11    Ubuntu Server
             │              │              │
             └──────────────┼──────────────┘
                            │
                         LAB-NET
                     192.168.56.0/24
                            │
             ┌──────────────┼──────────────┐
             │              │              │
        192.168.56.10  192.168.56.20  192.168.56.30
             │              │              │
          Kali Linux     Windows 11    Ubuntu Server

Network Design

Network| Purpose| Address Range
NAT| Internet access| "10.0.2.0/24"
LAB-NET| Isolated lab traffic| "192.168.56.0/24"

Lab IP Addresses

System| Role| LAB-NET IP
Kali Linux| Offensive Security| "192.168.56.10"
Windows 11| Windows Endpoint| "192.168.56.20"
Ubuntu Server| Linux Server| "192.168.56.30"

---

💻 Virtual Machines

⚔️ Kali Linux

Role: Offensive Security / Penetration Testing

Planned activities include:

- Network reconnaissance
- Service enumeration
- Vulnerability assessment
- Web security testing
- Password security testing
- Exploitation in the isolated lab
- Post-exploitation fundamentals
- Reporting and documentation

---

🪟 Windows 11 Enterprise

Role: Windows Endpoint / Defensive Security

Planned activities include:

- Windows administration
- PowerShell
- Windows networking
- Windows Firewall
- User and privilege management
- Event logs
- Security monitoring
- Endpoint hardening
- Attack detection

---

🐧 Ubuntu Server

Role: Linux Server / Infrastructure

Planned activities include:

- Linux administration
- SSH
- Networking
- User and permission management
- Service management
- Firewall configuration
- Log analysis
- Server hardening
- Security monitoring

---

🌐 Networking

The lab uses two network interfaces on each virtual machine.

NAT

Used for:

- Internet connectivity
- Package updates
- Software installation
- Access to external resources

LAB-NET

Used for:

- VM-to-VM communication
- Security testing
- Network analysis
- Attack simulations
- Defensive security experiments

The LAB-NET environment is isolated and intended only for authorized laboratory activities.

---

🧪 Labs & Experiments

This section will grow as I progress through my cybersecurity training.

🌐 Networking

- [ ] Network configuration
- [ ] IP addressing
- [ ] Subnetting
- [ ] Routing
- [ ] DNS
- [ ] Network troubleshooting
- [ ] Packet analysis

🐧 Linux Security

- [ ] Linux users and permissions
- [ ] SSH security
- [ ] Linux firewall
- [ ] Log analysis
- [ ] Server hardening

🪟 Windows Security

- [ ] Windows users and privileges
- [ ] PowerShell
- [ ] Windows Firewall
- [ ] Event Viewer
- [ ] Windows security hardening
- [ ] Endpoint monitoring

🛡️ Blue Team / SOC

- [ ] Log collection
- [ ] Security monitoring
- [ ] Alert investigation
- [ ] Threat detection
- [ ] Incident response
- [ ] SIEM fundamentals

⚔️ Offensive Security

- [ ] Reconnaissance
- [ ] Network scanning
- [ ] Enumeration
- [ ] Vulnerability assessment
- [ ] Exploitation
- [ ] Privilege escalation
- [ ] Post-exploitation
- [ ] Penetration testing reports

---

🛠️ Tools & Technologies

Tools will be added as they are learned and used in the lab.

- VirtualBox
- Kali Linux
- Ubuntu Server
- Windows 11
- Git & GitHub
- Nmap
- Wireshark
- PowerShell
- Linux CLI
- Windows Security Tools

---

🔧 Troubleshooting & Lessons Learned

Cybersecurity is not only about successful attacks or configurations.

This repository also documents:

- Configuration failures
- Connectivity problems
- Misconfigurations
- Troubleshooting processes
- Solutions
- Security lessons learned

Failures are treated as part of the learning process.

---

📸 Evidence

Screenshots, network diagrams, command outputs, configurations, and other relevant evidence will be added throughout the project.

Sensitive information such as passwords, private keys, tokens, credentials, and other secrets will never be committed to this repository.

---

## 📚 Documentation Structure

``text
cybersecurity-homelab/
│
├── 📄 README.md
├── 📄 CHANGELOG.md
│
├── 📁 architecture/
│   ├── 📄 lab-architecture.md
│   ├── 📄 virtualbox-networking.md
│   └── 🖼️ network-diagram.png
│
├── 📁 virtual-machines/
│   ├── 📄 kali.md
│   ├── 📄 windows.md
│   └── 📄 ubuntu.md
│
├── 📁 networking/
│   ├── 📄 addressing.md
│   ├── 📄 connectivity-tests.md
│   └── 📄 troubleshooting.md
│
├── 📁 linux/
│   └── 📄 ubuntu-server.md
│
├── 📁 windows/
│   └── 📄 windows-security.md
│
├── 📁 offensive-security/
│   ├── 📁 reconnaissance/
│   ├── 📁 enumeration/
│   ├── 📁 vulnerability-assessment/
│   ├── 📁 exploitation/
│   └── 📁 penetration-testing/
│
├── 📁 blue-team/
│   ├── 📁 monitoring/
│   ├── 📁 threat-detection/
│   ├── 📁 log-analysis/
│   └── 📁 endpoint-security/
│
├── 📁 incident-response/
│   ├── 📁 investigation/
│   ├── 📁 containment/
│   ├── 📁 eradication/
│   └── 📁 recovery/
│
├── 📁 screenshots/
│   ├── 📁 architecture/
│   ├── 📁 networking/
│   ├── 📁 linux/
│   ├── 📁 windows/
│   ├── 📁 offensive-security/
│   └── 📁 blue-team/
│
└── 📁 reports/
    ├── 📁 penetration-testing/
    ├── 📁 incident-response/
    └── 📁 security-assessments/


---

🚀 Project Status

Status: 🟢 Active Development

The homelab is continuously evolving as I progress through my cybersecurity training.

New systems, security controls, experiments, and documentation will be added over time.

---

📈 Learning Progress

Learning Area| Status
Cybersecurity Fundamentals| ✅
Networking Fundamentals| 🔄
Linux Administration| 🔄
Windows Administration| 🔄
Network Security| 🔄
Blue Team / SOC| 🔄
Incident Response| 🔄
Offensive Security| 🔄
Penetration Testing| 🔄
Advanced Security Operations| ⏳

Legend:

- ✅ Completed
- 🔄 In Progress
- ⏳ Upcoming

---

⚠️ Disclaimer

All security testing documented in this repository is performed in an authorized laboratory environment that I control.

No unauthorized systems, networks, accounts, or organizations are targeted.

---

👤 Author

Emmanuel Ngari

Cybersecurity | Network Security | Blue Team | Red Team

GitHub: @Emmanuel-Ngari
LinkedIn: @emmanuel-ngari

---

📝 Initial Commit

This repository begins with a professional Git workflow focused on documenting practical cybersecurity development rather than simply storing files.

Future commits will document the construction, configuration, testing, troubleshooting, and evolution of the cybersecurity homelab.
