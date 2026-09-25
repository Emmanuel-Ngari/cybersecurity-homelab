# Windows Advanced Security Auditing

## 1. Overview

This document records the advanced Windows security auditing and Wazuh monitoring implemented and verified on:

`LAB-WIN-01`

The objective was to expand endpoint telemetry beyond Sysmon, PowerShell logging, and File Integrity Monitoring by collecting authentication, process creation, privilege assignment, user account management, and privileged group membership activity.

The implementation was verified locally through the Windows Security Event Log and centrally through Wazuh.

---

# 2. Process Creation Auditing — Event ID 4688

## Initial State

The Windows Advanced Audit Policy for Process Creation was checked using:

`auditpol /get /subcategory:"Process Creation"`

Initial result:

`Process Creation    No Auditing`

Process Creation auditing was therefore enabled.

Command:

`auditpol /set /subcategory:"Process Creation" /success:enable`

Verification:

`auditpol /get /subcategory:"Process Creation"`

Result:

`Process Creation    Success`

---

## 3. Process Command-Line Logging

Windows was also configured to include command-line arguments inside process creation events.

Registry configuration:

`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit`

Value:

`ProcessCreationIncludeCmdLine_Enabled = 1`

Command used:

`reg.exe add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f`

Verification:

`Get-ItemPropertyValue -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" -Name ProcessCreationIncludeCmdLine_Enabled`

Result:

`1`

---

# 4. Controlled Event ID 4688 Test

A controlled PowerShell child process was created using:

`Start-Process powershell.exe -ArgumentList '-NoProfile','-Command','Write-Output "CYBERLAB_4688_TEST_20260925"' -Wait`

The Windows Security Event Log was queried for Event ID 4688.

Verified event:

- Event ID: `4688`
- Description: `A new process has been created.`
- User: `Emmanuel`
- New Process: `powershell.exe`
- Parent Process: `powershell.exe`
- Token Elevation Type: Full / elevated
- Command-line capture: enabled
- Marker: `CYBERLAB_4688_TEST_20260925`

The recorded command line contained:

`powershell.exe -NoProfile -Command Write-Output "CYBERLAB_4688_TEST_20260925"`

This confirmed that both Process Creation auditing and command-line capture were functioning correctly.

---

# 5. Wazuh Security Event Channel Collection

The Wazuh agent configuration was inspected at:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

The Security Event Channel was already configured:

`<location>Security</location>`

with:

`<log_format>eventchannel</log_format>`

The existing query excluded several noisy Windows Security Event IDs but did not exclude Event ID `4688`.

No Wazuh agent configuration change was required.

---

# 6. Event ID 4688 in Wazuh

The controlled process creation event was confirmed on `LAB-SIEM-01`.

Wazuh generated:

- Rule ID: `67027`
- Description: `A process was created.`
- Agent: `LAB-WIN-01`
- Windows Event ID: `4688`
- Command-line marker: `CYBERLAB_4688_TEST_20260925`

The event was also successfully located in the Wazuh Threat Hunting interface.

---

# 7. Sysmon Correlation

The same process creation was also independently detected through Sysmon Event ID 1.

Wazuh generated the more specific detection:

- Rule ID: `92027`
- Rule Level: `4`
- Description: `Powershell process spawned powershell instance`
- MITRE ATT&CK: `T1059.001`
- Tactic: `Execution`
- Technique: `PowerShell`

The Security Event 4688 process information correlated with the Sysmon Event ID 1 process information.

Observed correlation:

Security Event 4688:

- New Process ID: `0x13a0`
- Creator Process ID: `0x2464`

Sysmon:

- Process ID: `5024`
- Parent Process ID: `9316`

Hexadecimal conversion confirms:

`0x13a0 = 5024`

`0x2464 = 9316`

Both telemetry sources therefore described the same process execution.

This demonstrated the value of correlating multiple endpoint telemetry sources rather than depending on a single log source.

---

# 8. Authentication Auditing

The Windows Logon audit policy was checked:

`auditpol /get /subcategory:"Logon"`

Result:

`Logon    Success and Failure`

No configuration change was required.

The following authentication events were tested and verified.

---

# 9. Successful Logon — Event ID 4624

Successful Windows authentication events were observed using Event ID:

`4624`

A verified event for user:

`Emmanuel`

contained:

- Logon Type: `2`
- Logon Type Meaning: Interactive
- Authentication Package: `Negotiate`
- Workstation: `DESKTOP-7T1HFS1`
- Source Address: `127.0.0.1`
- Account: `Emmanuel`

Wazuh detection:

- Rule ID: `60118`
- Rule Level: `3`
- Description: `Windows Workstation Logon Success`
- MITRE ATT&CK: `T1078`
- Technique: `Valid Accounts`

The event was verified through Wazuh Threat Hunting.

---

# 10. Failed Logon — Event ID 4625

A controlled failed network authentication was generated using:

`net use \\127.0.0.1\IPC$ /user:.\CYBERLAB_FAIL_TEST WrongPassword123!`

The authentication failed as expected.

Windows generated:

- Event ID: `4625`
- Description: `An account failed to log on.`
- Target Account: `CYBERLAB_FAIL_TEST`
- Logon Type: `3`
- Logon Type Meaning: Network
- Authentication Package: `NTLM`
- Source Address: `127.0.0.1`
- Failure Reason: Unknown user name or bad password
- Status: `0xC000006D`
- Sub Status: `0xC0000064`

Wazuh generated:

- Rule ID: `60122`
- Rule Level: `5`
- Description: `Logon Failure - Unknown user or bad password`

The event was successfully verified through the Wazuh Threat Hunting interface.

---

# 11. Special Privilege Assignment — Event ID 4672

The Special Logon audit policy was checked:

`auditpol /get /subcategory:"Special Logon"`

Result:

`Special Logon    Success`

Event ID `4672` was observed for both SYSTEM and administrator sessions.

A useful administrator event was identified for:

- Account: `Emmanuel`
- Domain: `DESKTOP-7T1HFS1`
- Logon ID: `0x811B0`

Privileges included:

- `SeSecurityPrivilege`
- `SeTakeOwnershipPrivilege`
- `SeLoadDriverPrivilege`
- `SeBackupPrivilege`
- `SeRestorePrivilege`
- `SeDebugPrivilege`
- `SeSystemEnvironmentPrivilege`
- `SeImpersonatePrivilege`
- `SeDelegateSessionUserImpersonatePrivilege`

---

# 12. Logon ID Correlation

The earlier successful 4624 event contained:

- Target Logon ID: `0x811ED`
- Linked Logon ID: `0x811B0`

The Event ID 4672 privileged session contained:

- Logon ID: `0x811B0`

This allowed the authentication and privilege events to be correlated:

`4624 Interactive Logon`

→ `Linked administrative session`

→ `4672 Special Privileges Assigned`

This demonstrates how Logon IDs can be used during SOC investigations to connect authentication and privilege events.

The 4672 event was also verified in Wazuh Threat Hunting.

---

# 13. User Account Management Auditing

The User Account Management audit policy was checked:

`auditpol /get /subcategory:"User Account Management"`

Result:

`User Account Management    Success`

No policy modification was required.

A temporary lab account was created:

`CYBERLAB_AUDIT_TEST`

The account password was entered interactively using:

`net user CYBERLAB_AUDIT_TEST * /add`

The `*` option was intentionally used so the password would not appear inside process command-line telemetry.

---

# 14. Event ID 4720 — Account Created

Windows generated:

- Event ID: `4720`
- Description: `A user account was created.`
- Creator: `Emmanuel`
- Account: `CYBERLAB_AUDIT_TEST`

The event was verified locally and through Wazuh Threat Hunting.

---

# 15. Account Creation Sequence

Immediately following account creation, Windows generated:

`4720` — User account created

`4722` — User account enabled

`4738` — User account changed

The three events occurred during the same account initialization sequence.

---

# 16. Account Disable / Enable Lifecycle

The temporary account was disabled using:

`net user CYBERLAB_AUDIT_TEST /active:no`

Windows generated:

`4725` — User account disabled

The account was then re-enabled:

`net user CYBERLAB_AUDIT_TEST /active:yes`

Windows generated:

`4722` — User account enabled

Windows also generated Event ID `4738` during the associated state changes.

---

# 17. Account Deletion

The temporary account was deleted using:

`net user CYBERLAB_AUDIT_TEST /delete`

Windows generated:

`4726` — User account deleted

The complete observed account lifecycle was:

`4720` — Created

`4722` — Enabled

`4738` — Changed

`4725` — Disabled

`4738` — Changed

`4722` — Re-enabled

`4738` — Changed

`4726` — Deleted

The lifecycle was verified locally and centrally through Wazuh Threat Hunting.

---

# 18. Security Group Management Auditing

The Security Group Management audit policy was checked:

`auditpol /get /subcategory:"Security Group Management"`

Result:

`Security Group Management    Success`

No configuration change was required.

A temporary account was created:

`CYBERLAB_GROUP_TEST`

The account was used to test privileged local-group membership monitoring.

---

# 19. Event ID 4732 — Added to Administrators

The test user was added to the local Administrators group:

`net localgroup Administrators CYBERLAB_GROUP_TEST /add`

Windows generated:

`4732` — A member was added to a security-enabled local group

Observed fields:

- Performed by: `Emmanuel`
- Group: `Administrators`
- Group SID: `S-1-5-32-544`
- Member SID ending in: `-1003`

Windows did not populate the account name directly in the Member field.

The SID was therefore resolved using:

`Get-LocalUser CYBERLAB_GROUP_TEST | Select-Object Name,SID`

Result:

`CYBERLAB_GROUP_TEST`

mapped to:

`S-1-5-21-3188534369-2127354081-4186760499-1003`

This confirmed that the 4732 event represented the test account being added to the local Administrators group.

---

# 20. Event ID 4733 — Removed from Administrators

The account was removed from the local Administrators group:

`net localgroup Administrators CYBERLAB_GROUP_TEST /delete`

Windows generated:

`4733` — A member was removed from a security-enabled local group

Observed:

- Performed by: `Emmanuel`
- Group: `Administrators`
- Group SID: `S-1-5-32-544`
- Member SID ending in: `-1003`

Both Event IDs `4732` and `4733` were verified through Wazuh Threat Hunting by filtering on the member SID.

---

# 21. Final Test Account Cleanup

The temporary group-membership test account was deleted:

`net user CYBERLAB_GROUP_TEST /delete`

Windows generated:

`4726` — User account deleted

Verified:

- Account: `CYBERLAB_GROUP_TEST`
- Performed by: `Emmanuel`
- Target SID ending in: `-1003`

No temporary audit accounts remain on the system.

---

# 22. SOC Investigation Skills Practiced

The exercises demonstrated several important SOC investigation techniques:

- Filtering Wazuh events by structured fields
- Searching by Windows Event ID
- Searching by account name
- Searching by SID
- Correlating multiple telemetry sources
- Correlating events using Logon IDs
- Comparing successful and failed authentication
- Distinguishing interactive and network logons
- Investigating privileged sessions
- Tracking user account lifecycle activity
- Tracking privileged group membership changes
- Resolving SIDs when account names are unavailable
- Using MITRE ATT&CK mappings
- Comparing local Windows evidence with central SIEM evidence

The general investigation workflow used was:

`Host`

→ `Time`

→ `Event ID`

→ `User / SID`

→ `Logon ID`

→ `Process / Command`

→ `Context`

→ `Disposition`

---

# 23. Verified Windows Security Event Coverage

The following Windows Security Event IDs have now been tested and verified:

| Event ID | Description |
| --- | --- |
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4672 | Special privileges assigned to new logon |
| 4688 | New process created |
| 4720 | User account created |
| 4722 | User account enabled |
| 4725 | User account disabled |
| 4726 | User account deleted |
| 4732 | Member added to security-enabled local group |
| 4733 | Member removed from security-enabled local group |
| 4738 | User account changed |

---

# 24. Endpoint Telemetry Layers

`LAB-WIN-01` now provides several complementary telemetry sources:

- Windows Security Event Log
- Sysmon
- PowerShell Script Block Logging — Event ID 4104
- PowerShell Module Logging — Event ID 4103
- PowerShell Transcription
- Wazuh File Integrity Monitoring
- Windows Process Creation auditing — Event ID 4688
- Authentication auditing
- Privileged-logon auditing
- User account management auditing
- Security group management auditing

These telemetry sources are centrally collected and investigated through Wazuh.

---

# 25. Verified Monitoring Pipeline

The tested telemetry path is:

`LAB-WIN-01`

→ `Windows Security / Sysmon`

→ `Wazuh Agent`

→ `CORPNET`

→ `OPNsense`

→ `SOCNET`

→ `LAB-SIEM-01`

→ `Wazuh Manager`

→ `Filebeat`

→ `Wazuh Indexer`

→ `Wazuh Threat Hunting Dashboard`

The complete pipeline was successfully verified.

---

# 26. Snapshot Milestone

After all temporary accounts were removed and the endpoint returned to a clean state, the following powered-off milestone snapshot was created:

`Windows - Advanced Security Auditing Verified`

This snapshot represents the verified endpoint state after:

- Process creation auditing
- Command-line capture
- Authentication monitoring
- Privileged-logon monitoring
- User account management monitoring
- Security group membership monitoring
- Wazuh Threat Hunting verification

---

# 27. Snapshot Storage Maintenance

During this phase, host storage dropped to approximately:

`17.2 GB free`

A review showed excessive VirtualBox snapshot chains consuming significant disk capacity.

Snapshots were removed using `VBoxManage snapshot ... delete`.

Snapshot `.vdi`, `.sav`, and VirtualBox snapshot directories were never manually deleted.

After cleanup, host storage increased to approximately:

`139.15 GB free`

This reinforced the operational rule:

`Snapshots are rollback points, not backups.`

Only meaningful milestone snapshots should be retained.

---

# 28. Current Windows Snapshot Strategy

After cleanup, the primary Windows snapshot chain was reduced to meaningful milestones:

`CORPNET Final Cutover Verified`

→ `Windows - PowerShell Transcription + Wazuh FIM Verified`

→ `Windows - Advanced Security Auditing Verified`

This substantially reduces snapshot sprawl while preserving useful recovery points.

---

# 29. Security Lessons Learned

### Process telemetry

Knowing that `powershell.exe` ran is useful.

Knowing:

`powershell.exe -NoProfile -Command ...`

is much more useful.

Command-line capture significantly improves process investigation.

### Multiple telemetry sources

The same PowerShell process was independently observed through:

- Windows Security Event ID 4688
- Sysmon Event ID 1

Independent telemetry sources can be correlated to increase investigation confidence.

### Authentication context

A successful or failed logon alone is insufficient.

Analysts should examine:

- user
- source
- logon type
- authentication package
- status
- privilege assignment
- associated process activity

### Privileged-group changes

Administrators-group membership changes are important security events.

Even when Windows provides only a SID, the SID can be resolved and correlated with other events.

### Temporary credentials

Passwords should not be placed directly into command lines on systems where process command-line auditing is enabled.

Interactive password entry was used during test-account creation to avoid recording lab credentials in telemetry.

---

# 30. Result

Advanced Windows auditing is operational on `LAB-WIN-01`.

Verified capabilities include:

- Process creation auditing
- Process command-line capture
- Security Event ID 4688
- Sysmon process correlation
- Successful logon monitoring
- Failed logon monitoring
- Special privilege monitoring
- Account creation monitoring
- Account enable/disable monitoring
- Account modification monitoring
- Account deletion monitoring
- Local Administrators membership monitoring
- SID-based investigation
- Logon-ID correlation
- Wazuh Threat Hunting verification
- MITRE ATT&CK context
- Endpoint-to-SIEM telemetry verification

---

# 31. Next Phase

The next Windows/security-monitoring phase will continue with additional telemetry such as:

- Microsoft Defender Operational events
- Windows Firewall logging
- Service installation monitoring
- Scheduled task monitoring
- Persistence-related Windows events

After Windows telemetry is complete, the broader homelab roadmap continues with:

`Detection Engineering`

→ `OPNsense Log Centralization`

→ `Suricata IDS`

→ `Linux Security Monitoring`

→ `Active Directory`

→ `DMZ`

→ `Attack / Detect / Investigate / Respond`

→ `Threat Hunting`

→ `Incident Response`

→ `Final Capstone`
