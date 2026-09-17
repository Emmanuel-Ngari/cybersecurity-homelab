# Firewall Evidence

This directory contains visual evidence related to the OPNsense firewall and network-security architecture of the cybersecurity homelab.

Evidence may include:

- OPNsense interface configuration
- Security-zone configuration
- Firewall aliases
- Firewall rules
- Inter-zone traffic restrictions
- Internet-access policies
- Wazuh communication rules
- Firewall logging
- Segmentation verification
- Troubleshooting and recovery

## Current Security Zones

- WAN — External / VirtualBox NAT
- CORPNET — `10.10.20.0/24`
- REDNET — `10.10.10.0/24`
- SERVERNET — `10.10.30.0/24`
- SOCNET — `10.10.40.0/24`

Future zones will be documented when implemented.

## Evidence Requirements

Screenshots should demonstrate a specific configuration or verified security behavior.

Use descriptive filenames such as:

`opnsense-security-zones.png`

`corpnet-firewall-rules.png`

`rednet-segmentation-block.png`

`servernet-wazuh-rule.png`

Before publication, every screenshot must be reviewed for credentials, secrets, personal information, or other sensitive data.

OPNsense configuration backup XML files must never be committed to this evidence directory.
