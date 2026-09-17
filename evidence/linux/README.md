# Linux Security Evidence

This directory contains visual evidence related to Linux administration, security configuration, monitoring, testing, detection, and investigation within the cybersecurity homelab.

## Current Linux Systems

### LAB-UBUNTU-01

- Role: Protected Linux server
- Network: SERVERNET
- IP Address: `10.10.30.30`
- Default Gateway: `10.10.30.1`
- SIEM: `LAB-SIEM-01`
- Wazuh Manager: `10.10.40.10`

### LAB-SIEM-01

- Role: Wazuh SIEM server
- Network: SOCNET
- IP Address: `10.10.40.10`

## Evidence Scope

Evidence stored in this directory may demonstrate:

- Linux network configuration
- User and group administration
- File and directory permissions
- SSH configuration
- SSH authentication
- `sudo` activity
- Linux authentication logs
- Service management
- Process investigation
- Network connections
- Firewall configuration
- File Integrity Monitoring
- `auditd`
- Security hardening
- Wazuh agent operation
- Log collection
- Persistence detection
- Linux incident investigation
- Troubleshooting and recovery

## Evidence Standard

Linux evidence should demonstrate a meaningful configuration, test, detection, or investigation result.

Where practical, evidence should show both:

- Linux endpoint-side activity
- Centralized Wazuh visibility

This helps demonstrate the complete telemetry path:

`Linux Activity`

→ `Linux Security Telemetry`

→ `Wazuh Agent`

→ `SERVERNET`

→ `OPNsense`

→ `SOCNET`

→ `LAB-SIEM-01`

→ `Detection / Investigation`

## Recommended Filenames

Use descriptive filenames such as:

`ubuntu-servernet-network-configuration.png`

`ubuntu-wazuh-agent-active.png`

`ubuntu-ssh-service-running.png`

`linux-authentication-event.png`

`linux-sudo-event.png`

`linux-auditd-event.png`

`linux-fim-detection.png`

`wazuh-dashboard-linux-investigation.png`

Avoid generic filenames such as:

`Screenshot1.png`

`Screenshot2.png`

`image.png`

## Evidence Security

Before screenshots are committed, inspect them for:

- Passwords
- Credentials
- Authentication tokens
- API keys
- Private keys
- SSH private keys
- Sensitive configuration files
- Personal information
- Unnecessary command history
- Other confidential information

Private keys and credentials must never be committed as evidence.

## Future Linux Security Evidence

As the homelab progresses, this directory may contain verified evidence related to:

- SSH security monitoring
- Failed authentication attempts
- Successful authentication
- `sudo` monitoring
- `auditd`
- File Integrity Monitoring
- Process monitoring
- Network monitoring
- Service changes
- User and group changes
- Persistence techniques
- Linux attack-and-detection exercises
- Incident investigation
- Threat hunting

Only capabilities that have actually been implemented and tested should be represented as verified.
