# Active Directory Domain Services & Domain Controller Security Monitoring

## Purple Team CyberLab

### Project Phase
Active Directory Infrastructure, Domain Integration, Group Policy Hardening, Wazuh Monitoring, and Sysmon Telemetry

### Status
COMPLETED AND VERIFIED

---

# 1. Overview

This phase introduced a dedicated Microsoft Active Directory environment into the Purple Team CyberLab.

The objective was not simply to install Active Directory Domain Services, but to build a realistic enterprise identity environment that could later support:

- Centralized authentication
- Domain user management
- Group-based access control
- Group Policy enforcement
- Kerberos authentication
- Windows endpoint domain membership
- Domain Controller security auditing
- Wazuh SIEM monitoring
- PowerShell telemetry
- Sysmon endpoint telemetry
- Active Directory attack detection
- Purple Team simulations

The final environment provides a monitored Windows domain with a dedicated Domain Controller, a domain-joined Windows workstation, segmented firewall access, advanced Windows auditing, PowerShell monitoring, Wazuh integration, and tuned Sysmon telemetry.

---

# 2. Active Directory Network Architecture

The CyberLab uses segmented networks controlled by OPNsense.

Active Directory was placed in its own dedicated security zone.

Network:

ADNET: 10.10.50.0/24

Domain Controller:

Hostname: LAB-DC-01
IP Address: 10.10.50.10
Subnet Mask: 255.255.255.0
Default Gateway: 10.10.50.1
Role: Active Directory Domain Controller / DNS Server
Operating System: Windows Server 2025 Standard Evaluation

Domain:

cyberlab.internal

NetBIOS Domain:

CYBERLAB

The existing workstation resides in:

CORPNET: 10.10.20.0/24

Windows workstation:

Hostname: LAB-WIN-01
IP Address: 10.10.20.20

The SIEM resides in:

SOCNET: 10.10.40.0/24

Wazuh server:

Hostname: LAB-SIEM-01
IP Address: 10.10.40.10

---

# 3. Domain Controller Deployment

Windows Server 2025 was deployed as:

LAB-DC-01

The server was assigned the static IP:

10.10.50.10/24

Gateway:

10.10.50.1

The server was configured with:

- Active Directory Domain Services
- DNS Server
- Global Catalog
- Kerberos authentication services
- SYSVOL
- NETLOGON
- Group Policy Management

A new Active Directory forest was created.

Forest root domain:

cyberlab.internal

NetBIOS domain:

CYBERLAB

Forest functional level:

Windows Server 2025

Domain functional level:

Windows Server 2025

The server was promoted to the first Domain Controller in the CyberLab forest.

After promotion, the system was verified using:

whoami

Expected result:

cyberlab\administrator

Domain verification:

%USERDNSDOMAIN%

Expected result:

CYBERLAB.INTERNAL

Domain Controller discovery was tested using:

nltest /dsgetdc:cyberlab.internal

The result successfully identified:

LAB-DC-01.cyberlab.internal

at:

10.10.50.10

SYSVOL and NETLOGON shares were also verified.

---

# 4. Domain Controller Health Verification

Initial Domain Controller diagnostics were performed using:

dcdiag

Initial warnings were related to recently initialized DFSR/SYSVOL events and the fact that Secure Boot is disabled in the virtual machine.

Focused health tests were then performed.

The following tests completed successfully:

Advertising
SysVolCheck
NetLogons
Services
DNS

Commands used included:

dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

and:

dcdiag /test:dns /q

No output from these /q tests indicated successful completion.

---

# 5. Active Directory Organizational Unit Structure

A dedicated parent Organizational Unit was created:

CyberLab

Inside it, the following OUs were created:

CyberLab
├── Admins
├── Users
├── Groups
├── Service Accounts
├── Servers
└── Workstations

This structure separates identities and systems by role and allows Group Policy to be targeted precisely.

---

# 6. Active Directory Security Groups

The following Global Security Groups were created under:

CyberLab → Groups

Groups:

GG_Employees
GG_Helpdesk
GG_IT_Admins
GG_SOC_Analysts
GG_Domain_Admins

Group configuration:

Scope: Global
Type: Security

The GG_Domain_Admins group is currently treated as a classification group and is not automatically nested into Domain Admins.

Privileged access was kept explicit rather than hidden behind unnecessary nested group membership.

---

# 7. Domain User Accounts

## Standard User

A normal domain user was created under:

CyberLab → Users

User:

Emmanuel Patrick

Logon name:

emmanuel.patrick

UPN:

emmanuel.patrick@cyberlab.internal

Membership:

Domain Users
GG_Employees
GG_SOC_Analysts

This account is used for normal workstation activity.

It is not a Domain Administrator.

---

# 8. Privileged Administrative Account

A separate privileged account was created under:

CyberLab → Admins

Display name:

Emmanuel Ngari

Logon name:

emmanuel.ngari-adm

UPN:

emmanuel.ngari-adm@cyberlab.internal

Membership:

Domain Users
GG_IT_Admins
GG_Domain_Admins
Domain Admins

This account is used only when administrative elevation is required.

Normal day-to-day use continues under:

emmanuel.patrick

This separation follows the principle of least privilege and avoids using Domain Administrator credentials for routine activities.

---

# 9. Firewall Segmentation for Active Directory

OPNsense was used to restrict traffic between CORPNET and ADNET.

Rather than allowing unrestricted access between the two networks, only required Active Directory client traffic was permitted.

The Domain Controller alias was created:

LAB_DC_01

Address:

10.10.50.10

Active Directory TCP alias:

AD_CLIENT_TCP

Ports:

53
88
135
389
445
464
3268
49152:65535

Active Directory UDP alias:

AD_CLIENT_UDP

Ports:

53
88
123
389
464

CORPNET rules were created allowing clients to communicate only with LAB_DC_01 on the required Active Directory ports.

Rules:

CORPNET - Allow AD TCP to LAB-DC-01
CORPNET - Allow AD UDP to LAB-DC-01

These rules were positioned before the broader private-network blocking rule.

This prevented a broad CORPNET-to-ADNET allow rule from being introduced.

---

# 10. Workstation DNS Configuration

Before joining the workstation to the domain, LAB-WIN-01 was configured to use the Domain Controller for DNS.

Preferred DNS:

10.10.50.10

DNS resolution was verified using:

nslookup cyberlab.internal 10.10.50.10

Result:

cyberlab.internal → 10.10.50.10

Domain Controller SRV records were verified using:

_ldap._tcp.dc._msdcs.cyberlab.internal

The query successfully returned:

LAB-DC-01.cyberlab.internal

Port:

389

IP:

10.10.50.10

Domain Controller discovery from the workstation was verified using:

nltest /dsgetdc:cyberlab.internal

---

# 11. LAB-WIN-01 Domain Join

The workstation was successfully joined to:

cyberlab.internal

Administrative credentials used:

CYBERLAB\emmanuel.ngari-adm

After restart, the workstation computer object was moved from the default Computers container into:

CyberLab → Workstations

The workstation was then restarted again so computer-targeted Group Policy could apply correctly.

The normal domain user logged in successfully as:

emmanuel.patrick@cyberlab.internal

Identity verification:

whoami

Result:

cyberlab\emmanuel.patrick

Logon server:

\\LAB-DC-01

Group membership verification confirmed:

GG_Employees
GG_SOC_Analysts

---

# 12. Workstation Rename

The original workstation hostname was:

DESKTOP-7T1HFS1

It was renamed to:

LAB-WIN-01

After restart, verification was performed using:

hostname

Result:

LAB-WIN-01

Active Directory Users and Computers also automatically reflected the new computer name.

The computer object remained inside:

CyberLab → Workstations

---

# 13. Domain Secure Channel Verification

The workstation-domain trust relationship was verified from an elevated Command Prompt.

Command:

nltest /sc_verify:cyberlab.internal

Result:

Trusted DC Name:
\\LAB-DC-01.cyberlab.internal

Trusted DC Connection Status:
Status = 0 0x0 NERR_Success

Trust Verification Status:
Status = 0 0x0 NERR_Success

Authentication Service:

Kerberos

This confirmed that the workstation rename did not break the domain secure channel.

---

# 14. Workstation Group Policy Baseline

A custom Group Policy Object was created:

GPO-Workstation-Security-Baseline

It was linked specifically to:

CyberLab → Workstations

The built-in Default Domain Policy and Default Domain Controllers Policy were not modified for workstation hardening.

Configured controls included:

## Windows Firewall

Domain Profile:

Firewall: On
Inbound: Block
Outbound: Allow

Private Profile:

Firewall: On
Inbound: Block
Outbound: Allow

Public Profile:

Firewall: On
Inbound: Block
Outbound: Allow

## Advanced Audit Policy

Audit Logon:

Success
Failure

Audit Process Creation:

Success

Audit Special Logon:

Success

## Process Command-Line Auditing

Policy:

Include command line in process creation events

Status:

Enabled

## PowerShell Script Block Logging

Policy:

Turn on PowerShell Script Block Logging

Status:

Enabled

Invocation start/stop logging was left disabled.

## PowerShell Module Logging

Policy:

Turn on Module Logging

Status:

Enabled

Module name:

*

---

# 15. Workstation GPO Verification

Computer-side Group Policy was verified using:

gpresult /scope computer /r

Result:

Applied Group Policy Objects:

GPO-Workstation-Security-Baseline
Default Domain Policy

The workstation object was confirmed at:

CN=LAB-WIN-01,OU=Workstations,OU=CyberLab,DC=cyberlab,DC=internal

Policy was applied from:

LAB-DC-01.cyberlab.internal

---

# 16. Domain Controller Security Baseline GPO

A separate GPO was created specifically for Domain Controllers:

GPO-DomainController-Security-Baseline

It was linked to:

Domain Controllers OU

The workstation baseline was not reused for Domain Controllers.

This allowed Domain Controller monitoring to remain independent and easier to troubleshoot.

---

# 17. Domain Controller Advanced Audit Policy

The following Advanced Audit Policy settings were configured.

## Account Logon

Audit Kerberos Authentication Service:

Success and Failure

Audit Kerberos Service Ticket Operations:

Success and Failure

Audit Credential Validation:

Success and Failure

## Account Management

Audit User Account Management:

Success and Failure

Audit Security Group Management:

Success and Failure

Audit Computer Account Management:

Success and Failure

## Directory Service Access

Audit Directory Service Changes:

Success

## Logon/Logoff

Audit Logon:

Success and Failure

Audit Special Logon:

Success

## Detailed Tracking

Audit Process Creation:

Success

## Policy Change

Audit Audit Policy Change:

Success and Failure

---

# 18. Advanced Audit Policy Enforcement

The following Security Option was enabled:

Audit: Force audit policy subcategory settings
(Windows Vista or later) to override audit policy category settings

This prevents legacy audit-policy settings from overriding the more precise Advanced Audit Policy subcategories.

---

# 19. Domain Controller Process Command-Line Auditing

The following policy was enabled:

Computer Configuration
→ Administrative Templates
→ System
→ Audit Process Creation
→ Include command line in process creation events

Status:

Enabled

This enriches Event ID 4688 with the actual command line used to start a process.

---

# 20. Domain Controller PowerShell Logging

The following PowerShell controls were enabled through Group Policy.

Script Block Logging:

Enabled

Primary Event ID:

4104

Module Logging:

Enabled

Module Names:

*

Primary Event ID:

4103

This allows detailed PowerShell activity to be captured for later analysis and detection engineering.

---

# 21. Domain Controller GPO Verification

The Domain Controller baseline was applied using:

gpupdate /force

Computer policy and user policy both completed successfully.

Effective Group Policy was verified using:

gpresult /scope computer /r

Applied GPOs included:

Default Domain Controllers Policy
GPO-DomainController-Security-Baseline
Default Domain Policy

---

# 22. Effective Audit Policy Verification

The actual Windows audit configuration was verified using auditpol.

Account Logon:

Kerberos Service Ticket Operations:
Success and Failure

Kerberos Authentication Service:
Success and Failure

Credential Validation:
Success and Failure

Account Management:

Computer Account Management:
Success and Failure

Security Group Management:
Success and Failure

User Account Management:
Success and Failure

Logon/Logoff:

Logon:
Success and Failure

Special Logon:
Success

Detailed Tracking:

Process Creation:
Success

Directory Service Access:

Directory Service Changes:
Success

Policy Change:

Audit Policy Change:
Success and Failure

---

# 23. Registry Verification of Logging Controls

Process command-line auditing was verified using:

HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit

Value:

ProcessCreationIncludeCmdLine_Enabled = 0x1

PowerShell Script Block Logging:

HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging

Value:

EnableScriptBlockLogging = 0x1

PowerShell Module Logging:

HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging

Value:

EnableModuleLogging = 0x1

Module logging wildcard:

*

---

# 24. Wazuh Firewall Integration for ADNET

Existing OPNsense aliases were reused.

Domain Controller:

LAB_DC_01
10.10.50.10

Wazuh Server:

WAZUH_SERVER
10.10.40.10

Wazuh agent ports:

WAZUH_AGENT_PORTS

Ports:

1514
1515

A dedicated ADNET firewall rule was created:

ADNET - Allow LAB-DC-01 to Wazuh

Source:

LAB_DC_01

Destination:

WAZUH_SERVER

Destination ports:

WAZUH_AGENT_PORTS

Protocol:

TCP

This preserved network segmentation while allowing only the required SIEM communication.

Connectivity was verified from LAB-DC-01:

TCP 1514: True
TCP 1515: True

---

# 25. Wazuh Agent Deployment on LAB-DC-01

Wazuh Agent version:

4.14.7

Manager:

10.10.40.10

Agent name:

LAB-DC-01

Agent group:

default

The agent was installed and the service verified.

Service:

WazuhSvc

Status:

Running

The Wazuh dashboard confirmed:

LAB-DC-01
10.10.50.10
Active

---

# 26. Windows Security Event Collection

The Wazuh agent successfully collected the Domain Controller Security event channel.

Process creation telemetry was verified using Event ID:

4688

Wazuh successfully displayed:

A process was created.

The event contained:

Process executable
Parent process
Command line
Account
Domain
Computer
Security context

Command-line auditing was therefore verified end-to-end.

---

# 27. PowerShell Event Collection in Wazuh

Windows was generating PowerShell events locally, but the PowerShell Operational event channel was not included in the Wazuh agent configuration by default.

The following block was added to:

C:\Program Files (x86)\ossec-agent\ossec.conf

Configuration:

<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

The XML configuration was validated before restart.

WazuhSvc was restarted successfully.

PowerShell Event ID 4104 then appeared in Wazuh Threat Hunting.

Script Block Logging:

4104
Verified

Module Logging:

4103
Verified locally

Both use:

Microsoft-Windows-PowerShell/Operational

The event-channel collection path was therefore confirmed.

---

# 28. Kerberos Authentication Monitoring

Kerberos telemetry was validated using real domain activity.

## Event ID 4768

Description:

A Kerberos authentication ticket (TGT) was requested.

Verified locally on:

LAB-DC-01

## Event ID 4769

Description:

A Kerberos service ticket was requested.

A service ticket was generated by accessing:

\\LAB-DC-01\SYSVOL

Verified successfully.

## Event ID 4771

Description:

Kerberos pre-authentication failed.

A controlled incorrect password attempt was made using:

CYBERLAB\emmanuel.patrick

Wazuh detected the event.

Target user:

emmanuel.patrick

Source:

10.10.20.20

Status:

0x18

0x18 indicates Kerberos pre-authentication failure caused by an incorrect password.

This confirmed the ability to identify failed domain authentication attempts.

---

# 29. Active Directory User Lifecycle Monitoring

A temporary user was created:

telemetry.test

Display name:

Telemetry Test

This produced:

Event ID 4720

Description:

A user account was created.

Wazuh detected the event and correctly identified:

Target user:

telemetry.test

Account creator:

Administrator

The temporary user was later deleted.

This produced:

Event ID 4726

Description:

A user account was deleted.

Wazuh successfully detected the deletion.

The temporary account was removed after testing.

---

# 30. Security Group Membership Monitoring

A temporary Global Security Group was created:

GG_Telemetry_Test

Group type:

Security

Group scope:

Global

The normal account:

emmanuel.patrick

was temporarily added to the group.

This generated:

Event ID 4728

Description:

A member was added to a security-enabled global group.

Wazuh successfully detected the membership addition.

The account was then removed.

This generated:

Event ID 4729

Description:

A member was removed from a security-enabled global group.

Wazuh successfully detected the removal.

The temporary group was deleted after testing.

---

# 31. Privileged Logon Monitoring

Special Logon auditing generated:

Event ID 4672

Description:

Special privileges assigned to new logon.

Wazuh successfully detected the event for:

Domain:

CYBERLAB

Account:

Administrator

Computer:

LAB-DC-01.cyberlab.internal

The event included high-value privileges related to:

Backup
Restore
Ownership
Debugging
Impersonation
Security operations

This confirmed visibility into privileged logon sessions on the Domain Controller.

---

# 32. Audit Policy Change Monitoring

Audit policy changes generated:

Event ID 4719

Description:

System audit policy was changed.

Historical Event ID 4719 records were successfully confirmed on LAB-DC-01.

This provides visibility into attempts to weaken or modify Windows auditing.

---

# 33. Directory Service Change Monitoring

Directory Service Change auditing was enabled.

Event ID:

5136

Description:

A directory service object was modified.

Historical 5136 events were confirmed on LAB-DC-01.

Recent temporary user/group tests did not generate new 5136 events because Directory Service Change auditing also depends on appropriate SACL configuration on the target Active Directory objects.

A broad domain-wide SACL was intentionally not introduced because it would create unnecessary noise.

Targeted SACL monitoring can be introduced later during dedicated Active Directory detection engineering.

---

# 34. Sysmon Deployment

Microsoft Sysinternals Sysmon was deployed to LAB-DC-01.

Sysmon version:

15.22

Configuration schema:

4.91

Installation directory:

C:\Tools\Sysmon

Files included:

Sysmon.exe
Sysmon64.exe
Sysmon64a.exe
Eula.txt

A Domain Controller-specific Sysmon configuration was created:

C:\Tools\Sysmon\sysmon-dc.xml

Sysmon was installed using:

Sysmon64.exe

Service:

Sysmon64

Status:

Running

Driver:

SysmonDrv

Status:

Running

Sysmon Operational log:

Microsoft-Windows-Sysmon/Operational

---

# 35. Initial Sysmon Monitoring Scope

The initial configuration included monitoring for:

Process Creation
Network Connections
Driver Loading
CreateRemoteThread
LSASS Process Access
Script/File Creation
Registry Persistence
DNS Queries
Process Tampering

High-value persistence paths included:

CurrentVersion\Run
CurrentVersion\RunOnce
SYSTEM\CurrentControlSet\Services

Credential-access monitoring included process access to:

lsass.exe

Suspicious file types included:

.ps1
.psm1
.bat
.cmd
.vbs
.js
.hta

---

# 36. Sysmon Wazuh Integration

The following event channel was added to the Wazuh agent configuration:

<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

The configuration was validated before restarting Wazuh.

Wazuh successfully began receiving Sysmon events from:

LAB-DC-01

Sysmon Event ID 1:

Process Create

was successfully observed in Wazuh Threat Hunting.

Wazuh also began applying rules to Sysmon process activity.

---

# 37. Sysmon Noise Reduction

The first Sysmon configuration captured every process creation event.

This produced unnecessary duplication because Windows Security Event ID 4688 was already providing broad process creation visibility.

Instead of keeping redundant high-volume telemetry, Sysmon was tuned to focus on higher-value executables.

A backup was created before tuning:

C:\Tools\Sysmon\sysmon-dc-before-tuning.xml

The tuned ProcessCreate rule focuses on:

powershell.exe
pwsh.exe
cmd.exe
rundll32.exe
regsvr32.exe
mshta.exe
certutil.exe
bitsadmin.exe
wscript.exe
cscript.exe
schtasks.exe
nltest.exe
ntdsutil.exe
whoami.exe
net.exe
net1.exe

DNS telemetry was also restricted to higher-risk interpreters and utilities including:

powershell.exe
pwsh.exe
cmd.exe
certutil.exe
mshta.exe
wscript.exe
cscript.exe

The tuning reduced duplicate process noise while preserving high-value activity.

---

# 38. Sysmon Tuning Verification

A controlled validation was performed.

Normal application:

notepad.exe

Result:

No new Sysmon Event ID 1

High-value command interpreter:

cmd.exe

Test:

SYSMON-TUNING-TEST

Result:

Sysmon Event ID 1 generated successfully.

This proved that the tuned configuration successfully filtered ordinary application activity while retaining higher-value command-shell telemetry.

---

# 39. Final Wazuh Endpoint State

The Wazuh environment currently has three active monitored endpoints:

LAB-WIN-01
10.10.20.20
Windows
Active

LAB-UBUNTU-01
10.10.30.30
Linux
Active

LAB-DC-01
10.10.50.10
Windows Server / Domain Controller
Active

---

# 40. Key Verified Windows / Active Directory Events

The following telemetry was verified during this phase:

4103
PowerShell Module Logging

4104
PowerShell Script Block Logging

4672
Special privileges assigned to new logon

4688
A new process has been created

4719
System audit policy was changed

4720
User account created

4726
User account deleted

4728
Member added to security-enabled global group

4729
Member removed from security-enabled global group

4768
Kerberos authentication ticket requested

4769
Kerberos service ticket requested

4771
Kerberos pre-authentication failed

5136
Directory service object modified

Sysmon 1
Process creation

Sysmon 10
Process access

---

# 41. Security Design Decisions

Several deliberate security decisions were made during this phase.

## Separate Administrative Account

Routine user activity is performed using:

emmanuel.patrick

Administrative activity uses:

emmanuel.ngari-adm

This limits exposure of privileged credentials.

## Domain Controller Is Not Used as a General Admin Workstation

LAB-DC-01 is reserved for:

Active Directory
DNS
Group Policy
Domain Controller configuration
Security telemetry

General browsing and administration are performed from:

LAB-WIN-01

## No Broad CORPNET-to-ADNET Rule

Only required Active Directory ports are permitted.

## No Broad Domain-Wide Directory SACL

Directory Service auditing can generate significant noise.

Targeted SACL monitoring will be introduced only where security value justifies it.

## Sysmon Is Tuned Rather Than Left in Log-Everything Mode

Windows Event 4688 already provides broad process monitoring.

Sysmon is used to add higher-value telemetry rather than duplicate everything.

## Default Microsoft GPOs Were Not Used for Custom Hardening

Custom policies were created:

GPO-Workstation-Security-Baseline

GPO-DomainController-Security-Baseline

This improves troubleshooting, auditability, and future change control.

---

# 42. Snapshots / Recovery Points

Important snapshots created during this phase included:

Pre-ADDS Baseline

Pre-Domain-Admin-Delegation

Pre-AD-GPO-Baseline

AD-GPO-Workstation-Baseline-Configured

Pre-CORPNET-AD-Firewall-Rules

Pre-CYBERLAB-Domain-Join

Domain-Join-GPO-Verified

AD-Domain-Joined-GPO-Verified

AD-Workstation-Integration-Complete

AD-DC-Security-Baseline-Verified

Pre-ADNET-Wazuh-Rules

Pre-DC-PowerShell-Wazuh-Collection

Pre-DC-Sysmon-Deployment

DC-Sysmon-Wazuh-Verified-Pre-Tuning

AD-DC-Monitoring-Complete

These snapshots provide rollback points before major Active Directory, Group Policy, firewall, Wazuh, and Sysmon changes.

---

# 43. Lessons Learned

## Verify Before Changing

A missing event in Wazuh does not automatically mean the underlying Windows configuration is broken.

For example:

- 4103 existed locally even when it was not visible as an alert.
- 4768 existed locally even though normal successful Kerberos authentication did not generate a Wazuh Threat Hunting alert.
- 5136 requires more than simply enabling the audit policy.

The correct troubleshooting process is:

Generate event
→ Verify locally
→ Verify collection
→ Verify SIEM alerting

These are separate layers.

## Alerts Are Not the Same as Raw Events

Wazuh Threat Hunting mainly presents events that match rules and become alerts.

A valid Windows event may exist locally and may even be collected without necessarily appearing as a meaningful alert.

## Domain Controllers Require Focused Telemetry

Logging everything on a Domain Controller creates excessive noise.

Telemetry should prioritize:

Authentication
Privilege changes
Account lifecycle
Group membership
PowerShell
Credential access
Persistence
High-risk process activity
Audit-policy changes

## Network Segmentation Should Be Preserved

Security tooling should not be used as an excuse to create broad firewall rules.

Wazuh connectivity was provided using dedicated aliases and specific ports rather than opening ADNET to SOCNET.

---

# 44. Current Active Directory Phase Status

Active Directory Forest:
COMPLETE

Domain Controller:
COMPLETE

DNS:
COMPLETE

OU Structure:
COMPLETE

Domain Users:
COMPLETE

Security Groups:
COMPLETE

Privileged Account Separation:
COMPLETE

Workstation Domain Join:
COMPLETE

Workstation Rename:
COMPLETE

Kerberos Secure Channel:
COMPLETE

Workstation Security GPO:
COMPLETE

Domain Controller Security GPO:
COMPLETE

Advanced Audit Policy:
COMPLETE

PowerShell Logging:
COMPLETE

Wazuh Agent Deployment:
COMPLETE

AD Authentication Monitoring:
COMPLETE

Account Management Monitoring:
COMPLETE

Security Group Monitoring:
COMPLETE

Privileged Logon Monitoring:
COMPLETE

Audit Policy Monitoring:
COMPLETE

Sysmon Deployment:
COMPLETE

Sysmon Wazuh Integration:
COMPLETE

Sysmon Noise Tuning:
COMPLETE

Final Snapshot:
AD-DC-Monitoring-Complete

---

# 45. Next Phase

The next major infrastructure phase is the DMZ.

Planned DMZ network:

10.10.60.0/24

Planned gateway:

10.10.60.1

The DMZ phase will include:

- Dedicated OPNsense interface
- DMZ server
- Restricted inbound/outbound firewall policy
- Wazuh monitoring
- Suricata visibility
- Controlled exposed services
- REDNET attack simulation
- DMZ-to-internal segmentation testing
- Detection validation

After the DMZ is completed, the entire Purple Team CyberLab infrastructure will undergo final validation and documentation before transitioning into realistic SOC and Purple Team operational simulations.

---

# Final Result

The CyberLab now contains a functioning and monitored Active Directory environment with:

- Dedicated Windows Server 2025 Domain Controller
- Segmented AD network
- Domain DNS
- Kerberos authentication
- Domain-joined Windows workstation
- Role-based user/group structure
- Separate privileged administration
- Workstation and Domain Controller Group Policy baselines
- Advanced Windows auditing
- PowerShell telemetry
- Wazuh SIEM integration
- Active Directory security event monitoring
- Tuned Sysmon telemetry
- Verified end-to-end security visibility

This Active Directory environment can now support realistic attack simulations, threat hunting, incident response, detection engineering, and Purple Team exercises.
