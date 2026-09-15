# Windows PowerShell Script Block Logging and Wazuh Integration

## 1. Overview

This document describes the deployment and verification of PowerShell Script Block Logging on `LAB-WIN-01` and its integration with the Wazuh SIEM running on `LAB-SIEM-01`.

The objective was to improve visibility into PowerShell activity by collecting Event ID `4104` from:

`Microsoft-Windows-PowerShell/Operational`

The final verified telemetry path is:

`PowerShell Activity -> Event ID 4104 -> Windows PowerShell Operational Log -> Wazuh Agent -> OPNsense -> SOCNET -> Wazuh Manager`

The pipeline was successfully verified end-to-end.

---

# 2. Endpoint Information

Host:

`LAB-WIN-01`

IP address:

`10.10.20.20`

Security zone:

`CORPNET`

SIEM server:

`LAB-SIEM-01`

SIEM IP:

`10.10.40.10`

Security zone:

`SOCNET`

---

# 3. Existing Security State

Before this phase, LAB-WIN-01 already had:

- Wazuh Agent
- Microsoft Sysmon
- Sysmon Operational event collection
- Verified Wazuh forwarding
- Verified end-to-end Sysmon telemetry

Existing snapshot:

`Windows - Sysmon Wazuh Integration Verified`

Because this was already a known-good powered-off state, no additional pre-change snapshot was required.

---

# 4. Checking Existing Script Block Logging

PowerShell Script Block Logging was checked using:

    if (Test-Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging") { Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" } else { "Script Block Logging: Not configured" }

Result:

`Script Block Logging: Not configured`

This confirmed the endpoint had no existing Script Block Logging policy.

---

# 5. Enabling Script Block Logging

Script Block Logging was enabled using:

    New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force | Out-Null; Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name EnableScriptBlockLogging -Type DWord -Value 1

The value was then verified:

    Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" | Select-Object EnableScriptBlockLogging

Result:

`EnableScriptBlockLogging = 1`

---

# 6. First PowerShell Test

A harmless PowerShell command was launched in a new PowerShell process:

    powershell.exe -NoProfile -Command "Write-Output 'PS_SCRIPTBLOCK_TEST_20260913'"

Output:

`PS_SCRIPTBLOCK_TEST_20260913`

The PowerShell Operational log was then queried:

    Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational';Id=4104;StartTime=(Get-Date).AddMinutes(-5)} | Where-Object {$_.Message -match 'PS_SCRIPTBLOCK_TEST_20260913'} | Select-Object -First 3 TimeCreated,Id,Message

Result:

Event ID:

`4104`

Message:

`Creating Scriptblock text (1 of 1)`

This confirmed that Script Block Logging was functioning locally.

---

# 7. Wazuh PowerShell Event Channel Check

The Wazuh agent configuration was searched for existing PowerShell Operational collection:

    Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" -Pattern "Microsoft-Windows-PowerShell/Operational" -Context 2,2

No existing configuration was found.

---

# 8. Wazuh Configuration Backup

Before editing the agent configuration, a backup was created:

    Copy-Item "C:\Program Files (x86)\ossec-agent\ossec.conf" "C:\Program Files (x86)\ossec-agent\ossec.conf.pre-powershell.bak" -Force

This preserved the known-good Sysmon-integrated configuration.

---

# 9. Adding PowerShell Operational Collection

The following Wazuh block was added inside `<ossec_config>`:

    <!-- PowerShell Operational event collection -->
    <localfile>
      <location>Microsoft-Windows-PowerShell/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

The block was inserted using PowerShell.

---

# 10. Wazuh XML Validation

The modified Wazuh configuration was validated before restarting the service:

    try { [xml]$xml = Get-Content "C:\Program Files (x86)\ossec-agent\ossec.conf" -Raw; "XML VALID" } catch { $_.Exception.Message }

Result:

`XML VALID`

---

# 11. Wazuh Agent Restart Issue

An initial restart attempt using:

    Restart-Service WazuhSvc

returned an error indicating that the service could not be stopped.

The service state was checked:

    Get-Service WazuhSvc | Select-Object Status,Name,CanStop,CanShutdown,StartType

Result:

`Status: Stopped`

The service had actually stopped despite the restart error.

It was then started manually:

    Start-Service WazuhSvc

Service verification:

    Get-Service WazuhSvc

Result:

`Running`

This was a useful reminder to verify actual service state rather than relying only on command errors.

---

# 12. Wazuh PowerShell Channel Verification

The Wazuh agent log was searched:

    Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.log" -Pattern "Microsoft-Windows-PowerShell/Operational" | Select-Object -Last 10

The log confirmed:

`Analyzing event log: 'Microsoft-Windows-PowerShell/Operational'.`

This proved the Wazuh agent had opened the PowerShell Operational channel successfully.

---

# 13. PowerShell-to-Wazuh Test

A new harmless test command was generated:

    powershell.exe -NoProfile -Command "Write-Output 'POWERSHELL_WAZUH_TEST_20260913'"

Output:

`POWERSHELL_WAZUH_TEST_20260913`

The local PowerShell Operational log was checked:

    Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational';Id=4104;StartTime=(Get-Date).AddMinutes(-5)} | Where-Object {$_.Message -match 'POWERSHELL_WAZUH_TEST_20260913'} | Select-Object -First 3 TimeCreated,Id,Message

Result:

Event ID:

`4104`

This confirmed the test was captured locally.

---

# 14. Initial SIEM Alert Search

The SIEM alert log was searched for the PowerShell marker.

The marker was visible through Sysmon process creation telemetry, but not through the PowerShell provider.

This highlighted an important distinction:

Wazuh may receive an event without necessarily generating an alert for it.

The absence of the PowerShell event from the alert log did not prove collection failure.

---

# 15. PowerShell Registry Test

A harmless registry test was created under the current user's profile:

    New-Item -Path "HKCU:\Software\CyberLab-WazuhTest" -Force | Out-Null; New-ItemProperty -Path "HKCU:\Software\CyberLab-WazuhTest" -Name "DetectionTest" -Value 1 -PropertyType DWord -Force

Because the existing PowerShell session had been open before Script Block Logging was enabled, the first attempt did not generate a new 4104 event.

A fresh PowerShell process was used:

    powershell.exe -NoProfile -Command "New-Item -Path 'HKCU:\Software\CyberLab-WazuhTest' -Force | Out-Null; New-ItemProperty -Path 'HKCU:\Software\CyberLab-WazuhTest' -Name 'DetectionTest2' -Value 2 -PropertyType DWord -Force"

The local PowerShell Operational log confirmed:

`Event ID 4104`

This demonstrated that newly launched PowerShell processes were applying the Script Block Logging policy correctly.

---

# 16. Why Raw Event Archiving Was Needed

Wazuh alert logs only contain events that trigger alert rules.

To prove the PowerShell 4104 event was actually reaching the manager, raw JSON event archiving was temporarily enabled on `LAB-SIEM-01`.

Before changing the SIEM configuration, disk space was checked:

    df -h /

Approximate result:

- Root filesystem: 53 GB
- Used: 25 GB
- Available: 26 GB
- Usage: 50%

This confirmed enough temporary storage was available.

---

# 17. Wazuh Manager Configuration Backup

Before modifying raw-event settings, the manager configuration was backed up:

    sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.pre-powershell-test.bak

---

# 18. Existing Raw Archive Settings

The manager configuration was checked:

    sudo grep -nE 'logall|logall_json' /var/ossec/etc/ossec.conf

Initial values:

    <logall>no</logall>
    <logall_json>no</logall_json>

This confirmed raw event archiving was disabled by default.

---

# 19. Temporary Raw JSON Archiving

Several initial `sed` attempts failed due to command syntax.

The working command used was:

    sudo sed -i '/<logall_json>/s/>no</>yes</' /var/ossec/etc/ossec.conf

The configuration was then verified:

    sudo grep -nE 'logall|logall_json' /var/ossec/etc/ossec.conf

Result:

    <logall>no</logall>
    <logall_json>yes</logall_json>

Only JSON raw-event archiving was enabled.

---

# 20. Wazuh Manager Restart

The manager was restarted:

    sudo systemctl restart wazuh-manager

Health was verified:

    sudo systemctl is-active wazuh-manager

Result:

`active`

---

# 21. Final Raw Archive Test

A new PowerShell test was generated:

    powershell.exe -NoProfile -Command "Write-Output 'POWERSHELL_RAWARCHIVE_TEST_20260913'"

The local system produced a new 4104 event.

---

# 22. Wazuh Raw Archive Discovery

After raw archiving was enabled, the archive location was identified:

    sudo find /var/ossec/logs -maxdepth 3 -iname '*archive*' -print

Results included:

`/var/ossec/logs/archives`

`/var/ossec/logs/archives/archives.json`

`/var/ossec/logs/archives/archives.log`

---

# 23. Definitive SIEM Verification

The raw archive was searched for the test marker and filtered for the PowerShell provider:

    sudo grep -F "POWERSHELL_RAWARCHIVE_TEST_20260913" /var/ossec/logs/archives/archives.json | grep -F "Microsoft-Windows-PowerShell"

The output showed the actual event from:

Agent:

`LAB-WIN-01`

IP:

`10.10.20.20`

Provider:

`Microsoft-Windows-PowerShell`

Channel:

`Microsoft-Windows-PowerShell/Operational`

Event ID:

`4104`

Message included:

`Creating Scriptblock text (1 of 1)`

and the exact test marker:

`POWERSHELL_RAWARCHIVE_TEST_20260913`

This conclusively proved the complete telemetry path.

---

# 24. Verified PowerShell Telemetry Path

The final verified path is:

    LAB-WIN-01
         |
    PowerShell
         |
    Event ID 4104
         |
    Microsoft-Windows-PowerShell/Operational
         |
      Wazuh Agent
         |
       CORPNET
         |
      OPNsense
         |
       SOCNET
         |
    LAB-SIEM-01
         |
    Wazuh Manager
         |
    Raw Event Archive

This path was independently verified.

---

# 25. Raw Archiving Disabled Again

Because raw event archiving can generate significant disk usage, it was disabled immediately after verification.

Command:

    sudo sed -i '/<logall_json>/s/>yes</>no</' /var/ossec/etc/ossec.conf

Configuration was verified:

    sudo grep -nE 'logall|logall_json' /var/ossec/etc/ossec.conf

Final values:

    <logall>no</logall>
    <logall_json>no</logall_json>

The manager was restarted:

    sudo systemctl restart wazuh-manager

Health verification:

    sudo systemctl is-active wazuh-manager

Result:

`active`

---

# 26. Temporary Registry Test Cleanup

The temporary registry test key was removed:

    Remove-Item "HKCU:\Software\CyberLab-WazuhTest" -Recurse -Force

This returned the Windows endpoint to a clean state.

---

# 27. Post-Deployment Snapshot

LAB-WIN-01 was shut down normally.

A powered-off snapshot was created:

`Windows - PowerShell 4104 Wazuh Verified`

This snapshot represents the known-good Windows endpoint state after:

- Sysmon integration
- PowerShell Script Block Logging
- Wazuh PowerShell event collection
- Verified 4104 telemetry
- End-to-end SIEM transport validation

---

# 28. Current Windows Telemetry Capabilities

LAB-WIN-01 now provides:

- Sysmon telemetry
- Process creation logging
- PowerShell Script Block Logging
- Event ID 4104 collection
- Wazuh endpoint forwarding
- Cross-zone telemetry through OPNsense
- SIEM ingestion
- Verified raw-event transport
- Detection-ready PowerShell telemetry

---

# 29. Key Lessons

## Alerts Are Not the Same as Raw Events

A telemetry event may reach Wazuh without triggering an alert rule.

This is why raw-event verification was required.

## Validate Every Layer

The troubleshooting process verified:

1. PowerShell generated the event
2. Windows logged Event ID 4104
3. Wazuh Agent opened the PowerShell channel
4. The event crossed the network
5. Wazuh Manager received the event
6. Raw archive proved the actual event arrived

## Avoid Leaving Raw Archiving Enabled

`logall_json=yes` is useful for controlled troubleshooting, but can create significant disk growth.

It should remain disabled unless specifically needed.

## New PowerShell Processes Matter

After enabling Script Block Logging, a new PowerShell process may be required before the policy takes effect reliably.

---

# 30. Current Status

Script Block Logging:

`Enabled`

Registry policy:

`Verified`

Event ID 4104:

`Verified`

PowerShell Operational log:

`Operational`

Wazuh PowerShell channel:

`Verified`

Wazuh Agent:

`Running`

Wazuh Manager:

`Active`

Raw event verification:

`Completed`

Raw JSON archiving:

`Disabled after testing`

Temporary registry key:

`Removed`

Post-deployment snapshot:

`Created`

End-to-end PowerShell telemetry:

`Verified`

---

# 31. Next Improvements

Future Windows telemetry improvements may include:

- PowerShell Module Logging
- PowerShell Transcription
- Process command-line auditing
- Advanced Audit Policy
- Logon/logoff auditing
- Account-management auditing
- Windows Defender event collection
- Scheduled-task monitoring
- Service-install monitoring
- Custom Wazuh PowerShell rules
- MITRE ATT&CK mapping
- Controlled PowerShell attack simulations
- Detection engineering
- SOC investigation exercises

---

# 32. PowerShell Module Logging Deployment

## 32.1 Objective

After successfully deploying and verifying PowerShell Script Block Logging, the next Windows telemetry improvement was PowerShell Module Logging.

PowerShell Module Logging provides additional visibility into PowerShell command and module execution.

The primary event used during this phase was:

`Event ID 4103`

The objective was to verify the complete telemetry path:

`PowerShell Module Activity -> Event ID 4103 -> PowerShell Operational Log -> Wazuh Agent -> CORPNET -> OPNsense -> SOCNET -> Wazuh Manager`

---

## 32.2 Initial Module Logging State

The existing Module Logging configuration was checked on `LAB-WIN-01`.

Result:

`PowerShell Module Logging: Not configured`

This established the pre-deployment baseline.

---

## 32.3 Enabling Module Logging

The following registry path was created:

`HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging`

The `ModuleNames` subkey was also created:

`HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames`

Module Logging was enabled with:

`EnableModuleLogging = 1`

A wildcard module configuration was created:

`* = *`

This configured Module Logging for all PowerShell modules.

---

## 32.4 Configuration Verification

The Module Logging policy was queried after configuration.

Result:

`EnableModuleLogging = 1`

The `ModuleNames` configuration was also queried.

Result:

`* : *`

This confirmed that Module Logging was enabled and configured to monitor all modules.

---

## 32.5 Local Event Generation Test

A fresh Windows PowerShell process was launched with:

`powershell.exe -NoProfile -Command "Get-Process | Select-Object -First 3"`

The PowerShell Operational event log was queried for Event ID 4103.

Events were successfully generated, including:

`CommandInvocation(Get-Process)`

This provided initial confirmation that Module Logging was functioning locally.

---

## 32.6 Initial Wazuh Verification Attempt

The Wazuh raw archive was initially searched for Event ID 4103.

No matching output was observed.

The Wazuh alerts log was also checked, but no 4103 alert was generated.

This did not prove telemetry failure because Wazuh alert logs contain rule-triggering events rather than every event received by the manager.

Further investigation was therefore performed.

---

## 32.7 Wazuh Agent Troubleshooting

The Windows Wazuh agent log was inspected using PowerShell.

Historical entries showed that on September 14 the agent had temporarily experienced connectivity problems to:

`10.10.40.10:1514/tcp`

Historical messages included:

`Unable to connect`

and:

`Process locked due to agent is offline`

A historical message also indicated that the Windows Event Log service had temporarily been unavailable.

However, current September 15 logs showed:

`Analyzing event log: 'Microsoft-Windows-PowerShell/Operational'.`

This confirmed that the Wazuh agent was currently monitoring the required PowerShell event channel.

---

## 32.8 Current Wazuh Connection Verification

The Wazuh agent connection state was checked.

The current agent log showed:

`Connected to the server ([10.10.40.10]:1514/tcp).`

This confirmed that:

- LAB-WIN-01 could reach LAB-SIEM-01
- TCP 1514 connectivity was operational
- OPNsense policy allowed the required Wazuh traffic
- The Wazuh agent was connected to the manager

The earlier connectivity errors were therefore historical rather than an active failure.

---

## 32.9 Fresh 4103 Test

A new PowerShell process was used to generate fresh Module Logging telemetry:

`powershell.exe -NoProfile -Command "Get-Service WazuhSvc | Select-Object Name,Status"`

Result:

`WazuhSvc Running`

The local PowerShell Operational log was queried immediately afterward.

Multiple Event ID 4103 records were observed, including:

`CommandInvocation(Get-Service)`

This definitively confirmed local Module Logging operation.

---

## 32.10 Controlled Raw Archive Verification

Because the 4103 events did not necessarily trigger Wazuh alert rules, raw JSON archiving was temporarily enabled again for controlled troubleshooting.

The Wazuh Manager configuration was changed temporarily from:

`<logall_json>no</logall_json>`

to:

`<logall_json>yes</logall_json>`

The configuration was verified before restarting Wazuh Manager.

After the restart:

`sudo systemctl is-active wazuh-manager`

returned:

`active`

A new 4103 test event was then generated from LAB-WIN-01.

---

## 32.11 Case-Sensitive Search Troubleshooting

An initial Linux `grep` search appeared to return no PowerShell events.

The issue was traced to capitalization in the search string.

Linux `grep` performs case-sensitive matching by default.

For example:

`PowerShell`

and:

`Powershell`

are treated as different strings.

After correcting the capitalization, Wazuh raw archive data containing PowerShell Module Logging events was successfully located.

This provided an important Linux command-line lesson:

`grep -F` performs fixed-string, case-sensitive matching.

For case-insensitive searches, the `-i` option can be used, for example:

`grep -Fi 'powershell' file`

---

## 32.12 Definitive Event ID 4103 Verification

The Wazuh raw archive contained events with:

- Agent: `LAB-WIN-01`
- Agent IP: `10.10.20.20`
- Provider: `Microsoft-Windows-PowerShell`
- Channel: `Microsoft-Windows-PowerShell/Operational`
- Event ID: `4103`
- Location: `EventChannel`

Module Logging information included PowerShell command invocation data such as:

`CommandInvocation(Set-StrictMode)`

and other PowerShell module activity.

This conclusively proved that Event ID 4103 telemetry was reaching Wazuh.

---

## 32.13 Verified Module Logging Telemetry Path

The final verified path is:

`LAB-WIN-01`

↓

`PowerShell Module Logging`

↓

`Event ID 4103`

↓

`Microsoft-Windows-PowerShell/Operational`

↓

`Wazuh Agent`

↓

`CORPNET - 10.10.20.0/24`

↓

`OPNsense`

↓

`SOCNET - 10.10.40.0/24`

↓

`LAB-SIEM-01`

↓

`Wazuh Manager`

↓

`Raw Event Archive`

Status:

`VERIFIED`

---

## 32.14 Raw Archive Cleanup

Raw JSON archiving was enabled only for controlled verification.

After successful 4103 verification, the configuration was restored to:

`<logall>no</logall>`

`<logall_json>no</logall_json>`

Wazuh Manager was restarted.

Final health check:

`sudo systemctl is-active wazuh-manager`

Result:

`active`

Raw JSON collection therefore remains disabled during normal lab operation to prevent unnecessary SIEM disk growth.

---

## 32.15 Post-Deployment Snapshot

LAB-WIN-01 was shut down normally after successful verification.

A powered-off VirtualBox snapshot was created:

`Windows - PowerShell 4103 Module Logging Verified`

This provides a known-good rollback point containing:

- Wazuh Agent
- Sysmon
- PowerShell Script Block Logging
- Event ID 4104 collection
- PowerShell Module Logging
- Event ID 4103 collection
- Verified Wazuh telemetry

---

# 33. Current PowerShell Security Telemetry

LAB-WIN-01 currently provides two complementary PowerShell telemetry sources.

### Script Block Logging

Event ID:

`4104`

Status:

`Enabled and end-to-end verified`

Purpose:

Provides visibility into PowerShell script-block content.

### Module Logging

Event ID:

`4103`

Status:

`Enabled and end-to-end verified`

Purpose:

Provides visibility into PowerShell module and command invocation activity.

Together, these provide significantly stronger PowerShell visibility than the default Windows logging configuration.

---

# 34. Troubleshooting Lessons

This phase reinforced several operational principles.

### Historical Errors Must Be Distinguished From Current Errors

Old Wazuh connectivity errors were present in the agent log, but current entries confirmed that the agent had recovered.

Timestamps must therefore be considered before diagnosing an active failure.

### Verify Every Layer Independently

The investigation checked:

`Command execution -> Local event -> Event channel -> Wazuh collector -> Agent connection -> Network path -> SIEM raw event`

This prevented incorrect conclusions about where the problem existed.

### Linux Is Case-Sensitive

Search strings must use the correct capitalization unless case-insensitive matching is explicitly requested.

### Alerts and Events Are Different

An event can reach Wazuh successfully without generating an alert.

Raw-event verification can therefore be valuable during controlled troubleshooting.

### Configuration Changes Require Verification

The workflow used throughout this phase was:

`Backup -> Change -> Verify -> Restart -> Verify Service -> Generate Test -> Verify Telemetry -> Restore Temporary Settings`

---

# 35. Module Logging Milestone Status

PowerShell Module Logging:

`ENABLED`

All-module wildcard:

`CONFIGURED`

Event ID 4103 generated locally:

`VERIFIED`

PowerShell Operational channel:

`MONITORED`

Wazuh Agent connection:

`VERIFIED`

Event ID 4103 received by Wazuh:

`VERIFIED`

Raw archive troubleshooting:

`COMPLETED`

Raw JSON archiving returned to disabled:

`VERIFIED`

Wazuh Manager:

`ACTIVE`

Powered-off snapshot:

`Windows - PowerShell 4103 Module Logging Verified`

Overall status:

`POWERShell MODULE LOGGING END-TO-END VERIFIED`

---

# 36. Next Phase

The next Windows telemetry phase is:

`PowerShell Transcription`

After PowerShell telemetry is completed, planned Windows monitoring improvements include:

- Process Creation auditing
- Command-line auditing
- Advanced Windows Audit Policy
- Windows Defender Operational telemetry
- Windows Firewall telemetry
- Authentication monitoring
- Account-management monitoring
- Scheduled-task monitoring
- Service-install monitoring

The long-term objective remains:

`Generate -> Collect -> Forward -> Detect -> Investigate -> Respond -> Document`
