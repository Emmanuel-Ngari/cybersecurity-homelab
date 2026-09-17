# Cybersecurity Homelab Evidence

This directory contains visual evidence collected during the implementation, testing, verification, investigation, and documentation of the cybersecurity homelab.

Evidence is organized by technology and security function.

## Evidence Categories

- `firewall/` — OPNsense configuration, segmentation, and firewall-policy evidence
- `networking/` — Network configuration and connectivity verification
- `wazuh/` — Wazuh SIEM, endpoint telemetry, FIM, Sysmon, and PowerShell evidence
- `windows/` — Windows endpoint security configuration and verification
- `linux/` — Linux server security configuration and verification
- `attack-detection/` — Attack, detection, investigation, and response evidence

## Evidence Standard

Screenshots stored in this repository should demonstrate meaningful security milestones such as:

- Security configuration
- Successful testing
- Detection verification
- SIEM investigation
- Network segmentation
- Troubleshooting and recovery
- Incident-response exercises

Screenshots should use descriptive filenames rather than generic names such as `Screenshot1.png`.

Example:

`powershell-transcript-fim-modified.png`

Before screenshots are committed, they must be reviewed for sensitive information including:

- Passwords
- API keys
- Authentication tokens
- Private keys
- Personal information
- Sensitive configuration backups
- Other credentials or secrets

OPNsense configuration backup XML files must not be published as evidence.

The evidence workflow used throughout this project is:

`Snapshot → Change → Test → Verify → Capture Evidence → Review → Document → Commit → Continue`
