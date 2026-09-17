# Wazuh SIEM Evidence

This directory contains visual evidence related to the deployment, configuration, testing, detection, investigation, and operation of Wazuh within the cybersecurity homelab.

## Wazuh Architecture

The Wazuh SIEM server is:

- Hostname: `LAB-SIEM-01`
- Network: SOCNET
- IP Address: `10.10.40.10`
- Role: Centralized security monitoring and SIEM

Current monitored endpoints include:

- `LAB-WIN-01` — Windows endpoint on CORPNET
- `LAB-UBUNTU-01` — Ubuntu server on SERVERNET

## Evidence Categories

Wazuh evidence is organized into the following areas:

- `agents/` — Agent enrollment, connectivity, status, and communication
- `fim/` — File Integrity Monitoring configuration and detections
- `powershell/` — PowerShell telemetry and investigation evidence
- `sysmon/` — Windows Sysmon telemetry and Wazuh integration

Additional categories may be introduced as the SOC environment expands.

## Evidence Scope

Evidence may demonstrate:

- Wazuh manager operation
- Wazuh agent connectivity
- Windows endpoint monitoring
- Linux endpoint monitoring
- Sysmon event collection
- PowerShell Event ID 4103 monitoring
- PowerShell Event ID 4104 monitoring
- PowerShell transcript FIM
- File creation detection
- File modification detection
- File deletion detection
- Wazuh rule identification
- Threat hunting
- Event filtering
- Dashboard investigations
- Detection engineering
- MITRE ATT&CK mapping
- Incident investigation

## CLI and Dashboard Evidence

The project uses both command-line and graphical investigation workflows.

CLI evidence may demonstrate:

- Wazuh service status
- Alert searches
- Raw-event verification
- JSON analysis
- `grep`
- `jq`
- Troubleshooting

Dashboard evidence may demonstrate:

- Agent status
- Security events
- FIM detections
- Event filtering
- Rule information
- Event details
- Threat hunting
- Visualizations
- Detection investigations

Where appropriate, both CLI and Dashboard evidence should be preserved to demonstrate understanding of the underlying telemetry as well as SIEM analyst workflows.

## Evidence Naming Standard

Screenshots should use descriptive filenames.

Examples:

`wazuh-windows-agent-active.png`

`powershell-4104-wazuh-verification.png`

`powershell-4103-wazuh-verification.png`

`powershell-transcript-fim-added.png`

`powershell-transcript-fim-deleted.png`

`powershell-transcript-fim-modified.png`

`wazuh-dashboard-powershell-fim-investigation.png`

Avoid generic names such as:

`Screenshot1.png`

`Screenshot2.png`

`image.png`

## Evidence Security

Before screenshots are committed, they must be reviewed for:

- Passwords
- Authentication tokens
- API keys
- Private keys
- Sensitive personal information
- Credentials
- Session information
- Unnecessary sensitive configuration data

Evidence should prove the security milestone without exposing information that should remain private.

## Current Verified Wazuh Capabilities

The homelab has currently verified:

- Wazuh 4.14.7 all-in-one SIEM deployment
- Windows Wazuh agent connectivity
- Ubuntu Wazuh agent connectivity
- Sysmon telemetry collection
- PowerShell Script Block Logging telemetry
- PowerShell Module Logging telemetry
- PowerShell Transcription
- Real-time FIM of the PowerShell transcript repository
- File creation detection
- File deletion detection
- Genuine PowerShell transcript modification detection
- Structured alert investigation using `jq`

Future evidence will include Dashboard-based SOC investigation, custom detection rules, MITRE ATT&CK analysis, firewall telemetry, IDS telemetry, and attack-detection-response exercises.
