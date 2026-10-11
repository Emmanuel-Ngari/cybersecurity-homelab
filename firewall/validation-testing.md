# Firewall Validation and Testing

## Overview

This document records the firewall and segmentation validation performed throughout the Cybersecurity Home Lab, from the earliest flat-network phase through the final six-zone OPNsense architecture.

The purpose of validation was to confirm that firewall policy behaved as intended in practice, not merely that rules appeared correct in the OPNsense interface.

Testing covered:

- basic host connectivity
- allowed service access
- blocked inter-zone access
- firewall rule ordering
- pending OPNsense configuration changes
- Active Directory communication
- Wazuh connectivity
- Suricata visibility
- DMZ exposure
- restricted SSH administration
- default-deny behaviour
- final six-zone segmentation

The final environment was considered validated only after expected allowed paths succeeded and expected denied paths failed.

---

# Historical Testing Approach

## Original Flat Lab

The first version of the lab used:

`192.168.56.0/24`

Primary hosts included:

- Kali — `192.168.56.10`
- Windows — `192.168.56.20`
- Ubuntu — `192.168.56.30`

Testing at this stage focused primarily on host-to-host reachability and service behaviour.

Examples included:

- ICMP reachability
- SSH access
- Nmap scanning
- Hydra authentication testing
- fail2ban response
- service exposure
- port-state changes

Because the network was largely flat, there was limited firewall segmentation to validate.

---

# Transition to OPNsense

When OPNsense was introduced, testing changed significantly.

The key question was no longer simply:

**Can Host A reach Host B?**

The new questions became:

- Should Host A be able to reach Host B?
- Which service should be allowed?
- Which traffic should be blocked?
- Which firewall rule processed the packet?
- Was the firewall change actually applied?
- Was the traffic visible to the SIEM?
- Was the result consistent with least privilege?

This marked the beginning of policy-driven validation.

---

# Initial Four-Zone Validation

The first OPNsense architecture introduced:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

Testing was performed from each source zone according to its intended role.

---

# CORPNET Early Validation

Source:

`LAB-WIN-01`

Network:

`10.10.20.0/24`

The corporate workstation was used to validate approved enterprise and administrative connectivity.

Tests included:

- OPNsense management access
- DNS
- Internet access
- selected SERVERNET administration
- isolation from REDNET

---

## CORPNET → OPNsense Management

Expected:

`ALLOW`

Purpose:

Permit the approved workstation to administer the firewall.

Validation method:

Web access to the OPNsense management interface.

Result:

**PASS**

---

## CORPNET → DNS

Expected:

`ALLOW`

Purpose:

Provide name resolution.

Result:

**PASS**

---

## CORPNET → SERVERNET SSH

Destination:

`LAB-UBUNTU-01`

Port:

`TCP/22`

Expected:

`ALLOW`

Purpose:

Provide approved administration of the internal Linux server during the early segmented phase.

Result:

**PASS**

---

## CORPNET → REDNET

Expected:

`BLOCK`

Purpose:

Prevent unnecessary corporate-to-attacker communication.

Result:

**PASS**

---

# REDNET Early Validation

Source:

`LAB-KALI-01`

Network:

`10.10.10.0/24`

REDNET represented the offensive network.

The objective was to preserve controlled testing paths while preventing broad internal access.

---

## REDNET → DNS

Expected:

`ALLOW`

Result:

**PASS**

---

## REDNET → Internet

Expected:

`ALLOW`

Purpose:

Permit updates and security-tool access.

Result:

**PASS**

---

## REDNET → SERVERNET

Selected attack paths were allowed during controlled exercises.

Examples included:

- Nmap
- SSH enumeration
- Hydra
- fail2ban validation

Result:

**PASS**

---

## REDNET → CORPNET

Expected:

`BLOCK`

Purpose:

Prevent Kali from freely accessing the Windows workstation.

Result:

**PASS**

---

## REDNET → OPNsense Management

Expected:

`BLOCK`

Purpose:

Prevent the attacker network from administering the firewall.

Result:

**PASS**

---

# SERVERNET Early Validation

Source:

`LAB-UBUNTU-01`

Network:

`10.10.30.0/24`

The server was tested for required infrastructure access and restricted east-west communication.

Validated behaviour included:

- DNS allowed
- NTP allowed
- Internet access allowed where required
- unsolicited CORPNET access restricted
- unsolicited REDNET access restricted

These tests confirmed the move away from the original flat network.

---

# SOCNET Validation

SOCNET was introduced when Wazuh became the central SIEM.

Primary host:

`LAB-SIEM-01 — 10.10.40.10`

Required Wazuh paths included:

- TCP/1514
- TCP/1515 where enrollment was required

Endpoints were tested before and after Wazuh deployment to distinguish:

- network failures
- agent failures
- enrollment failures
- manager-side failures

---

# Wazuh Port Testing

A common validation pattern became:

    nc -zv -w 3 10.10.40.10 1514

or on Windows:

    Test-NetConnection 10.10.40.10 -Port 1514

This was more useful than relying only on ping.

---

# ICMP vs Service Connectivity

One important lesson occurred when:

`LAB-UBUNTU-01`

could not ping:

`LAB-SIEM-01`

but could connect successfully to:

`TCP/1514`

The correct interpretation was:

- ICMP blocked
- Wazuh service path healthy

This reinforced the rule:

**Validate the actual required service, not only ICMP.**

---

# OPNsense Firewall Log Validation

OPNsense Live View became a core validation tool.

It was used to confirm:

- traffic reached the firewall
- the expected interface processed the traffic
- the expected rule matched
- the firewall action was allow or block
- rule order was correct

This was especially important when service state and routing appeared healthy but connectivity still failed.

---

# Firewall Rule Order Validation

A repeated pattern throughout the build was:

    Specific Allow
    Broad Deny

Example:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

Testing confirmed that placing the broad deny above the exception prevented the specific allow from functioning.

This demonstrated that rule order is part of firewall policy, not merely presentation.

---

# OPNsense Apply Changes Validation

One of the most important troubleshooting events occurred during REDNET-to-DMZ HTTP exposure.

The rule was:

- defined correctly
- placed correctly
- saved correctly

but traffic was still blocked.

OPNsense Live View showed the old deny behaviour.

The cause was that the pending ruleset had not been applied.

The red:

`Apply`

button remained visible.

After applying the configuration, the connection succeeded immediately.

From that point forward, firewall validation included:

1. Save.
2. Confirm rule order.
3. Apply changes.
4. Confirm the pending-change banner disappears.
5. Generate fresh traffic.
6. Review Live View.
7. Confirm expected result.

---

# Firewall Logging to Wazuh Validation

OPNsense firewall telemetry was later forwarded into Wazuh.

This allowed deny events to be investigated centrally.

A firewall correlation rule was successfully validated:

`Wazuh Rule 87702`

This confirmed that firewall policy could be observed from the SOC environment rather than only through the OPNsense GUI.

---

# Suricata Validation

Suricata was deployed on OPNsense after basic firewall segmentation was operational.

Primary monitored interface:

`em2`

This corresponded to REDNET traffic.

Initial Suricata validation confirmed the IDS could generate alerts from controlled traffic.

---

# Suricata Signature Test

A known test condition produced:

`GPL ATTACK_RESPONSE id check returned root`

Signature ID:

`2100498`

This became a useful controlled validation signature.

---

# Suricata Telemetry to Wazuh

Suricata EVE JSON events were forwarded to:

`LAB-SIEM-01`

using:

`UDP/5514`

and written to:

`/var/log/suricata/eve.json`

Wazuh monitored that file using:

`json`

log format.

The Suricata event later generated:

`Wazuh Rule 86601`

This confirmed the end-to-end IDS pipeline.

---

# ADNET Validation

ADNET was introduced later in the project.

Network:

`10.10.50.0/24`

Primary host:

`LAB-DC-01 — 10.10.50.10`

The objective was to permit legitimate domain functionality while preserving identity isolation.

---

# Active Directory Connectivity Testing

The corporate workstation was used to validate required domain services.

Final sampled tests included:

## DNS

    Test-NetConnection 10.10.50.10 -Port 53

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

## Kerberos

    Test-NetConnection 10.10.50.10 -Port 88

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

## LDAP

    Test-NetConnection 10.10.50.10 -Port 389

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

# Broader Active Directory Rule Set

Although the final spot checks directly tested DNS, Kerberos, and LDAP, the firewall supported the wider service set required for normal Active Directory operation.

Important TCP services included:

- `53`
- `88`
- `135`
- `389`
- `445`
- `464`
- `3268`
- `49152:65535`

Important UDP services included:

- `53`
- `88`
- `123`
- `389`
- `464`

The successful domain join, Group Policy operation, authentication, SYSVOL, and `dcdiag` health checks provided additional validation beyond individual port testing.

---

# Domain Controller Health Validation

Targeted validation used:

    dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

Result:

No output.

With `dcdiag /q`, no output indicated no detected failures for the requested tests.

Additional DNS validation was also completed successfully.

Status:

**PASS**

---

# REDNET → ADNET Validation

Test:

    nc -zv -w 3 10.10.50.10 389

Expected:

`BLOCK`

Result:

Connection timed out.

Status:

**PASS**

This confirmed that the offensive network could not directly access LDAP on the Domain Controller.

---

# ADNET → SOCNET Validation

Test:

    Test-NetConnection 10.10.40.10 -Port 1514

Expected:

`ALLOW`

Result:

    TcpTestSucceeded : True

Status:

**PASS**

This confirmed Domain Controller telemetry could reach Wazuh.

---

# DMZ Introduction

The DMZ was the final major security zone added to the firewall.

Network:

`10.10.60.0/24`

Primary host:

`LAB-DMZ-01 — 10.10.60.10`

Primary service:

Nginx on TCP/80.

The DMZ was validated in multiple stages:

1. base network
2. outbound connectivity
3. internal isolation
4. Wazuh connectivity
5. REDNET web exposure
6. CORPNET SSH administration
7. Suricata visibility
8. final six-zone validation

---

# DMZ Base Networking Validation

The DMZ host was configured with:

    IP: 10.10.60.10
    Gateway: 10.10.60.1
    DNS: 10.10.60.1

Validation confirmed:

- interface up
- correct route
- DNS operation
- Internet access
- internal blocking

---

# DMZ → Wazuh Pre-Installation Validation

Before installing the Wazuh agent, the required ports were tested.

Tests:

    nc -zv -w 3 10.10.40.10 1514
    nc -zv -w 3 10.10.40.10 1515

Results:

Both succeeded.

This confirmed that firewall policy was correct before introducing agent software.

---

# DMZ → CORPNET Block Validation

Test:

    nc -zv -w 3 10.10.20.20 445

Expected:

`BLOCK`

Result:

Timed out.

Status:

**PASS**

---

# DMZ → ADNET Block Validation

Test:

    nc -zv -w 3 10.10.50.10 389

Expected:

`BLOCK`

Result:

Timed out.

Status:

**PASS**

---

# DMZ → Internet HTTPS Validation

Test:

    curl -I --connect-timeout 5 https://packages.wazuh.com

Result:

`HTTP/2 403`

Interpretation:

The remote application denied the request, but the network test still proved:

- DNS resolution
- routing
- TCP/443
- TLS
- remote server reachability

Status:

**PASS**

---

# DMZ Default-Deny Validation

An outbound connection using an unapproved TCP port was blocked by:

`Default deny / state violation rule`

This confirmed that the DMZ did not have unrestricted outbound Internet access.

---

# REDNET → DMZ HTTP Validation

Source:

`LAB-KALI-01 — 10.10.10.10`

Destination:

`LAB-DMZ-01 — 10.10.60.10`

Service:

`TCP/80`

Test:

    curl -I http://10.10.60.10

Expected:

`ALLOW`

Final result:

`HTTP/1.1 200 OK`

Status:

**PASS**

---

# REDNET → DMZ SSH Validation

Test:

    nc -zv -w 3 10.10.60.10 22

Expected:

`BLOCK`

Result:

Connection timed out.

OPNsense logging confirmed the traffic was blocked by the REDNET private-network deny policy.

Status:

**PASS**

---

# REDNET → CORPNET SMB Validation

Test:

    nc -zv -w 3 10.10.20.20 445

Expected:

`BLOCK`

Result:

Connection timed out.

Status:

**PASS**

---

# CORPNET → DMZ SSH Validation

Initial test from:

`LAB-WIN-01`

to:

`LAB-DMZ-01:22`

timed out.

This confirmed there was no pre-existing management path.

A narrow rule was then introduced:

`ADMIN_WORKSTATION → LAB_DMZ_01 TCP/22`

After applying the rule, SSH succeeded.

Status:

**PASS**

---

# SSH Network Restriction Validation

Final source restrictions were tested.

## CORPNET Admin Workstation

    LAB-WIN-01 → LAB-DMZ-01:22

Result:

`ALLOW`

## REDNET

    LAB-KALI-01 → LAB-DMZ-01:22

Result:

`BLOCK`

## ADNET

    LAB-DC-01 → LAB-DMZ-01:22

Result:

`BLOCK`

This confirmed that SSH administration was limited to the intended source.

---

# SSH Authentication Validation

After network access was restricted, host-level authentication was hardened.

Effective SSH policy:

    permitrootlogin no
    pubkeyauthentication yes
    passwordauthentication no
    kbdinteractiveauthentication no

Public-key authentication from `LAB-WIN-01` succeeded.

Password-only authentication returned:

    Permission denied (publickey).

This provided defense in depth:

- firewall source restriction
- key-only SSH
- no root SSH
- no password SSH

---

# Nginx Monitoring Validation

A controlled request was generated from REDNET:

    GET /wazuh-nginx-verify-01 HTTP/1.1

Source:

`10.10.10.10`

Nginx returned:

`404`

The request appeared in:

`/var/log/nginx/access.log`

Wazuh generated:

- Rule `31101`
- Description: `Web server 400 error code.`
- Agent: `LAB-DMZ-01`
- Source IP: `10.10.10.10`
- Decoder: `web-accesslog`

Status:

**PASS**

---

# REDNET → DMZ Suricata Visibility Test

A controlled HTTP resource was temporarily created on the DMZ containing:

    uid=0(root) gid=0(root) groups=0(root)

Kali requested the resource.

The request successfully traversed:

    REDNET
       |
       v
    OPNsense
       |
       v
    DMZ

Suricata generated:

- Event type: `alert`
- Interface: `em2`
- Signature ID: `2100498`
- Signature: `GPL ATTACK_RESPONSE id check returned root`
- Source: `10.10.60.10:80`
- Destination: `10.10.10.10`
- Direction: `to_client`

Status:

**PASS**

---

# Suricata → Wazuh Validation

The same controlled event was verified on the SIEM.

Wazuh generated:

- Rule ID: `86601`
- Description: `Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

Status:

**PASS**

This confirmed the full chain:

    Kali
      |
      v
    OPNsense Firewall
      |
      v
    Suricata
      |
      v
    DMZ
      |
      v
    EVE JSON
      |
      v
    Wazuh
      |
      v
    SOC Alert

---

# SERVERNET Final Validation

Source:

`LAB-UBUNTU-01 — 10.10.30.30`

## SERVERNET → SOCNET Wazuh

    nc -zv -w 3 10.10.40.10 1514

Result:

Connection succeeded.

Status:

**PASS**

---

## SERVERNET → CORPNET SMB

    nc -zv -w 3 10.10.20.20 445

Result:

Timed out.

Status:

**PASS**

---

## SERVERNET → DMZ SSH

    nc -zv -w 3 10.10.60.10 22

Result:

Timed out.

Status:

**PASS**

---

# CORPNET Final Validation

Source:

`LAB-WIN-01 — 10.10.20.20`

Validated:

    CORPNET → ADNET DNS/53        PASS
    CORPNET → ADNET Kerberos/88   PASS
    CORPNET → ADNET LDAP/389      PASS
    CORPNET → SOCNET Wazuh/1514   PASS
    LAB-WIN-01 → DMZ SSH/22       PASS

---

# ADNET Final Validation

Source:

`LAB-DC-01 — 10.10.50.10`

Validated:

    ADNET → SOCNET Wazuh/1514   PASS
    ADNET → DMZ SSH/22          BLOCKED AS DESIGNED

Domain Controller health:

`PASS`

---

# DMZ Final Validation

Source:

`LAB-DMZ-01 — 10.10.60.10`

Validated:

    DMZ → SOCNET Wazuh/1514   PASS
    DMZ → CORPNET SMB/445     BLOCKED
    DMZ → ADNET LDAP/389      BLOCKED
    DMZ → Internet HTTPS      PASS

---

# Wazuh Agent Final Validation

Final agent state:

    ID: 001, Name: LAB-WIN-01, IP: any, Active
    ID: 002, Name: LAB-UBUNTU-01, IP: any, Active
    ID: 003, Name: LAB-DC-01, IP: any, Active
    ID: 004, Name: LAB-DMZ-01, IP: any, Active

Status:

**PASS**

---

# Wazuh Stack Validation

The following services were confirmed active:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Status:

**PASS**

---

# SIEM Storage Validation

Observed final state:

    Filesystem: /dev/mapper/ubuntu--vg-ubuntu--lv
    Size: 53G
    Used: 25G
    Available: 26G
    Usage: 49%

Status:

**PASS**

---

# Final Six-Zone Validation Matrix

| Source | Destination | Service | Expected | Result |
|---|---|---|---|---|
| REDNET | DMZ | HTTP/80 | Allow | PASS |
| REDNET | DMZ | SSH/22 | Block | PASS |
| REDNET | CORPNET | SMB/445 | Block | PASS |
| REDNET | ADNET | LDAP/389 | Block | PASS |
| CORPNET | ADNET | DNS/53 | Allow | PASS |
| CORPNET | ADNET | Kerberos/88 | Allow | PASS |
| CORPNET | ADNET | LDAP/389 | Allow | PASS |
| CORPNET | SOCNET | Wazuh/1514 | Allow | PASS |
| LAB-WIN-01 | DMZ | SSH/22 | Allow | PASS |
| SERVERNET | SOCNET | Wazuh/1514 | Allow | PASS |
| SERVERNET | CORPNET | SMB/445 | Block | PASS |
| SERVERNET | DMZ | SSH/22 | Block | PASS |
| ADNET | SOCNET | Wazuh/1514 | Allow | PASS |
| ADNET | DMZ | SSH/22 | Block | PASS |
| DMZ | SOCNET | Wazuh/1514 | Allow | PASS |
| DMZ | CORPNET | SMB/445 | Block | PASS |
| DMZ | ADNET | LDAP/389 | Block | PASS |
| DMZ | Internet | HTTPS/443 | Allow | PASS |

---

# Validation Philosophy

The project adopted several validation principles.

## Test Both Allowed and Blocked Traffic

A firewall is not considered validated by testing only permitted connections.

Denied paths must also be tested.

---

## Use the Real Service

Where possible, test the actual application port rather than relying only on ping.

---

## Review Firewall Logs

A timeout alone does not prove which control blocked the traffic.

OPNsense logs were used to identify the responsible rule.

---

## Confirm Source Host

The same destination can behave differently depending on the source zone.

---

## Generate Fresh Traffic

After changing firewall policy, new connections were generated instead of relying on existing states.

---

## Validate End to End

For monitored services, success meant more than network reachability.

Example:

    Attack
      |
      v
    Firewall
      |
      v
    IDS
      |
      v
    Application
      |
      v
    Log
      |
      v
    SIEM
      |
      v
    Detection

---

# Snapshot and Validation Workflow

Significant firewall changes followed the workflow:

    Snapshot
        |
        v
    Configure
        |
        v
    Save
        |
        v
    Apply
        |
        v
    Test
        |
        v
    Review Logs
        |
        v
    Verify Detection
        |
        v
    Document

This helped preserve known-good states throughout the build.

---

# Historical vs Current Validation

This document intentionally records validation from the beginning of the project through the final architecture.

Earlier sections therefore include tests that relate to:

- the original flat lab
- the initial four-zone design
- temporary attacker-to-server paths
- early workstation administration
- pre-Active Directory networking

These were valid at their respective stages.

The current authoritative validation is the final six-zone matrix.

---

# Final Status

Firewall and segmentation validation is complete.

The final environment demonstrated:

- correct inter-zone routing
- functioning least-privilege policy
- REDNET containment
- protected CORPNET
- protected SERVERNET
- centralized SOC connectivity
- functioning Active Directory paths
- ADNET isolation
- DMZ isolation
- restricted administrative SSH
- controlled Internet access
- firewall telemetry
- Suricata visibility
- Suricata-to-Wazuh detection
- Nginx-to-Wazuh monitoring
- healthy Wazuh agents
- healthy SIEM services
- validated Domain Controller health

The firewall is now considered a stable control plane for future SOC and Purple Team scenarios.
