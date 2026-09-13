# Professional Cybersecurity Homelab — Master Build & Learning Journal

## Project Owner

**GitHub:** Emmanuel-Ngari  
**Repository:** cybersecurity-homelab

---

# 1. Purpose of This Journal

This document records the complete development of my professional cybersecurity homelab from the beginning.

The objective is not only to document what exists in the lab, but also to record:

- What I installed
- Why I installed it
- How the network evolved
- Commands used
- Configuration decisions
- Firewall policies
- Troubleshooting
- Errors and failures
- Recovery procedures
- Testing
- Snapshots
- Lessons learned
- Security architecture decisions
- Future improvements

This journal is also intended to help me explain my laboratory confidently during:

- Cybersecurity interviews
- Portfolio reviews
- Technical discussions
- Certification preparation
- SOC interviews
- Network security interviews
- Penetration testing interviews
- Personal revision

Where an exact command from an early stage of the project was not preserved, I document the verified action rather than inventing a command.

---

# 2. Project Philosophy

The laboratory is being built around the following learning cycle:

**Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat**

The operational workflow used for major changes is:

**Snapshot → Change → Test → Verify → Backup if needed → Document → Continue**

The purpose of this workflow is to make the environment:

- Recoverable
- Repeatable
- Documented
- Professionally managed
- Safe for experimentation

---

# 3. Original Lab Goal

The original goal was to build a cybersecurity homelab for both offensive and defensive security.

## Offensive Security Goals

The offensive side of the lab is intended to support:

- Linux security
- Reconnaissance
- Network enumeration
- Vulnerability scanning
- Ethical hacking
- Penetration testing
- Web security
- Credential attack simulations
- Active Directory attacks
- Network attacks
- Red-team exercises

## Defensive Security Goals

The defensive side is intended to support:

- Firewall administration
- Network segmentation
- Endpoint monitoring
- Log analysis
- Security Information and Event Management
- IDS/IPS
- Threat detection
- Vulnerability detection
- Incident investigation
- Incident response
- Security hardening
- SOC analyst exercises
- MITRE ATT&CK mapping

The long-term objective is to create a realistic attack-and-defense cyber range.

---

# 4. Physical Host Computer

The main laboratory runs on a Windows laptop.

Known hardware specifications include:

- CPU: 12th Gen Intel Core i7-1260P
- Physical cores: 12
- Logical processors: 16
- RAM: approximately 16 GB
- Virtualization platform: Oracle VirtualBox
- Host operating system: Windows

The Windows host has changed editions during the project, including Windows 11 Home and Windows 11 Enterprise Evaluation environments.

Hardware virtualization support was required for VirtualBox.

---

# 5. Virtualization Preparation

Before building the environment, hardware virtualization had to be available.

Virtualization support was checked and enabled where necessary through system firmware/BIOS.

This was important because VirtualBox relies on processor virtualization technologies such as Intel VT-x.

---

# 6. Installing Oracle VirtualBox

Oracle VirtualBox was downloaded and installed on the Windows host.

VirtualBox became the primary hypervisor for the cybersecurity environment.

The lab initially started with three main virtual machines:

- Kali Linux
- Windows
- Ubuntu Server

The environment later expanded to include:

- OPNsense firewall
- Dedicated SIEM server
- Multiple isolated security networks

Current major VM roles include:

| VM | Role |
| --- | --- |
| LAB-FW-01 | Firewall / Router |
| LAB-KALI-01 | Offensive Security Workstation |
| LAB-WIN-01 | Windows Endpoint / Administration |
| LAB-UBUNTU-01 | Linux Server |
| LAB-SIEM-01 | SOC / Wazuh SIEM |

Future systems are planned for:

- Active Directory
- Windows Server
- DMZ services
- Vulnerable targets
- Additional monitoring systems

---

# 7. Initial Three-VM Architecture

The first laboratory architecture was much simpler than the current environment.

The original internal VirtualBox network was:

`LAB-NET`

Network:

`192.168.56.0/24`

Original IP addresses included:

| Machine | IP Address |
| --- | --- |
| Kali | 192.168.56.10 |
| Windows | 192.168.56.20 |
| Ubuntu | 192.168.56.30 |

VirtualBox NAT was also used for Internet access.

Typical VirtualBox NAT addressing included:

`10.0.2.0/24`

Typical VirtualBox NAT gateway:

`10.0.2.2`

The early design looked approximately like:

    Internet
       |
    VirtualBox NAT
       |
    -------------------------
    |           |           |
    Kali      Windows     Ubuntu
    |           |           |
    ------- LAB-NET ---------
        192.168.56.0/24

This architecture was useful for learning VM networking but did not provide realistic enterprise segmentation.

---

# 8. Kali Linux Deployment

Kali became the offensive-security workstation.

Final VM name:

`LAB-KALI-01`

During the early environment Kali had two network interfaces.

Examples included:

`10.0.2.15/24`

for NAT connectivity and:

`192.168.56.10/24`

for the original laboratory network.

Important Linux networking commands used during the project included:

    ip addr

Short form:

    ip a

Routing information:

    ip route

NetworkManager information:

    nmcli

Connectivity testing:

    ping <destination-IP>

These commands were used repeatedly to understand:

- Interface addressing
- Default routes
- Network connectivity
- VirtualBox adapter behavior
- Routing through the lab

---

# 9. Windows VM Deployment

The Windows VM became the primary corporate-style endpoint.

Final VM name:

`LAB-WIN-01`

Original laboratory IP:

`192.168.56.20/24`

The Windows VM was later migrated behind OPNsense and became:

`10.10.20.20/24`

Gateway:

`10.10.20.1`

DNS:

`10.10.20.1`

Windows PowerShell became an important troubleshooting and verification tool.

A major command used throughout the environment was:

    Test-NetConnection <destination> -Port <port>

For example:

    Test-NetConnection 10.10.40.10 -Port 443

This allowed TCP connectivity to be tested between security zones.

---

# 10. Ubuntu Server Deployment

Ubuntu became the protected Linux server in the environment.

Final VM name:

`LAB-UBUNTU-01`

The Ubuntu system was used to practice:

- Linux administration
- Networking
- Service management
- Server security
- Wazuh monitoring
- Cross-zone firewall testing

Some important commands used during the Linux learning stage included:

    uname -a

This displays kernel and architecture information.

Memory information:

    free -h

CPU information:

    lscpu

Network interfaces:

    ip addr

Routing:

    ip route

The original Ubuntu address was:

`192.168.56.30/24`

It was later migrated to:

`10.10.30.30/24`

Gateway:

`10.10.30.1`

DNS:

`10.10.30.1`

---

# 11. Why the Original Flat Network Was Replaced

The original LAB-NET architecture allowed the machines to communicate too freely.

That architecture was good for learning basic networking but not ideal for simulating a professional environment.

The decision was made to introduce a real firewall/router between networks.

OPNsense was selected.

The design changed from:

    Kali
    Windows
    Ubuntu
       |
    LAB-NET

to:

                    Internet
                       |
                VirtualBox NAT
                       |
                   OPNsense
                       |
        --------------------------------
        |              |               |
      REDNET         CORPNET        SERVERNET
        |              |               |
      Kali          Windows          Ubuntu

This was one of the most important architectural improvements in the project.

---

# 12. OPNsense Firewall Deployment

A dedicated OPNsense VM was created.

VM name:

`LAB-FW-01`

Firewall platform:

`OPNsense 26.7 amd64`

Approximate resources:

- 2 vCPU
- 4 GB RAM
- 32 GB virtual disk

Hostname:

`lab-fw-01.cyberlab.internal`

The firewall became responsible for:

- Routing
- Internet access
- Inter-zone traffic control
- Security segmentation
- DNS forwarding
- NTP access
- Administrative restrictions
- Logging
- Future IDS/IPS integration

---

# 13. OPNsense Interfaces

The original firewall interfaces included:

## WAN

Interface:

`em0`

Address:

`10.0.2.15/24`

Upstream gateway:

`10.0.2.2`

VirtualBox network type:

`NAT`

---

## CORPNET

Interface:

`em1`

Address:

`10.10.20.1/24`

Network:

`10.10.20.0/24`

Purpose:

Trusted endpoint and administration network.

---

## REDNET

Interface:

`em2`

Address:

`10.10.10.1/24`

Network:

`10.10.10.0/24`

Purpose:

Offensive-security / attacker network.

---

## SERVERNET

Interface:

`em3`

Address:

`10.10.30.1/24`

Network:

`10.10.30.0/24`

Purpose:

Protected server network.

---

# 14. Known OPNsense Interface MAC Addresses

Known interface mappings include:

| Interface | MAC Address |
| --- | --- |
| em0 | 08:00:27:3B:19:9B |
| em1 | 08:00:27:04:74:7C |
| em2 | 08:00:27:6C:10:29 |
| em3 | 08:00:27:93:06:8C |
| em4 | 08:00:27:59:A7:82 |

These mappings were useful when identifying VirtualBox adapters inside OPNsense.

---

# 15. Security Zone Design

The lab was intentionally divided into different trust zones.

## REDNET

Network:

`10.10.10.0/24`

Gateway:

`10.10.10.1`

Primary host:

`LAB-KALI-01`

Purpose:

- Offensive security
- Attack simulation
- Reconnaissance
- Penetration testing

Trust level:

Low / untrusted internal zone.

---

## CORPNET

Network:

`10.10.20.0/24`

Gateway:

`10.10.20.1`

Primary host:

`LAB-WIN-01`

Purpose:

- Corporate-style endpoint
- Administration workstation
- Wazuh dashboard access
- Windows monitoring

Trust level:

Trusted user / management network.

---

## SERVERNET

Network:

`10.10.30.0/24`

Gateway:

`10.10.30.1`

Primary host:

`LAB-UBUNTU-01`

Purpose:

- Linux server services
- Protected server workloads
- Defensive monitoring
- Controlled attack target

---

## SOCNET

Network:

`10.10.40.0/24`

Gateway:

`10.10.40.1`

Primary host:

`LAB-SIEM-01`

Purpose:

- Security monitoring
- SIEM
- SOC services
- Log collection
- Detection engineering

---

## Future AD-NET

Planned network:

`10.10.50.0/24`

Purpose:

- Active Directory
- Domain Controller
- Windows enterprise services

---

## Future DMZ

Planned network:

`10.10.60.0/24`

Purpose:

- Public-facing test services
- Vulnerable applications
- Web servers
- Attack simulations

---

# 16. Final Endpoint Addressing

Current main endpoints:

| Host | Zone | IP |
| --- | --- | --- |
| LAB-KALI-01 | REDNET | 10.10.10.10 |
| LAB-WIN-01 | CORPNET | 10.10.20.20 |
| LAB-UBUNTU-01 | SERVERNET | 10.10.30.30 |
| LAB-SIEM-01 | SOCNET | 10.10.40.10 |

Default gateways:

| Zone | Gateway |
| --- | --- |
| REDNET | 10.10.10.1 |
| CORPNET | 10.10.20.1 |
| SERVERNET | 10.10.30.1 |
| SOCNET | 10.10.40.1 |

The endpoints use OPNsense for DNS.

---

# 17. Removing Direct NAT From Internal Systems

One of the major improvements was removing direct VirtualBox NAT access from internal hosts.

Instead of:

    Endpoint → VirtualBox NAT → Internet

the final model became:

    Endpoint
       |
    Internal Security Zone
       |
    OPNsense
       |
    WAN
       |
    VirtualBox NAT
       |
    Internet

This allows OPNsense to control and observe network traffic.

---

# 18. Firewall Alias Design

Aliases were created in OPNsense to make policies easier to understand and maintain.

Known aliases include:

## ADMIN_WORKSTATION

Address:

`10.10.20.20`

Represents:

`LAB-WIN-01`

---

## KALI_ATTACKER

Address:

`10.10.10.10`

Represents:

`LAB-KALI-01`

---

## UBUNTU_SERVER

Address:

`10.10.30.30`

Represents:

`LAB-UBUNTU-01`

---

## LAB_INTERNAL_NETS

Used to represent internal laboratory networks.

---

## PRIVATE_NETS

Contains private IPv4 ranges including:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

---

## SERVER_ADMIN_PORTS

Used for selected administrative services such as:

- TCP 22
- TCP 443

---

## DNS_PORT

Port:

`53`

---

## NTP_PORT

Port:

`123`

---

# 19. CORPNET Firewall Policy

Custom CORPNET firewall rules were created.

The intended rule structure included:

1. ALLOW Admin Workstation to OPNsense HTTPS
2. ALLOW CORPNET to OPNsense DNS
3. ALLOW CORPNET to OPNsense NTP
4. BLOCK Unauthorized CORPNET Access to Firewall
5. ALLOW Admin Workstation to SERVERNET Admin Services
6. BLOCK CORPNET to Unauthorized Private Networks
7. ALLOW CORPNET to Internet

The default broad VirtualBox-style internal trust model was removed.

The original OPNsense rule:

`Default allow LAN to any`

was disabled after the custom rules were tested.

This moved the environment toward:

**Explicit allow + default deny**

instead of allowing unrestricted internal communication.

---

# 20. REDNET Firewall Policy

REDNET was treated as an attacker/untrusted zone.

Rules included:

1. ALLOW DNS to firewall
2. ALLOW NTP to firewall
3. ALLOW Kali ICMP to firewall
4. BLOCK REDNET to OPNsense Management
5. BLOCK REDNET to CORPNET
6. ALLOW Kali to Ubuntu Lab Testing
7. BLOCK REDNET to Unauthorized Private Networks
8. ALLOW REDNET to Internet

This allows Kali to perform controlled security exercises while preventing unrestricted access to trusted systems.

---

# 21. SERVERNET Firewall Policy

SERVERNET was protected with restrictions including:

1. ALLOW DNS
2. ALLOW NTP
3. ALLOW Ubuntu ICMP
4. BLOCK SERVERNET to OPNsense Management
5. BLOCK SERVERNET to CORPNET
6. BLOCK SERVERNET to REDNET
7. BLOCK SERVERNET to Unauthorized Private Networks
8. ALLOW SERVERNET to Internet

Later, specific exceptions were added for communication with the Wazuh SIEM.

---

# 22. Network Segmentation Testing

Connectivity was tested after migrating the machines behind OPNsense.

Linux tests included:

    ping <destination>

TCP port testing used:

    nc -zv -w 5 <destination-IP> <port>

Windows TCP testing used:

    Test-NetConnection <destination-IP> -Port <port>

Examples of expected policy behavior:

- Kali → Windows: blocked
- Ubuntu → Windows: blocked
- Ubuntu → Kali: blocked
- Kali → authorized Ubuntu services: allowed
- Windows → authorized Ubuntu administrative services: allowed
- Internal systems → Internet: allowed where explicitly permitted

These tests verified that OPNsense was enforcing network segmentation.

---

# 23. Stateful Firewall Behavior

OPNsense is a stateful firewall.

This means that if a permitted system initiates an authorized connection, return traffic for that connection can pass through the existing state.

This allows controlled communication without requiring dangerous broad bidirectional rules.

Understanding stateful inspection became an important networking lesson during the project.

---

# 24. GitHub Repository

The laboratory is documented publicly in:

`Emmanuel-Ngari/cybersecurity-homelab`

Known documentation created during the project includes:

- `architecture/lab-architecture.md`
- `architecture/virtualbox-networking.md`
- `virtual-machines/kali.md`
- `virtual-machines/windows.md`
- `firewall/opnsense-firewall.md`
- `firewall/security-zones.md`
- `firewall/policy-matrix.md`
- `firewall/firewall-rules.md`

The repository will continue growing as new phases are completed.

---

# 25. Known Git Commit Messages

Known commits include:

`docs: document OPNsense firewall deployment and initial policy`

`docs: document OPNsense security zones and trust model`

`docs: add firewall policy matrix and segmentation rules`

`docs: document implemented OPNsense firewall rules`

Future documentation should continue using clear professional commit messages.

---

# 26. Adding SOCNET

A dedicated Security Operations network was added after the primary network segmentation was stable.

New interface:

`em4`

OPNsense internal identifier:

`opt3`

Description:

`SOCNET`

OPNsense address:

`10.10.40.1/24`

Network:

`10.10.40.0/24`

The interface was enabled and configured as part of the firewall.

The route was verified as:

`10.10.40.0/24`

through:

`em4`

SOCNET was also included in outbound NAT so systems in SOCNET could access the Internet for updates and package installations.

---

# 27. SOCNET Security Model

SOCNET is intended to be a protected monitoring zone.

The design principle is:

- Endpoints may send explicitly authorized monitoring traffic to SOCNET.
- SOCNET should not automatically receive unrestricted access to every internal zone.
- Administrative access should remain controlled.
- Monitoring services should use specific required ports.

This reduces the risk of turning the SIEM server into an unrestricted pivot point.

---

# 28. Building LAB-SIEM-01

A dedicated Ubuntu Server VM was created.

VM name:

`LAB-SIEM-01`

Hostname:

`lab-siem-01`

User:

`emmanuel`

Operating system:

`Ubuntu Server 26.04.1 LTS`

Architecture:

`x86_64`

Approximate allocated memory:

`8 GB`

Virtual disk:

`60 GB`

Network adapter:

Internal Network

Network name:

`SOC-NET`

IP address:

`10.10.40.10/24`

Gateway:

`10.10.40.1`

DNS:

`10.10.40.1`

---

# 29. LAB-SIEM-01 Storage Configuration

Ubuntu's installer initially allocated only approximately 29 GB of the virtual disk to the root logical volume.

Because SIEM systems can generate significant log and index data, the root logical volume was expanded during installation.

Final root filesystem capacity became approximately:

`53–54 GB`

A small amount of unused space remained in the volume group.

This was a useful lesson in Linux LVM configuration and SIEM storage planning.

---

# 30. SIEM Network Verification

After installation, connectivity was tested.

Gateway test:

    ping 10.10.40.1

External connectivity:

    ping 1.1.1.1

DNS resolution was also tested.

HTTPS connectivity was tested using `curl`.

The SIEM server successfully reached:

- OPNsense gateway
- Internet
- DNS

At the same time, firewall segmentation prevented unauthorized SOCNET initiation into:

- CORPNET
- REDNET
- SERVERNET

Management access to OPNsense from SOCNET was also restricted.

---

# 31. SSH Administration of LAB-SIEM-01

SSH was installed during Ubuntu Server deployment.

From Windows, connectivity was tested with:

    Test-NetConnection 10.10.40.10 -Port 22

The connection succeeded.

SSH login was then tested:

    ssh emmanuel@10.10.40.10

This proved the authorized path:

    LAB-WIN-01
        |
    CORPNET
        |
    OPNsense
        |
    SOCNET
        |
    LAB-SIEM-01

---

# 32. Installing Wazuh

Wazuh was selected as the primary SIEM/XDR platform.

Version installed:

`4.14.7`

The official installer was downloaded from:

`https://packages.wazuh.com/4.14/wazuh-install.sh`

The all-in-one installation was executed using:

    sudo bash ./wazuh-install.sh -a

The installation deployed:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Ubuntu 26.04 produced a compatibility warning because it was not included in the installer's recommended Ubuntu versions.

Because the system is an isolated laboratory and not production infrastructure, the installation was allowed to continue and was thoroughly tested.

---

# 33. Initial Wazuh Verification

Major Wazuh services were verified after installation.

Relevant service checks included:

    sudo systemctl status wazuh-manager

    sudo systemctl status wazuh-indexer

    sudo systemctl status wazuh-dashboard

Filebeat was also confirmed running.

The Wazuh dashboard became accessible at:

`https://10.10.40.10`

---

# 34. Wazuh Communication Ports

Important Wazuh communication ports currently used include:

## Agent communication

TCP:

`1514`

---

## Agent registration/enrollment

TCP:

`1515`

---

## Wazuh Dashboard

TCP:

`443`

---

## Wazuh Indexer / OpenSearch

TCP:

`9200`

The indexer is primarily accessed locally by Wazuh components.

---

# 35. Wazuh Firewall Aliases

Additional aliases were created for Wazuh.

Known aliases include:

- `WAZUH_SERVER`
- `WAZUH_AGENT_PORTS`
- `WAZUH_DASHBOARD_PORT`

These were used to make the firewall configuration readable and maintainable.

---

# 36. CORPNET Wazuh Rules

Rules were added to permit the Windows workstation to communicate with Wazuh.

Examples included:

`ALLOW - Windows Endpoint to Wazuh Agent Services`

Source:

`ADMIN_WORKSTATION`

Destination:

`WAZUH_SERVER`

Ports:

`WAZUH_AGENT_PORTS`

Another rule allowed:

`ADMIN_WORKSTATION`

to access:

`WAZUH_SERVER`

using:

`WAZUH_DASHBOARD_PORT`

These rules were placed above broad internal private-network blocking rules.

---

# 37. Windows Wazuh Agent Installation

Wazuh Agent version:

`4.14.7`

was installed on:

`LAB-WIN-01`

The first MSI installation attempt contained a command problem.

The installation command was corrected.

The successful installation returned:

`Exit code 0`

The Wazuh Windows service was inspected with:

    Get-Service WazuhSvc

The service initially showed:

`Stopped`

It was started using:

    Start-Service WazuhSvc

The service then showed:

`Running`

---

# 38. Windows Wazuh Connectivity Tests

Windows successfully tested communication with the Wazuh server on agent ports.

The following paths were verified:

`10.10.20.20 → 10.10.40.10:1514`

and:

`10.10.20.20 → 10.10.40.10:1515`

The Windows agent eventually appeared in Wazuh as:

`LAB-WIN-01`

IP:

`10.10.20.20`

Status:

`Active`

Agent version:

`4.14.7`

---

# 39. Windows Wazuh Telemetry

The Windows endpoint began providing security data to Wazuh.

Observed information included:

- System inventory
- Security Configuration Assessment
- Compliance information
- Vulnerability information
- Security events

This proved the monitoring path:

    LAB-WIN-01
         |
    CORPNET
         |
    OPNsense
         |
    SOCNET
         |
    LAB-SIEM-01

---

# 40. Preparing Ubuntu for Wazuh

Before modifying the Ubuntu server, a snapshot was created:

`Ubuntu - Pre Wazuh Agent`

Dependencies were installed for repository configuration.

Known package installation:

    sudo apt install gnupg apt-transport-https

Repository configuration produced several real troubleshooting issues.

These included:

- GPG import failure
- DNS timeouts
- Incorrect keyring path
- Typing mistakes
- IPv4/IPv6 connectivity complications

---

# 41. Wazuh GPG Key Failure

The first Wazuh signing-key import failed with:

`gpg: no valid OpenPGP data found`

Instead of assuming the key was successfully imported, the problem was investigated.

DNS connectivity toward:

`packages.wazuh.com`

was tested.

---

# 42. Wazuh DNS Troubleshooting

DNS was tested using:

    resolvectl query packages.wazuh.com

The query initially timed out.

Later attempts returned valid DNS responses.

IPv4 was forced when retrieving the signing key.

Command used:

    curl -4 -fsSL --max-time 30 https://packages.wazuh.com/key/GPG-KEY-WAZUH | head

Successful output began with:

`-----BEGIN PGP PUBLIC KEY BLOCK-----`

This proved that the key was actually being retrieved.

---

# 43. Wazuh Signing Key Verification

The valid Wazuh key was placed at:

`/usr/share/keyrings/wazuh.gpg`

The key was verified using:

    gpg --show-keys /usr/share/keyrings/wazuh.gpg

The output identified:

`Wazuh.com (Wazuh Signing Key)`

Email:

`support@wazuh.com`

The signing key was RSA 4096-bit.

---

# 44. Repository Typing Error

A repository configuration problem occurred because:

`keystrings`

was typed instead of:

`keyrings`

This caused repository configuration problems.

The path was corrected.

This became an important lesson:

Always verify repository paths and filenames instead of assuming installation failures are caused by networking or the package vendor.

---

# 45. Correct Wazuh Repository

The repository configuration was corrected to:

`deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main`

APT metadata was refreshed using IPv4:

    sudo apt-get -o Acquire::ForceIPv4=true update

The repository update completed without signature errors.

---

# 46. SERVERNET to SOCNET Connectivity Failure

Before installing the Ubuntu Wazuh agent, communication was tested.

Port 1514:

    nc -zv -w 5 10.10.40.10 1514

Port 1515:

    nc -zv -w 5 10.10.40.10 1515

Initially the connections failed.

OPNsense Live View showed the traffic being blocked.

An unexpected firewall label appeared similar to:

`BLOCK - SERVERNET to REDNET`

even though the destination was the SOCNET Wazuh server.

This indicated a firewall rule configuration problem.

---

# 47. Firewall Troubleshooting Lesson

The SERVERNET firewall rules were reviewed.

A rule was corrected so that intended Wazuh traffic to SOCNET would not accidentally match a blocking rule.

After correction:

    nc -zv -w 5 10.10.40.10 1514

succeeded.

The second test:

    nc -zv -w 5 10.10.40.10 1515

also succeeded.

This demonstrated why firewall logging is important.

The troubleshooting process was:

1. Test connectivity
2. Observe failure
3. Check firewall logs
4. Identify matching rule
5. Review rule configuration
6. Correct policy
7. Retest
8. Verify success

---

# 48. Ubuntu Wazuh Agent Installation

The Ubuntu Wazuh agent was installed using:

    sudo env WAZUH_MANAGER="10.10.40.10" WAZUH_REGISTRATION_SERVER="10.10.40.10" WAZUH_AGENT_NAME="LAB-UBUNTU-01" apt-get install -y wazuh-agent

Installed version:

`4.14.7-1`

Systemd was refreshed:

    sudo systemctl daemon-reload

The service was enabled:

    sudo systemctl enable wazuh-agent

The service was started:

    sudo systemctl start wazuh-agent

Service state was checked:

    sudo systemctl is-active wazuh-agent

Result:

`active`

---

# 49. Ubuntu Agent Enrollment Verification

Wazuh agent logs confirmed the enrollment sequence.

Observed events included:

- Requesting authentication key
- No authentication password provided
- Authentication key received
- Attempting connection to `10.10.40.10:1514/tcp`
- Successful connection to `10.10.40.10:1514/tcp`

This proved:

    LAB-UBUNTU-01
          |
      SERVERNET
          |
       OPNsense
          |
        SOCNET
          |
      LAB-SIEM-01

---

# 50. Snapshot Management

Snapshots became part of the lab change-management workflow.

Known snapshots include:

`LAB-SIEM-01 - Wazuh + Windows Agent Verified`

`Windows - Wazuh Agent Active`

`Ubuntu - Pre Wazuh Agent`

Later recovery snapshots included:

`SIEM - Pre Indexer Timeout Recovery`

and:

`SIEM - Wazuh Windows Ubuntu Agents Verified`

Snapshots are created before significant changes whenever practical.

They are not a replacement for proper backups but provide useful rollback points during experimentation.

---

# 51. OPNsense Configuration Backup

An OPNsense configuration backup was exported privately.

The OPNsense XML backup must not be uploaded publicly because it can contain sensitive configuration information.

Sensitive configuration files are intentionally excluded from the public GitHub repository.

---

# 52. Host Disk-Full Incident

A major laboratory incident occurred when the physical Windows host completely ran out of disk space.

The C: drive showed approximately:

`0.00 GB free`

VirtualBox reported:

`Host system reported disk full. VM execution is suspended.`

VirtualBox error identifier:

`DrvVD_DISKFULL`

Affected machines included:

- LAB-WIN-01
- LAB-SIEM-01

VirtualBox suspended the machines to avoid further write failures.

---

# 53. Host Storage Investigation

Temporary files were initially cleared.

This recovered very little space.

A large-file investigation was then performed.

Large files discovered included approximately:

- 45.7 GB FIFA archive
- 6.61 GB Windows ISO copies
- 4.17 GB Kali ISO
- Multiple approximately 2.73 GB Ubuntu ISOs
- Other game files
- Partial download files

The largest unnecessary file identified was:

`C:\Users\USER\Downloads\Quick Share\FIFA23-SteamRIP.com.rar`

Approximate size:

`45.7 GB`

---

# 54. Safe Storage Cleanup

Only the unnecessary 45.7 GB archive was deleted at that stage.

The host then showed approximately:

`45.70 GB free`

Important safety rule:

The following VirtualBox items were NOT manually deleted:

- `.vdi` files
- `.vbox` files
- VM folders
- Snapshot folders
- Snapshot disk files

Deleting those manually could destroy or corrupt virtual machines.

---

# 55. Wazuh Problem After Disk-Full Incident

After resuming the SIEM VM, the Wazuh dashboard displayed:

`Wazuh dashboard server is not ready yet`

Dashboard logs showed repeated errors:

`connect ECONNREFUSED 127.0.0.1:9200`

This indicated that the dashboard could not communicate with the Wazuh Indexer/OpenSearch service.

---

# 56. Checking Wazuh Indexer Port

The local OpenSearch port was checked using:

    sudo ss -lntp | grep ':9200'

Initially no listener was returned.

This confirmed that OpenSearch was not yet listening on TCP 9200.

---

# 57. Inspecting Wazuh Indexer Service

The indexer was inspected using:

    sudo systemctl status wazuh-indexer --no-pager -l

The service showed a failure related to startup timeout.

The Java process had exited with status:

`143`

The indexer had been consuming significant CPU and memory while attempting startup.

---

# 58. Inspecting Indexer Journal

The indexer journal was inspected using:

    sudo journalctl -u wazuh-indexer -n 80 --no-pager

The journal showed:

- OpenSearch startup
- Java warnings
- SecurityManager deprecation warnings
- Plugin initialization
- Systemd startup timeout

No clear evidence of catastrophic index corruption was visible.

---

# 59. Inspecting the Wazuh Indexer Log

The indexer application log was inspected using:

    sudo tail -n 100 /var/log/wazuh-indexer/wazuh-cluster.log

The log showed many OpenSearch modules and plugins loading.

Examples included modules relating to:

- OpenSearch Dashboards
- Painless
- Percolator
- Reindex
- Search pipeline
- Transport
- Alerting
- Anomaly detection
- Security
- SQL
- k-NN
- Neural search
- Reports

Warnings and an audit logging configuration message were visible.

However, the log did not show a clear catastrophic corruption failure.

The important conclusion was:

Do not destroy or modify index data without evidence.

---

# 60. Systemd Timeout Investigation

The service timeout configuration was investigated.

Command:

    systemctl show wazuh-indexer -p TimeoutStartUSec -p TimeoutStopUSec

Observed value:

`TimeoutStopUSec=infinity`

Another check was performed:

    systemctl show wazuh-indexer --property=TimeoutStartUSec

No useful startup timeout value was returned.

The actual service file was then inspected:

    systemctl cat wazuh-indexer

Important values included:

`Type=notify`

and:

`OPENSEARCH_SD_NOTIFY=true`

The unit also contained:

`TimeoutStopSec=0`

A systemd override was considered but not applied because more evidence was needed first.

---

# 61. Recovery Snapshot Before Changes

Before attempting any potentially significant recovery change, a snapshot was created:

`SIEM - Pre Indexer Timeout Recovery`

This created a rollback point before modifying the Wazuh service.

---

# 62. Decision Not to Make Unnecessary Changes

Potential recovery options included:

- Increasing systemd timeout
- Editing Wazuh configuration
- Deleting locks
- Rebuilding indexes
- Restoring a snapshot
- Reinstalling Wazuh

None of these were immediately performed.

The reason was that the logs showed continued OpenSearch initialization and no confirmed corruption.

A clean shutdown/reboot was selected as the safer next diagnostic step.

---

# 63. Clean Shutdown Procedure

The virtual machines were shut down cleanly.

Linux shutdown command:

    sudo poweroff

Windows was shut down normally through the operating system.

OPNsense was powered off properly.

VirtualBox was then closed.

The physical Windows host was shut down normally.

This was preferable to using VirtualBox Save State after the disk-full incident.

---

# 64. Lab Startup Order

When the lab was started again, systems were brought online in dependency order.

First:

`LAB-FW-01`

The firewall was allowed to fully boot.

Verified networks included:

- CORPNET `10.10.20.1`
- REDNET `10.10.10.1`
- SERVERNET `10.10.30.1`
- SOCNET `10.10.40.1`

Then:

`LAB-SIEM-01`

was started.

This ensured routing and DNS were available before Wazuh started.

---

# 65. Wazuh Indexer Recovery

After the clean reboot, the Wazuh Indexer was checked again:

    sudo systemctl status wazuh-indexer --no-pager -l

This time the service reported:

`active (running)`

The Java/OpenSearch process was active.

Approximate memory usage was around:

`1.7 GB`

This proved that the indexer could recover without modifying the systemd timeout.

---

# 66. Verifying TCP 9200 After Recovery

The indexer listener was checked:

    sudo ss -lntp | grep ':9200'

The output showed the Java process listening on:

`127.0.0.1:9200`

This confirmed that OpenSearch was now operational.

---

# 67. Wazuh Dashboard Recovery

The dashboard service was checked:

    sudo systemctl status wazuh-dashboard --no-pager -l

Result:

`active (running)`

The dashboard completed plugin initialization and began listening on HTTPS.

The previous:

`ECONNREFUSED 127.0.0.1:9200`

condition was resolved.

---

# 68. Windows to Wazuh Dashboard Test

LAB-WIN-01 was started.

PowerShell was used to verify dashboard connectivity:

    Test-NetConnection 10.10.40.10 -Port 443

Observed values included:

`ComputerName : 10.10.40.10`

`RemoteAddress : 10.10.40.10`

`RemotePort : 443`

`InterfaceAlias : CORP-NET`

`SourceAddress : 10.10.20.20`

`TcpTestSucceeded : True`

This proved the complete path:

    LAB-WIN-01
    10.10.20.20
         |
      CORPNET
         |
      OPNsense
         |
       SOCNET
         |
    LAB-SIEM-01
    10.10.40.10:443

---

# 69. Dashboard Browser Verification

Microsoft Edge on LAB-WIN-01 was used to access:

`https://10.10.40.10`

The Wazuh dashboard loaded successfully.

The dashboard showed security-event information and normal operation.

This confirmed successful recovery of:

- Indexer
- Dashboard
- HTTPS access
- Firewall connectivity

---

# 70. Ubuntu Agent Persistence Test

LAB-UBUNTU-01 was started after the full shutdown.

The agent was checked using:

    sudo systemctl is-active wazuh-agent

Result:

`active`

This proved that the Wazuh agent was correctly configured to start automatically after reboot.

---

# 71. Final Agent Verification

The Wazuh dashboard was opened on LAB-WIN-01.

Two agents appeared.

## LAB-WIN-01

IP:

`10.10.20.20`

Operating system:

Windows 11 Enterprise

Agent version:

`4.14.7`

Status:

`Active`

---

## LAB-UBUNTU-01

IP:

`10.10.30.30`

Operating system:

Ubuntu 26.04.1 LTS

Agent version:

`4.14.7`

Status:

`Active`

Dashboard summary showed:

- Active agents: 2
- Disconnected agents: 0
- Pending agents: 0

This completed the first multi-zone Wazuh deployment.

---

# 72. Final SIEM Recovery Snapshot

After both endpoints were verified active, a new recovery snapshot was created:

`SIEM - Wazuh Windows Ubuntu Agents Verified`

This snapshot represents the current known-good SIEM state.

---

# 73. Current Architecture

The current environment is approximately:

                         INTERNET
                            |
                     VirtualBox NAT
                            |
                       LAB-FW-01
                        OPNsense
                            |
       ------------------------------------------------
       |              |              |                |
     REDNET         CORPNET       SERVERNET         SOCNET
  10.10.10.0/24  10.10.20.0/24 10.10.30.0/24   10.10.40.0/24
       |              |              |                |
 LAB-KALI-01      LAB-WIN-01    LAB-UBUNTU-01   LAB-SIEM-01
 10.10.10.10      10.10.20.20    10.10.30.30     10.10.40.10
       |              |              |                |
 Offensive        Corporate        Server            SOC
 Security         Endpoint       Monitoring          Wazuh

---

# 74. Current OPNsense Addressing

| Interface | Network | OPNsense IP |
| --- | --- | --- |
| WAN | VirtualBox NAT | 10.0.2.15 |
| REDNET | 10.10.10.0/24 | 10.10.10.1 |
| CORPNET | 10.10.20.0/24 | 10.10.20.1 |
| SERVERNET | 10.10.30.0/24 | 10.10.30.1 |
| SOCNET | 10.10.40.0/24 | 10.10.40.1 |

---

# 75. Current Host Roles

## LAB-FW-01

Role:

- Firewall
- Router
- NAT gateway
- Segmentation enforcement
- DNS path
- Future IDS/IPS platform

---

## LAB-KALI-01

Role:

- Offensive security workstation
- Reconnaissance
- Penetration testing
- Controlled attacker system

---

## LAB-WIN-01

Role:

- Corporate endpoint
- Windows monitoring target
- Administrative workstation
- Wazuh dashboard access system

---

## LAB-UBUNTU-01

Role:

- Linux server
- Defensive monitoring target
- Authorized attack target
- Server security laboratory

---

## LAB-SIEM-01

Role:

- Security Operations server
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat
- Centralized endpoint monitoring

---

# 76. Major Skills Practiced So Far

The project has provided practical experience with:

## Virtualization

- VirtualBox
- VM creation
- Virtual disks
- VM snapshots
- Internal networking
- NAT
- Resource allocation

## Networking

- IPv4 addressing
- Subnetting
- Default gateways
- Routing
- DNS
- NAT
- Stateful firewalling
- Network segmentation
- TCP port testing
- ICMP testing

## Linux

- Ubuntu Server
- Kali Linux
- systemd
- APT
- GPG signing keys
- NetworkManager
- Linux networking
- LVM
- Log inspection
- Service troubleshooting

## Windows

- PowerShell
- Service management
- Network testing
- Windows endpoint monitoring

## Firewall Administration

- OPNsense
- Interface assignments
- Security zones
- Firewall aliases
- Explicit allow rules
- Explicit block rules
- Stateful firewall behavior
- Log analysis
- Cross-zone troubleshooting

## SIEM

- Wazuh
- Endpoint agents
- Agent enrollment
- Indexer
- Dashboard
- Manager
- Filebeat
- Security Configuration Assessment
- Vulnerability information
- Endpoint inventory

## Troubleshooting

- DNS failures
- Firewall rule mismatch
- Package signing issues
- Repository problems
- Typing errors
- TCP connectivity issues
- VirtualBox disk exhaustion
- OpenSearch startup problems
- Service dependency problems

## Documentation

- GitHub
- Markdown
- Architecture documentation
- Firewall documentation
- Change history
- Technical journaling

---

# 77. Important Commands Learned

## Linux System Information

    uname -a

    free -h

    lscpu

---

## Linux Networking

    ip addr

    ip a

    ip route

    nmcli

    ping <destination>

---

## DNS Testing

    resolvectl query <hostname>

Example:

    resolvectl query packages.wazuh.com

---

## HTTP/HTTPS Testing

    curl <URL>

Force IPv4:

    curl -4 <URL>

Wazuh key test:

    curl -4 -fsSL --max-time 30 https://packages.wazuh.com/key/GPG-KEY-WAZUH | head

---

## TCP Testing with Netcat

    nc -zv -w 5 <IP> <port>

Examples:

    nc -zv -w 5 10.10.40.10 1514

    nc -zv -w 5 10.10.40.10 1515

---

## PowerShell TCP Testing

    Test-NetConnection <IP> -Port <port>

Example:

    Test-NetConnection 10.10.40.10 -Port 443

---

## SSH

    ssh <username>@<IP>

Example:

    ssh emmanuel@10.10.40.10

---

## Linux Service Status

    sudo systemctl status <service>

---

## Check Whether Service Is Active

    sudo systemctl is-active <service>

Example:

    sudo systemctl is-active wazuh-agent

---

## Enable Service at Boot

    sudo systemctl enable <service>

Example:

    sudo systemctl enable wazuh-agent

---

## Start Service

    sudo systemctl start <service>

Example:

    sudo systemctl start wazuh-agent

---

## Reload systemd

    sudo systemctl daemon-reload

---

## View Service Logs

    sudo journalctl -u <service> -n 80 --no-pager

Example:

    sudo journalctl -u wazuh-indexer -n 80 --no-pager

---

## Check Listening Ports

    sudo ss -lntp

Specific port:

    sudo ss -lntp | grep ':9200'

---

## View Application Log

    sudo tail -n 100 /var/log/wazuh-indexer/wazuh-cluster.log

---

## Inspect systemd Unit

    systemctl cat wazuh-indexer

---

## Inspect systemd Properties

    systemctl show wazuh-indexer -p TimeoutStartUSec -p TimeoutStopUSec

---

## Verify GPG Key

    gpg --show-keys /usr/share/keyrings/wazuh.gpg

---

## Refresh APT Using IPv4

    sudo apt-get -o Acquire::ForceIPv4=true update

---

## Linux Shutdown

    sudo poweroff

---

## Windows Wazuh Service

Check:

    Get-Service WazuhSvc

Start:

    Start-Service WazuhSvc

---

# 78. Important Wazuh Commands Used

Wazuh all-in-one installation:

    sudo bash ./wazuh-install.sh -a

Ubuntu Wazuh Agent installation:

    sudo env WAZUH_MANAGER="10.10.40.10" WAZUH_REGISTRATION_SERVER="10.10.40.10" WAZUH_AGENT_NAME="LAB-UBUNTU-01" apt-get install -y wazuh-agent

Enable agent:

    sudo systemctl enable wazuh-agent

Start agent:

    sudo systemctl start wazuh-agent

Check agent:

    sudo systemctl is-active wazuh-agent

Check indexer:

    sudo systemctl status wazuh-indexer --no-pager -l

Check dashboard:

    sudo systemctl status wazuh-dashboard --no-pager -l

Check OpenSearch TCP 9200:

    sudo ss -lntp | grep ':9200'

Inspect indexer logs:

    sudo journalctl -u wazuh-indexer -n 80 --no-pager

Inspect Wazuh cluster log:

    sudo tail -n 100 /var/log/wazuh-indexer/wazuh-cluster.log

---

# 79. Important Troubleshooting Principles Learned

## Do Not Guess

When something fails:

1. Reproduce the problem
2. Inspect logs
3. Verify networking
4. Check configuration
5. Make one controlled change
6. Test again

---

## Use Firewall Logs

The SERVERNET → SOCNET Wazuh issue showed that firewall logs can identify exactly which rule is blocking traffic.

Do not simply add broad allow rules.

---

## Verify Downloaded Keys

A command completing does not automatically mean a valid signing key was received.

The downloaded content should be inspected and the imported key verified.

---

## Check Paths Carefully

Small typing errors such as:

`keystrings`

instead of:

`keyrings`

can break an entire package installation process.

---

## Avoid Destructive Recovery Without Evidence

During the Wazuh Indexer incident, no data directories, indexes or lock files were deleted because there was no proof that they were corrupt.

---

## Clean Restart Before Deep Repair

The Wazuh Indexer ultimately recovered after a clean shutdown and restart.

A restart should not always be the first troubleshooting step, but once logs and state were understood, it was a safe diagnostic action.

---

# 80. Storage Management Lesson

The host disk-full incident demonstrated that virtualization requires substantial storage planning.

A professional lab must account for:

- VM disks
- Snapshots
- ISO files
- Log growth
- SIEM index growth
- Downloads
- Temporary files

The physical host should maintain significant free space.

The lab should avoid reaching zero free disk space again.

Future storage management should include:

- Removing duplicate ISOs safely
- Archiving unused installers
- Monitoring disk usage
- Reviewing snapshot growth
- Avoiding unnecessary large downloads on the VM host
- Never manually deleting active VirtualBox disk or snapshot files

---

# 81. Snapshot Philosophy

Snapshots should be taken before:

- Major network changes
- Firewall redesign
- Security-agent installation
- SIEM configuration
- IDS/IPS installation
- Active Directory changes
- Major upgrades
- Risky troubleshooting

Snapshots should also be created after major milestones are verified.

The working process is:

**Snapshot → Change → Test → Verify → Document**

---

# 82. Security Lessons Learned

The lab has demonstrated several important security principles.

## Segmentation

Systems with different trust levels should not automatically share one flat network.

---

## Least Privilege Networking

Only required traffic should be allowed.

---

## Default Deny

Broad internal allow rules should be replaced with specific policies where possible.

---

## Defense in Depth

Security is being implemented through multiple layers:

- Network segmentation
- Firewall controls
- Endpoint monitoring
- SIEM
- Future IDS/IPS
- Future Active Directory monitoring

---

## Logging Matters

Logs are required to understand:

- Why something was blocked
- Why a service failed
- Whether an attack occurred
- Whether a security rule is working

---

# 83. How I Can Explain the Lab in an Interview

A concise explanation:

I started my cybersecurity homelab with three VirtualBox machines: Kali Linux, Windows and Ubuntu Server on a shared internal network.

After learning basic virtualization and networking, I redesigned the environment into a segmented enterprise-style architecture using OPNsense as the central firewall and router.

I created separate security zones for offensive systems, corporate endpoints, servers and security monitoring.

Kali is isolated inside REDNET, Windows is in CORPNET, Ubuntu is in SERVERNET, and a dedicated Wazuh SIEM server is located in SOCNET.

All inter-zone traffic passes through OPNsense. I removed direct NAT access from the internal machines so routing and Internet access are controlled by the firewall.

I created explicit firewall policies and aliases to restrict communication between zones while permitting required services.

I then deployed Wazuh as an all-in-one SIEM solution and installed agents on both Windows and Ubuntu systems.

The Windows endpoint sends telemetry from CORPNET to SOCNET, while the Ubuntu server sends telemetry from SERVERNET to SOCNET.

During the project I troubleshot firewall rule matching, DNS problems, Linux package-signing keys, repository configuration, agent enrollment, VirtualBox disk exhaustion and a Wazuh Indexer startup problem.

I use snapshots before major changes, verify each configuration with connectivity tests and logs, and document the environment in GitHub.

The next stages of the project include Sysmon, enhanced Linux auditing, OPNsense log forwarding, Suricata IDS, Active Directory, a DMZ, vulnerable systems and complete attack-detection-response exercises.

---

# 84. Current Project Status

## Completed

- VirtualBox installed
- Kali deployed
- Windows deployed
- Ubuntu Server deployed
- Basic networking learned
- Original LAB-NET created
- OPNsense deployed
- REDNET created
- CORPNET created
- SERVERNET created
- SOCNET created
- Static addressing configured
- Direct endpoint NAT removed
- OPNsense routing configured
- OPNsense outbound NAT working
- Firewall aliases created
- Firewall policies implemented
- Cross-zone testing completed
- GitHub repository created
- Architecture documentation started
- Firewall documentation created
- LAB-SIEM-01 deployed
- Wazuh 4.14.7 deployed
- Windows Wazuh agent deployed
- Ubuntu Wazuh agent deployed
- Windows endpoint active in Wazuh
- Ubuntu endpoint active in Wazuh
- Wazuh dashboard reachable through OPNsense
- Wazuh Indexer recovery completed
- Recovery snapshots created

---

# 85. Current Lab Maturity

Current approximate maturity:

## Virtualization Foundation

Complete.

## Network Segmentation

Complete for current networks.

## OPNsense Firewall

Operational.

## Offensive Workstation

Operational.

## Windows Endpoint

Operational.

## Ubuntu Server

Operational.

## SOC Network

Operational.

## Wazuh SIEM

Operational.

## Endpoint Monitoring

Operational for Windows and Ubuntu.

## IDS/IPS

Not yet deployed.

## Active Directory

Not yet deployed.

## DMZ

Not yet deployed.

## Vulnerable Targets

Not yet deployed.

## Attack Detection Exercises

Not yet started.

## Incident Response Exercises

Not yet started.

---

# 86. Planned Next Phase — Enhanced Endpoint Telemetry

The next defensive phase should improve endpoint visibility.

Planned Windows improvements include:

- Sysmon
- Enhanced Windows event collection
- PowerShell logging
- Process creation monitoring
- Network connection telemetry
- Wazuh integration
- Detection validation

Planned Linux improvements include:

- auditd
- Authentication monitoring
- SSH monitoring
- Privilege escalation monitoring
- File integrity monitoring
- Service monitoring

---

# 87. Planned OPNsense Log Integration

Future work will forward OPNsense logs to the SIEM.

This will allow Wazuh to receive information about:

- Firewall blocks
- Allowed connections
- Authentication activity
- Network-policy violations
- Possible reconnaissance
- Attack traffic

This will improve the SOC view of the network.

---

# 88. Planned Suricata Deployment

Suricata will later be added as the network intrusion detection platform.

Planned goals:

- IDS monitoring
- Signature-based detection
- Suspicious traffic alerts
- Network attack detection
- Wazuh alert integration

Suricata should initially operate in IDS mode before considering IPS blocking.

---

# 89. Planned Active Directory Environment

A future network is reserved:

`AD-NET`

Network:

`10.10.50.0/24`

Planned components include:

- Windows Server
- Active Directory Domain Services
- DNS
- Domain Controller
- Domain users
- Group Policy
- Windows client domain membership

This environment will support both attack and defense exercises.

---

# 90. Planned DMZ

Future DMZ network:

`10.10.60.0/24`

Potential systems include:

- Web servers
- Vulnerable applications
- Linux servers
- Security training targets

The DMZ will allow realistic external-facing service simulations.

---

# 91. Planned Attack and Detection Exercises

Once monitoring is mature, controlled attacks will be performed from Kali.

Examples may include:

- Port scanning
- Failed login attempts
- SSH brute-force simulations
- Suspicious PowerShell
- Credential attacks
- Web attacks
- Privilege escalation
- Lateral movement
- Active Directory attacks

The goal will not simply be successful exploitation.

Every exercise should include:

1. Attack
2. Detection
3. Alert review
4. Investigation
5. Response
6. Documentation

---

# 92. Planned Incident Response Workflow

Future incidents will follow a structured process.

Example:

1. Preparation
2. Detection
3. Triage
4. Investigation
5. Containment
6. Eradication
7. Recovery
8. Lessons learned
9. Documentation

The lab will eventually be used to write realistic SOC incident reports.

---

# 93. Professional Development Connection

The homelab supports my broader cybersecurity learning path.

Current certification roadmap includes:

1. Google Cybersecurity Certificate — completed
2. CCNA
3. CompTIA Security+
4. Microsoft SC-200
5. OSCP+ later

ISC2 CC is considered optional because of overlap with existing foundational training.

The laboratory is intended to provide hands-on experience alongside certification study.

---

# 94. Learning Roadmap

The broader learning sequence is:

Cybersecurity Fundamentals  
↓  
Networking Fundamentals  
↓  
Linux and Windows Administration  
↓  
Network Security  
↓  
SOC Operations  
↓  
Blue Team  
↓  
Incident Response and Threat Detection  
↓  
Offensive Security and Ethical Hacking  
↓  
Penetration Testing  
↓  
Advanced Red Team  
↓  
Advanced Cybersecurity Professional

---

# 95. Rules for Continuing the Project

Every future major change should follow:

1. Understand the objective
2. Take a snapshot where appropriate
3. Make one controlled change
4. Test the result
5. Inspect logs if something fails
6. Verify expected security behavior
7. Create a stable snapshot after major milestones
8. Update GitHub
9. Update this journal

Avoid randomly installing tools without understanding their role in the architecture.

---

# 96. Documentation Security Rules

The public repository must not contain:

- Passwords
- Private keys
- Authentication tokens
- Wazuh passwords
- OPNsense backup XML
- Sensitive certificates
- Personal credentials
- Secret configuration values

Screenshots should be reviewed before uploading publicly.

---

# 97. Major Lessons From the Project So Far

The most important lessons include:

1. Building the VM is only the beginning.
2. Networking knowledge is essential for cybersecurity.
3. Flat networks do not represent realistic security architecture.
4. Firewalls should enforce trust boundaries.
5. Security monitoring requires carefully designed communication paths.
6. Logs are more useful than guessing.
7. A successful installation does not guarantee a healthy service.
8. DNS problems can appear like package-manager problems.
9. Small configuration typos can cause major failures.
10. Disk capacity is critical for virtualization and SIEM systems.
11. Never make destructive recovery changes without evidence.
12. Snapshots make experimentation much safer.
13. Documentation is part of professional cybersecurity work.
14. Troubleshooting is itself a cybersecurity skill.
15. Testing both allowed and blocked traffic is necessary.
16. A professional lab should simulate architecture, not just collect tools.
17. Every attack exercise should eventually connect to detection and response.

---

# 98. Current Known-Good State

As of the latest verified milestone:

OPNsense:

`Operational`

REDNET:

`Operational`

CORPNET:

`Operational`

SERVERNET:

`Operational`

SOCNET:

`Operational`

LAB-KALI-01:

`Operational`

LAB-WIN-01:

`Operational`

LAB-UBUNTU-01:

`Operational`

LAB-SIEM-01:

`Operational`

Wazuh Manager:

`Operational`

Wazuh Indexer:

`Operational`

Wazuh Dashboard:

`Operational`

Filebeat:

`Operational`

LAB-WIN-01 Wazuh Agent:

`Active`

LAB-UBUNTU-01 Wazuh Agent:

`Active`

Dashboard HTTPS access:

`Verified`

Windows → SOCNET monitoring path:

`Verified`

Ubuntu → SOCNET monitoring path:

`Verified`

Network segmentation:

`Operational`

---

# 99. Latest Verified Snapshot

Current verified SIEM snapshot:

`SIEM - Wazuh Windows Ubuntu Agents Verified`

This represents the current stable recovery point before continuing with additional defensive-security capabilities.

---

# 100. Project Motto

**Learn → Build → Break → Troubleshoot → Fix → Test → Document → Repeat**

This journal will continue to grow as the cybersecurity homelab evolves into a complete offensive and defensive cyber range.
