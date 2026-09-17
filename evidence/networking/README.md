# Networking Evidence

This directory contains visual evidence related to the network architecture, connectivity, routing, and segmentation of the cybersecurity homelab.

## Evidence Scope

Evidence stored here may include:

- VirtualBox network configuration
- OPNsense interface connectivity
- Static IP configuration
- Default gateway configuration
- DNS configuration
- Routing verification
- Inter-network connectivity tests
- Network segmentation tests
- Allowed and blocked traffic verification
- TCP/UDP connectivity testing
- Network troubleshooting

## Current Network Architecture

The laboratory currently uses the following primary security networks:

- REDNET — `10.10.10.0/24`
- CORPNET — `10.10.20.0/24`
- SERVERNET — `10.10.30.0/24`
- SOCNET — `10.10.40.0/24`

OPNsense operates as the firewall and router between these security zones.

## Current Key Hosts

- `LAB-KALI-01` — `10.10.10.10`
- `LAB-WIN-01` — `10.10.20.20`
- `LAB-UBUNTU-01` — `10.10.30.30`
- `LAB-SIEM-01` — `10.10.40.10`

## Evidence Examples

Useful evidence may include screenshots demonstrating:

- Windows network configuration
- Kali network configuration
- Ubuntu network configuration
- SIEM network configuration
- OPNsense routes and interfaces
- Successful permitted connections
- Failed connections caused by firewall policy
- DNS resolution
- Internet connectivity
- Wazuh agent connectivity across security zones

Use descriptive filenames such as:

`windows-corpnet-ip-configuration.png`

`kali-rednet-ip-configuration.png`

`ubuntu-servernet-ip-configuration.png`

`siem-socnet-ip-configuration.png`

`kali-to-ubuntu-ssh-allowed.png`

`rednet-to-corpnet-blocked.png`

## Evidence Requirements

Every screenshot should demonstrate a specific network configuration, test, or verified security behavior.

Before publication, screenshots must be reviewed for:

- Passwords
- Credentials
- Authentication tokens
- Personal information
- Unnecessary host information
- Other sensitive data

Screenshots should support the written documentation rather than replace it.
