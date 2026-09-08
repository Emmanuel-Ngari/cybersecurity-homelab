Cybersecurity Homelab

A hands-on cybersecurity laboratory built to develop and demonstrate practical skills across network security, Linux, Windows, defensive security, offensive security, and security operations.

This repository documents my journey from building the lab infrastructure to conducting security experiments, troubleshooting failures, testing defenses, and documenting the results.

---

Objectives

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

Learning Methodology

My cybersecurity learning process follows:

«Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat»

I use this methodology to turn theoretical knowledge into practical, demonstrable cybersecurity skills.

---

Lab Architecture

The lab consists of three virtual machines connected through an isolated laboratory network.

                         INTERNET
                            │
                            │
                           NAT
                            │
             ┌──────────────┼──────────────┐
             │              │              │
        Kali Linux      Windows 11    Ubuntu Server
        NAT: DHCP       NAT: DHCP       NAT: DHCP
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
        Kali Linux      Windows 11    Ubuntu Server
       Attack Machine    Endpoint       Linux Server

Network Design

Network| Purpose| Address Range
NAT| Internet connectivity, package updates, and software installation| "10.0.2.0/24"
LAB-NET| Isolated VM-to-VM communication and security testing| "192.168.56.0/24"

Lab IP Addresses

System| Role| LAB-NET IP
Kali Linux| Offensive Security| "192.168.56.10"
Windows 11 Enterprise| Windows Endpoint| "192.168.56.20"
Ubuntu Server| Linux Server| "192.168.56.30"

«Note: NAT addresses may be dynamically assigned by VirtualBox. The LAB-NET addresses are the static laboratory addresses used for VM-to-VM communication.»

---

Virtual Machines

Kali Linux

Role: Offensive Security / Penetration Testing

Planned activities include:

- Network reconnaissance
- Service enumeration
- Vulnerability assessment
- Web security testing
- Password security testing
- Exploitation in the isolated lab
- Post-exploitation fundamentals
- Penetration testing methodology
- Security reporting and documentation

---

Windows 11 Enterprise

Role: Windows Endpoint / Defensive Security

Planned activities include:

- Windows administration
- PowerShell
- Windows networking
- Windows Firewall
- User and privilege management
- Event Logs
- Security monitoring
- Endpoint hardening
- Attack detection

---

Ubuntu Server

Role: Linux Server / Infrastructure

Planned activities include:

- Linux administration
- Network configuration
- SSH
- User and permission management
- Service management
- Firewall configuration
- Log analysis
- Server hardening
- Security monitoring

---

Networking

Each virtual machine uses two network interfaces.

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

The "LAB-NET" environment is isolated and intended only for authorized laboratory activities.

---

Labs & Experiments

This section will grow as I progress through my cybersecurity training.

🌐 Networking

- [ ] Network configuration
- [ ] IP addressing
- [ ] Subnetting
- [ ] Routing
- [ ] DNS
- [ ] Network troubleshooting
- [ ] Packet analysis

Linux Security

- [ ] Linux users and permissions
- [ ] SSH security
- [ ] Linux firewall
- [ ] Log analysis
- [ ] Server hardening

Windows Security

- [ ] Windows users and privileges
- [ ] PowerShell
- [ ] Windows Firewall
- [ ] Event Viewer
- [ ] Windows security hardening
- [ ] Endpoint monitoring

Blue Team / SOC

- [ ] Log collection
- [ ] Security monitoring
- [ ] Alert investigation
- [ ] Threat detection
- [ ] Incident response
- [ ] SIEM fundamentals

Offensive Security

- [ ] Reconnaissance
- [ ] Network scanning
- [ ] Enumeration
- [ ] Vulnerability assessment
- [ ] Exploitation
- [ ] Privilege escalation
- [ ] Post-exploitation
- [ ] Penetration testing reports

---

Tools & Technologies

Tools will be added as they are learned and used in the laboratory.

- Category| Tools / Technologies
- Virtualization| VirtualBox
- Operating Systems| Kali Linux, Ubuntu Server, Windows 11 Enterprise
- Version Control| Git, GitHub
- Network Security| Nmap, Wireshark
- Windows Security| PowerShell, Windows Security Tools
- Linux Security| Linux CLI, SSH, Firewall Tools

---

Troubleshooting & Lessons Learned

Cybersecurity is not only about successful attacks or configurations.

This repository also documents:

- Configuration failures
- Connectivity problems
- Misconfigurations
- Troubleshooting processes
- Solutions
- Security lessons learned
- Improvements made to the laboratory

«Failures are treated as part of the learning process.»

---

Evidence

Screenshots, network diagrams, command outputs, configurations, and other relevant evidence will be added throughout the project.

Sensitive information such as:

- Passwords
- Private keys
- API tokens
- Credentials
- Personal information
- Other secrets

will never be committed to this repository.

---

Documentation Structure

cybersecurity-homelab/
│
├── README.md
├── CHANGELOG.md
│
├── architecture/
│   ├── network-diagram.png
│   ├── lab-architecture.md
│   └── virtualbox-networking.md
│
├── virtual-machines/
│   ├── kali.md
│   ├── windows.md
│   └── ubuntu.md
│
├── networking/
│   ├── addressing.md
│   ├── connectivity-tests.md
│   └── troubleshooting.md
│
├── linux/
│   └── ubuntu-server.md
│
├── windows/
│   └── windows-security.md
│
├── offensive-security/
│
├── blue-team/
│
├── incident-response/
│
└── screenshots/

---

Project Status

Status: 🟢 Active Development

The homelab is continuously evolving as I progress through my cybersecurity training.

New systems, security controls, experiments, and documentation will be added over time.

---

📈 Learning Progress

Area| Status
- Cybersecurity Fundamentals| ✅ Completed
- Networking Fundamentals| 🔄 In Progress
- Linux Administration| 🔄 In Progress
- Windows Administration| 🔄 In Progress
- Network Security| 🔄 In Progress
- Blue Team / SOC| 🔄 In Progress
- Incident Response| 🔄 In Progress
- Offensive Security| 🔄 In Progress
- Penetration Testing| 🔄 In Progress
- Advanced Security Operations| ⏳ Planned

---

⚠️ Disclaimer

All security testing documented in this repository is performed in an authorized laboratory environment that I control.

No unauthorized systems, networks, accounts, or organizations are targeted.

The offensive security activities documented here are intended strictly for education, defensive security development, and authorized testing.

---

👤 Author

Emmanuel Ngari

Cybersecurity | Network Security | Blue Team | Red Team

GitHub: "@Emmanuel-Ngari" (https://github.com/Emmanuel-Ngari)

---

📌 Repository Philosophy

«Build it. Understand it. Break it. Defend it. Document it.»
