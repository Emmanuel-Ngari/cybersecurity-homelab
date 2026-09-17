# Sysmon Security Monitoring Evidence

This directory contains visual evidence related to Microsoft Sysmon deployment, configuration, telemetry generation, Wazuh integration, and SOC investigation within the cybersecurity homelab.

## Monitored Endpoint

- Host: `LAB-WIN-01`
- Network: CORPNET
- IP Address: `10.10.20.20`
- SIEM: `LAB-SIEM-01`
- Wazuh Manager: `10.10.40.10`

## Sysmon Deployment

Microsoft Sysmon was deployed on `LAB-WIN-01` using the official Microsoft Sysinternals distribution.

The Sysmon configuration was based on the SwiftOnSecurity Sysmon configuration and successfully validated against the installed Sysmon version.

Verified Sysmon service:

`Sysmon64`

Service status:

`Running`

## Current Telemetry Pipeline

The verified Sysmon telemetry path is:

`Windows Activity`

→ `Sysmon`

→ `Microsoft-Windows-Sysmon/Operational`

→ `Wazuh Agent — LAB-WIN-01`

→ `CORPNET`

→ `OPNsense`

→ `SOCNET`

→ `Wazuh Manager — LAB-SIEM-01`

→ `Security Investigation`

## Verified Sysmon Telemetry

Local Sysmon events were successfully generated and verified on `LAB-WIN-01`.

Observed Event IDs included:

- Event ID `1` — Process creation
- Event ID `4` — Sysmon service state changed
- Event ID `16` — Sysmon configuration state changed

The Windows Wazuh agent was configured to collect:

`Microsoft-Windows-Sysmon/Operational`

The Wazuh agent subsequently confirmed that the Sysmon Operational channel was being analyzed.

## End-to-End Verification

A controlled marker was used during Sysmon testing:

`SYSMON_WAZUH_TEST_20260913`

The resulting Sysmon process creation telemetry successfully reached the Wazuh SIEM.

This verified the complete endpoint-to-SIEM telemetry pipeline.

## Evidence Scope

Evidence stored in this directory may demonstrate:

- Sysmon installation
- Sysmon service status
- Sysmon configuration
- Sysmon Operational events
- Event ID 1 process creation
- Wazuh Sysmon event collection
- Controlled telemetry tests
- CLI-based Sysmon investigation
- Wazuh Dashboard investigation
- Process-tree analysis
- Suspicious process detection
- Future detection engineering
- MITRE ATT&CK mapping

## Recommended Filenames

Use descriptive filenames such as:

`sysmon-service-running.png`

`sysmon-event-id-1-local.png`

`sysmon-wazuh-agent-collection.png`

`sysmon-wazuh-process-creation.png`

`wazuh-dashboard-sysmon-investigation.png`

`sysmon-process-tree-investigation.png`

Avoid generic filenames such as:

`Screenshot1.png`

`image.png`

## Evidence Standard

Whenever practical, evidence should demonstrate both:

- Endpoint-side Sysmon telemetry
- SIEM-side verification

This provides stronger evidence than showing only that Sysmon is installed.

The objective is to demonstrate:

`Activity → Telemetry → Collection → Transport → Detection/Investigation`

## Future Sysmon Evidence

As the homelab progresses, this directory may contain evidence related to:

- Suspicious PowerShell process execution
- Command-line analysis
- Parent-child process relationships
- Network connections
- File creation
- Registry modifications
- Process injection telemetry
- Persistence activity
- Lateral movement
- Credential-access simulations
- Attack-and-detection exercises

Only telemetry actually implemented and tested should be presented as verified.

## Security Review

Before publication, screenshots must be checked for:

- Passwords
- Credentials
- Authentication tokens
- API keys
- Private keys
- Personal information
- Sensitive command-line arguments
- Other unnecessary sensitive information

Controlled benign markers should be used whenever possible when generating portfolio evidence.
