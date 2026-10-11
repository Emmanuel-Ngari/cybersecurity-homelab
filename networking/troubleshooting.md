# Network Troubleshooting and Lessons Learned

## Overview

This document records significant networking and security troubleshooting encountered while building the six-zone Cybersecurity Home Lab.

The purpose is not to document every temporary error encountered during construction. Instead, it captures the issues that produced useful technical lessons about:

- OPNsense firewall behaviour
- VirtualBox networking
- Wazuh agent connectivity
- Suricata telemetry
- Active Directory communication
- DMZ access
- SSH administration
- Linux service behaviour
- validation methodology

The final environment was fully validated after these issues were resolved.

---

# Final Network Reference

| Zone | Subnet | Gateway | Primary Host |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `10.10.10.1` | `LAB-KALI-01` |
| CORPNET | `10.10.20.0/24` | `10.10.20.1` | `LAB-WIN-01` |
| SERVERNET | `10.10.30.0/24` | `10.10.30.1` | `LAB-UBUNTU-01` |
| SOCNET | `10.10.40.0/24` | `10.10.40.1` | `LAB-SIEM-01` |
| ADNET | `10.10.50.0/24` | `10.10.50.1` | `LAB-DC-01` |
| DMZ | `10.10.60.0/24` | `10.10.60.1` | `LAB-DMZ-01` |

All inter-zone traffic is routed through:

`LAB-FW-01`

---

# Troubleshooting Methodology

The troubleshooting process followed a consistent sequence:

1. Confirm the source host.
2. Confirm the destination host.
3. Confirm IP addressing.
4. Confirm local service state.
5. Confirm routing.
6. Confirm firewall policy.
7. Confirm destination port reachability.
8. Confirm application behaviour.
9. Confirm telemetry.
10. Change only one layer at a time.

This approach prevented application problems from being incorrectly treated as firewall problems and prevented firewall issues from being incorrectly treated as endpoint failures.

---

# OPNsense Rule Changes Not Taking Effect

## Symptom

A firewall rule had been created or reordered correctly in the OPNsense interface, but traffic continued matching an older deny rule.

Example:

REDNET traffic to:

`LAB-DMZ-01 — TCP/80`

continued timing out even after an HTTP allow rule was placed above the broad REDNET private-network block.

Firewall logs showed traffic still matching:

`BLOCK - REDNET to Unauthorized Private`

---

## Root Cause

The rule had been saved but the pending OPNsense configuration had not yet been applied.

The red:

`Apply`

button was still visible.

OPNsense had therefore not loaded the new packet-filter configuration.

---

## Resolution

After clicking:

`Apply Changes`

the rule became active.

A new request from Kali returned:

`HTTP/1.1 200 OK`

---

## Lesson Learned

In OPNsense:

**Save does not necessarily mean the active firewall ruleset has been reloaded.**

After changing firewall policy:

1. Save the rule.
2. Verify the rule order.
3. Click Apply.
4. Confirm the pending-change banner disappears.
5. Retest using fresh traffic.

This became a standard validation step for subsequent firewall changes.

---

# Firewall Rule Order

## Symptom

An intended allow rule existed but traffic was still blocked.

---

## Cause

OPNsense processes interface firewall rules according to rule order.

A broad block rule placed above a specific allow rule can prevent the allow rule from ever being reached.

---

## Example

Correct order:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

Incorrect order:

    BLOCK - REDNET to Unauthorized Private Networks
    ALLOW - REDNET to LAB-DMZ-01 HTTP

---

## Lesson Learned

Specific exceptions must appear before broader deny rules.

Every new inter-zone exception should therefore be checked for:

- source
- destination
- protocol
- port
- interface
- direction
- position
- logging
- active application of the changes

---

# VirtualBox DMZ Adapter Limitation

## Requirement

The final architecture required six internal security zones plus an upstream WAN interface.

The OPNsense VM therefore required more adapters than were conveniently exposed during the normal GUI workflow.

---

## Solution

The additional DMZ interface was configured using `VBoxManage`.

The resulting OPNsense VM used:

    NIC1  NAT
    NIC2  CORP-NET
    NIC3  RED-NET
    NIC4  SERVER-NET
    NIC5  SOC-NET
    NIC6  AD-NET
    NIC7  DMZ-NET
    NIC8  Disabled

The new adapter was detected by OPNsense as:

`em6`

and configured as the DMZ interface.

---

## Lesson Learned

VirtualBox GUI limitations do not necessarily represent hypervisor limitations.

`VBoxManage` provides additional control for advanced virtual network configurations.

---

# REDNET HTTP Initially Timing Out

## Symptom

Requests from:

`LAB-KALI-01 — 10.10.10.10`

to:

`LAB-DMZ-01 — 10.10.60.10:80`

timed out.

Nginx was confirmed active on the DMZ server.

---

## Investigation

The following were checked:

- Nginx service state
- listener on TCP/80
- DMZ host addressing
- OPNsense interface state
- firewall rule configuration
- OPNsense Live View logs

Firewall logs identified the actual problem.

Traffic was being blocked by:

`BLOCK - REDNET to Unauthorized Private`

---

## Resolution

The specific REDNET → DMZ HTTP allow rule was correctly positioned above the broader deny rule and the OPNsense configuration was applied.

---

## Final Result

    REDNET → DMZ HTTP/80

Status:

**Allowed**

---

# REDNET SSH Validation

The same validation process was used to ensure the offensive network did not gain administrative access to the DMZ.

Test:

    nc -zv -w 3 10.10.60.10 22

Result:

    Connection timed out

OPNsense logging confirmed the traffic was blocked by policy.

---

## Lesson Learned

A successful security test is not always a successful connection.

For deny-policy validation, a timeout or rejected connection can represent the expected successful result.

---

# CORPNET DMZ SSH Administration

## Requirement

Long commands entered directly through the Ubuntu Server VirtualBox console were inconvenient.

A secure remote-management path was therefore required.

---

## Initial Test

From `LAB-WIN-01`:

    ssh emmanuel@10.10.60.10

Result:

    Connection timed out

---

## Cause

CORPNET had no approved SSH path into the DMZ.

This was expected based on the segmentation policy.

---

## Design Decision

Rather than opening SSH from all internal networks, a narrow rule was created:

    Source:
    ADMIN_WORKSTATION
    10.10.20.20

    Destination:
    LAB_DMZ_01
    10.10.60.10

    Protocol:
    TCP

    Destination Port:
    22

The rule was placed above the broad CORPNET private-network deny policy.

---

## Final Result

SSH from `LAB-WIN-01` succeeded.

SSH from REDNET remained blocked.

SSH from ADNET remained blocked.

---

## Security Lesson

Administrative access should be explicitly scoped by:

- approved source
- approved destination
- approved protocol
- approved port

A broad:

`CORPNET → DMZ`

allow rule was unnecessary and would have weakened segmentation.

---

# Why the Domain Controller Was Not Used as a Jump Host

Technically, `LAB-DC-01` could have been allowed to SSH into the DMZ.

This design was rejected.

The Domain Controller is critical identity infrastructure and should not serve as a general-purpose administrative workstation.

Using it as a jump host would:

- increase its attack surface
- mix identity and administration functions
- create unnecessary trust paths
- reduce architecture realism

The approved administrative host remains:

`LAB-WIN-01`

A future dedicated privileged-access workstation could provide stronger administrative separation.

---

# SSH Key Authentication

## Objective

After confirming the restricted network path, host-level SSH security was strengthened.

---

## Initial Effective Configuration

    permitrootlogin prohibit-password
    pubkeyauthentication yes
    passwordauthentication yes
    kbdinteractiveauthentication no

Password authentication remained enabled.

---

## Investigation

Ubuntu configuration included:

    /etc/ssh/sshd_config.d/50-cloud-init.conf

with:

    PasswordAuthentication yes

---

## Resolution

A dedicated security override was created:

`/etc/ssh/sshd_config.d/00-dmz-hardening.conf`

Effective configuration became:

    permitrootlogin no
    pubkeyauthentication yes
    passwordauthentication no
    kbdinteractiveauthentication no

---

## Validation

Public-key authentication from `LAB-WIN-01` succeeded.

Password-only authentication was explicitly tested and returned:

    Permission denied (publickey).

---

## Lesson Learned

On modern Linux systems, inspecting only:

`/etc/ssh/sshd_config`

may not reveal the effective configuration.

Drop-in directories such as:

`/etc/ssh/sshd_config.d/`

must also be considered.

The command:

    sshd -T

is useful for validating the final effective SSH configuration.

---

# SSH Test Run from the Wrong Host

## Symptom

A key-only SSH test returned:

    Permission denied (publickey)

after key-based authentication had previously succeeded.

---

## Cause

The test command had accidentally been executed from inside:

`LAB-DMZ-01`

instead of:

`LAB-WIN-01`.

The DMZ server did not possess the Windows workstation's private SSH key.

---

## Resolution

The test was repeated from the intended management workstation.

Key authentication succeeded.

---

## Lesson Learned

Always verify the command prompt and source machine before interpreting network-test results.

In a multi-VM environment, the source host is part of the test.

---

# Wazuh Agent Appeared Disconnected

## Symptom

During final validation:

`LAB-UBUNTU-01`

appeared as:

`Disconnected`

in Wazuh.

---

## Initial Checks

On `LAB-UBUNTU-01`:

- IP address was correct
- `wazuh-agent` was active
- TCP/1514 to `LAB-SIEM-01` succeeded
- ICMP to the SIEM failed

---

## Important Observation

The failed ICMP test did not indicate that Wazuh connectivity was broken.

The required Wazuh TCP port remained reachable.

---

## Agent Log Review

The Wazuh agent log contained historical failures such as:

    Unable to connect to server
    Unable to connect to enrollment service

but the most recent entries showed:

    Connected to the server ([10.10.40.10]:1514/tcp)
    Server responded. Releasing lock.
    Agent is now online.

---

## Final Verification

From `LAB-SIEM-01`:

    LAB-UBUNTU-01 ... Active

---

## Lesson Learned

Do not troubleshoot from historical log errors alone.

Always correlate:

- timestamps
- current service state
- current port reachability
- current manager status

The most recent event state is what determines the current condition.

---

# ICMP Failure Did Not Equal Service Failure

Several validation tests showed:

- ping failed
- required TCP service succeeded

Example:

    LAB-UBUNTU-01 → LAB-SIEM-01 ICMP

failed while:

    TCP/1514

succeeded.

---

## Lesson Learned

Ping is not a universal connectivity test.

A firewall may intentionally block ICMP while allowing required application traffic.

Validation should target the actual service port.

Examples:

    nc -zv <host> 1514

or:

    Test-NetConnection <host> -Port 1514

provide more relevant results than relying only on ICMP.

---

# Suricata URI Search Returned Nothing

## Symptom

A benign HTTP request from REDNET to the DMZ succeeded, but searching:

`/var/log/suricata/eve.json`

for the URI returned no result.

---

## Cause

The Suricata telemetry pipeline being collected on the SIEM primarily contained security alert events rather than every benign HTTP transaction.

The absence of the URI therefore did not prove that Suricata lacked visibility.

---

## Resolution

A controlled response was created on the DMZ containing:

    uid=0(root) gid=0(root) groups=0(root)

Kali requested the resource.

Suricata generated:

    signature_id: 2100498
    GPL ATTACK_RESPONSE id check returned root

Traffic was observed between:

    10.10.60.10:80
        →
    10.10.10.10

---

## Lesson Learned

IDS validation should trigger a known detection rather than assuming every network transaction will produce an alert.

---

# Suricata Wazuh Search Initially Failed

## Symptom

The Suricata alert existed in:

`/var/log/suricata/eve.json`

but an initial search in:

`/var/ossec/logs/alerts/alerts.json`

did not find it using the expected raw fields.

---

## Investigation

Wazuh configuration correctly contained:

    <localfile>
      <log_format>json</log_format>
      <location>/var/log/suricata/eve.json</location>
    </localfile>

The Wazuh logcollector also reported:

    Analyzing file: '/var/log/suricata/eve.json'.

---

## Root Cause

The Wazuh Suricata alert had:

    "full_log": null

Searching only the `full_log` field therefore missed the event.

---

## Correct Validation

Filtering by Wazuh alert location showed:

    rule_id: 86601
    description:
    Suricata: Alert - GPL ATTACK_RESPONSE id check returned root

---

## Lesson Learned

Wazuh alert structure can differ between decoders.

Investigation should consider fields such as:

- `location`
- `rule.id`
- `rule.description`
- decoded fields
- agent information
- timestamps

rather than assuming all events retain identical `full_log` content.

---

# jq Parse Error on Active Wazuh Alert File

## Symptom

While querying:

`/var/ossec/logs/alerts/alerts.json`

`jq` returned:

    parse error: Unfinished string at EOF

while still returning valid previous events.

---

## Likely Cause

The Wazuh alert file was being actively written while `jq` attempted to parse it.

The newest JSON line may therefore have been incomplete at the exact moment it was read.

---

## Impact

No evidence indicated corruption of historical alerts.

Required Suricata and DMZ events remained retrievable.

---

## Lesson Learned

Active log files can be read while a writer is appending data.

For production-grade analysis, ingestion through the SIEM/indexer is preferable to repeatedly parsing a live JSON file directly.

---

# Suricata eve.json jq Parse Failure

## Symptom

Running `jq` against:

`/var/log/suricata/eve.json`

returned:

    parse error: Invalid numeric literal

---

## Investigation

Direct inspection of the file showed valid JSON alert records.

---

## Lesson Learned

When structured parsing fails:

1. Do not immediately modify the pipeline.
2. Inspect several raw lines first.
3. Confirm whether the issue is formatting, prefixes, partial writes, or malformed records.
4. Preserve working ingestion until the root cause is understood.

The Suricata → Wazuh pipeline was ultimately confirmed healthy.

---

# Nginx Test URI Confusion

## Symptom

A grep for:

`wazuh-test-2`

returned no result.

---

## Investigation

The most recent Nginx access log showed a different validation request:

    GET /wazuh-nginx-verify-01 HTTP/1.1

---

## Cause

The test URI had changed between validation steps.

Nginx itself was functioning normally.

---

## Lesson Learned

Use unique, consistent test identifiers and verify timestamps before interpreting an empty search result.

---

# Wazuh Nginx Log Format

Nginx access logs were configured using:

    <log_format>apache</log_format>

because the standard Nginx access format is compatible with Wazuh web-access decoding.

The error log was configured separately using:

    <log_format>syslog</log_format>

This allowed Nginx HTTP activity to be decoded by Wazuh.

A controlled request produced:

    Rule 31101
    Web server 400 error code.

---

# Wazuh Manager and Agent Version Compatibility

Before installing the DMZ Wazuh agent, the manager version was checked.

Manager:

`v4.14.7`

The DMZ agent was installed using:

`4.14.7-1`

This avoided introducing a newer agent version than the manager.

---

## Lesson Learned

Security-agent deployment should include version validation rather than automatically installing the newest available package.

---

# DMZ Wazuh Port Validation

Before installing the Wazuh agent, connectivity from `LAB-DMZ-01` was tested to:

    10.10.40.10:1514
    10.10.40.10:1515

Both connections succeeded.

This isolated potential firewall problems before application installation.

---

## Lesson Learned

Validate required network paths before installing software that depends on those paths.

This reduces troubleshooting complexity.

---

# Linux Unattended Upgrade Shutdown Inhibitor

## Symptom

`LAB-UBUNTU-01` refused a normal poweroff before the final snapshot.

The system reported:

    Operation inhibited by "APT"
    reason is "APT is installing or removing packages"

---

## Investigation

`systemctl status unattended-upgrades`

showed the unattended-upgrade shutdown service.

A process inspection showed that no active:

- `apt`
- `dpkg`
- package installation

process remained.

The remaining process was:

    unattended-upgrade-shutdown --wait-for-signal

---

## Resolution

The helper service was stopped normally:

    sudo systemctl stop unattended-upgrades.service

The VM was then powered off normally.

---

## Lesson Learned

Do not bypass package-manager shutdown inhibitors without first confirming whether package activity is actually occurring.

Avoid:

    systemctl poweroff -i

unless there is a justified reason to ignore inhibitors.

---

# Ubuntu Soft Lockup Messages

## Observation

`LAB-UBUNTU-01` displayed kernel messages such as:

    watchdog: BUG: soft lockup
    CPU stuck

during periods when several VirtualBox VMs were running simultaneously.

---

## Assessment

The VM remained responsive.

The condition was consistent with significant CPU scheduling contention on the physical host rather than an immediate indication of filesystem or application failure.

---

## Response

No unnecessary kernel modification was performed.

The issue was treated as a host-resource consideration unless it persisted under normal VM load.

---

## Lesson Learned

Virtualized lab environments can generate guest timing warnings when the physical host is heavily oversubscribed.

Resource contention should be considered before modifying a healthy guest operating system.

---

# Active Directory Validation

Domain Controller health checks were performed using targeted `dcdiag` tests.

Command:

    dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

Result:

No output.

Previous DNS testing also passed.

---

## Lesson Learned

With `dcdiag /q`, silence can indicate successful validation.

Not every successful diagnostic produces explicit success messages.

---

# Reverse DNS Warnings During nc Testing

`nc` occasionally displayed:

    inverse host lookup failed: Unknown host

during connectivity testing.

Example:

    10.10.60.10: inverse host lookup failed

The actual TCP test still completed correctly.

---

## Meaning

Netcat attempted reverse DNS resolution for the target IP and did not obtain a hostname.

This did not affect the port test itself.

---

## Lesson Learned

Separate name-resolution warnings from transport-layer results.

A reverse-DNS warning does not automatically indicate a network failure.

---

# HTTP 403 During Internet Test

## Test

From the DMZ:

    curl -I https://packages.wazuh.com

Result:

    HTTP/2 403

---

## Interpretation

Although the application returned `403 Forbidden`, the test still proved:

- DNS resolution worked
- routing worked
- TCP/443 worked
- TLS worked
- the remote web server received the request

---

## Lesson Learned

Application response codes must be interpreted in context.

A `403` can still prove successful network connectivity.

---

# Snapshot Cleanup

Snapshots were created before major security and infrastructure changes throughout the build.

By project completion, many temporary checkpoints existed.

Examples included:

- pre-domain-join
- pre-GPO
- pre-Sysmon
- pre-DMZ
- pre-web exposure
- pre-Wazuh monitoring
- pre-SSH hardening

After final six-zone validation, obsolete snapshots were deleted and VirtualBox merged their differencing disks.

Every VM now retains only:

`Final-Six-Zone-Validated-Baseline`

---

## Lesson Learned

Snapshots are valuable rollback controls during change management but should not become permanent clutter.

A clean final baseline simplifies:

- future exercises
- storage management
- recovery
- documentation
- troubleshooting

---

# Final Troubleshooting Principles

The build reinforced several general security-engineering lessons.

## Verify Before Changing

Collect evidence before modifying configuration.

## Test the Actual Service

Use service-specific port tests rather than relying only on ping.

## Check the Source Host

The origin of a connection matters in a segmented environment.

## Read Firewall Logs

Firewall logs often identify the exact rule responsible for a connection result.

## Confirm Rule Order

A correctly defined rule can still fail if positioned incorrectly.

## Apply Pending Firewall Changes

Saved configuration may not yet be active.

## Use Effective Configuration

Tools such as:

    sshd -T

show the configuration actually being enforced.

## Correlate Timestamps

Historical errors should not be treated as current failures.

## Validate End to End

A detection pipeline is not considered working merely because one component generated a log.

Validation should prove the full chain.

Example:

    Attack
      →
    Network
      →
    IDS
      →
    Log Pipeline
      →
    SIEM
      →
    Detection

## Preserve Least Privilege

Troubleshooting should not be solved by creating broad allow rules.

The preferred solution is the narrowest rule that satisfies the legitimate requirement.

---

# Final Status

All significant networking issues encountered during the infrastructure build were resolved.

The final environment successfully validated:

- inter-zone routing
- least-privilege firewall policy
- REDNET isolation
- Active Directory communication
- Wazuh endpoint connectivity
- Suricata IDS visibility
- Suricata-to-Wazuh integration
- Nginx monitoring
- restricted DMZ administration
- SSH key authentication
- password SSH rejection
- DMZ isolation
- SOC telemetry
- Domain Controller health

The network is now treated as a stable platform for future SOC and Purple Team exercises rather than an infrastructure build in progress.
