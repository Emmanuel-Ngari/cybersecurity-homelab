# PowerShell Security Monitoring Evidence

This directory contains visual evidence related to PowerShell security logging, telemetry collection, SIEM verification, and investigation within the cybersecurity homelab.

## Monitored Endpoint

- Host: `LAB-WIN-01`
- Network: CORPNET
- IP Address: `10.10.20.20`
- SIEM: `LAB-SIEM-01`
- Wazuh Manager: `10.10.40.10`

## Current PowerShell Monitoring Stack

The Windows endpoint currently provides multiple complementary PowerShell telemetry sources.

### PowerShell Module Logging

Primary Event ID:

`4103`

Module Logging provides visibility into PowerShell module and cmdlet execution.

### PowerShell Script Block Logging

Primary Event ID:

`4104`

Script Block Logging provides visibility into PowerShell script block content.

### PowerShell Transcription

System-wide PowerShell Transcription is enabled.

Transcript repository:

`C:\ProgramData\CyberLab\PowerShellTranscripts`

Transcripts provide text-based records of PowerShell sessions, commands, and command output.

### Wazuh File Integrity Monitoring

The PowerShell transcript repository is monitored by Wazuh using real-time File Integrity Monitoring.

This provides visibility into transcript file:

- Creation
- Modification
- Deletion

## Verified Capabilities

The following PowerShell security capabilities have been implemented and verified:

- Event ID `4103` generation
- Event ID `4103` delivery to Wazuh
- Event ID `4104` generation
- Event ID `4104` delivery to Wazuh
- Automatic PowerShell transcript generation
- Transcript command capture
- Transcript output capture
- Real-time Wazuh monitoring of the transcript repository
- Genuine transcript modification detection
- CLI-based Wazuh investigation
- Structured Wazuh JSON analysis using `jq`

## Evidence Scope

Evidence stored in this directory may demonstrate:

- PowerShell policy configuration
- Event ID 4103
- Event ID 4104
- PowerShell Operational event logs
- PowerShell transcript generation
- Controlled PowerShell test markers
- Wazuh PowerShell event collection
- CLI investigation
- Wazuh Dashboard investigation
- PowerShell-related detection rules
- MITRE ATT&CK mapping
- Future suspicious PowerShell detection exercises

## Recommended Filenames

Use descriptive filenames such as:

`powershell-4103-local-verification.png`

`powershell-4103-wazuh-verification.png`

`powershell-4104-local-verification.png`

`powershell-4104-wazuh-verification.png`

`powershell-transcription-policy.png`

`powershell-transcript-local-verification.png`

`powershell-transcript-wazuh-fim.png`

`wazuh-dashboard-powershell-investigation.png`

## Evidence Standard

Whenever practical, PowerShell evidence should demonstrate the complete telemetry chain:

`PowerShell Activity`

→ `Windows PowerShell Logging`

→ `Wazuh Agent`

→ `CORPNET`

→ `OPNsense`

→ `SOCNET`

→ `Wazuh Manager`

→ `CLI or Dashboard Investigation`

This demonstrates not only that PowerShell logging is enabled, but that the resulting telemetry successfully reaches the centralized monitoring platform.

## Security Considerations

PowerShell evidence requires careful review before publication.

Screenshots must be checked for:

- Passwords
- Credentials
- API keys
- Authentication tokens
- Private keys
- Sensitive command arguments
- Personal information
- Sensitive transcript output

Real credentials or secrets must never be intentionally entered into test PowerShell commands.

Controlled markers and benign commands should be used when generating portfolio evidence.
