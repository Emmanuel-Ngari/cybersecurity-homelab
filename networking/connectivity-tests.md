# Homelab Connectivity Testing

## 1. Overview

This document records the connectivity testing performed within my cybersecurity homelab.

The purpose of these tests is to verify that the virtual machines can communicate correctly across the isolated LAB-NET network while maintaining separate NAT connectivity for Internet access.

Connectivity testing is an important part of validating the laboratory architecture before beginning offensive security, defensive security, monitoring, and incident response exercises.

---

## 2. Laboratory Systems

The current cybersecurity homelab contains three primary virtual machines.

| System | Role | LAB-NET Address |
|---|---|---|
| Kali Linux | Offensive Security Workstation | `192.168.56.10/24` |
| Windows 11 Enterprise | Windows Endpoint / Blue Team | `192.168.56.20/24` |
| Ubuntu Server | Linux Server / Blue Team | `192.168.56.30/24` |

All systems are connected to the isolated:

`192.168.56.0/24`

LAB-NET network.

---

## 3. Testing Objectives

The connectivity tests were performed to verify:

- Correct IPv4 addressing
- Correct subnet configuration
- LAB-NET communication
- NAT connectivity
- Routing functionality
- Network interface configuration
- Windows Firewall behavior
- Ubuntu network configuration
- VirtualBox network configuration
- Communication between offensive and defensive systems

Successful connectivity provides the foundation required for future security exercises.

---

## 4. Network Design

The laboratory uses two network interfaces on each virtual machine.

### NAT

Used for:

- Internet connectivity
- Operating system updates
- Package installation
- Tool downloads
- Access to legitimate learning resources

### LAB-NET

Used for:

- Communication between laboratory systems
- Network scanning
- Service enumeration
- Packet analysis
- Security monitoring
- Controlled exploitation
- Incident response exercises

LAB-NET uses:

`192.168.56.0/24`

---

## 5. Kali Linux Network Configuration

Kali Linux uses two interfaces.

### NAT Interface

**Interface:** `eth0`

**IPv4 Address:** `10.0.2.15/24`

### LAB-NET Interface

**Interface:** `eth1`

**IPv4 Address:** `192.168.56.10/24`

Kali serves as the primary offensive security workstation.

---

## 6. Windows Network Configuration

Windows 11 Enterprise uses two network adapters.

### NAT Adapter

**Adapter Name:** `NAT`

**IPv4 Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

### LAB-NET Adapter

**Adapter Name:** `LAB-NET`

**IPv4 Address:** `192.168.56.20/24`

**Default Gateway:** None

Windows serves as the primary Windows endpoint for security monitoring and defensive exercises.

---

## 7. Ubuntu Network Configuration

Ubuntu Server uses two network interfaces.

### NAT Interface

**Interface:** `enp0s3`

**IPv4 Address:** `10.0.2.15/24`

**Default Gateway:** `10.0.2.2`

### LAB-NET Interface

**Interface:** `enp0s8`

**IPv4 Address:** `192.168.56.30/24`

**Default Gateway:** None

Ubuntu serves as the Linux server within the cybersecurity laboratory.

---

## 8. Kali to Windows Connectivity Test

Connectivity between Kali Linux and Windows 11 was tested across LAB-NET.

### Source

Kali Linux:

`192.168.56.10`

### Destination

Windows 11:

`192.168.56.20`

### Test Command

`ping -c 4 192.168.56.20`

### Result

**Status:** 🟢 Successful

Kali Linux was able to communicate with the Windows endpoint after network and firewall troubleshooting.

This confirmed that both systems were correctly connected to LAB-NET.

---

## 9. Windows to Kali Connectivity Test

Windows connectivity to Kali Linux was tested using ICMP.

### Source

Windows 11:

`192.168.56.20`

### Destination

Kali Linux:

`192.168.56.10`

### Test Command

`ping 192.168.56.10`

### Result

**Status:** 🟢 Successful

The test completed successfully with responses received from Kali Linux.

This confirmed bidirectional communication between Windows and Kali.

---

## 10. Ubuntu to Kali Connectivity Test

Ubuntu Server connectivity to Kali Linux was tested across LAB-NET.

### Source

Ubuntu Server:

`192.168.56.30`

### Destination

Kali Linux:

`192.168.56.10`

### Test Command

`ping -c 4 192.168.56.10`

### Result

**Status:** 🟢 Successful

**Packet Loss:** `0%`

Ubuntu successfully communicated with Kali Linux across the isolated network.

---

## 11. Ubuntu to Windows Connectivity Test

Ubuntu Server connectivity to Windows 11 was tested across LAB-NET.

### Source

Ubuntu Server:

`192.168.56.30`

### Destination

Windows 11:

`192.168.56.20`

### Test Command

`ping -c 4 192.168.56.20`

### Result

**Status:** 🟢 Successful

**Packet Loss:** `0%`

The successful test confirmed connectivity between the Linux server and Windows endpoint.

---

## 12. Ubuntu NAT Gateway Test

Ubuntu Server was tested against the VirtualBox NAT gateway.

### Source

Ubuntu NAT Interface:

`10.0.2.15`

### Destination

VirtualBox NAT Gateway:

`10.0.2.2`

### Test Command

`ping -c 4 10.0.2.2`

### Result

**Status:** 🟢 Successful

**Packet Loss:** `0%`

This confirmed that Ubuntu's NAT interface and default routing configuration were functioning correctly.

---

## 13. Ubuntu Persistence Test

Ubuntu's LAB-NET address was initially configured manually.

A persistent Netplan configuration was later created for:

`192.168.56.30/24`

After applying the configuration, the Ubuntu Server was rebooted.

The following command was used after reboot:

`ip addr show enp0s8`

The interface continued to show:

`192.168.56.30/24`

### Result

**Status:** 🟢 Successful

This confirmed that the LAB-NET configuration survived the reboot and was persistent.

---

## 14. Routing Verification

Routing was checked to ensure that Internet traffic and laboratory traffic used the correct interfaces.

### Ubuntu

Command:

`ip route`

The routing table confirmed:

- Default Internet route through `enp0s3`
- Default gateway through `10.0.2.2`
- LAB-NET traffic through `enp0s8`
- Connected route for `192.168.56.0/24`

### Kali

Command:

`ip route`

The routing configuration confirmed:

- NAT traffic through `eth0`
- LAB-NET traffic through `eth1`

### Windows

PowerShell command:

`Get-NetRoute`

The routing table confirmed:

- Default route through the NAT interface
- Connected LAB-NET route for `192.168.56.0/24`

This verified that traffic was being routed according to the intended network design.

---

## 15. Initial Windows Connectivity Problem

During the initial configuration, Kali Linux was unable to successfully communicate with Windows.

Windows initially had an automatically assigned APIPA address within:

`169.254.0.0/16`

on the LAB-NET interface.

The required address was:

`192.168.56.20/24`

The Windows LAB-NET interface was manually configured with the correct static IPv4 address.

---

## 16. Windows Firewall Troubleshooting

After correcting the IP addressing configuration, Windows security settings were also investigated.

The troubleshooting process included checking:

- Windows Firewall profiles
- Network profile configuration
- ICMP behavior
- Inbound firewall rules
- Network adapter state
- IP configuration
- Routing information

The Windows network profile was identified as:

`Public`

After the network and firewall issues were investigated and corrected, communication between Kali and Windows succeeded.

This exercise demonstrated that successful networking depends on more than IP addressing alone.

Endpoint firewall policies can also affect connectivity.

---

## 17. Ubuntu Netplan Troubleshooting

Ubuntu's LAB-NET address was initially configured temporarily using:

`sudo ip addr add 192.168.56.30/24 dev enp0s8`

Because this type of configuration does not normally survive a reboot, a persistent Netplan configuration was created.

During the process, a YAML formatting error produced:

`Error in network definition: expected sequence`

The error was caused by incorrect YAML formatting around the address configuration.

After correcting the indentation and address list syntax, the configuration was validated using:

`sudo netplan generate`

It was then safely tested using:

`sudo netplan try`

The configuration was successfully applied and later verified after reboot.

This provided practical experience troubleshooting Linux network configuration and YAML syntax.

---

## 18. Connectivity Troubleshooting Methodology

The following troubleshooting process was developed during the laboratory setup.

### Step 1 — Verify Interfaces

Linux:

`ip addr`

Windows:

`Get-NetAdapter`

### Step 2 — Verify IP Addresses

Linux:

`ip addr`

Windows:

`Get-NetIPAddress`

### Step 3 — Verify Routing

Linux:

`ip route`

Windows:

`Get-NetRoute`

### Step 4 — Test Local Network Connectivity

Use:

`ping`

to test communication between systems.

### Step 5 — Inspect Firewall Configuration

Windows:

`Get-NetFirewallProfile`

Linux firewall configuration can be inspected using tools such as:

`sudo ufw status`

when UFW is enabled.

### Step 6 — Verify VirtualBox Configuration

Confirm that:

- NAT adapters are enabled
- LAB-NET adapters are enabled
- Correct VirtualBox network modes are selected
- Virtual network adapters are connected

### Step 7 — Retest

Repeat the connectivity tests after every configuration change.

### Step 8 — Document the Resolution

Record:

- The original problem
- Commands used
- Configuration changes
- Test results
- Final solution

This provides a structured troubleshooting methodology that can be reused during future cybersecurity exercises.

---

## 19. Current Connectivity Status

**Overall Network Status:** 🟢 Operational

Current laboratory systems:

| System | LAB-NET Address | Status |
|---|---|---|
| Kali Linux | `192.168.56.10` | 🟢 Online |
| Windows 11 | `192.168.56.20` | 🟢 Online |
| Ubuntu Server | `192.168.56.30` | 🟢 Online |

Verified tests include:

| Test | Result |
|---|---|
| Kali → Windows | 🟢 Successful |
| Windows → Kali | 🟢 Successful |
| Ubuntu → Kali | 🟢 Successful |
| Ubuntu → Windows | 🟢 Successful |
| Ubuntu → NAT Gateway | 🟢 Successful |
| Ubuntu static IP after reboot | 🟢 Successful |

The laboratory networking foundation is operational and ready for future cybersecurity exercises.

---

## 20. Security and Testing Disclaimer

All connectivity and network security testing documented in this repository is performed within my personally controlled cybersecurity homelab.

Testing is limited to systems that I own or systems for which I have explicit authorization.

The laboratory is designed for:

- Cybersecurity education
- Network administration
- Network troubleshooting
- Defensive security
- Ethical hacking
- Security monitoring
- Incident response
- Authorized penetration testing

No unauthorized external systems, networks, accounts, or organizations are targeted.

The purpose of this environment is to develop practical cybersecurity skills through controlled, documented, and ethical experimentation.
