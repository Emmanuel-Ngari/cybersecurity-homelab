# LAB-DC-01 — Active Directory Domain Controller

## Purple Team Cybersecurity Homelab

**Hostname:** `LAB-DC-01`  
**Role:** Active Directory Domain Controller / DNS Server / Identity Infrastructure  
**Operating System:** Windows Server 2025 Standard Evaluation  
**Network Zone:** ADNET  
**IPv4 Address:** `10.10.50.10/24`  
**Default Gateway:** `10.10.50.1`  
**Domain:** `cyberlab.internal`  
**NetBIOS Domain:** `CYBERLAB`  
**SIEM:** Wazuh  
**Wazuh Manager:** `10.10.40.10`  
**Status:** Complete, monitored, verified, and snapshotted  

---

# 1. Purpose

`LAB-DC-01` provides centralized identity and authentication services for the CyberLab.

The server was deployed to provide:

- Active Directory Domain Services
- Domain DNS
- Kerberos authentication
- Centralized user and computer management
- Organizational Units
- Security groups
- Group Policy
- Domain workstation authentication
- Privileged account management
- Active Directory security auditing
- Windows Security event telemetry
- PowerShell telemetry
- Sysmon telemetry
- Wazuh SIEM integration

The server is intentionally isolated inside the dedicated `ADNET` network rather than sharing a subnet with endpoints or general-purpose servers.

---

# 2. Network Configuration

Network:

`ADNET`

Subnet:

`10.10.50.0/24`

Domain Controller:

`10.10.50.10`

Default Gateway:

`10.10.50.1`

The gateway is provided by:

`LAB-FW-01`

OPNsense provides routing and firewall enforcement between ADNET and the other CyberLab security zones.

---

# 3. CyberLab Network Context

Current lab segmentation:

| Zone | Network | Purpose |
|---|---|---|
| REDNET | `10.10.10.0/24` | Offensive / attacker network |
| CORPNET | `10.10.20.0/24` | Corporate workstation network |
| SERVERNET | `10.10.30.0/24` | Internal server network |
| SOCNET | `10.10.40.0/24` | SIEM / security operations |
| ADNET | `10.10.50.0/24` | Active Directory infrastructure |
| DMZ | `10.10.60.0/24` | Planned perimeter services |

Important systems:

| Host | IP | Role |
|---|---|---|
| LAB-DC-01 | `10.10.50.10` | Domain Controller / DNS |
| LAB-WIN-01 | `10.10.20.20` | Domain workstation |
| LAB-UBUNTU-01 | `10.10.30.30` | Linux server |
| LAB-SIEM-01 | `10.10.40.10` | Wazuh SIEM |
| LAB-FW-01 | Multiple interfaces | OPNsense firewall |

---

# 4. Active Directory Deployment

The following Windows Server roles were installed:

- Active Directory Domain Services
- DNS Server

`LAB-DC-01` was promoted as the first Domain Controller of a new forest.

Forest:

`cyberlab.internal`

Domain:

`cyberlab.internal`

NetBIOS name:

`CYBERLAB`

Forest functional level:

`Windows Server 2025`

Domain functional level:

`Windows Server 2025`

Additional settings:

- Global Catalog: Enabled
- DNS Server: Enabled
- DNS delegation: Not configured
- AD database: `C:\Windows\NTDS`
- SYSVOL: `C:\Windows\SYSVOL`

---

# 5. Domain Verification

Domain identity was verified using:

    whoami

Expected result:

    cyberlab\administrator

Domain environment:

    echo %USERDNSDOMAIN%

Expected result:

    CYBERLAB.INTERNAL

Domain Controller discovery:

    nltest /dsgetdc:cyberlab.internal

The command successfully identified:

    LAB-DC-01.cyberlab.internal
    10.10.50.10

SYSVOL and NETLOGON shares were also verified.

---

# 6. Domain Controller Health Checks

Domain Controller diagnostics were performed using `dcdiag`.

Focused validation included:

    dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

Result:

No output.

This indicates successful completion of the requested tests.

DNS health was verified using:

    dcdiag /test:dns /q

Result:

No output.

The following Domain Controller functions were therefore verified:

- Advertising
- SYSVOL availability
- NETLOGON availability
- Required services
- DNS functionality

---

# 7. Organizational Unit Structure

The following OU hierarchy was created:

    cyberlab.internal
    └── CyberLab
        ├── Admins
        ├── Groups
        ├── Servers
        ├── Service Accounts
        ├── Users
        └── Workstations

This structure separates identities and systems according to function and allows targeted Group Policy application.

---

# 8. Security Groups

Global Security Groups created under:

`CyberLab → Groups`

Groups:

- `GG_Employees`
- `GG_Helpdesk`
- `GG_IT_Admins`
- `GG_SOC_Analysts`
- `GG_Domain_Admins`

Configuration:

    Scope: Global
    Type: Security

`GG_Domain_Admins` is retained as a classification group and is not automatically nested into the built-in `Domain Admins` group.

Privileged membership is kept explicit.

---

# 9. Domain Accounts

## Standard User

Account:

`emmanuel.patrick`

UPN:

`emmanuel.patrick@cyberlab.internal`

Location:

`CyberLab → Users`

Membership:

- Domain Users
- GG_Employees
- GG_SOC_Analysts

This account is used for standard workstation activity.

It is not a Domain Administrator.

## Privileged Administrative Account

Account:

`emmanuel.ngari-adm`

UPN:

`emmanuel.ngari-adm@cyberlab.internal`

Location:

`CyberLab → Admins`

Membership:

- Domain Users
- GG_IT_Admins
- GG_Domain_Admins
- Domain Admins

This account is used only when privileged administration is required.

The normal account and administrative account are intentionally separated to reduce privileged credential exposure.

---

# 10. Domain Workstation Integration

The Windows workstation was joined to:

`cyberlab.internal`

Current hostname:

`LAB-WIN-01`

IP address:

`10.10.20.20`

The workstation's preferred DNS server was changed to:

`10.10.50.10`

DNS validation:

    nslookup cyberlab.internal 10.10.50.10

Kerberos / Domain Controller discovery:

    nltest /dsgetdc:cyberlab.internal

The workstation computer object is stored under:

`CyberLab → Workstations`

---

# 11. Workstation Secure Channel

The domain trust relationship was verified from an elevated session on `LAB-WIN-01`.

Command:

    nltest /sc_verify:cyberlab.internal

Verified result:

    Trusted DC Name \\LAB-DC-01.cyberlab.internal
    Trusted DC Connection Status Status = 0 0x0 NERR_Success
    Trust Verification Status = 0 0x0 NERR_Success
    Authentication Service: Kerberos

This confirmed that the workstation-domain trust remained healthy after the workstation rename.

---

# 12. Firewall Rules Supporting Active Directory

OPNsense aliases used:

    LAB_DC_01 = 10.10.50.10
    AD_CLIENT_TCP
    AD_CLIENT_UDP

Active Directory TCP ports:

    53
    88
    135
    389
    445
    464
    3268
    49152:65535

Active Directory UDP ports:

    53
    88
    123
    389
    464

CORPNET rules permit required Active Directory traffic specifically to `LAB_DC_01`.

Broad unrestricted CORPNET-to-ADNET access was intentionally avoided.

---

# 13. Workstation Security GPO

Custom GPO:

`GPO-Workstation-Security-Baseline`

Linked to:

`CyberLab → Workstations`

Important settings include:

- Windows Firewall enabled
- Default inbound traffic blocked
- Default outbound traffic allowed
- Audit Logon: Success and Failure
- Audit Process Creation: Success
- Audit Special Logon: Success
- Include command line in process creation events
- PowerShell Script Block Logging
- PowerShell Module Logging

Computer policy verification:

    gpresult /scope computer /r

Confirmed:

    GPO-Workstation-Security-Baseline
    Default Domain Policy

---

# 14. Domain Controller Security GPO

Custom GPO:

`GPO-DomainController-Security-Baseline`

Linked to:

`Domain Controllers`

The default Microsoft GPOs were not modified to contain custom CyberLab hardening settings.

This maintains clearer separation between:

- Microsoft defaults
- CyberLab security configuration

---

# 15. Domain Controller Advanced Audit Policy

The following subcategories are enabled.

## Account Logon

    Kerberos Authentication Service       Success and Failure
    Kerberos Service Ticket Operations    Success and Failure
    Credential Validation                 Success and Failure

## Account Management

    User Account Management       Success and Failure
    Security Group Management     Success and Failure
    Computer Account Management   Success and Failure

## Logon / Logoff

    Logon           Success and Failure
    Special Logon   Success

## Detailed Tracking

    Process Creation    Success

## Directory Service Access

    Directory Service Changes    Success

## Policy Change

    Audit Policy Change    Success and Failure

---

# 16. Advanced Audit Policy Enforcement

Security option enabled:

`Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings`

This ensures Advanced Audit Policy subcategories take precedence over legacy audit settings.

---

# 17. Process Command-Line Auditing

The following policy is enabled:

`Computer Configuration → Administrative Templates → System → Audit Process Creation → Include command line in process creation events`

Registry validation:

`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit`

Value:

    ProcessCreationIncludeCmdLine_Enabled = 0x1

This enriches Security Event ID `4688` with process command-line data.

---

# 18. PowerShell Logging

PowerShell Script Block Logging is enabled.

Primary event:

`4104`

PowerShell Module Logging is enabled.

Primary event:

`4103`

Module wildcard:

`*`

Registry validation confirmed:

    EnableScriptBlockLogging = 0x1
    EnableModuleLogging      = 0x1

---

# 19. GPO Verification

Policy refresh was performed using:

    gpupdate /force

Effective computer policy:

    gpresult /scope computer /r

Confirmed applied GPOs:

    Default Domain Controllers Policy
    GPO-DomainController-Security-Baseline
    Default Domain Policy

---

# 20. Effective Audit Policy Verification

Audit policy was verified using:

    auditpol /get /category:"Account Logon"
    auditpol /get /category:"Account Management"
    auditpol /get /category:"Logon/Logoff"
    auditpol /get /category:"Detailed Tracking"
    auditpol /get /category:"DS Access"
    auditpol /get /category:"Policy Change"

The expected configured subcategories were confirmed at the operating-system level.

This verified that the GPO settings were not merely configured but were actually effective.

---

# 21. Wazuh Network Access

Wazuh Server:

`LAB-SIEM-01`

IP:

`10.10.40.10`

OPNsense aliases:

    LAB_DC_01
    WAZUH_SERVER
    WAZUH_AGENT_PORTS

Wazuh agent ports:

    1514
    1515

Dedicated firewall rule:

`ADNET - Allow LAB-DC-01 to Wazuh`

Connectivity testing from `LAB-DC-01`:

    Test-NetConnection 10.10.40.10 -Port 1514
    Test-NetConnection 10.10.40.10 -Port 1515

Results:

    TcpTestSucceeded : True
    TcpTestSucceeded : True

---

# 22. Wazuh Agent

Installed version:

`4.14.7`

Agent name:

`LAB-DC-01`

Manager:

`10.10.40.10`

Service:

`WazuhSvc`

Status:

`Running`

Wazuh dashboard status:

`Active`

Current monitored endpoints include:

- LAB-WIN-01
- LAB-UBUNTU-01
- LAB-DC-01

---

# 23. Windows Security Event Monitoring

Windows Security Event ID `4688` was verified in Wazuh.

Description:

`A new process has been created.`

Wazuh successfully received:

- Process executable
- Parent process
- Account
- Domain
- Command line
- Computer name
- Security context

This confirmed the telemetry path:

    Windows Security Log
    → Wazuh Agent
    → LAB-SIEM-01
    → Threat Hunting

---

# 24. PowerShell Wazuh Collection

The PowerShell Operational channel was added to:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

Configuration:

    <localfile>
      <location>Microsoft-Windows-PowerShell/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

The XML configuration was validated before restarting Wazuh.

Event ID `4104` was subsequently observed in Wazuh.

Event ID `4103` was confirmed locally.

This verified PowerShell security telemetry.

---

# 25. Kerberos Monitoring

Kerberos telemetry verified:

## Event ID 4768

Description:

`A Kerberos authentication ticket (TGT) was requested.`

Verified locally.

## Event ID 4769

Description:

`A Kerberos service ticket was requested.`

Generated by accessing:

`\\LAB-DC-01\SYSVOL`

Verified locally.

## Event ID 4771

Description:

`Kerberos pre-authentication failed.`

A controlled wrong-password attempt generated a Wazuh alert.

Observed data:

    Target user: emmanuel.patrick
    Source:      10.10.20.20
    Status:      0x18

`0x18` represented an incorrect password / Kerberos pre-authentication failure.

---

# 26. User Account Lifecycle Monitoring

Temporary account:

`telemetry.test`

Creation generated:

`4720`

Description:

`A user account was created.`

Wazuh correctly identified the new account.

Deletion generated:

`4726`

Description:

`A user account was deleted.`

Wazuh successfully detected the deletion.

The temporary account was removed after testing.

---

# 27. Security Group Monitoring

Temporary group:

`GG_Telemetry_Test`

Account added:

`emmanuel.patrick`

Addition generated:

`4728`

Description:

`A member was added to a security-enabled global group.`

Removal generated:

`4729`

Description:

`A member was removed from a security-enabled global group.`

Both events were detected in Wazuh.

The temporary group was deleted after testing.

---

# 28. Privileged Logon Monitoring

Event ID:

`4672`

Description:

`Special privileges assigned to new logon.`

Wazuh captured the event for:

    Domain:   CYBERLAB
    Account:  Administrator
    Computer: LAB-DC-01.cyberlab.internal

The event contained privileged rights including:

- Backup
- Restore
- Ownership
- Debugging
- Impersonation
- Security-related privileges

---

# 29. Audit Policy Change Monitoring

Event ID:

`4719`

Description:

`System audit policy was changed.`

Multiple historical events were confirmed locally on `LAB-DC-01`.

This provides visibility into attempts to modify Windows auditing configuration.

---

# 30. Directory Service Change Monitoring

Policy:

`Audit Directory Service Changes`

Status:

`Success`

Event ID:

`5136`

Historical `5136` events were confirmed on the Domain Controller.

Recent test objects did not automatically generate `5136` because Directory Service Change logging also depends on SACL configuration on the affected AD objects.

A broad domain-wide SACL was intentionally not configured to avoid unnecessary telemetry volume.

---

# 31. Sysmon Installation

Sysmon version:

`15.22`

Schema:

`4.91`

Location:

`C:\Tools\Sysmon`

Important files:

- `Sysmon.exe`
- `Sysmon64.exe`
- `Sysmon64a.exe`
- `sysmon-dc.xml`

Service:

`Sysmon64`

Status:

`Running`

Driver:

`SysmonDrv`

The Sysmon Operational log is located at:

`Microsoft-Windows-Sysmon/Operational`

---

# 32. Sysmon Monitoring Scope

The Domain Controller Sysmon configuration includes visibility into:

- Selected process creation
- Suspicious network connections
- Driver loading
- Remote thread creation
- LSASS process access
- Suspicious script/file creation
- Registry persistence
- Selected DNS queries
- Process tampering

LSASS access monitoring provides telemetry useful for detecting credential-dumping activity.

Persistence monitoring includes paths such as:

- `CurrentVersion\Run`
- `CurrentVersion\RunOnce`
- `SYSTEM\CurrentControlSet\Services`

Suspicious script/file extensions include:

- `.ps1`
- `.psm1`
- `.bat`
- `.cmd`
- `.vbs`
- `.js`
- `.hta`

---

# 33. Sysmon Wazuh Integration

The Sysmon event channel was added to:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

Configuration:

    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

The configuration was validated before restarting the Wazuh service.

Wazuh subsequently received Sysmon telemetry from:

`LAB-DC-01`

---

# 34. Sysmon Noise Tuning

The initial Sysmon configuration logged every process creation.

This duplicated large amounts of information already available through Windows Security Event ID `4688`.

A backup was created:

`C:\Tools\Sysmon\sysmon-dc-before-tuning.xml`

Process creation was then restricted to higher-value executables:

- `powershell.exe`
- `pwsh.exe`
- `cmd.exe`
- `rundll32.exe`
- `regsvr32.exe`
- `mshta.exe`
- `certutil.exe`
- `bitsadmin.exe`
- `wscript.exe`
- `cscript.exe`
- `schtasks.exe`
- `nltest.exe`
- `ntdsutil.exe`
- `whoami.exe`
- `net.exe`
- `net1.exe`

DNS monitoring was restricted to:

- `powershell.exe`
- `pwsh.exe`
- `cmd.exe`
- `certutil.exe`
- `mshta.exe`
- `wscript.exe`
- `cscript.exe`

This reduced routine Domain Controller noise while retaining higher-value security telemetry.

---

# 35. Sysmon Tuning Verification

A controlled comparison was performed.

Test 1:

`notepad.exe`

Result:

`No new Sysmon Event ID 1`

Test 2:

`cmd.exe /c echo SYSMON-TUNING-TEST`

Result:

`Sysmon Event ID 1 generated`

This confirmed that the tuned configuration successfully filters ordinary processes while retaining higher-risk command interpreters.

---

# 36. Important Security Events Verified

| Event ID | Description | Verification |
|---|---|---|
| 4103 | PowerShell Module Logging | Verified locally |
| 4104 | PowerShell Script Block Logging | Verified in Wazuh |
| 4672 | Special privileges assigned to new logon | Verified in Wazuh |
| 4688 | Process creation | Verified in Wazuh |
| 4719 | Audit policy changed | Verified locally |
| 4720 | User account created | Verified in Wazuh |
| 4726 | User account deleted | Verified in Wazuh |
| 4728 | Member added to security group | Verified in Wazuh |
| 4729 | Member removed from security group | Verified in Wazuh |
| 4768 | Kerberos TGT requested | Verified locally |
| 4769 | Kerberos service ticket requested | Verified locally |
| 4771 | Kerberos pre-authentication failed | Verified in Wazuh |
| 5136 | Directory object modified | Historical verification |
| Sysmon 1 | Process Create | Verified locally and in Wazuh |
| Sysmon 10 | Process Access | Verified locally |

---

# 37. Security Design Decisions

## Dedicated AD Network

The Domain Controller is placed in:

`ADNET`

instead of CORPNET or SERVERNET.

This isolates identity infrastructure from ordinary hosts.

## Dedicated Administrative Account

Normal work:

`emmanuel.patrick`

Privileged administration:

`emmanuel.ngari-adm`

This reduces unnecessary use of Domain Administrator credentials.

## Domain Controller Is Not a Browsing Workstation

`LAB-DC-01` is not used for:

- General web browsing
- Wazuh dashboard access
- OPNsense administration
- Routine user activity

Management interfaces are accessed from:

`LAB-WIN-01`

## Firewall Access Is Explicit

Only required ports are opened between security zones.

No broad CORPNET-to-ADNET or ADNET-to-SOCNET allow-any rules were introduced.

## Sysmon Is Tuned

Sysmon complements rather than duplicates native Windows auditing.

Windows Event `4688` provides broad process telemetry.

Sysmon provides additional higher-value endpoint visibility.

---

# 38. Recovery Points

Important VirtualBox snapshots associated with the Domain Controller phase include:

- `Pre-ADDS Baseline`
- `Pre-Domain-Admin-Delegation`
- `Pre-AD-GPO-Baseline`
- `AD-GPO-Workstation-Baseline-Configured`
- `AD-Workstation-Integration-Complete`
- `AD-DC-Security-Baseline-Verified`
- `Pre-DC-PowerShell-Wazuh-Collection`
- `Pre-DC-Sysmon-Deployment`
- `DC-Sysmon-Wazuh-Verified-Pre-Tuning`
- `AD-DC-Monitoring-Complete`

Additional firewall and workstation snapshots were created before corresponding integration changes.

Snapshots are treated as recovery points rather than normal configuration-management tools.

---

# 39. Current Operational State

`LAB-DC-01` is currently:

    Active Directory Domain Controller     HEALTHY
    DNS Server                             HEALTHY
    Kerberos                               VERIFIED
    SYSVOL                                 VERIFIED
    NETLOGON                               VERIFIED
    Group Policy                           VERIFIED
    Advanced Audit Policy                  VERIFIED
    Command-Line Auditing                  VERIFIED
    PowerShell Logging                     VERIFIED
    Wazuh Agent                            ACTIVE
    Windows Security Telemetry             VERIFIED
    AD Account Monitoring                  VERIFIED
    AD Group Monitoring                    VERIFIED
    Privileged Logon Monitoring            VERIFIED
    Kerberos Failure Detection             VERIFIED
    Sysmon                                 RUNNING
    Sysmon Wazuh Integration               VERIFIED
    Sysmon Noise Tuning                    VERIFIED
    Final Monitoring Snapshot              COMPLETE

---

# 40. Known / Intentional Limitations

The following are intentional and are not currently considered faults:

- Successful Kerberos events such as `4768` may exist locally without becoming Wazuh alerts.
- PowerShell `4103` events may exist locally without generating a Wazuh alert.
- Directory Service Event `5136` requires appropriate SACL configuration on monitored AD objects.
- A broad domain-wide SACL has not been enabled.
- `GG_Domain_Admins` is not automatically nested into the built-in `Domain Admins` group.
- `LAB-DC-01` is not used as a general-purpose administrative workstation.

These choices reduce noise and preserve least-privilege design.

---

# 41. Final Status

The `LAB-DC-01` build is complete.

The system now provides:

- Active Directory
- DNS
- Kerberos
- Domain authentication
- Centralized identity management
- Group Policy
- Advanced Windows auditing
- PowerShell telemetry
- Wazuh SIEM integration
- Active Directory security monitoring
- Tuned Sysmon endpoint telemetry

The Domain Controller is now suitable for future Purple Team scenarios involving:

- Password spraying detection
- Kerberos abuse
- Kerberoasting investigation
- Privilege escalation
- Unauthorized group membership changes
- Suspicious account creation
- PowerShell abuse
- Credential-access activity
- Persistence
- Lateral movement
- Domain Controller attack simulation
- Incident response
- Detection engineering

---

# 42. Next Infrastructure Phase

The next major build phase is the CyberLab DMZ.

Planned network:

`10.10.60.0/24`

Planned gateway:

`10.10.60.1`

The Domain Controller should remain at its current validated baseline while DMZ development proceeds.
