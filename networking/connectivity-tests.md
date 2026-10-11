# Connectivity and Segmentation Validation

## Overview

This document records the final connectivity and segmentation validation performed across the six-zone Cybersecurity Home Lab.

The objective was to confirm that:

- required services remain reachable
- unauthorized east-west communication is blocked
- security monitoring paths remain operational
- Active Directory connectivity works correctly
- DMZ access follows least-privilege policy
- REDNET remains isolated from protected enterprise services
- all monitored endpoints remain connected to Wazuh

The validation was completed after the firewall, Active Directory, SIEM, Suricata, endpoint-monitoring and DMZ phases were finished.

---

## Final Network Architecture

| Zone | Subnet | Gateway | Primary Host |
|---|---|---|---|
| REDNET | `10.10.10.0/24` | `10.10.10.1` | `LAB-KALI-01` |
| CORPNET | `10.10.20.0/24` | `10.10.20.1` | `LAB-WIN-01` |
| SERVERNET | `10.10.30.0/24` | `10.10.30.1` | `LAB-UBUNTU-01` |
| SOCNET | `10.10.40.0/24` | `10.10.40.1` | `LAB-SIEM-01` |
| ADNET | `10.10.50.0/24` | `10.10.50.1` | `LAB-DC-01` |
| DMZ | `10.10.60.0/24` | `10.10.60.1` | `LAB-DMZ-01` |

All inter-zone traffic is routed through `LAB-FW-01`.

---

# Validation Method

Connectivity testing was performed using:

- `curl`
- `nc`
- `Test-NetConnection`
- `dcdiag`
- Wazuh `agent_control`
- direct service validation
- firewall logging
- Suricata alerts
- Wazuh alert verification

The tests were intentionally selected to confirm both approved and denied communication paths.

A successful timeout or failed connection was considered a pass when the firewall policy required the traffic to be blocked.

---

# REDNET Validation

Source host:

`LAB-KALI-01 — 10.10.10.10`

REDNET represents the offensive and least-trusted network.

The objective was to ensure Kali could reach the intended DMZ web service while remaining isolated from administrative and internal enterprise services.

## REDNET → DMZ HTTP

Test:

    curl -s -o /dev/null -w "%{http_code}\n" http://10.10.60.10/

Result:

    HTTP 200

Status:

**PASS — Allowed as designed**

Purpose:

This path supports controlled web attack simulations against the DMZ.

---

## REDNET → DMZ SSH

Test:

    nc -zv -w 3 10.10.60.10 22

Result:

    Connection timed out

Status:

**PASS — Blocked as designed**

This confirms the offensive workstation cannot use SSH to administer the DMZ server.

---

## REDNET → CORPNET SMB

Test:

    nc -zv -w 3 10.10.20.20 445

Result:

    Connection timed out

Status:

**PASS — Blocked as designed**

This prevents direct SMB access from the offensive network to the corporate workstation.

---

## REDNET → ADNET LDAP

Test:

    nc -zv -w 3 10.10.50.10 389

Result:

    Connection timed out

Status:

**PASS — Blocked as designed**

This confirms Kali cannot directly query Active Directory through LDAP.

---

# CORPNET Validation

Source host:

`LAB-WIN-01 — 10.10.20.20`

CORPNET represents the enterprise workstation environment.

The workstation must be able to communicate with Active Directory and the Wazuh SIEM while also retaining the approved DMZ administration path.

---

## CORPNET → ADNET DNS

Test:

    Test-NetConnection 10.10.50.10 -Port 53

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

## CORPNET → ADNET Kerberos

Test:

    Test-NetConnection 10.10.50.10 -Port 88

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

## CORPNET → ADNET LDAP

Test:

    Test-NetConnection 10.10.50.10 -Port 389

Result:

    TcpTestSucceeded : True

Status:

**PASS**

These three services confirm that the corporate workstation retains the core communication required for Active Directory domain membership.

---

## CORPNET → SOCNET Wazuh

Test:

    Test-NetConnection 10.10.40.10 -Port 1514

Result:

    TcpTestSucceeded : True

Status:

**PASS**

This confirms endpoint telemetry can reach the Wazuh manager.

---

## CORPNET → DMZ Administrative SSH

Test:

    Test-NetConnection 10.10.60.10 -Port 22

Result:

    TcpTestSucceeded : True

Status:

**PASS**

This path is intentionally restricted to the approved administrative workstation.

Approved path:

    LAB-WIN-01
        |
        | TCP/22
        v
    LAB-DMZ-01

---

# SERVERNET Validation

Source host:

`LAB-UBUNTU-01 — 10.10.30.30`

SERVERNET represents protected internal Linux server infrastructure.

---

## SERVERNET → SOCNET Wazuh

Test:

    nc -zv -w 3 10.10.40.10 1514

Result:

    Connection succeeded

Status:

**PASS — Allowed**

---

## SERVERNET → CORPNET SMB

Test:

    nc -zv -w 3 10.10.20.20 445

Result:

    Connection timed out

Status:

**PASS — Blocked**

This reduces unnecessary lateral movement between the internal server and workstation zones.

---

## SERVERNET → DMZ SSH

Test:

    nc -zv -w 3 10.10.60.10 22

Result:

    Connection timed out

Status:

**PASS — Blocked**

SERVERNET is not authorized to administer the DMZ server.

---

# ADNET Validation

Source host:

`LAB-DC-01 — 10.10.50.10`

ADNET contains critical identity infrastructure.

The Domain Controller should communicate with SOCNET for telemetry but should not be used as a general-purpose jump host.

---

## ADNET → SOCNET Wazuh

Test:

    Test-NetConnection 10.10.40.10 -Port 1514

Result:

    TcpTestSucceeded : True

Status:

**PASS**

---

## ADNET → DMZ SSH

Test:

    Test-NetConnection 10.10.60.10 -Port 22

Result:

    TcpTestSucceeded : False

Status:

**PASS — Blocked**

This confirms the Domain Controller cannot be used to SSH directly into the DMZ.

---

# Active Directory Health Validation

The following targeted Domain Controller checks were executed:

    dcdiag /test:Advertising /test:SysVolCheck /test:NetLogons /test:Services /q

Result:

No output.

With `dcdiag /q`, no output indicates that no failures were detected by the requested tests.

Validated components:

- Advertising
- SYSVOL
- NetLogons
- Services

Previous DNS validation also completed successfully.

Status:

**PASS**

---

# DMZ Validation

Source host:

`LAB-DMZ-01 — 10.10.60.10`

The DMZ must communicate with SOCNET for monitoring and the Internet for approved updates while remaining isolated from protected internal services.

---

## DMZ → SOCNET Wazuh

Test:

    nc -zv -w 3 10.10.40.10 1514

Result:

    Connection succeeded

Status:

**PASS — Allowed**

---

## DMZ → CORPNET SMB

Test:

    nc -zv -w 3 10.10.20.20 445

Result:

    Connection timed out

Status:

**PASS — Blocked**

---

## DMZ → ADNET LDAP

Test:

    nc -zv -w 3 10.10.50.10 389

Result:

    Connection timed out

Status:

**PASS — Blocked**

These results confirm the DMZ cannot freely initiate connections into protected workstation or identity infrastructure.

---

## DMZ → Internet HTTPS

Test:

    curl -I --connect-timeout 5 https://packages.wazuh.com

Result:

    HTTP/2 403

Status:

**PASS — Network connectivity confirmed**

The `403` response confirms that:

- DNS resolution succeeded
- TCP connectivity succeeded
- TLS negotiation succeeded
- the remote HTTPS server received the request

The application-level denial does not indicate a network failure.

---

# DMZ SSH Administration Validation

The administrative path to `LAB-DMZ-01` was tested independently from the CORPNET workstation.

## Approved Source

`LAB-WIN-01 — 10.10.20.20`

## Destination

`LAB-DMZ-01 — 10.10.60.10`

## Service

`TCP/22`

Key-based authentication was configured using an Ed25519 SSH key.

The following effective SSH settings were validated:

    permitrootlogin no
    pubkeyauthentication yes
    passwordauthentication no
    kbdinteractiveauthentication no

Key-only authentication from `LAB-WIN-01` succeeded.

Password-only authentication was then tested using:

    ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password emmanuel@10.10.60.10

Result:

    Permission denied (publickey).

Status:

**PASS**

This confirms:

- SSH key authentication works
- password SSH is disabled
- root SSH is disabled
- the approved administrative path remains functional

---

# Wazuh Agent Validation

Wazuh agent status was verified from `LAB-SIEM-01`.

Final result:

    ID: 001, Name: LAB-WIN-01, IP: any, Active
    ID: 002, Name: LAB-UBUNTU-01, IP: any, Active
    ID: 003, Name: LAB-DC-01, IP: any, Active
    ID: 004, Name: LAB-DMZ-01, IP: any, Active

Status:

**PASS**

All monitored endpoints were connected during final validation.

---

# Wazuh Stack Validation

The following services were checked on `LAB-SIEM-01`:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Result:

    active
    active
    active
    active

Status:

**PASS**

---

# SIEM Storage Validation

Filesystem:

`/dev/mapper/ubuntu--vg-ubuntu--lv`

Final observed state:

    Size: 53G
    Used: 25G
    Available: 26G
    Usage: 49%

Status:

**PASS**

The SIEM had sufficient free storage during final infrastructure validation.

---

# Nginx Monitoring Validation

A controlled request was generated from REDNET to the DMZ:

    GET /wazuh-nginx-verify-01 HTTP/1.1

Source:

`10.10.10.10`

Destination:

`10.10.60.10`

Nginx response:

`404`

The request appeared in:

`/var/log/nginx/access.log`

Wazuh generated:

- Rule ID: `31101`
- Description: `Web server 400 error code.`
- Agent: `LAB-DMZ-01`
- Source IP: `10.10.10.10`
- Decoder: `web-accesslog`

Status:

**PASS**

This proves the application-layer monitoring path:

    Kali
      |
      v
    Nginx
      |
      v
    access.log
      |
      v
    Wazuh Agent
      |
      v
    LAB-SIEM-01

---

# Suricata Validation

A controlled IDS validation was performed between REDNET and the DMZ.

A temporary test resource returned:

    uid=0(root) gid=0(root) groups=0(root)

The request originated from:

`LAB-KALI-01 — 10.10.10.10`

The HTTP response originated from:

`LAB-DMZ-01 — 10.10.60.10`

Suricata generated:

- Event type: `alert`
- Interface: `em2`
- Signature ID: `2100498`
- Signature: `GPL ATTACK_RESPONSE id check returned root`
- Category: `Potentially Bad Traffic`
- Direction: `to_client`

Observed response flow:

    10.10.60.10:80
        |
        v
    10.10.10.10

Status:

**PASS**

---

# Suricata → Wazuh Validation

Suricata events are forwarded to:

`LAB-SIEM-01`

and written to:

`/var/log/suricata/eve.json`

Wazuh monitors the file using:

    <log_format>json</log_format>

The controlled IDS event generated:

- Wazuh Rule ID: `86601`
- Description: `Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

Status:

**PASS**

This confirms the full detection path:

    LAB-KALI-01
        |
        v
    LAB-FW-01 / Suricata
        |
        v
    LAB-DMZ-01
        |
        v
    Suricata Alert
        |
        v
    EVE JSON
        |
        v
    LAB-SIEM-01
        |
        v
    Wazuh Rule 86601

---

# Final Segmentation Matrix

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

# Security Conclusions

The validation confirms that the lab behaves as a segmented enterprise-style network rather than a flat home lab.

Key outcomes include:

- REDNET cannot freely access enterprise infrastructure
- Active Directory services are reachable only where required
- the DMZ is isolated from protected internal systems
- SSH administration is restricted to an approved workstation
- password SSH is disabled
- Wazuh monitoring paths are operational
- all endpoint agents are active
- Suricata sees REDNET-to-DMZ attack traffic
- Suricata alerts reach Wazuh
- Nginx activity reaches Wazuh
- Domain Controller health is validated
- SIEM services and storage are healthy

---

# Final Validation Status

**Six-zone infrastructure validation: COMPLETE**

Validated zones:

- REDNET
- CORPNET
- SERVERNET
- SOCNET
- ADNET
- DMZ

Following successful validation, every virtual machine was placed at the common snapshot:

`Final-Six-Zone-Validated-Baseline`

This baseline represents the final verified infrastructure state before realistic SOC and Purple Team operational scenarios begin.
