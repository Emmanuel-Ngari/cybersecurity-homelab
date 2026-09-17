# Wazuh Agent Evidence

This directory contains visual evidence related to Wazuh agent deployment, enrollment, connectivity, status, and communication with the Wazuh manager.

## Current Monitored Agents

### LAB-WIN-01

- Operating System: Windows
- Network: CORPNET
- IP Address: `10.10.20.20`
- Wazuh Manager: `10.10.40.10`
- Status: Active and verified

### LAB-UBUNTU-01

- Operating System: Ubuntu
- Network: SERVERNET
- IP Address: `10.10.30.30`
- Wazuh Manager: `10.10.40.10`
- Status: Active and verified

## Evidence Scope

Evidence stored here may demonstrate:

- Wazuh agent installation
- Agent enrollment
- Agent service status
- Manager connectivity
- Agent status in the Wazuh Dashboard
- Agent communication across OPNsense security zones
- Agent reconnection after service or network interruption
- Agent troubleshooting
- Endpoint inventory visibility

## Evidence Examples

Recommended descriptive filenames include:

`wazuh-windows-agent-active.png`

`wazuh-ubuntu-agent-active.png`

`wazuh-dashboard-agents-overview.png`

`windows-wazuh-service-running.png`

`ubuntu-wazuh-agent-running.png`

## Verification Standard

Where possible, agent evidence should demonstrate both:

- Endpoint-side verification
- Wazuh manager or Dashboard-side verification

This provides evidence of the complete communication path rather than only proving that the local agent service is running.

## Security Review

Before committing screenshots, verify that they do not expose:

- Passwords
- Authentication keys
- Enrollment secrets
- API credentials
- Session tokens
- Private keys
- Personal information
- Other unnecessary sensitive data
