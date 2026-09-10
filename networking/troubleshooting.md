# Homelab Network Troubleshooting

## 1. Overview

This document records the major networking problems encountered while building my cybersecurity homelab and the troubleshooting methods used to resolve them.

Rather than only documenting the final working configuration, this file focuses on the problems, symptoms, investigation process, commands used, root causes, fixes, and lessons learned.

Troubleshooting is an essential cybersecurity skill because analysts, administrators, penetration testers, and incident responders must be able to distinguish between configuration problems, network failures, firewall restrictions, routing problems, and security controls.

---

## 2. Troubleshooting Environment

The cybersecurity homelab contains three primary systems:

| System | Role | LAB-NET Address |
|---|---|---|
| Kali Linux | Offensive Security Workstation | `192.168.56.10/24` |
| Windows 11 Enterprise | Windows Endpoint / Blue Team | `192.168.56.20/24` |
| Ubuntu Server | Linux Server / Blue Team | `192.168.56.30/24` |

The environment uses two network paths:

- NAT for Internet connectivity
- LAB-NET for isolated cybersecurity laboratory communication

LAB-NET:

`192.168.56.0/24`

---

## 3. Troubleshooting Methodology

A structured troubleshooting process is used when network communication fails.

The general methodology is:

1. Identify the problem.
2. Verify the virtual machine is running.
3. Verify the VirtualBox network adapters.
4. Check interface status.
5. Check IP addressing.
6. Check subnet configuration.
7. Check the routing table.
8. Test connectivity.
9. Check firewall configuration.
10. Check operating system network profiles.
11. Inspect services if required.
12. Make one controlled change.
13. Retest.
14. Verify persistence.
15. Document the final resolution.

This approach reduces random configuration changes and makes troubleshooting repeatable.

---

## 4. Case Study 1 — Windows APIPA Address

### Problem

The Windows LAB-NET adapter did not initially have the required laboratory IPv4 address.

Instead, Windows assigned an APIPA address.

The address was within:

`169.254.0.0/16`

### Expected Configuration

The Windows LAB-NET adapter should have been configured as:

`192.168.56.20/24`

### Impact

Because Windows was not correctly addressed on the `192.168.56.0/24` network, communication with Kali Linux and Ubuntu Server could not function as intended.

---

## 5. Understanding APIPA

APIPA stands for Automatic Private IP Addressing.

Windows can assign itself an address from:

`169.254.0.0/16`

when an interface is configured to obtain an IPv4 address automatically but cannot obtain one from a DHCP server.

In this laboratory, LAB-NET does not depend on DHCP for endpoint addressing.

Static addresses are used instead.

Therefore, the APIPA address indicated that the LAB-NET interface had not yet been configured correctly.

---

## 6. Windows APIPA Investigation

The Windows network configuration was investigated using PowerShell.

Useful commands included:

`Get-NetAdapter`

`Get-NetIPConfiguration`

`Get-NetIPAddress`

`Get-NetRoute`

These commands helped identify:

- Available network adapters
- Interface names
- Assigned IP addresses
- Interface indexes
- Default gateways
- Connected routes

The LAB-NET adapter was identified and configured separately from the NAT adapter.

---

## 7. Windows APIPA Resolution

The Windows LAB-NET adapter was assigned the static address:

`192.168.56.20/24`

No default gateway was configured on LAB-NET.

No DNS server was required on LAB-NET.

The NAT adapter remained responsible for Internet connectivity.

### Final Design

NAT:

`10.0.2.15/24`

Gateway:

`10.0.2.2`

LAB-NET:

`192.168.56.20/24`

Gateway:

None

After the static configuration was applied, Windows had a valid address on the laboratory network.

---

## 8. Case Study 2 — Kali to Windows Ping Failure

### Problem

After configuring the laboratory network, Kali Linux was initially unable to successfully ping the Windows endpoint.

### Source

Kali Linux:

`192.168.56.10`

### Destination

Windows 11:

`192.168.56.20`

### Expected Result

Kali should receive ICMP Echo Replies from Windows.

### Initial Result

Connectivity was unsuccessful.

This required investigation beyond basic IP addressing.

---

## 9. Kali to Windows Investigation

The troubleshooting process included checking:

- Kali interface configuration
- Windows interface configuration
- LAB-NET addressing
- Routing tables
- VirtualBox adapter configuration
- Windows Firewall
- Windows network profile
- ICMP traffic

Kali networking was checked using:

`ip addr`

`ip route`

Windows networking was checked using:

`Get-NetAdapter`

`Get-NetIPAddress`

`Get-NetIPConfiguration`

`Get-NetRoute`

The network configuration confirmed that both systems were intended to be on:

`192.168.56.0/24`

---

## 10. Windows Firewall and Network Profile

Windows Firewall was investigated because correct IP addressing alone does not guarantee successful communication.

The Windows network profile was identified as:

`Public`

The Public profile is generally more restrictive than a Private network profile.

Firewall profiles were inspected using:

`Get-NetFirewallProfile`

Firewall and ICMP behavior were considered during troubleshooting.

This demonstrated an important principle:

A failed ping does not automatically mean that the network itself is broken.

Traffic can also be blocked by endpoint security controls.

---

## 11. Kali to Windows Resolution

After correcting the Windows network configuration and investigating the relevant firewall settings, connectivity was restored.

Kali successfully pinged:

`192.168.56.20`

The reverse test was also performed from Windows to Kali:

`ping 192.168.56.10`

### Final Result

**Kali → Windows:** Successful

**Windows → Kali:** Successful

This confirmed bidirectional LAB-NET communication between both systems.

---

## 12. Case Study 3 — Ubuntu LAB-NET Missing IPv4 Address

### Problem

Ubuntu Server's second network interface:

`enp0s8`

initially did not have the required LAB-NET IPv4 address.

The interface needed:

`192.168.56.30/24`

Without this address, Ubuntu could not fully participate in LAB-NET communication.

---

## 13. Temporary Ubuntu Configuration

A temporary address was assigned using:

`sudo ip addr add 192.168.56.30/24 dev enp0s8`

The interface was then checked using:

`ip addr show enp0s8`

The routing table was checked using:

`ip route`

The resulting connected LAB-NET route included:

`192.168.56.0/24 dev enp0s8`

Connectivity testing confirmed that the temporary configuration worked.

However, configuration performed using the `ip` command is not intended to survive a reboot.

A persistent solution was therefore required.

---

## 14. Ubuntu Connectivity Testing

After assigning the temporary LAB-NET address, Ubuntu was tested against the other laboratory systems.

Ubuntu → Kali:

`ping -c 4 192.168.56.10`

Ubuntu → Windows:

`ping -c 4 192.168.56.20`

Ubuntu → NAT Gateway:

`ping -c 4 10.0.2.2`

### Results

All three tests completed successfully with:

`0% packet loss`

This confirmed that:

- LAB-NET communication worked
- NAT connectivity remained functional
- Routing was functioning correctly

---

## 15. Case Study 4 — Ubuntu Netplan YAML Error

### Objective

The temporary Ubuntu LAB-NET configuration needed to be made persistent.

The existing Netplan configuration was inspected before making changes.

The Netplan directory was checked using:

`ls -la /etc/netplan/`

The configuration was inspected using:

`sudo cat /etc/netplan/*.yaml`

The active configuration file was:

`/etc/netplan/00-installer-config.yaml`

A backup was created before modification.

---

## 16. Netplan Validation Error

After editing the Netplan configuration, validation produced the error:

`Error in network definition: expected sequence`

The problem was related to YAML formatting of the address configuration.

YAML is indentation-sensitive.

The LAB-NET address needed to be defined as a list under:

`addresses:`

The corrected configuration used:

`addresses:`

followed by:

`- 192.168.56.30/24`

with the required YAML indentation.

After correcting the formatting, the configuration was validated again.

---

## 17. Safe Netplan Testing and Application

The corrected configuration was first checked using:

`sudo netplan generate`

The command completed without an error.

The configuration was then tested safely using:

`sudo netplan try`

This approach provided rollback protection in case the new configuration caused network connectivity problems.

After confirmation, Ubuntu was checked using:

`ip addr show enp0s8`

and:

`ip route`

The LAB-NET address was present as:

`192.168.56.30/24`

The LAB-NET route remained:

`192.168.56.0/24`

The NAT default route remained through:

`10.0.2.2`

---

## 18. Persistence Verification

After confirming that the new configuration worked, Ubuntu Server was rebooted.

The purpose of the reboot was to verify that the LAB-NET configuration was genuinely persistent.

After reboot, the following command was used:

`ip addr show enp0s8`

The address remained:

`192.168.56.30/24`

Routing was checked again using:

`ip route`

Connectivity to Kali was tested using:

`ping -c 4 192.168.56.10`

The test succeeded.

### Result

**Ubuntu persistent LAB-NET configuration:** Successful

This confirmed that the Netplan configuration survived a reboot.

---

## 19. Snapshot and Recovery Strategy

VirtualBox snapshots are used as recovery points before significant configuration changes.

Snapshots were created for the laboratory virtual machines so that major changes can be reversed if necessary.

For Ubuntu Server, a snapshot was taken before continuing with persistent networking and future security configuration.

Snapshots are useful before activities such as:

- Network configuration changes
- Firewall configuration
- System hardening
- Security tool installation
- Vulnerability testing
- Exploitation exercises
- Privilege escalation labs
- Logging configuration
- SIEM integration
- Major operating system changes

Snapshots are not a replacement for proper backups, but they are extremely useful in a controlled virtual laboratory.

---

## 20. Key Lessons Learned

The network troubleshooting process provided several important practical lessons.

### IP Addressing Matters

Systems must have valid addresses within the correct subnet before communication can occur.

### APIPA Is a Troubleshooting Indicator

An address within:

`169.254.0.0/16`

can indicate that Windows could not obtain the expected IPv4 configuration.

### Routing Must Be Verified

A correct IP address does not guarantee that traffic will follow the intended path.

Routing tables should always be checked.

### Firewalls Affect Connectivity

A failed ping does not necessarily mean that the network is incorrectly configured.

Endpoint firewall policies may block traffic.

### Network Profiles Matter

Windows can apply different firewall behavior depending on whether a network is categorized as:

- Public
- Private
- Domain

### Linux Configuration Can Be Temporary or Persistent

Commands such as:

`ip addr add`

can be useful for testing but do not replace persistent network configuration.

### YAML Formatting Matters

Netplan depends on valid YAML syntax.

Incorrect indentation can prevent a network configuration from being generated.

### Validate Before Applying

Commands such as:

`sudo netplan generate`

and:

`sudo netplan try`

provide a safer workflow than immediately applying an unverified configuration.

### Test Both Directions

Bidirectional testing helps distinguish one-way filtering from complete network failure.

### Change One Thing at a Time

Controlled troubleshooting makes it easier to determine which configuration change actually solved the problem.

### Reboot Testing Matters

A configuration is not fully verified as persistent until it survives a restart.

### Documentation Is Part of Troubleshooting

Recording:

- Symptoms
- Commands
- Findings
- Root causes
- Changes
- Results

turns troubleshooting into repeatable technical knowledge.

---

## Final Troubleshooting Status

**Overall Status:** 🟢 Resolved

The current laboratory network is operational.

### Kali Linux

LAB-NET:

`192.168.56.10/24`

### Windows 11 Enterprise

LAB-NET:

`192.168.56.20/24`

### Ubuntu Server

LAB-NET:

`192.168.56.30/24`

The systems can communicate across the isolated LAB-NET environment.

Ubuntu's static LAB-NET configuration has been verified after reboot.

The troubleshooting experience documented here forms part of the practical networking foundation for future offensive security, defensive security, SOC, and incident response exercises.

---

## Security Disclaimer

All troubleshooting, scanning, network testing, and security experimentation documented in this repository is performed within a controlled cybersecurity homelab.

Testing is limited to systems that I own or systems for which I have explicit authorization.

No unauthorized external systems, networks, accounts, or organizations are targeted.
