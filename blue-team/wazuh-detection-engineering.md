# Wazuh Detection Engineering and OPNsense Firewall Integration

## 1. Overview

This phase extended the CyberLab SIEM from basic log collection into active detection engineering and network-security monitoring.

The work covered:

- Custom Wazuh detection rules
- MITRE ATT&CK mapping
- Detection severity tuning
- False-positive and authorized-test suppression
- Windows service-installation monitoring
- Windows scheduled-task monitoring
- Windows Firewall blocked-connection telemetry
- OPNsense remote syslog integration
- OPNsense/PF firewall log decoding
- Firewall-event correlation and noise reduction

The goal was to move beyond simply collecting logs and begin building a SOC workflow where telemetry is converted into meaningful security detections.

---

## 2. Environment

### Wazuh SIEM

- Hostname: `lab-siem-01`
- IP address: `10.10.40.10`
- Network: `SOCNET`
- Wazuh version: `4.14.7`

### Windows Endpoint

- Hostname: `LAB-WIN-01`
- IP address: `10.10.20.20`
- Network: `CORPNET`

### Kali Linux

- Hostname: `LAB-KALI-01`
- IP address: `10.10.10.10`
- Network: `REDNET`

### OPNsense Firewall

- Hostname: `lab-fw-01.cyberlab.internal`
- SOCNET interface: `10.10.40.1`
- CORPNET gateway: `10.10.20.1`
- REDNET gateway: `10.10.10.1`
- SERVERNET gateway: `10.10.30.1`

---

# Part 1 — Windows Extended Security Telemetry

## 3. Windows Defender Operational Logging

The Microsoft Defender Operational event channel was verified as enabled:

`Microsoft-Windows-Windows Defender/Operational`

The Wazuh Windows agent was configured to collect this channel.

A controlled EICAR antivirus test was used to generate Defender telemetry.

Verified Defender events included:

- Event ID `1116` — malware/threat detected
- Event ID `1117` — Defender action/remediation

The events successfully reached Wazuh.

---

## 4. Scheduled Task Monitoring

The following Windows event channel was enabled:

`Microsoft-Windows-TaskScheduler/Operational`

Advanced Audit Policy was also enabled for:

`Other Object Access Events`

This allowed Windows Security events relating to scheduled tasks to be monitored.

Important Security Event IDs include:

- `4698` — scheduled task created
- `4699` — scheduled task deleted

A controlled scheduled task named:

`\CyberLab\RuleTest`

was created and deleted to verify telemetry.

Wazuh built-in rule:

`60228`

successfully detected Event ID `4698`.

---

## 5. Windows Service Installation Monitoring

Advanced Audit Policy was enabled for:

`Security System Extension`

A temporary service was created using `sc.exe` to generate service-installation telemetry.

Observed events included:

- Security Event ID `4697`
- System Event ID `7045`

Wazuh built-in rule:

`61138`

identified Event ID `7045` as:

`New Windows Service Created`

The test service was removed immediately after testing.

---

## 6. Windows Firewall Monitoring

Windows Firewall logging was enabled for:

- Domain
- Private
- Public

Both allowed and blocked traffic logging were enabled.

The original text firewall log was configured at:

`C:\ProgramData\CyberLab\Firewall\pfirewall.log`

Although Windows correctly wrote firewall entries and the Wazuh agent opened the file, reliable forwarding of the plain-text firewall log was not achieved.

Instead of relying on text-log parsing, the architecture was improved by using structured Windows Filtering Platform telemetry.

The following audit category was enabled:

`Filtering Platform Connection`

Failure auditing was enabled so Windows generates Event ID:

`5157`

Event `5157` means:

`The Windows Filtering Platform has blocked a connection.`

The existing Wazuh Security-channel filter was modified so Event ID `5157` was no longer excluded.

A controlled outbound block to:

`1.1.1.1:443`

was generated.

Verification confirmed:

- Windows generated Event ID `5157`
- Wazuh received the event
- Wazuh rule `60104` processed the event

The final Windows Firewall monitoring path became:

`Windows Firewall → Security Event 5157 → Wazuh Agent → Wazuh Manager → Alert`

This structured event-channel approach was retained instead of the plain-text firewall-log approach.

---

# Part 2 — Custom Wazuh Detection Engineering

## 7. Custom Rule File

A dedicated rule file was created:

`/var/ossec/etc/rules/cyberlab_rules.xml`

Before modification, the existing Wazuh rule configuration was backed up.

Custom rule IDs were allocated from the local/custom rule range.

---

## 8. Custom Windows Service Detection

Custom rule:

`100100`

was created for Windows service-installation activity.

The rule inherits from built-in Wazuh rule:

`61138`

Severity:

`Level 10`

MITRE ATT&CK mapping:

`T1543.003 — Windows Service`

Description:

`CyberLab: Windows service installation detected`

A controlled service named:

`CYBERLAB-RULE-TEST`

was created to verify the rule.

The custom rule successfully generated a Wazuh alert.

---

## 9. Custom Scheduled Task Detection

Custom rule:

`100101`

was created for scheduled-task creation.

The rule inherits from built-in Wazuh rule:

`60228`

Severity:

`Level 9`

MITRE ATT&CK mapping:

`T1053.005 — Scheduled Task`

Description:

`CyberLab: Scheduled task creation detected`

A controlled task named:

`\CyberLab\RuleTest`

was created.

The custom rule successfully fired.

---

## 10. Detection Tuning and Noise Suppression

Detection engineering also included tuning known authorized test activity.

Suppression rules were created for:

- `CYBERLAB-RULE-TEST`
- `\CyberLab\RuleTest`

These suppress only the known authorized CyberLab test objects.

After tuning:

- New test executions did not create additional `100100` or `100101` alerts.
- Other related telemetry such as Event ID `4688` process-creation activity remained visible.

This demonstrated an important SOC principle:

> Detection tuning should reduce known noise without blindly suppressing the surrounding telemetry required for investigation.

---

# Part 3 — OPNsense Remote Syslog Integration

## 11. Wazuh Remote Syslog Listener

The existing Wazuh secure agent connection remained unchanged:

- Protocol: TCP
- Port: `1514`

A separate remote syslog listener was created for OPNsense.

Configuration:

- Connection type: `syslog`
- Protocol: UDP
- Port: `514`
- Allowed sender: `10.10.40.1`
- Local listener address: `10.10.40.10`

This restricted the syslog listener to the OPNsense SOCNET interface.

After restarting Wazuh, verification with `ss` confirmed:

`10.10.40.10:514`

was listening under:

`wazuh-remoted`

---

## 12. OPNsense Remote Logging

OPNsense remote logging was configured to send firewall logs to:

`10.10.40.10:514/UDP`

Application:

`filterlog`

The destination represents the Wazuh manager on SOCNET.

A packet capture on the SIEM confirmed live traffic:

`10.10.40.1 → 10.10.40.10:514/UDP`

This verified the network path independently of the SIEM parsing layer.

---

## 13. Wazuh PF Decoder

Temporary JSON archive logging was enabled during troubleshooting:

`<logall_json>yes</logall_json>`

This allowed raw OPNsense syslog events to be inspected.

Wazuh automatically recognized OPNsense `filterlog` messages using its built-in:

`pf`

decoder.

Decoded fields included:

- `action`
- `protocol`
- `srcip`
- `srcport`
- `dstip`
- `dstport`
- `id`
- `length`

Example decoded traffic:

`10.10.20.20 → 10.10.20.1:53 UDP PASS`

Because OPNsense is an agentless syslog source, the event appears under Wazuh manager agent ID `000`.

The remote sender can be identified through:

`location = 10.10.40.1`

---

## 14. Controlled Firewall Block Test

To test segmentation and detection, Kali on REDNET attempted to connect to the Windows SMB service on CORPNET:

`10.10.10.10 → 10.10.20.20:445`

The connection was blocked by OPNsense as expected.

Wazuh received and decoded the firewall events as:

- Protocol: TCP
- Action: block
- Source: `10.10.10.10`
- Destination: `10.10.20.20`
- Destination port: `445`

This verified the complete path:

`Kali → REDNET → OPNsense → Block → Syslog → Wazuh → PF Decoder`

---

## 15. Built-In OPNsense/PF Detection Rules

Testing with `wazuh-logtest` revealed that Wazuh already provides built-in PF firewall detection rules.

### Rule 87701

Rule:

`87701`

Severity:

`Level 5`

Description:

`pfSense firewall drop event.`

Although the description refers to pfSense, the rule also works with OPNsense because both produce compatible PF/filterlog firewall records.

The rule contains:

`<options>no_log</options>`

This means individual firewall blocks are recognized internally but are intentionally not written as ordinary alerts.

This design prevents the SIEM from generating an alert for every routine firewall drop.

---

## 16. Firewall Correlation — Rule 87702

Wazuh rule:

`87702`

correlates repeated firewall blocks.

The built-in rule uses:

- Frequency: `18`
- Timeframe: `45 seconds`
- Ignore period: `240 seconds`
- Same-source-IP correlation

Twenty controlled SMB connection attempts were generated from Kali:

`10.10.10.10 → 10.10.20.20:445`

OPNsense blocked the attempts.

Wazuh successfully generated:

- Rule ID: `87702`
- Severity: `Level 10`
- Source IP: `10.10.10.10`

Description:

`Multiple pfSense firewall blocks events from same source.`

This proved that Wazuh correlates repeated low-level firewall events into a higher-severity security alert.

---

## 17. Detection Engineering Lesson

The OPNsense integration demonstrated the difference between:

`Event → Detection → Correlation → Alert`

A firewall block is not automatically evidence of an attack.

Generating an alert for every dropped packet would create excessive SOC noise.

Instead:

1. OPNsense records the firewall block.
2. Wazuh decodes the event.
3. Rule `87701` tracks individual drops without logging every one as an alert.
4. Rule `87702` identifies repeated activity from the same source.
5. Wazuh generates a higher-severity correlated alert.

This provides a more realistic SOC monitoring model.

---

## 18. Raw Archive Cleanup

`logall_json` was enabled only temporarily while inspecting OPNsense events.

After verification it was returned to:

`<logall>no</logall>`

`<logall_json>no</logall_json>`

This prevents unnecessary archive growth while preserving normal Wazuh alert generation.

---

## 19. Verified Architecture

The verified network-security monitoring path is now:

`Kali / Windows / Ubuntu`
↓
`Segmented CyberLab Networks`
↓
`OPNsense Firewall`
↓
`Firewall Allow / Block Decision`
↓
`filterlog`
↓
`UDP 514 Remote Syslog`
↓
`LAB-SIEM-01`
↓
`Wazuh remoted`
↓
`PF Decoder`
↓
`Built-In + Custom Detection Rules`
↓
`Correlation`
↓
`SOC Alert / Investigation`

---

## 20. Milestone Snapshots

The following verified snapshots were created:

### Windows

`Windows - Extended Security Telemetry Verified`

### Wazuh

`LAB-SIEM-01 - Custom Detection Rules Verified`

`LAB-SIEM-01 - OPNsense Firewall Detection Verified`

### OPNsense

`OPNsense - Wazuh Syslog Integration Verified`

---

## 21. Current Security Capabilities

At this milestone, CyberLab can monitor:

- Windows process creation
- Sysmon telemetry
- PowerShell activity
- PowerShell transcription integrity
- Windows authentication
- Privileged logons
- User-account changes
- Local-group membership changes
- Service installation
- Scheduled task creation
- Microsoft Defender detections
- Windows Firewall blocked connections
- File Integrity Monitoring
- OPNsense firewall traffic
- Cross-zone blocked connections
- Repeated firewall-block correlation
- Custom Wazuh detections
- MITRE ATT&CK mappings
- Detection tuning and suppression

The next phase extends network visibility from firewall decisions into network intrusion detection using Suricata IDS.

---

## 22. Next Phase

### Phase 4 — Suricata IDS

Planned workflow:

`Network Traffic → OPNsense → Suricata IDS → Alert → Wazuh → Investigation`

Objectives:

- Deploy Suricata on OPNsense
- Select the correct monitoring interfaces
- Configure IDS mode before IPS
- Configure and update Suricata rules
- Generate controlled IDS test traffic
- Understand Suricata signatures
- Forward Suricata alerts to Wazuh
- Correlate Suricata network alerts with endpoint and firewall telemetry
- Document the completed IDS architecture
