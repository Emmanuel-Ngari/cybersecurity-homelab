# Windows Sysmon Deployment and Wazuh Integration

## 1. Overview

This document describes the deployment of Microsoft Sysmon on `LAB-WIN-01` and its integration with the Wazuh SIEM running on `LAB-SIEM-01`.

The objective was to improve Windows endpoint visibility beyond the standard Windows event logs by collecting detailed telemetry such as:

- Process creation
- Network activity
- Service activity
- File activity
- Registry activity
- Driver activity
- Process relationships
- Security-relevant endpoint behavior

The complete telemetry path is:

`Windows Activity -> Sysmon -> Windows Event Log -> Wazuh Agent -> OPNsense -> SOCNET -> Wazuh Manager -> Wazuh Alerts`

This path was successfully verified end-to-end.

---

# 2. Endpoint Information

Host:

`LAB-WIN-01`

IP address:

`10.10.20.20`

Security zone:

`CORPNET`

Operating system:

`Windows 11 Enterprise`

Wazuh agent version:

`4.14.7`

SIEM server:

`LAB-SIEM-01`

SIEM address:

`10.10.40.10`

SIEM zone:

`SOCNET`

---

# 3. Pre-Deployment Snapshot

Before installing Sysmon, a VirtualBox snapshot was created:

`Windows - Pre Sysmon Deployment`

The snapshot was created before modifying the endpoint-monitoring configuration.

---

# 4. Sysmon Download

PowerShell was opened as Administrator.

The official Microsoft Sysinternals Sysmon archive was downloaded using:

    Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "$env:TEMP\Sysmon.zip"

The download completed successfully.

---

# 5. Sysmon Extraction

A permanent tools directory was created:

`C:\Tools\Sysmon`

The downloaded archive was extracted using:

    New-Item -ItemType Directory -Path "C:\Tools\Sysmon" -Force | Out-Null; Expand-Archive -Path "$env:TEMP\Sysmon.zip" -DestinationPath "C:\Tools\Sysmon" -Force

The extracted files included:

- `Eula.txt`
- `Sysmon.exe`
- `Sysmon64.exe`
- `Sysmon64a.exe`

Because LAB-WIN-01 is a 64-bit x86 Windows system, `Sysmon64.exe` was selected.

---

# 6. Binary Signature Verification

Before executing the downloaded security tool, its Authenticode signature was verified.

Command:

    Get-AuthenticodeSignature "C:\Tools\Sysmon\Sysmon64.exe" | Select-Object Status,@{N='Signer';E={$_.SignerCertificate.Subject}}

Result:

`Status: Valid`

Signer:

`Microsoft Windows Publisher`

Publisher:

`Microsoft Corporation`

This confirmed that the executable was digitally signed by Microsoft.

---

# 7. Sysmon Configuration

Instead of installing Sysmon with an empty/default configuration, a detection-focused community configuration was downloaded.

The SwiftOnSecurity Sysmon configuration was retrieved using:

    Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\Sysmon\sysmonconfig.xml"

The downloaded file identified itself as:

`sysmon-config`

Source project:

`SwiftOnSecurity/sysmon-config`

The configuration source version displayed in the file was:

`74`

Source date:

`2021-07-08`

---

# 8. Sysmon Configuration Inspection

The beginning of the configuration was inspected using:

    Get-Content "C:\Tools\Sysmon\sysmonconfig.xml" -TotalCount 20

This confirmed that an XML Sysmon configuration had been downloaded rather than an HTML error page or failed web response.

---

# 9. XML Validation

PowerShell was used to parse the configuration as XML.

Command:

    [xml]$cfg = Get-Content "C:\Tools\Sysmon\sysmonconfig.xml" -Raw; $cfg.Sysmon.schemaversion

Result:

`4.50`

The configuration therefore parsed successfully.

---

# 10. Sysmon Installation

Sysmon was installed using the downloaded configuration.

Command:

    & "C:\Tools\Sysmon\Sysmon64.exe" -accepteula -i "C:\Tools\Sysmon\sysmonconfig.xml"

Installation output confirmed:

- Configuration file validated
- Sysmon64 installed
- SysmonDrv installed
- SysmonDrv started
- Sysmon64 started

Installed Sysmon version:

`15.22`

Sysmon supported schema version:

`4.91`

Configuration schema version:

`4.50`

The older configuration schema was accepted successfully by the installed Sysmon version.

---

# 11. Sysmon Service Verification

The Windows Sysmon service was checked using:

    Get-Service Sysmon64

Result:

`Running`

This confirmed the Sysmon service was operational.

---

# 12. Local Sysmon Event Verification

The Sysmon Operational event channel was queried using:

    Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5 | Select-Object TimeCreated,Id,ProviderName

Recent events included:

- Event ID 1
- Event ID 4
- Event ID 16

Provider:

`Microsoft-Windows-Sysmon`

This confirmed Sysmon was actively producing Windows event telemetry.

---

# 13. Example Sysmon Event IDs Observed

## Event ID 1

Process creation.

Useful for detecting:

- Command execution
- PowerShell activity
- Suspicious child processes
- Malware execution
- LOLBins
- Attack tooling

## Event ID 4

Sysmon service state change.

Useful for detecting changes to the monitoring service.

## Event ID 16

Sysmon configuration change.

Useful for detecting modification or replacement of the Sysmon configuration.

---

# 14. Wazuh Agent Configuration Location

The Wazuh agent configuration path was verified using:

    Test-Path "C:\Program Files (x86)\ossec-agent\ossec.conf"

Result:

`True`

Configuration file:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

---

# 15. Wazuh Configuration Backup

Before editing the Wazuh configuration, a backup was created.

Command:

    Copy-Item "C:\Program Files (x86)\ossec-agent\ossec.conf" "C:\Program Files (x86)\ossec-agent\ossec.conf.pre-sysmon.bak" -Force

Backup:

`ossec.conf.pre-sysmon.bak`

This provides a rollback point if the event-channel configuration causes problems.

---

# 16. Existing Sysmon Configuration Check

The Wazuh configuration was searched for any existing Sysmon configuration.

Command:

    Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" -Pattern "Sysmon" -Context 3,3

No existing Sysmon configuration was found.

This prevented duplicate event-channel definitions.

---

# 17. Wazuh Configuration Inspection

The end of the existing Wazuh configuration was inspected using:

    Get-Content "C:\Program Files (x86)\ossec-agent\ossec.conf" -Tail 30

The closing:

`</ossec_config>`

element was identified.

The Sysmon collection block needed to be placed inside the main Wazuh configuration before this closing element.

---

# 18. Adding Sysmon Event Collection to Wazuh

The following configuration was added inside `<ossec_config>`:

    <!-- Sysmon event collection -->
    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

The configuration instructs the Wazuh agent to monitor:

`Microsoft-Windows-Sysmon/Operational`

The block was inserted using PowerShell:

    $path="C:\Program Files (x86)\ossec-agent\ossec.conf"; $content=Get-Content $path -Raw; $block="`r`n  <!-- Sysmon event collection -->`r`n  <localfile>`r`n    <location>Microsoft-Windows-Sysmon/Operational</location>`r`n    <log_format>eventchannel</log_format>`r`n  </localfile>`r`n"; $content=$content -replace '\r?\n</ossec_config>',"$block`r`n</ossec_config>"; Set-Content -Path $path -Value $content -Encoding UTF8

---

# 19. Wazuh XML Validation

Before restarting the Wazuh service, the modified configuration was parsed as XML.

Command:

    try { [xml]$xml = Get-Content "C:\Program Files (x86)\ossec-agent\ossec.conf" -Raw; "XML VALID" } catch { $_.Exception.Message }

Result:

`XML VALID`

This reduced the risk of restarting Wazuh with malformed XML.

---

# 20. Wazuh Agent Restart

The Wazuh agent was restarted:

    Restart-Service WazuhSvc

The service state was then checked:

    Get-Service WazuhSvc

Result:

`Running`

This confirmed the Wazuh agent accepted the modified configuration and restarted successfully.

---

# 21. Wazuh Sysmon Channel Verification

The Wazuh agent log was searched for references to Sysmon.

Command:

    Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.log" -Pattern "Sysmon|Microsoft-Windows-Sysmon|eventchannel" | Select-Object -Last 20

The log contained:

`Analyzing event log: 'Microsoft-Windows-Sysmon/Operational'.`

This confirmed that the Wazuh agent had successfully opened the Sysmon Operational event channel.

At this stage the local collection chain was:

`Windows Activity -> Sysmon -> Windows Event Log -> Wazuh Agent`

---

# 22. Sysmon Test Event

A harmless test process was generated to verify telemetry collection.

Command:

    Start-Process cmd.exe -ArgumentList '/c echo SYSMON_WAZUH_TEST_20260913 > C:\Windows\Temp\sysmon-wazuh-test.txt' -Wait

This command simply:

1. Started `cmd.exe`
2. Wrote a test string to a temporary file
3. Created identifiable process telemetry

Test marker:

`SYSMON_WAZUH_TEST_20260913`

---

# 23. Local Test Verification

The recent Sysmon process-creation events were searched for the test marker.

Command:

    Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational';Id=1;StartTime=(Get-Date).AddMinutes(-5)} | Where-Object {$_.Message -match 'SYSMON_WAZUH_TEST_20260913|sysmon-wazuh-test\.txt'} | Select-Object -First 3 TimeCreated,Id,Message

Result included:

`Event ID 1`

Provider:

`Microsoft-Windows-Sysmon`

This confirmed:

`Test Process -> Sysmon Event`

---

# 24. SIEM-Side Verification

After starting `LAB-SIEM-01`, Wazuh alert logs were inspected.

Initial marker search:

    sudo grep -R "SYSMON_WAZUH_TEST_20260913" /var/ossec/logs/alerts/ /var/ossec/logs/archives/ 2>/dev/null | tail -n 20

A broader Sysmon-provider search was also performed:

    sudo grep -R "Microsoft-Windows-Sysmon" /var/ossec/logs/alerts/ 2>/dev/null | tail -n 10

The Wazuh alert log contained events with:

`Microsoft-Windows-Sysmon`

and:

`Microsoft-Windows-Sysmon/Operational`

The test process data was visible inside the received event, including the distinctive test command.

This confirmed the complete telemetry pipeline.

---

# 25. Verified Telemetry Path

The final verified path is:

    LAB-WIN-01
    Windows activity
          |
        Sysmon
          |
    Microsoft-Windows-Sysmon/Operational
          |
      Wazuh Agent
          |
       CORPNET
    10.10.20.0/24
          |
       OPNsense
          |
        SOCNET
    10.10.40.0/24
          |
     LAB-SIEM-01
          |
      Wazuh Manager
          |
      Wazuh Alerts

Every major stage was independently tested.

---

# 26. Firewall Architecture

No broad CORPNET-to-SOCNET access was required.

Existing least-privilege Wazuh firewall rules allow the Windows endpoint to communicate with the Wazuh server using the required Wazuh services.

This maintains the segmented security architecture.

---

# 27. Post-Deployment Snapshot

After Sysmon and Wazuh integration were fully verified, Windows was shut down normally.

A powered-off VirtualBox snapshot was then created:

`Windows - Sysmon Wazuh Integration Verified`

This snapshot represents the current known-good Windows endpoint monitoring state.

---

# 28. Snapshot Storage Lesson

During this phase, a major VirtualBox storage-management problem was also identified.

The physical host previously reached approximately:

`0 GB free`

Investigation showed that multiple long VirtualBox snapshot chains had created large differencing disks.

Examples included snapshot VDI files tens of gigabytes in size.

The environment was shut down safely and obsolete snapshots were consolidated using VirtualBox itself.

Snapshot files were never manually deleted.

The cleanup covered:

- LAB-WIN-01
- LAB-UBUNTU-01
- LAB-KALI-01
- LAB-SIEM-01
- LAB-FW-01

The host eventually recovered to more than:

`100 GB free`

Additional cleanup of obsolete non-lab files increased free storage further.

---

# 29. New Snapshot Policy

The following snapshot policy was adopted:

- Maintain approximately 2–3 meaningful snapshots per VM where practical
- Prefer powered-off snapshots
- Avoid snapshots for every minor change
- Keep major verified milestones
- Remove obsolete pre-change snapshots after the newer state is proven stable
- Monitor host free space regularly
- Never manually delete `.vdi`, `.sav`, or VirtualBox snapshot files
- Treat snapshots as temporary rollback points, not backups

---

# 30. Security Value of Sysmon

Sysmon significantly improves endpoint visibility because standard Windows event logging does not provide the same level of detailed endpoint telemetry.

Sysmon will later support detection of activity such as:

- Suspicious PowerShell
- Command-shell execution
- Malware execution
- Credential-access tooling
- Persistence techniques
- Suspicious network connections
- Registry modification
- Process injection indicators
- Living-off-the-land binaries
- Attack-tool execution
- Privilege escalation activity

This provides the foundation for future attack-and-detection exercises.

---

# 31. Current Status

Sysmon download:

`Verified`

Microsoft digital signature:

`Valid`

Sysmon configuration:

`Validated`

Sysmon service:

`Running`

Sysmon driver:

`Running`

Sysmon Operational event log:

`Generating events`

Wazuh configuration backup:

`Completed`

Wazuh XML configuration:

`Valid`

Wazuh service:

`Running`

Sysmon event channel collection:

`Verified`

Local Event ID 1 generation:

`Verified`

Wazuh SIEM ingestion:

`Verified`

End-to-end telemetry path:

`Operational`

Post-deployment snapshot:

`Created`

---

# 32. Next Improvements

Future Windows telemetry improvements may include:

- PowerShell Script Block Logging
- PowerShell Module Logging
- PowerShell Transcription
- Process creation command-line auditing
- Advanced Windows Audit Policy
- Windows Defender event monitoring
- Authentication auditing
- Account-management auditing
- Scheduled-task monitoring
- Service-installation monitoring
- Custom Wazuh rules
- MITRE ATT&CK mapping
- Detection validation using Kali
- SOC investigation exercises

The objective is to progressively turn LAB-WIN-01 into a realistic monitored enterprise endpoint.
