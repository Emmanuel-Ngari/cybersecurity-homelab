# Windows Endpoint Security Evidence

This directory contains visual evidence related to the configuration, hardening, monitoring, testing, and investigation of Windows endpoints within the cybersecurity homelab.

## Current Windows Endpoint

- Hostname: `LAB-WIN-01`
- Network: CORPNET
- IP Address: `10.10.20.20`
- Default Gateway: `10.10.20.1`
- SIEM: `LAB-SIEM-01`
- Wazuh Manager: `10.10.40.10`

## Evidence Scope

Evidence stored in this directory may demonstrate:

- Windows network configuration
- Windows security configuration
- Windows Event Logging
- Advanced Audit Policy
- Process creation auditing
- Command-line process auditing
- Authentication and logon events
- Account-management events
- Windows Defender telemetry
- Windows Firewall configuration and logging
- Service installation monitoring
- Scheduled task monitoring
- Registry security configuration
- Security-policy changes
- Endpoint troubleshooting
- Security validation

Specialized telemetry evidence may also be stored within the relevant Wazuh evidence directories.

For example:

`evidence/wazuh/sysmon/`

`evidence/wazuh/powershell/`

`evidence/wazuh/fim/`

## Evidence Standard

Windows evidence should demonstrate a meaningful configuration, test, or security result.

Where possible, evidence should show both:

- The Windows-side configuration or event
- The corresponding centralized SIEM visibility

This helps demonstrate the complete security-monitoring architecture rather than only endpoint configuration.

## Recommended Filenames

Use descriptive filenames such as:

`windows-corpnet-network-configuration.png`

`windows-advanced-audit-policy.png`

`windows-process-creation-auditing.png`

`windows-event-4688.png`

`windows-defender-operational-log.png`

`windows-firewall-logging.png`

`windows-logon-event.png`

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
- Personal information
- Sensitive command-line arguments
- Unnecessary host information
- Other confidential data

Controlled test accounts, benign commands, and test markers should be used whenever practical.

## Future Windows Security Evidence

As the lab progresses, this directory may contain verified evidence related to:

- Advanced Audit Policy
- Event ID 4688 process creation
- Command-line auditing
- Windows Defender
- Windows Firewall
- Authentication monitoring
- Account-management auditing
- Scheduled tasks
- Windows services
- Persistence detection
- Active Directory domain membership
- Group Policy
- Attack-and-detection exercises

Only capabilities that have actually been implemented and tested should be represented as verified.
