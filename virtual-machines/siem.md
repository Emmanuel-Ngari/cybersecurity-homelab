# LAB-SIEM-01 — Wazuh SIEM / Security Operations Server

## Purple Team Cybersecurity Homelab

**Hostname:** `LAB-SIEM-01`  
**Role:** Security Information and Event Management / XDR / Central Security Monitoring  
**Operating System:** Ubuntu Server  
**Network Zone:** SOCNET  
**IPv4 Address:** `10.10.40.10/24`  
**Default Gateway:** `10.10.40.1`  
**Primary Platform:** Wazuh  
**Status:** Operational, monitored, integrated, and verified

---

# 1. Purpose

`LAB-SIEM-01` is the central monitoring and detection platform for the Purple Team CyberLab.

It provides:

- Centralized endpoint telemetry
- Windows Security event monitoring
- Linux security monitoring
- Sysmon collection
- PowerShell telemetry
- Active Directory security monitoring
- OPNsense firewall log ingestion
- Suricata IDS alert ingestion
- File Integrity Monitoring
- Auditd monitoring
- Detection engineering
- Threat hunting
- Alert correlation
- Security dashboarding

The SIEM resides in the dedicated `SOCNET` security zone to separate security infrastructure from normal endpoints, servers, Active Directory systems, and attacker networks.

---

# 2. Network Configuration

Network:

`SOCNET`

Subnet:

`10.10.40.0/24`

SIEM Server:

`10.10.40.10`

Default Gateway:

`10.10.40.1`

The gateway is provided by:

`LAB-FW-01`

OPNsense controls traffic entering and leaving SOCNET.

---

# 3. CyberLab Network Context

Current network architecture:

| Zone | Network | Purpose |
|---|---|---|
| REDNET | `10.10.10.0/24` | Offensive / attacker systems |
| CORPNET | `10.10.20.0/24` | Corporate Windows endpoints |
| SERVERNET | `10.10.30.0/24` | Internal Linux/server systems |
| SOCNET | `10.10.40.0/24` | Security monitoring infrastructure |
| ADNET | `10.10.50.0/24` | Active Directory infrastructure |
| DMZ | `10.10.60.0/24` | Planned perimeter services |

Important monitored systems:

| Host | IP | Role |
|---|---|---|
| LAB-WIN-01 | `10.10.20.20` | Windows domain workstation |
| LAB-UBUNTU-01 | `10.10.30.30` | Linux endpoint/server |
| LAB-SIEM-01 | `10.10.40.10` | Wazuh SIEM |
| LAB-DC-01 | `10.10.50.10` | Domain Controller / DNS |
| LAB-FW-01 | Multiple interfaces | OPNsense firewall |

---

# 4. Wazuh Architecture

The Wazuh stack installed on `LAB-SIEM-01` includes:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

These services provide the core SIEM pipeline:

    Endpoint / Firewall / IDS
            ↓
        Wazuh Manager
            ↓
          Filebeat
            ↓
        Wazuh Indexer
            ↓
       Wazuh Dashboard

The Wazuh Dashboard is accessed from the management workstation rather than directly from the SIEM console.

---

# 5. Wazuh Service Verification

The following Wazuh components were verified as active:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

The Wazuh environment is currently receiving telemetry from multiple security zones.

---

# 6. Current Wazuh Agents

The following monitored agents are currently active:

| Agent | IP | Platform | Status |
|---|---|---|---|
| LAB-WIN-01 | `10.10.20.20` | Windows 11 | Active |
| LAB-UBUNTU-01 | `10.10.30.30` | Ubuntu Linux | Active |
| LAB-DC-01 | `10.10.50.10` | Windows Server / Domain Controller | Active |

This provides centralized visibility across:

- Workstation telemetry
- Linux server telemetry
- Active Directory telemetry

---

# 7. Windows Endpoint Monitoring

`LAB-WIN-01` forwards Windows security telemetry to Wazuh.

Monitoring includes:

- Windows Security events
- Sysmon
- PowerShell logging
- Microsoft Defender
- Windows Firewall
- File Integrity Monitoring
- Advanced Windows auditing

Examples of monitored Windows activity include:

- Process creation
- PowerShell execution
- Firewall blocks
- Authentication activity
- Suspicious command execution
- Endpoint security changes

---

# 8. Linux Endpoint Monitoring

`LAB-UBUNTU-01` forwards Linux telemetry to Wazuh.

Linux monitoring includes:

- Auditd
- SSH security activity
- User account changes
- File Integrity Monitoring
- Authentication activity
- System security events

Custom Wazuh rules were also created for native Linux account-management events.

Examples include:

`100200`

Purpose:

Native Linux user creation detection.

`100201`

Purpose:

Native Linux user deletion detection.

These detections were tested using controlled account lifecycle activity.

---

# 9. Active Directory Monitoring

`LAB-DC-01` is monitored by Wazuh through the dedicated ADNET-to-SOCNET firewall path.

The Domain Controller forwards:

- Windows Security events
- PowerShell Operational events
- Sysmon Operational events
- Active Directory authentication telemetry
- Account-management events
- Security-group changes
- Privileged logons
- Audit-policy changes

Important verified events include:

| Event ID | Description |
|---|---|
| 4103 | PowerShell Module Logging |
| 4104 | PowerShell Script Block Logging |
| 4672 | Special privileges assigned to new logon |
| 4688 | Process creation |
| 4719 | System audit policy changed |
| 4720 | User account created |
| 4726 | User account deleted |
| 4728 | Member added to global security group |
| 4729 | Member removed from global security group |
| 4768 | Kerberos TGT requested |
| 4769 | Kerberos service ticket requested |
| 4771 | Kerberos pre-authentication failed |
| 5136 | Directory service object modified |

---

# 10. Active Directory Authentication Detection

A controlled incorrect password attempt was performed using:

`CYBERLAB\emmanuel.patrick`

The Domain Controller generated:

`Event ID 4771`

Description:

`Kerberos pre-authentication failed`

Wazuh successfully detected the event.

Observed source:

`10.10.20.20`

Observed target:

`emmanuel.patrick`

Kerberos status:

`0x18`

This demonstrated the ability to detect failed Kerberos authentication activity.

---

# 11. Active Directory Account Lifecycle Detection

A temporary account was created:

`telemetry.test`

The Domain Controller generated:

`4720`

Wazuh identified the account creation.

The account was then deleted.

The Domain Controller generated:

`4726`

Wazuh identified the deletion.

The temporary account was removed after testing.

---

# 12. Active Directory Group Monitoring

A temporary Global Security Group was created:

`GG_Telemetry_Test`

The account:

`emmanuel.patrick`

was temporarily added.

This generated:

`4728`

The user was then removed.

This generated:

`4729`

Both events were detected in Wazuh.

The temporary security group was deleted after testing.

---

# 13. Privileged Logon Monitoring

Wazuh successfully detected:

`4672`

Description:

`Special privileges assigned to new logon`

The verified event involved:

    Domain: CYBERLAB
    Account: Administrator
    Computer: LAB-DC-01.cyberlab.internal

This provides visibility into privileged sessions occurring on the Domain Controller.

---

# 14. PowerShell Monitoring

PowerShell telemetry is collected from Windows systems.

On `LAB-DC-01`, the following channel was explicitly added to Wazuh:

    Microsoft-Windows-PowerShell/Operational

Wazuh configuration:

    <localfile>
      <location>Microsoft-Windows-PowerShell/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

PowerShell Script Block Logging:

`4104`

Verified in Wazuh.

PowerShell Module Logging:

`4103`

Verified locally.

---

# 15. Sysmon Monitoring

Sysmon telemetry is collected from Windows systems.

On `LAB-DC-01`, the following event channel is collected:

    Microsoft-Windows-Sysmon/Operational

Wazuh configuration:

    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

Sysmon Event ID `1` process creation events were successfully observed in Wazuh.

The Domain Controller Sysmon configuration was tuned to reduce unnecessary process noise while preserving higher-value activity.

---

# 16. OPNsense Firewall Log Integration

OPNsense firewall logs are forwarded to Wazuh.

The firewall telemetry allows Wazuh to monitor:

- Permitted traffic
- Blocked traffic
- Network segmentation violations
- Security-zone interaction
- Firewall policy enforcement

A custom firewall correlation rule was created and verified:

`87702`

This confirmed that OPNsense firewall events could be processed and correlated inside Wazuh.

---

# 17. Suricata IDS Integration

Suricata IDS runs on OPNsense.

Suricata monitors network traffic and produces EVE JSON telemetry.

The final architecture is:

    Network Traffic
        ↓
    OPNsense Suricata
        ↓
    EVE JSON
        ↓
    Remote Syslog UDP/5514
        ↓
    LAB-SIEM-01 rsyslog
        ↓
    /var/log/suricata/eve.json
        ↓
    Wazuh Logcollector
        ↓
    Wazuh Rules
        ↓
    Wazuh Dashboard

---

# 18. Suricata Syslog Receiver

A dedicated rsyslog listener was configured on:

`UDP/5514`

Configuration file:

`/etc/rsyslog.d/30-suricata-remote.conf`

Suricata logs are written to:

`/var/log/suricata/eve.json`

Directory ownership:

    syslog:wazuh

Directory permissions:

    750

Log file permissions:

    0640

The dedicated port avoids conflicts with other syslog traffic.

---

# 19. Suricata Detection Verification

A Suricata alert was successfully observed for:

Signature ID:

`2100498`

Description:

`GPL ATTACK_RESPONSE id check returned root`

Category:

`Potentially Bad Traffic`

Severity:

`2`

Example source:

`217.160.0.187:80`

Destination:

`10.10.10.10`

The alert was successfully decoded and processed by Wazuh.

Wazuh rule:

`86601`

Description:

Suricata alert.

This verified the IDS-to-SIEM monitoring pipeline.

---

# 20. Suricata Decoder Development

During development, Suricata events initially failed to decode correctly and fell back to:

Wazuh rule:

`1002`

Custom decoder work was performed while troubleshooting.

The environment was eventually simplified to use pure EVE JSON and Wazuh's built-in JSON / Suricata processing.

The temporary custom wrapper decoder was retired after the cleaner integration was confirmed.

This reduced unnecessary custom parsing and improved maintainability.

---

# 21. rsyslog Verification

rsyslog was verified as active.

Observed version:

`8.2512.0`

The service listens on:

`UDP/5514`

IPv4:

`0.0.0.0:5514`

IPv6:

`[::]:5514`

This listener is dedicated to Suricata telemetry.

---

# 22. Wazuh Agent Communication Ports

The lab uses:

`1514/TCP`

for Wazuh agent communication.

`1515/TCP`

for enrollment / registration.

OPNsense alias:

`WAZUH_AGENT_PORTS`

contains:

    1514
    1515

Security zones are granted access to these ports only where endpoint monitoring is required.

---

# 23. Wazuh Dashboard Access

Wazuh Dashboard is accessed from:

`LAB-WIN-01`

rather than using the SIEM server itself as a browsing workstation.

Dashboard address:

`https://10.10.40.10`

This maintains separation between:

- SIEM infrastructure
- Administrative workstation activity

---

# 24. SOCNET Security Design

SOCNET is:

`10.10.40.0/24`

The network contains security infrastructure such as:

`LAB-SIEM-01`

SOCNET is treated as a protected security zone.

Other networks do not receive unrestricted access to SOCNET.

Required communication is provided using explicit OPNsense rules.

Examples include:

- Wazuh agent communication
- Administrative dashboard access
- Security telemetry forwarding

---

# 25. Wazuh Agent Aliases

Relevant OPNsense aliases include:

`WAZUH_SERVER`

Address:

`10.10.40.10`

`WAZUH_AGENT_PORTS`

Ports:

    1514
    1515

`WAZUH_DASHBOARD_PORT`

Port:

`443`

`WAZUH_API_PORT`

Port:

`55000`

These aliases make firewall policy easier to read and maintain.

---

# 26. SIEM Storage Recovery

`LAB-SIEM-01` previously experienced a storage-related boot issue.

The VM was recovered successfully.

Logical volume:

`/dev/mapper/ubuntu--vg-ubuntu--lv`

Observed filesystem state after recovery:

    Total: approximately 53 GB
    Used: approximately 21 GB
    Free: approximately 30 GB

The logical volume was confirmed active.

No failed systemd services were present after recovery.

The Wazuh stack returned to normal operation.

---

# 27. Snapshot Recovery Policy

An older SIEM snapshot should not be restored during normal operations because the current SIEM state is healthy and contains newer configuration and monitoring data.

Rollback should only occur if a genuine infrastructure failure requires it.

The preferred workflow remains:

    Snapshot
    → Change
    → Verify
    → Document
    → Continue

---

# 28. Detection Engineering

The SIEM is used not only to collect telemetry but also to build and validate custom detections.

Examples completed include:

- Windows endpoint detections
- Windows Firewall detections
- Linux account lifecycle detections
- OPNsense firewall correlation
- Suricata IDS processing
- Active Directory account changes
- Active Directory security-group changes
- Kerberos authentication failures
- Privileged logons
- PowerShell telemetry
- Sysmon telemetry

The lab therefore supports both:

- Defensive monitoring
- Detection engineering

---

# 29. Current Monitoring Coverage

Current visibility includes:

## Windows Workstation

- Security logs
- Process creation
- PowerShell
- Sysmon
- Defender
- Windows Firewall
- File Integrity Monitoring
- Advanced auditing

## Linux Server

- Auditd
- SSH activity
- Account management
- File Integrity Monitoring
- Authentication
- System security events

## Domain Controller

- Kerberos
- Account creation/deletion
- Group membership
- Privileged logons
- Process creation
- PowerShell
- Sysmon
- Audit policy changes

## Network Security

- OPNsense firewall logs
- Suricata IDS
- Network segmentation telemetry

---

# 30. Current Operational State

`LAB-SIEM-01` is currently:

    Wazuh Manager              ACTIVE
    Wazuh Indexer              ACTIVE
    Wazuh Dashboard            ACTIVE
    Filebeat                   ACTIVE
    rsyslog                    ACTIVE
    Wazuh Agents               3 ACTIVE
    Windows Telemetry          VERIFIED
    Linux Telemetry            VERIFIED
    Active Directory Telemetry VERIFIED
    OPNsense Firewall Logs     VERIFIED
    Suricata IDS               VERIFIED
    Threat Hunting             OPERATIONAL
    Detection Engineering      OPERATIONAL

---

# 31. Active Agents

Current monitored agents:

    LAB-WIN-01      10.10.20.20    ACTIVE
    LAB-UBUNTU-01   10.10.30.30    ACTIVE
    LAB-DC-01       10.10.50.10    ACTIVE

Future monitored systems will include the planned DMZ server.

---

# 32. Security Design Decisions

## Dedicated SOC Network

The SIEM is isolated inside SOCNET rather than being placed with normal servers.

## Explicit Firewall Rules

Security zones receive only the Wazuh connectivity required for monitoring.

## Management from LAB-WIN-01

The dashboard is accessed from the management workstation rather than through local browsing on critical infrastructure.

## Native Formats Preferred

Suricata EVE JSON is used directly instead of maintaining unnecessary wrapper decoders.

## Alerting and Raw Events Are Treated Separately

Not every collected event becomes an alert.

Troubleshooting follows:

    Generate event
    → Verify locally
    → Verify collection
    → Verify alert rule

This prevents unnecessary configuration changes when telemetry exists but does not meet an alert condition.

---

# 33. Known Considerations

The following are expected and are not considered failures:

- Some successful Windows authentication events may remain raw events without becoming Wazuh alerts.
- Some PowerShell events may be collected without generating alert-level detections.
- Sysmon requires tuning to avoid excessive noise.
- Wazuh Threat Hunting represents detection output rather than every possible raw operating-system event.
- SIEM storage should continue to be monitored as additional agents and telemetry sources are added.

---

# 34. Future Work

Future SIEM work will include:

- DMZ endpoint monitoring
- Additional Active Directory detection engineering
- Kerberoasting detection
- Password-spraying detection
- Lateral-movement detection
- Credential-access detections
- Network-to-endpoint correlation
- Threat hunting exercises
- Incident-response simulations
- Detection tuning
- False-positive reduction
- Analyst investigation workflows

---

# 35. Final Status

`LAB-SIEM-01` is the operational security-monitoring core of the Purple Team CyberLab.

The server currently provides centralized visibility across:

- Windows endpoints
- Linux endpoints
- Active Directory
- OPNsense firewall
- Suricata IDS
- PowerShell telemetry
- Sysmon telemetry
- Auditd telemetry
- File Integrity Monitoring

The SIEM is now capable of supporting realistic SOC and Purple Team activities including:

- Detection
- Investigation
- Threat hunting
- Correlation
- Incident response
- Detection engineering
- Attack simulation validation

The next major monitored security zone will be the CyberLab DMZ.
