# Wazuh File Integrity Monitoring Evidence

This directory contains visual evidence related to Wazuh File Integrity Monitoring (FIM) within the cybersecurity homelab.

## Purpose

File Integrity Monitoring provides visibility into filesystem changes occurring on monitored endpoints.

The current implementation includes real-time monitoring of the PowerShell Transcription repository on `LAB-WIN-01`.

Monitored directory:

`C:\ProgramData\CyberLab\PowerShellTranscripts`

## Current Architecture

PowerShell transcript FIM follows this telemetry path:

`PowerShell Session`

→ `Automatic PowerShell Transcript`

→ `C:\ProgramData\CyberLab\PowerShellTranscripts`

→ `Wazuh Real-Time FIM`

→ `Wazuh Agent — LAB-WIN-01`

→ `CORPNET`

→ `OPNsense`

→ `SOCNET`

→ `Wazuh Manager — LAB-SIEM-01`

→ `Wazuh FIM Alert`

## Verified FIM Events

The following behaviors have been successfully tested and verified.

### File Creation

Wazuh detected creation of a controlled test file.

Verified telemetry:

- Rule ID: `554`
- Description: `File added to the system.`
- Event: `added`
- Mode: `realtime`
- Agent: `LAB-WIN-01`

### File Deletion

Wazuh detected deletion of the controlled test file.

Verified telemetry:

- Rule ID: `553`
- Description: `File deleted.`
- Event: `deleted`
- Mode: `realtime`
- Agent: `LAB-WIN-01`

### Genuine PowerShell Transcript Modification

A genuine PowerShell session automatically generated a transcript.

Wazuh detected modification of the transcript as PowerShell continued writing session data.

Verified telemetry:

- Rule ID: `550`
- Description: `Integrity checksum changed.`
- Event: `modified`
- Mode: `realtime`
- Agent: `LAB-WIN-01`

## Evidence Types

Evidence stored in this directory may include:

- FIM configuration
- File creation detections
- File modification detections
- File deletion detections
- Real-time monitoring verification
- CLI investigation
- Structured JSON analysis
- Wazuh Dashboard FIM investigations
- FIM rule information
- FIM event details

## Recommended Filenames

Use descriptive filenames such as:

`powershell-transcript-fim-added.png`

`powershell-transcript-fim-deleted.png`

`powershell-transcript-fim-modified.png`

`wazuh-fim-cli-verification.png`

`wazuh-dashboard-fim-investigation.png`

Avoid generic filenames such as:

`Screenshot1.png`

`image.png`

## CLI and Dashboard Verification

Where practical, significant FIM detections should be demonstrated through both:

- Wazuh manager CLI analysis
- Wazuh Dashboard investigation

CLI verification demonstrates understanding of the underlying Wazuh alert data.

Dashboard verification demonstrates SOC analyst investigation and filtering skills.

## Evidence Security

Before committing screenshots, inspect them for:

- Passwords
- Authentication tokens
- API keys
- Private keys
- Personal information
- Sensitive command history
- Credentials accidentally captured inside PowerShell transcripts
- Other unnecessary sensitive information

PowerShell transcript contents require particular care because commands and their output may contain sensitive information.

Evidence should demonstrate the detection without unnecessarily exposing complete transcript contents.
