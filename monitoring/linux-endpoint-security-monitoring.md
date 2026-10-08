# Linux Endpoint Security Monitoring — LAB-UBUNTU-01

## 1. Overview

This document records the implementation, validation, tuning, and final operational state of Linux security monitoring in the Purple Team CyberLab.

The monitored Linux endpoint is:

- Hostname: LAB-UBUNTU-01
- Operating System: Ubuntu 26.04.1 LTS
- Network Zone: SERVERNET
- IP Address: 10.10.30.30/24
- Default Gateway: 10.10.30.1
- Wazuh Agent ID: 002
- Wazuh Manager: LAB-SIEM-01
- SIEM IP Address: 10.10.40.10
- Wazuh Agent Version: 4.14.7-1

The objective of this phase was to create production-style Linux endpoint visibility covering:

- Authentication activity
- SSH activity
- Privileged command execution
- Linux account creation
- Linux account modification
- Linux account deletion
- Audit subsystem monitoring
- File Integrity Monitoring
- Critical system configuration changes
- Package installation
- Service start/stop activity
- Wazuh detection engineering
- MITRE ATT&CK mapping
- Dashboard validation

The environment was intentionally built using native Linux telemetry and Wazuh capabilities rather than installing unnecessary additional monitoring platforms.

---

# 2. Architecture

Security telemetry follows this path:

LAB-UBUNTU-01
    |
    | Wazuh Agent
    |
    | TCP/1514
    v
LAB-SIEM-01
    |
    +-- Wazuh Manager
    +-- Wazuh Indexer
    +-- Wazuh Dashboard
    +-- Filebeat

Relevant network zones:

SERVERNET
    10.10.30.0/24

SOCNET
    10.10.40.0/24

LAB-UBUNTU-01
    10.10.30.30

LAB-SIEM-01
    10.10.40.10

Firewall segmentation is handled by LAB-FW-01 running OPNsense.

ICMP connectivity between SERVERNET and SOCNET was not used as a requirement for determining Wazuh health.

The Wazuh agent was confirmed connected and active despite ICMP being restricted.

---

# 3. Initial Linux Endpoint Baseline

The endpoint was verified before implementing additional security monitoring.

Hostname:

    lab-ubuntu-01

Operating system:

    Ubuntu 26.04.1 LTS

Primary SERVERNET interface:

    enp0s8
    10.10.30.30/24

Default route:

    default via 10.10.30.1

Storage:

    /dev/mapper/ubuntu--vg-ubuntu--lv
    Size: 19G
    Used: 5.3G
    Available: 13G
    Utilization: 30%

System health:

    No failed systemd services

SSH:

    Active
    Listening on TCP/22

Authentication log:

    /var/log/auth.log

Wazuh agent:

    Active

Agent connection on LAB-SIEM-01:

    ID: 002
    Name: LAB-UBUNTU-01
    Status: Active

A VirtualBox snapshot was taken before implementing Linux security monitoring:

    Pre-Linux-Security-Monitoring

---

# 4. Existing Wazuh Agent Configuration

The Linux agent was already connected to the SIEM using:

Manager:

    10.10.40.10

Port:

    1514

Protocol:

    TCP

Agent profile:

    ubuntu
    ubuntu26
    ubuntu26.04

Existing telemetry collection included:

- journald
- /var/log/dpkg.log
- Wazuh active response log
- disk utilization command monitoring
- network command monitoring
- login history collection

The existing File Integrity Monitoring configuration covered:

- /etc
- /usr/bin
- /usr/sbin
- /bin
- /sbin
- /boot

The original FIM scan frequency was:

    43200 seconds

Equivalent to:

    12 hours

---

# 5. Authentication and Sudo Monitoring

Wazuh was already collecting Linux journald events.

This allowed authentication and sudo activity to be detected without separately ingesting /var/log/auth.log and creating duplicate telemetry.

A sudo test was performed using:

    sudo -k
    sudo whoami

Wazuh successfully generated:

Rule:

    5402

Description:

    Successful sudo to ROOT executed.

MITRE ATT&CK:

    T1548.003
    Sudo and Sudo Caching

The event contained:

- Source user
- Destination user
- Terminal
- Current directory
- Executed command

Example:

    USER=root
    COMMAND=/usr/bin/whoami

This confirmed privileged command visibility through journald.

---

# 6. SSH Monitoring

SSH monitoring was validated using existing Wazuh Linux rules.

A connection attempt was generated from REDNET:

Source:

    10.10.10.10

Destination:

    10.10.30.30:22

A non-existent account was used:

    fakeuser

Wazuh detected the invalid login attempt.

Rule:

    5710

Level:

    5

Description:

    sshd: Attempt to login using a non-existent user

MITRE ATT&CK:

    T1110.001
    Password Guessing

    T1021.004
    SSH

The event correctly captured:

    Source IP: 10.10.10.10
    Username: fakeuser

PAM authentication failure was also detected.

Rule:

    5503

Level:

    5

Description:

    PAM: User login failed.

MITRE ATT&CK:

    T1110.001

Successful SSH authentication was also validated.

Rule:

    5715

Description:

    sshd: authentication success.

Example source:

    10.10.20.20

Destination user:

    ubuntu

MITRE ATT&CK:

    T1078
    Valid Accounts

    T1021
    Remote Services

PAM session creation was also visible through:

    Rule 5501
    PAM: Login session opened.

PAM session termination was visible through:

    Rule 5502
    PAM: Login session closed.

No custom SSH rule was created because the built-in Wazuh detections provided sufficient coverage.

---

# 7. Auditd Installation

Auditd was installed on LAB-UBUNTU-01:

    sudo apt install auditd audispd-plugins

Service validation:

    systemctl is-active auditd

Result:

    active

Audit subsystem status:

    enabled 1
    failure 1
    rate_limit 0
    backlog_limit 8192
    lost 0

Audit log:

    /var/log/audit/audit.log

---

# 8. Wazuh Auditd Integration

The Wazuh agent configuration was backed up before modification:

    /var/ossec/etc/ossec.conf.pre-auditd.bak

Auditd ingestion was added using:

    <ossec_config>
      <localfile>
        <log_format>audit</log_format>
        <location>/var/log/audit/audit.log</location>
      </localfile>
    </ossec_config>

The configuration was validated using:

    sudo /var/ossec/bin/wazuh-logcollector -t

The Wazuh agent was restarted and confirmed active.

Wazuh logs confirmed:

    Analyzing file: '/var/log/audit/audit.log'.

A temporary Auditd test rule was created to validate the complete telemetry path.

The test confirmed:

Linux Kernel
    ->
Auditd
    ->
/var/log/audit/audit.log
    ->
Wazuh Agent
    ->
LAB-SIEM-01

Wazuh rule 80705 was generated during Auditd rule changes:

    Auditd: Configuration changed.

The temporary test rule was removed after successful validation.

---

# 9. Production Auditd Security Policy

The initial policy used legacy Auditd watch syntax.

Example:

    -w /etc/passwd -p wa -k identity_changes

Auditd generated warnings that old-style watch rules were slower.

The policy was therefore modernized using syscall/path-based rules.

Final policy file:

    /etc/audit/rules.d/50-cyberlab-security.rules

Final contents:

    ## Purple Team CyberLab - Linux Security Audit Policy
    ## Modern syscall-based file and directory monitoring

    # Identity and authentication databases
    -a always,exit -F arch=b64 -F path=/etc/passwd -F perm=wa -k identity_changes
    -a always,exit -F arch=b64 -F path=/etc/group -F perm=wa -k identity_changes
    -a always,exit -F arch=b64 -F path=/etc/shadow -F perm=wa -k identity_changes
    -a always,exit -F arch=b64 -F path=/etc/gshadow -F perm=wa -k identity_changes

    # Privilege configuration
    -a always,exit -F arch=b64 -F path=/etc/sudoers -F perm=wa -k privilege_changes
    -a always,exit -F arch=b64 -F dir=/etc/sudoers.d/ -F perm=wa -k privilege_changes

    # SSH configuration
    -a always,exit -F arch=b64 -F path=/etc/ssh/sshd_config -F perm=wa -k ssh_config_changes
    -a always,exit -F arch=b64 -F dir=/etc/ssh/sshd_config.d/ -F perm=wa -k ssh_config_changes

    # Audit subsystem configuration
    -a always,exit -F arch=b64 -F dir=/etc/audit/ -F perm=wa -k audit_config_changes

Rules were loaded using:

    sudo augenrules --check
    sudo augenrules --load

Kernel-loaded rules were confirmed using:

    sudo auditctl -l

Final Auditd health:

    enabled 1
    lost 0

Active keys:

    identity_changes
    privilege_changes
    ssh_config_changes
    audit_config_changes

No legacy -w rules remain in the final CyberLab policy.

Snapshot taken before modernization:

    Pre-Auditd-Policy-Modernization

---

# 10. Linux Account Creation Detection

Initial testing showed that monitoring modifications to:

- /etc/passwd
- /etc/group
- /etc/shadow
- /etc/gshadow

generated multiple alerts for one user creation operation.

This was technically correct but too noisy for a SOC environment.

Native Auditd account events were investigated instead.

Auditd generated:

    type=ADD_USER

for successful user creation.

A custom Wazuh detection was therefore created using the semantic ADD_USER event.

Custom rule:

    100200

Level:

    8

Description:

    Linux user account created successfully via Auditd.

MITRE ATT&CK:

    T1136
    Create Account

Final rule:

    <rule id="100200" level="8">
      <if_sid>80700</if_sid>
      <field name="audit.type">^ADD_USER$</field>
      <field name="audit.res">^success$</field>
      <field name="audit.exe">useradd</field>
      <description>Linux user account created successfully via Auditd.</description>
      <mitre>
        <id>T1136</id>
      </mitre>
    </rule>

A live account creation test generated one clean alert.

Example account:

    purpletest4

Agent:

    LAB-UBUNTU-01

Decoder:

    auditd

Result:

    success

The rule was considered production-ready for the CyberLab environment.

Snapshot:

    Linux-Audit-Account-Creation-Working

---

# 11. Linux Account Deletion Detection

Linux account deletion was investigated using Auditd.

Native event:

    type=DEL_USER

Example executable:

    /usr/sbin/userdel

Successful deletion:

    res=success

Custom Wazuh rule:

    100201

Level:

    8

Description:

    Linux user account deleted successfully via Auditd.

MITRE ATT&CK:

    T1531
    Account Access Removal

Final rule:

    <rule id="100201" level="8">
      <if_sid>80700</if_sid>
      <field name="audit.type">^DEL_USER$</field>
      <field name="audit.res">^success$</field>
      <field name="audit.exe">userdel</field>
      <description>Linux user account deleted successfully via Auditd.</description>
      <mitre>
        <id>T1531</id>
      </mitre>
    </rule>

Live validation successfully detected deletion of:

    purpletest3

Example Wazuh data:

    audit.type = DEL_USER
    audit.res = success
    audit.exe = "/usr/sbin/userdel"

Agent:

    LAB-UBUNTU-01

The detection was confirmed live.

---

# 12. Linux Account Modification Detection

Account modification was generated using:

    sudo usermod -c "Purple Team Test Account" purpletest2

Auditd generated:

    type=USER_MGMT

Operation:

    changing-comment

Executable:

    /usr/sbin/usermod

Result:

    success

Wazuh decoded:

    audit.type = USER_MGMT
    audit.exe = "/usr/sbin/usermod"
    audit.res = success

Custom rule:

    100202

Level:

    8

Description:

    Linux user account modified successfully via Auditd.

MITRE ATT&CK:

    T1098
    Account Manipulation

Final rule:

    <rule id="100202" level="8">
      <if_sid>80700</if_sid>
      <field name="audit.type">^USER_MGMT$</field>
      <field name="audit.res">^success$</field>
      <field name="audit.exe">usermod</field>
      <description>Linux user account modified successfully via Auditd.</description>
      <mitre>
        <id>T1098</id>
      </mitre>
    </rule>

Live testing successfully produced:

    Rule ID: 100202
    Level: 8
    Agent: LAB-UBUNTU-01
    MITRE: T1098

---

# 13. Final Linux Account Detection Rules

Wazuh rule file:

    /var/ossec/etc/rules/linux_audit_rules.xml

Final contents:

    <!--
      Purple Team CyberLab - Linux Auditd Detection Rules
      Rule range: 100200+
    -->

    <group name="linux,audit,account_management,cyberlab,">

      <rule id="100200" level="8">
        <if_sid>80700</if_sid>
        <field name="audit.type">^ADD_USER$</field>
        <field name="audit.res">^success$</field>
        <field name="audit.exe">useradd</field>
        <description>Linux user account created successfully via Auditd.</description>
        <mitre>
          <id>T1136</id>
        </mitre>
      </rule>

      <rule id="100201" level="8">
        <if_sid>80700</if_sid>
        <field name="audit.type">^DEL_USER$</field>
        <field name="audit.res">^success$</field>
        <field name="audit.exe">userdel</field>
        <description>Linux user account deleted successfully via Auditd.</description>
        <mitre>
          <id>T1531</id>
        </mitre>
      </rule>

      <rule id="100202" level="8">
        <if_sid>80700</if_sid>
        <field name="audit.type">^USER_MGMT$</field>
        <field name="audit.res">^success$</field>
        <field name="audit.exe">usermod</field>
        <description>Linux user account modified successfully via Auditd.</description>
        <mitre>
          <id>T1098</id>
        </mitre>
      </rule>

    </group>

Snapshot taken before implementing these rules:

    Pre-Linux-Audit-Detection-Rules

---

# 14. File Integrity Monitoring

The original Wazuh configuration monitored /etc using scheduled scans.

Wazuh logs showed:

    File integrity monitoring scan frequency: 43200 seconds

This provided baseline integrity monitoring but was insufficient for rapid detection of sensitive configuration changes.

A targeted real-time FIM configuration was therefore added for security-sensitive directories.

Before changing the configuration, a backup was created:

    /var/ossec/etc/ossec.conf.pre-realtime-fim

Snapshot:

    Pre-Realtime-Linux-FIM

The following configuration was added inside the existing <syscheck> section:

    <directories realtime="yes" whodata="yes">/etc/ssh,/etc/sudoers.d</directories>

The existing scheduled /etc monitoring was retained.

This provides:

- Broad periodic monitoring of /etc
- Immediate monitoring of sensitive SSH configuration
- Immediate monitoring of sudo configuration

The configuration was validated using:

    sudo /var/ossec/bin/wazuh-syscheckd -t

The Wazuh agent was restarted and remained active.

---

# 15. Real-Time FIM Validation

A harmless test file was created:

    /etc/ssh/cyberlab-fim-test.txt

The file was modified with:

    Purple Team FIM test

Wazuh immediately generated a real-time integrity event.

Rule:

    550

Level:

    7

Description:

    Integrity checksum changed.

MITRE ATT&CK:

    T1565.001
    Stored Data Manipulation

Wazuh captured:

- File path
- Monitoring mode
- Previous size
- New size
- Previous modification time
- New modification time
- Previous MD5
- New MD5
- Previous SHA1
- New SHA1
- Previous SHA256
- New SHA256
- Owner
- Group
- Permissions
- Inode

Monitoring mode:

    realtime

File creation was also detected.

Rule:

    554

Description:

    File added to the system.

Deletion was detected after cleanup.

Rule:

    553

Level:

    7

Description:

    File deleted.

MITRE ATT&CK:

    T1070.004
    File Deletion

    T1485
    Data Destruction

The test file was removed after validation:

    sudo rm -f /etc/ssh/cyberlab-fim-test.txt

Real-time FIM was therefore successfully validated for:

    Create
    Modify
    Delete

---

# 16. Package Monitoring

LAB-UBUNTU-01 already collected:

    /var/log/dpkg.log

A harmless package installation was used to validate package telemetry:

    sudo apt install -y tree

Wazuh generated:

Rule:

    2901

Description:

    New dpkg (Debian Package) requested to install.

Package:

    tree

Additional installation events included:

Rule:

    2904

Level:

    7

Description:

    Dpkg (Debian Package) half configured.

Final installation detection:

Rule:

    2902

Level:

    7

Description:

    New dpkg (Debian Package) installed.

Package:

    tree

Architecture:

    amd64

This confirmed that software installation activity is visible to the SIEM.

---

# 17. Linux Service Monitoring

Systemd service monitoring was added to identify suspicious stopping of important security services.

Initial testing used:

    rsyslog.service

Native journal events included:

    Stopping rsyslog.service - System Logging Service...
    Stopped rsyslog.service - System Logging Service.
    Starting rsyslog.service - System Logging Service...
    Started rsyslog.service - System Logging Service.

Wazuh decoded these events using:

    decoder = systemd

Built-in parent rule:

    40700

Description:

    Systemd rules

---

# 18. Initial Systemd Detection Attempt

Initial custom service rules matched every service start and stop.

Rules:

    100203
    100204

This worked technically but resulted in excessive dashboard noise because normal background systemd activity triggered many alerts.

The detection strategy was therefore tuned.

This was an important detection-engineering lesson:

A detection is not considered good simply because it triggers.

A production-quality detection should also be:

- Relevant
- Actionable
- Low-noise
- Understandable by an analyst
- Appropriate for the environment

---

# 19. Tuned Critical Service Monitoring

Monitoring was restricted to important security-related Linux services:

    ssh
    auditd
    rsyslog
    wazuh-agent

Final Wazuh rule file:

    /var/ossec/etc/rules/linux_systemd_rules.xml

Final configuration:

    <!-- Purple Team CyberLab - Linux Critical Service Detection -->

    <group name="linux,systemd,service_management,cyberlab,">

      <rule id="100203" level="8">
        <if_sid>40700</if_sid>
        <regex type="pcre2">^Stopped (?:ssh|auditd|rsyslog|wazuh-agent)\.service</regex>
        <description>Critical Linux security service stopped.</description>
        <mitre>
          <id>T1489</id>
        </mitre>
      </rule>

      <rule id="100204" level="4">
        <if_sid>40700</if_sid>
        <regex type="pcre2">^Started (?:ssh|auditd|rsyslog|wazuh-agent)\.service</regex>
        <description>Critical Linux security service started.</description>
      </rule>

    </group>

The use of PCRE2 was required for grouped alternation in the service list.

Configuration validation:

    sudo /var/ossec/bin/wazuh-analysisd -t

Wazuh Manager restart:

    sudo systemctl restart wazuh-manager

Manager status:

    active

Snapshot taken before this detection engineering work:

    Pre-Linux-Systemd-Detection-Rules

---

# 20. Critical Service Detection Validation

A controlled restart was performed:

    sudo systemctl restart rsyslog

The following custom alerts were generated.

Stopped service:

    Rule ID: 100203
    Level: 8
    Description: Critical Linux security service stopped.
    MITRE: T1489
    Technique: Service Stop
    Agent: LAB-UBUNTU-01
    Service: rsyslog.service

Started service:

    Rule ID: 100204
    Level: 4
    Description: Critical Linux security service started.
    Agent: LAB-UBUNTU-01
    Service: rsyslog.service

This confirmed clean detection of critical Linux service lifecycle events without alerting on every systemd service.

---

# 21. Custom Linux Detection Summary

Custom Wazuh detection range:

    100200+

Final custom detections:

    100200
    Linux user account created successfully via Auditd.
    Level 8
    MITRE T1136

    100201
    Linux user account deleted successfully via Auditd.
    Level 8
    MITRE T1531

    100202
    Linux user account modified successfully via Auditd.
    Level 8
    MITRE T1098

    100203
    Critical Linux security service stopped.
    Level 8
    MITRE T1489

    100204
    Critical Linux security service started.
    Level 4

These rules complement rather than replace the existing Wazuh Linux ruleset.

---

# 22. Built-In Wazuh Rules Validated

The following Wazuh built-in detections were successfully observed during testing:

    5402
    Successful sudo to ROOT executed.

    5501
    PAM: Login session opened.

    5502
    PAM: Login session closed.

    5503
    PAM: User login failed.

    5710
    sshd: Attempt to login using a non-existent user.

    5715
    sshd: authentication success.

    550
    Integrity checksum changed.

    553
    File deleted.

    554
    File added to the system.

    2901
    New dpkg package requested to install.

    2902
    New dpkg package installed.

    2904
    Dpkg package half configured.

    80705
    Auditd: Configuration changed.

These rules provide broad Linux endpoint visibility without unnecessary custom rule duplication.

---

# 23. Test Account Cleanup

Several temporary Linux accounts were used during detection engineering.

Examples included:

    purpletest
    purpletest2
    purpletest3
    purpletest4

All temporary accounts were removed after testing.

LAB-UBUNTU-01 verification:

    Ubuntu test accounts: CLEAN

An accidental test account was also checked on LAB-SIEM-01.

Verification:

    SIEM accidental test account: CLEAN

No Purple Team test accounts remain.

---

# 24. Wazuh Dashboard Validation

The Linux endpoint detections were verified in the Wazuh Dashboard.

Custom rule search included:

    agent.id:"002" AND (rule.id:100200 OR rule.id:100201 OR rule.id:100202 OR rule.id:100203 OR rule.id:100204)

This confirmed visibility of:

    Account creation
    Account deletion
    Account modification
    Service stop
    Service start

Additional built-in telemetry was verified for:

    SSH authentication
    SSH failures
    PAM activity
    Sudo activity
    File integrity monitoring
    Package installation

During dashboard review, the original broad systemd service rules were identified as too noisy.

The rules were subsequently tuned to security-critical services only.

Historical alerts from the broad rules remain in the index but do not represent the final detection logic.

---

# 25. Security Content Assessment Note

The Wazuh agent attempted to load:

    cis_ubuntu22-04.yml

The endpoint runs:

    Ubuntu 26.04.1 LTS

The Ubuntu 22.04 benchmark was therefore skipped.

The older benchmark was intentionally NOT forced onto Ubuntu 26.04.

Reason:

Using an unsupported CIS benchmark can generate misleading compliance results and create false confidence.

A properly supported Ubuntu 26.04 security baseline should be introduced when compatible policy content is available or when a custom benchmark is intentionally designed and validated.

This remains a future hardening item and does not affect the endpoint telemetry completed in this phase.

---

# 26. Final Endpoint Health Validation

LAB-UBUNTU-01 final service state:

    wazuh-agent: active
    auditd: active
    ssh: active
    rsyslog: active

Audit subsystem:

    enabled 1
    lost 0

Storage:

    Filesystem: /dev/mapper/ubuntu--vg-ubuntu--lv
    Size: 19G
    Used: 5.3G
    Available: 13G
    Utilization: 30%

The endpoint was healthy at completion.

---

# 27. Final SIEM Health Validation

LAB-SIEM-01 final services:

    wazuh-manager: active
    wazuh-indexer: active
    wazuh-dashboard: active
    filebeat: active

Connected agents:

    ID 000
    lab-siem-01
    Active/Local

    ID 001
    LAB-WIN-01
    Active

    ID 002
    LAB-UBUNTU-01
    Active

This confirmed successful endpoint-to-SIEM communication at completion.

---

# 28. Snapshot History

Important snapshots associated with this phase include:

LAB-UBUNTU-01:

    Pre-Linux-Security-Monitoring
    Linux-Audit-Account-Creation-Working
    Pre-Realtime-Linux-FIM
    Pre-Auditd-Policy-Modernization
    Linux-Security-Monitoring-Complete

LAB-SIEM-01:

    Pre-Linux-Audit-Detection-Rules
    Pre-Linux-Systemd-Detection-Rules
    Linux-Detection-Rules-Complete

The final snapshots represent the stable recovery point for the completed Linux monitoring phase.

---

# 29. Final Monitoring Coverage

The Linux endpoint now provides visibility across the following security domains:

Authentication
    SSH successful authentication
    SSH invalid users
    PAM failures
    PAM sessions

Privilege escalation
    sudo execution
    command visibility

Account management
    Account creation
    Account modification
    Account deletion

File integrity
    Sensitive SSH configuration
    Sudo configuration
    File creation
    File modification
    File deletion
    Cryptographic hash changes

Audit subsystem
    Identity database changes
    Privilege configuration changes
    SSH configuration changes
    Audit configuration changes

Software management
    Package installation
    dpkg state changes

Service monitoring
    Critical security service stopping
    Critical security service starting

SIEM
    Wazuh decoding
    Custom detection rules
    MITRE ATT&CK mapping
    Dashboard visibility

---

# 30. Detection Engineering Lessons

Several important Purple Team principles were demonstrated during this phase.

## Native semantic events are preferable to noisy indirect indicators

Initial account creation monitoring relied on file changes to:

    /etc/passwd
    /etc/group
    /etc/shadow
    /etc/gshadow

One user creation therefore generated several events.

Using Auditd's native:

    ADD_USER
    DEL_USER
    USER_MGMT

events produced much cleaner detections.

## Existing rules should be reused when they already provide good coverage

Custom rules were not created for:

    SSH authentication
    PAM
    sudo
    dpkg
    standard FIM

because Wazuh already had useful built-in detections.

## Detection quality matters more than detection quantity

The initial systemd rule detected every service stop.

This created excessive noise.

The rule was therefore restricted to:

    ssh
    auditd
    rsyslog
    wazuh-agent

This produced more useful SOC telemetry.

## Network security controls should not be weakened for monitoring tests

ICMP between SERVERNET and SOCNET was not enabled simply to make ping succeed.

Wazuh agent connectivity was used to establish actual monitoring health.

## Benchmarks should match the operating system

The Ubuntu 22.04 CIS policy was not forced onto Ubuntu 26.04.

Unsupported compliance content should not be treated as authoritative.

---

# 31. Current Completion Status

Linux endpoint security monitoring is complete.

Completed capabilities:

    [COMPLETE] Wazuh agent connectivity
    [COMPLETE] Journald monitoring
    [COMPLETE] SSH authentication monitoring
    [COMPLETE] PAM monitoring
    [COMPLETE] sudo monitoring
    [COMPLETE] Auditd installation
    [COMPLETE] Auditd ingestion into Wazuh
    [COMPLETE] Account creation detection
    [COMPLETE] Account deletion detection
    [COMPLETE] Account modification detection
    [COMPLETE] Modern Auditd security policy
    [COMPLETE] Scheduled FIM
    [COMPLETE] Targeted real-time FIM
    [COMPLETE] FIM create/modify/delete validation
    [COMPLETE] Package installation monitoring
    [COMPLETE] Critical systemd service monitoring
    [COMPLETE] Detection tuning
    [COMPLETE] Dashboard verification
    [COMPLETE] Test account cleanup
    [COMPLETE] Final service health validation
    [COMPLETE] Final VirtualBox snapshots

---

# 32. Final Architecture State

The completed Purple Team CyberLab currently includes security telemetry from:

Windows Endpoint
    LAB-WIN-01

Linux Endpoint
    LAB-UBUNTU-01

Firewall
    LAB-FW-01 / OPNsense

Network IDS
    Suricata

SIEM
    LAB-SIEM-01 / Wazuh

Validated security telemetry currently includes:

Windows
    Sysmon
    PowerShell
    Windows auditing
    Defender
    Windows Firewall
    File Integrity Monitoring
    Custom detection engineering

Network
    OPNsense firewall logs
    Suricata EVE JSON
    IDS alerts
    Firewall correlation

Linux
    Auditd
    SSH
    PAM
    sudo
    Linux account management
    File integrity
    Package changes
    Critical system services

The next infrastructure phase is:

    AD-NET
    10.10.50.0/24

This phase will introduce:

    Windows Server
    Active Directory Domain Services
    DNS
    Domain users and groups
    Organizational Units
    Group Policy
    Domain-joined workstations
    Domain Controller telemetry
    Authentication monitoring
    Kerberos monitoring
    Privileged account monitoring
    Active Directory attack-and-detection validation

After AD-NET and the DMZ are completed, the environment will transition from infrastructure building into realistic SOC and Purple Team operational simulations.
