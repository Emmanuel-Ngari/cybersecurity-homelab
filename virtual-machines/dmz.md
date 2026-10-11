# LAB-DMZ-01 — Demilitarized Zone Web Server

## Overview

This document records the complete lifecycle of the DMZ system in the Cybersecurity Home Lab, from the period before any DMZ existed through the final validated `LAB-DMZ-01` deployment.

The DMZ was the final major infrastructure expansion of the lab.

The project did not begin with a DMZ.

The earliest environment used a simple flat network based around:

`192.168.56.0/24`

and later evolved into a segmented OPNsense architecture with:

- REDNET
- CORPNET
- SERVERNET
- SOCNET
- ADNET

Only after the core firewall, SIEM, endpoint monitoring, Suricata, and Active Directory infrastructure were already operational was a dedicated DMZ introduced.

The purpose of the DMZ was to create a realistic externally reachable server segment that could be:

- intentionally exposed to selected attacker traffic
- isolated from protected internal networks
- centrally monitored
- administered through a restricted management path
- inspected by Suricata
- used for application-layer detection

Final host:

`LAB-DMZ-01`

Final operating system:

`Ubuntu Server 26.04.1 LTS`

Final IP:

`10.10.60.10`

Final zone:

`DMZ`

Primary service:

`Nginx`

Primary role:

Monitored public-facing Linux web server

---

# Phase 1 — Before the DMZ

The original Cybersecurity Home Lab contained no dedicated public-facing network.

Early systems included:

- Kali Linux
- Windows
- Ubuntu Server

on:

`192.168.56.0/24`

Later, OPNsense introduced:

- REDNET
- CORPNET
- SERVERNET
- SOCNET

and Active Directory later added:

- ADNET

At this stage, the lab already had:

- OPNsense firewalling
- Wazuh
- Suricata
- Windows monitoring
- Linux monitoring
- Active Directory
- Sysmon
- firewall telemetry

The missing architectural component was a realistic externally exposed server zone.

---

# Why the DMZ Was Added

The DMZ was introduced to create a controlled environment for scenarios such as:

- web attacks
- reconnaissance
- IDS detection
- application logging
- network segmentation
- attacker-to-web-server traffic
- web-server monitoring
- restricted administrative access
- compromised-DMZ lateral-movement testing

The design goal was not simply to add another Ubuntu machine.

The DMZ needed to represent a distinctly lower-trust server zone.

---

# DMZ Network Creation

A sixth internal security zone was created:

`DMZ`

Network:

`10.10.60.0/24`

Gateway:

`10.10.60.1`

Primary server:

`LAB-DMZ-01 — 10.10.60.10`

This expanded the final architecture to six zones.

---

# OPNsense DMZ Interface

The firewall originally did not have a seventh network adapter.

To support the new DMZ, an additional VirtualBox NIC was added to:

`LAB-FW-01`

while the firewall VM was powered off.

The adapter was configured as:

    Adapter: NIC 7
    Mode: Internal Network
    Internal Network: DMZ-NET
    Adapter Type: Intel PRO/1000 MT Desktop (82540EM)
    Cable Connected: Yes

The VirtualBox configuration was created using `VBoxManage`.

OPNsense detected the new interface as:

`em6`

The interface was assigned as:

`DMZ`

---

# OPNsense DMZ Gateway

The new interface was configured with:

    Interface: DMZ
    Device: em6
    IPv4 Configuration: Static IPv4
    Address: 10.10.60.1/24
    IPv6: None
    Upstream Gateway: None

This address became the default gateway for DMZ systems.

---

# LAB-DMZ-01 Virtual Machine Creation

A new Ubuntu Server VM was created specifically for the DMZ.

Final resources:

    Operating System: Ubuntu Server 26.04.1 LTS
    Memory: 2 GB
    CPUs: 2
    Storage: 25 GB dynamically allocated VDI

The server was attached only to:

`DMZ-NET`

for its internal lab connectivity.

This ensured that DMZ communication remained under OPNsense control.

---

# Storage Layout

During Ubuntu installation, LVM was used.

The root logical volume was manually expanded to approximately:

`20G`

This provided sufficient working capacity while remaining lightweight for the lab environment.

---

# Final DMZ Network Configuration

The server was configured with a static address.

Final settings:

    Hostname: lab-dmz-01
    IP Address: 10.10.60.10
    Prefix: /24
    Gateway: 10.10.60.1
    DNS: 10.10.60.1

Observed interface:

`enp0s3`

This design ensures that routing and DNS are controlled through OPNsense.

---

# Initial Connectivity Validation

After installation, the following were verified:

- interface state
- static IP
- default route
- DNS configuration
- OPNsense gateway reachability
- Internet connectivity

The DMZ server successfully used:

`10.10.60.1`

as its network gateway.

---

# DMZ Firewall Design

The DMZ was deliberately configured with restrictive policy.

The final DMZ firewall sequence became:

    1. ALLOW - DMZ to OPNsense DNS
    2. ALLOW - DMZ to OPNsense NTP
    3. ALLOW - DMZ to Wazuh Agent Services
    4. BLOCK - DMZ to Unauthorized Private Networks
    5. ALLOW - DMZ HTTP HTTPS Outbound

This ordering is significant.

The Wazuh exception must exist above the broad private-network block because SOCNET is part of the protected internal address space.

---

# DMZ Firewall Aliases

Several aliases were created or reused for the DMZ.

## `LAB_DMZ_01`

Address:

`10.10.60.10`

Purpose:

Represent the DMZ server.

---

## `WAZUH_SERVER`

Address:

`10.10.40.10`

Purpose:

Represent `LAB-SIEM-01`.

---

## `WAZUH_AGENT_PORTS`

Ports:

- `1514`
- `1515`

Purpose:

Wazuh communication and enrollment.

---

## `HTTP_HTTPS_PORTS`

Ports:

- `80`
- `443`

Purpose:

Approved outbound web communication.

---

## `DNS_PORT`

Port:

`53`

---

## `NTP_PORT`

Port:

`123`

---

## `PRIVATE_NETS`

Purpose:

Represent protected internal networks for broad DMZ isolation.

---

## `ADMIN_WORKSTATION`

Address:

`10.10.20.20`

Represents:

`LAB-WIN-01`

Purpose:

Approved administrative source for DMZ SSH.

---

# DMZ to DNS

Rule:

`ALLOW - DMZ to OPNsense DNS`

Purpose:

Permit DNS resolution through OPNsense.

Protocol:

`TCP/UDP`

Port:

`53`

Status:

**VALIDATED**

---

# DMZ to NTP

Rule:

`ALLOW - DMZ to OPNsense NTP`

Purpose:

Permit time synchronization.

Protocol:

`UDP`

Port:

`123`

Status:

**VALIDATED**

---

# DMZ to Wazuh

Rule:

`ALLOW - DMZ to Wazuh Agent Services`

Destination:

`10.10.40.10`

Ports:

- `1514`
- `1515`

Protocol:

`TCP`

Purpose:

Allow:

- Wazuh agent enrollment
- ongoing telemetry

---

# Pre-Installation Wazuh Connectivity Testing

Before installing the Wazuh agent, connectivity was tested directly.

Tests included:

    nc -zv -w 3 10.10.40.10 1514
    nc -zv -w 3 10.10.40.10 1515

Both succeeded.

This confirmed the network and firewall path before introducing Wazuh software.

Status:

**PASS**

---

# DMZ Private-Network Block

Rule:

`BLOCK - DMZ to Unauthorized Private Networks`

Source:

`DMZ network`

Destination:

`PRIVATE_NETS`

Action:

`Block`

Logging:

`Enabled`

Purpose:

Prevent a compromised DMZ server from freely accessing protected networks.

---

# DMZ to CORPNET Isolation

Test:

    nc -zv -w 3 10.10.20.20 445

Expected:

`BLOCK`

Result:

Connection timed out.

Status:

**PASS**

This confirmed the DMZ server could not initiate SMB access to the corporate workstation.

---

# DMZ to ADNET Isolation

Test:

    nc -zv -w 3 10.10.50.10 389

Expected:

`BLOCK`

Result:

Connection timed out.

Status:

**PASS**

This confirmed the DMZ server could not directly access LDAP on the Domain Controller.

---

# DMZ HTTP and HTTPS Outbound

Rule:

`ALLOW - DMZ HTTP HTTPS Outbound`

Protocol:

`TCP`

Ports:

- `80`
- `443`

Destination:

`Any`

Purpose:

Permit:

- package downloads
- software installation
- updates
- Wazuh repository access
- approved web connectivity

---

# HTTPS Validation

A test was performed against:

`https://packages.wazuh.com`

The response returned:

`HTTP/2 403`

This was correctly interpreted as a successful connectivity test because it proved:

- DNS resolution worked
- routing worked
- TCP/443 worked
- TLS negotiation worked
- the remote application responded

The HTTP 403 represented application authorization behaviour, not network failure.

Status:

**PASS**

---

# Default-Deny Validation

A DMZ outbound TCP attempt using an unapproved destination port was observed in OPNsense.

It was blocked by:

`Default deny / state violation rule`

This confirmed that the DMZ did not have unrestricted outbound access.

---

# Nginx Installation

Nginx was installed to provide the primary DMZ web service.

Observed version:

`nginx/1.28.3 (Ubuntu)`

The service was enabled and confirmed active.

Nginx listens on:

`TCP/80`

This became the intended offensive testing service.

---

# Initial Nginx Validation

Local and remote HTTP testing confirmed that Nginx responded successfully.

The server later returned:

`HTTP 200`

to approved requests from REDNET.

---

# REDNET to DMZ Web Exposure

A specific firewall rule was created:

`ALLOW - REDNET to LAB-DMZ-01 HTTP`

Source:

`REDNET network`

Destination:

`LAB_DMZ_01`

Protocol:

`TCP`

Port:

`80`

Purpose:

Permit controlled attacker traffic from Kali to the DMZ web server.

---

# Rule Ordering Lesson

The REDNET HTTP exception needed to appear above:

`BLOCK - REDNET to Unauthorized Private Networks`

Correct order:

    ALLOW - REDNET to LAB-DMZ-01 HTTP
    BLOCK - REDNET to Unauthorized Private Networks

This was important because OPNsense processes the matching firewall rules in order.

---

# Apply Changes Troubleshooting

During implementation, the REDNET HTTP rule was:

- created correctly
- saved
- positioned above the broad block

but traffic still timed out.

OPNsense Live View showed requests still matching:

`BLOCK - REDNET to Unauthorized Private`

The issue was that the new ruleset had not yet been applied.

The red:

`Apply`

button remained visible in OPNsense.

After applying the configuration:

`curl`

from Kali succeeded immediately.

This became one of the most important OPNsense lessons in the project:

**Saving a rule does not necessarily make it active. Always apply pending changes and verify the banner disappears.**

---

# REDNET to DMZ HTTP Validation

Source:

`LAB-KALI-01 — 10.10.10.10`

Destination:

`LAB-DMZ-01 — 10.10.60.10`

Port:

`TCP/80`

Result:

`HTTP 200`

Status:

**PASS**

---

# REDNET to DMZ SSH Validation

Kali was not granted administrative access.

Test:

    nc -zv -w 3 10.10.60.10 22

Result:

Connection timed out.

OPNsense showed the request blocked by the REDNET private-network policy.

Status:

**PASS**

This confirmed that REDNET could attack the exposed web service without gaining SSH access.

---

# Administrative Access Requirement

Typing long commands directly in the Ubuntu VirtualBox console became inefficient.

A legitimate management path was therefore required.

The design objective was:

- allow administration
- preserve segmentation
- avoid broad CORPNET-to-DMZ access
- avoid using the Domain Controller as a jump host

---

# Initial CORPNET SSH Test

From:

`LAB-WIN-01`

to:

`LAB-DMZ-01:22`

the initial connection timed out.

This was expected.

No firewall exception yet existed.

Status:

**BLOCKED AS DESIGNED**

---

# Restricted Administrative Firewall Rule

A specific rule was created:

`ALLOW - ADMIN_WORKSTATION to LAB-DMZ-01 SSH`

Source:

`ADMIN_WORKSTATION`

Address:

`10.10.20.20`

Destination:

`LAB_DMZ_01`

Address:

`10.10.60.10`

Protocol:

`TCP`

Port:

`22`

Logging:

`Enabled`

The rule was positioned above:

`BLOCK - CORPNET to Unauthorized Private Networks`

---

# CORPNET to DMZ SSH Validation

After the rule was applied, SSH from:

`LAB-WIN-01`

to:

`LAB-DMZ-01`

succeeded.

Status:

**PASS**

This created a narrow administrative path rather than permitting all CORPNET systems.

---

# Why the Domain Controller Was Not Used

Using:

`LAB-DC-01`

as the DMZ administration system was considered and rejected.

The Domain Controller is critical identity infrastructure.

Using it as a routine SSH jump host would:

- increase attack surface
- mix administrative roles
- expose privileged credentials
- weaken identity isolation

The approved management source remains:

`LAB-WIN-01`

---

# ADNET to DMZ SSH Validation

Test:

`LAB-DC-01 → LAB-DMZ-01 TCP/22`

Expected:

`BLOCK`

Result:

`TcpTestSucceeded : False`

Status:

**PASS**

---

# SSH Key Deployment

After network-level SSH access was functioning, authentication was hardened.

An Ed25519 SSH key was generated on:

`LAB-WIN-01`

The public key was installed for the Ubuntu account:

`emmanuel`

on:

`LAB-DMZ-01`

---

# SSH Hardening

Final effective SSH settings include:

    PermitRootLogin no
    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no

A dedicated override file was used:

`/etc/ssh/sshd_config.d/00-dmz-hardening.conf`

This was important because cloud-init configuration included:

`PasswordAuthentication yes`

The custom hardening file took precedence.

---

# SSH Key Validation

Key-based login from:

`LAB-WIN-01`

succeeded.

Status:

**PASS**

---

# Password Authentication Validation

Password-only authentication was deliberately tested.

Result:

`Permission denied (publickey).`

Status:

**PASS**

This confirmed password SSH was disabled.

---

# Defense in Depth for DMZ Administration

DMZ SSH administration therefore requires two separate conditions:

    Approved Source Host
            |
            v
    OPNsense Firewall
            |
            v
    TCP/22
            |
            v
    Public-Key Authentication
            |
            v
    LAB-DMZ-01

This is significantly stronger than simply exposing SSH to the entire internal network.

---

# Wazuh Agent Installation

After connectivity to the SIEM had already been validated, the Wazuh agent was installed.

Agent version:

`4.14.7-1`

Manager:

`10.10.40.10`

Agent name:

`LAB-DMZ-01`

Final Wazuh agent:

    ID: 004
    Name: LAB-DMZ-01
    Status: Active

---

# Why Wazuh Version Was Checked

Before installing the agent, the manager version was confirmed as:

`v4.14.7`

The DMZ agent package was selected accordingly.

This reduced the risk of version compatibility problems.

---

# Nginx Monitoring Integration

Wazuh was configured to monitor the Nginx logs.

Access log:

`/var/log/nginx/access.log`

Log format:

`apache`

Error log:

`/var/log/nginx/error.log`

Log format:

`syslog`

After configuration, the Wazuh agent was restarted and remained active.

---

# Nginx Detection Validation

A controlled request was generated from Kali:

`/wazuh-nginx-verify-01`

Source:

`10.10.10.10`

Nginx recorded:

    "GET /wazuh-nginx-verify-01 HTTP/1.1" 404

Wazuh generated:

`Rule 31101`

Description:

`Web server 400 error code.`

Event details included:

- Agent ID: `004`
- Agent: `LAB-DMZ-01`
- Source IP: `10.10.10.10`
- Decoder: `web-accesslog`

Status:

**PASS**

This proved that web activity was visible at the application layer.

---

# Suricata DMZ Visibility

One of the final DMZ objectives was to prove that REDNET-to-DMZ traffic was visible to Suricata.

A controlled test file was created:

`/var/www/html/suricata-test.txt`

The content was:

    uid=0(root) gid=0(root) groups=0(root)

Kali then requested the file over HTTP.

---

# Suricata Detection

Suricata detected:

    Signature ID: 2100498
    Signature: GPL ATTACK_RESPONSE id check returned root
    Category: Potentially Bad Traffic
    Severity: 2
    Direction: to_client

Observed flow:

    Source:
    10.10.60.10:80

    Destination:
    10.10.10.10

Interface:

`em2`

This confirmed Suricata visibility into the REDNET ↔ DMZ traffic path.

---

# Wazuh Suricata Correlation

The same event reached Wazuh and generated:

`Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

Status:

**PASS**

This confirmed the end-to-end detection chain:

    LAB-KALI-01
        |
        v
    LAB-FW-01
        |
        v
    Suricata
        |
        v
    LAB-DMZ-01
        |
        v
    IDS Alert
        |
        v
    LAB-SIEM-01
        |
        v
    Wazuh

---

# Temporary Test File Cleanup

After Suricata validation, the temporary file:

`/var/www/html/suricata-test.txt`

was removed.

This ensured that a deliberately suspicious test resource was not left permanently available on the server.

---

# System Updating

The DMZ server was updated using:

`apt update`

and:

`apt upgrade -y`

A total of approximately:

`23 packages`

were upgraded during the final maintenance stage.

---

# Service Validation After Updates

After the update, the following were confirmed active:

- Nginx
- Wazuh Agent
- SSH

No reboot was required.

---

# Why apt autoremove Was Not Used

Unused packages or an older kernel were not automatically removed simply for cosmetic cleanup.

Because the system was stable, there was no operational need to introduce another change immediately before final validation.

This followed the project principle:

**Do not make unnecessary changes to a validated system.**

---

# Listening Services Validation

Final service checks showed the externally relevant listening services were primarily:

- TCP `22` — SSH
- TCP `80` — HTTP

Other local services such as DNS stub or time-related services remained bound locally where appropriate.

This helped verify that the DMZ server was not unexpectedly exposing additional network services.

---

# Post-Update SSH Validation

After updates, remote administration was tested again from:

`LAB-WIN-01`

The server returned:

- hostname `lab-dmz-01`
- SSH active
- Nginx active
- Wazuh agent active

This confirmed the update did not break the management or monitoring path.

---

# Final DMZ Connectivity Validation

## DMZ → SOCNET Wazuh

Port:

`TCP/1514`

Expected:

`ALLOW`

Result:

**PASS**

---

## DMZ → CORPNET SMB

Port:

`TCP/445`

Expected:

`BLOCK`

Result:

Timed out.

Status:

**PASS**

---

## DMZ → ADNET LDAP

Port:

`TCP/389`

Expected:

`BLOCK`

Result:

Timed out.

Status:

**PASS**

---

## DMZ → Internet HTTPS

Port:

`TCP/443`

Expected:

`ALLOW`

Result:

Remote HTTP response received.

Status:

**PASS**

---

# Final Inbound Validation

## REDNET → DMZ HTTP

Expected:

`ALLOW`

Result:

`HTTP 200`

Status:

**PASS**

---

## REDNET → DMZ SSH

Expected:

`BLOCK`

Result:

Timed out.

Status:

**PASS**

---

## LAB-WIN-01 → DMZ SSH

Expected:

`ALLOW`

Result:

Successful key-based SSH.

Status:

**PASS**

---

## ADNET → DMZ SSH

Expected:

`BLOCK`

Result:

Failed.

Status:

**PASS**

---

# DMZ Monitoring Architecture

The final DMZ monitoring design includes multiple layers.

    REDNET Traffic
         |
         v
    LAB-FW-01
      OPNsense
         |
         +--> Firewall Logging
         |
         +--> Suricata IDS
         |
         v
    LAB-DMZ-01
         |
         +--> Nginx Access Log
         |
         +--> Nginx Error Log
         |
         +--> Linux System Logs
         |
         +--> Wazuh Agent
         |
         v
    LAB-SIEM-01

This gives both:

- network visibility
- application visibility
- host visibility

---

# DMZ Security Model

The final DMZ uses defense in depth.

## Network Segmentation

OPNsense separates the DMZ from internal networks.

## Least Privilege

Only required communication paths are allowed.

## Restricted Administration

Only `LAB-WIN-01` can SSH to the DMZ.

## Strong Authentication

SSH uses public-key authentication only.

## No Root SSH

Root login is disabled.

## Application Monitoring

Nginx logs are sent to Wazuh.

## Host Monitoring

Wazuh monitors the server.

## Network Detection

Suricata inspects attacker traffic.

## Logged Denies

Important DMZ boundary violations are logged.

---

# Final DMZ Validation Summary

| Validation | Result |
|---|---|
| Static IP `10.10.60.10` | PASS |
| Gateway `10.10.60.1` | PASS |
| DNS through OPNsense | PASS |
| NTP through OPNsense | PASS |
| DMZ → Wazuh/1514 | PASS |
| DMZ → Wazuh/1515 | PASS |
| DMZ → CORPNET SMB blocked | PASS |
| DMZ → ADNET LDAP blocked | PASS |
| DMZ → Internet HTTPS | PASS |
| Unauthorized outbound port blocked | PASS |
| Nginx active | PASS |
| REDNET → DMZ HTTP | PASS |
| REDNET → DMZ SSH blocked | PASS |
| CORPNET admin → DMZ SSH | PASS |
| ADNET → DMZ SSH blocked | PASS |
| Public-key SSH | PASS |
| Password SSH blocked | PASS |
| Wazuh agent active | PASS |
| Nginx log ingestion | PASS |
| Wazuh Rule 31101 | PASS |
| Suricata SID 2100498 | PASS |
| Wazuh Rule 86601 | PASS |
| Post-update services healthy | PASS |

---

# Snapshot History

Snapshots were taken before major DMZ changes, including:

- base VM configuration
- network configuration
- firewall deployment
- Nginx installation
- Wazuh installation
- web exposure
- SSH administration
- SSH hardening
- Suricata validation
- final updates

These checkpoints provided safe rollback points during the build.

After the entire six-zone architecture passed final validation, temporary development snapshots were removed.

The remaining DMZ snapshot is:

`Final-Six-Zone-Validated-Baseline`

Snapshot UUID:

`ac27c1e7-c61f-41fd-ab11-b7e84883068c`

---

# Historical vs Current DMZ State

The project originally had no DMZ.

The DMZ appeared only after:

- the initial flat network
- OPNsense segmentation
- Wazuh
- Windows monitoring
- Linux monitoring
- Suricata
- Active Directory

The authoritative current DMZ state is:

    Hostname: LAB-DMZ-01
    OS: Ubuntu Server 26.04.1 LTS
    IP: 10.10.60.10
    Zone: DMZ
    Gateway: 10.10.60.1
    Web Server: Nginx
    SSH: Key-only
    Root SSH: Disabled
    Wazuh Agent: Active
    Nginx Monitoring: Active
    Suricata Visibility: Validated

---

# Evolution Summary

The DMZ evolved through the following stages:

    No DMZ
       |
       v
    DMZ Architecture Planning
       |
       v
    OPNsense NIC 7 Addition
       |
       v
    DMZ-NET Creation
       |
       v
    Ubuntu Server Deployment
       |
       v
    Static Networking
       |
       v
    Restrictive Firewall Policy
       |
       v
    Nginx Deployment
       |
       v
    REDNET Web Exposure
       |
       v
    Wazuh Integration
       |
       v
    Restricted CORPNET SSH
       |
       v
    Key-Only SSH Hardening
       |
       v
    Nginx Detection Validation
       |
       v
    Suricata Detection Validation
       |
       v
    Final Six-Zone Baseline

---

# Final Role

`LAB-DMZ-01` now serves as the lab’s intentionally exposed web server.

Its primary purposes are:

- controlled web attack simulation
- Suricata detection testing
- Nginx monitoring
- Wazuh application telemetry
- DMZ segmentation validation
- restricted administration
- lateral-movement testing
- Purple Team scenarios

---

# Current Status

The DMZ infrastructure phase is complete.

`LAB-DMZ-01` is:

- correctly segmented
- monitored by Wazuh
- visible to Suricata
- serving Nginx
- reachable from REDNET only on the intended web service
- administratively reachable only from the approved workstation
- protected from unauthorized internal access
- restricted in outbound communication
- hardened with key-only SSH
- fully validated as part of the final six-zone architecture

Future work should use the DMZ for realistic web, SOC, IDS, incident-response, and Purple Team exercises rather than continuing to modify the baseline infrastructure.
