# LAB-KALI-01 — Offensive Security Workstation

## Overview

This document records the complete lifecycle of the Kali Linux system used throughout the Cybersecurity Home Lab, from the earliest flat-network environment through its final role as `LAB-KALI-01` in the segmented six-zone architecture.

Kali Linux has been part of the project since the beginning.

Its role evolved from a general-purpose penetration-testing VM into a deliberately isolated offensive-security workstation used for controlled attack simulation, network validation, IDS testing, and Purple Team exercises.

The final host is:

`LAB-KALI-01`

Final address:

`10.10.10.10`

Final zone:

`REDNET`

Primary role:

Offensive security workstation

---

## Phase 1 — Original Kali Lab System

The earliest version of the lab used Kali Linux as the primary offensive system.

The original lab used a simple VirtualBox network based around:

`192.168.56.0/24`

Kali was assigned:

`192.168.56.10`

The other major systems were:

- Windows — `192.168.56.20`
- Ubuntu Server — `192.168.56.30`

At this stage, Kali was used for foundational offensive-security exercises.

These included:

- Nmap scanning
- SSH enumeration
- Hydra authentication testing
- service discovery
- attacker-to-server interaction
- fail2ban validation
- network troubleshooting

The environment was much simpler than the final architecture and did not yet include OPNsense-based trust zones.

---

# Early Kali Networking

In the early lab, Kali used a combination of VirtualBox networking modes.

The host typically had:

- a NAT interface for Internet access
- an internal or host-only-style interface for lab communication

A later configuration showed:

`eth0`

used for VirtualBox NAT with:

`10.0.2.15/24`

Gateway:

`10.0.2.2`

and:

`eth1`

used for the internal lab network with:

`192.168.56.10/24`

Routing included:

    default via 10.0.2.2
    192.168.56.0/24 via eth1

This configuration allowed Kali to:

- reach the Internet
- communicate with the internal lab
- perform attacker testing against Windows and Ubuntu

---

# Early Offensive Exercises

Kali was used extensively during the initial hands-on phase of the lab.

## Nmap

Nmap was used to identify:

- live hosts
- open ports
- service exposure
- changes in firewall state

Example observations included:

- SSH port open
- SSH port later filtered
- host availability changes

These tests helped validate host-level security changes.

---

## SSH Testing

Kali was used to connect to the Ubuntu server over SSH.

This created the basis for later attacker/defender testing.

The Ubuntu server was then hardened using:

- fail2ban
- SSH controls

Kali was used to verify the effect of those changes.

---

## Hydra Testing

Hydra was used in controlled authentication testing against SSH.

This was part of the early defensive validation process.

The objective was not merely to perform password attacks, but to observe:

- failed authentication behaviour
- fail2ban response
- source blocking
- service availability after repeated failures

This introduced the project to the concept of validating defenses from an attacker perspective.

---

## fail2ban Validation

The Ubuntu server was configured with fail2ban.

Kali generated repeated authentication failures.

The defensive response was then observed.

This demonstrated an early version of the Purple Team workflow:

    Attacker Activity
        |
        v
    Defensive Control
        |
        v
    Detection / Blocking
        |
        v
    Retest

This concept later became a core design philosophy for the entire lab.

---

# Transition Away from the Flat Network

As the project matured, the original `192.168.56.0/24` architecture became too limited.

The key problem was that Kali existed too close to the systems it was intended to attack.

The flat design did not adequately represent:

- trust boundaries
- enterprise segmentation
- controlled attacker access
- firewall policy
- lateral-movement restrictions
- network IDS monitoring

Kali therefore became one of the main reasons for introducing a dedicated attacker zone.

---

# Introduction of REDNET

When OPNsense was introduced, Kali was moved into:

`REDNET`

Network:

`10.10.10.0/24`

Gateway:

`10.10.10.1`

Final address:

`10.10.10.10`

The system was renamed and standardized as:

`LAB-KALI-01`

This transformed Kali from a general internal VM into a clearly defined adversarial system.

---

# Final Kali Network Role

Final configuration:

    Hostname: LAB-KALI-01
    IP Address: 10.10.10.10
    Network: REDNET
    Gateway: 10.10.10.1

REDNET is treated as the least-trusted internal security zone.

Kali no longer receives unrestricted access to enterprise systems.

All inter-zone communication must pass through:

`LAB-FW-01`

---

# OPNsense and Kali

OPNsense became the central enforcement point between Kali and the rest of the lab.

Kali traffic is therefore subject to:

- firewall rules
- private-network blocks
- explicit allow rules
- logging
- Suricata IDS inspection

This significantly improved the realism of offensive testing.

---

# Early REDNET Testing

The first REDNET phase allowed Kali to perform controlled testing against SERVERNET.

Examples included:

- Nmap
- SSH testing
- Hydra
- service discovery
- fail2ban validation

However, REDNET was prevented from freely reaching:

- CORPNET
- firewall management
- other protected systems

This was the first major step toward a safer attacker environment.

---

# REDNET Isolation

The final architecture restricts Kali significantly.

Validated blocked paths include:

    REDNET → DMZ SSH/22
    REDNET → CORPNET SMB/445
    REDNET → ADNET LDAP/389

These tests confirmed that Kali cannot freely administer or access protected systems.

---

# REDNET → DMZ Web Testing

The final architecture provides one intentional attack path:

    LAB-KALI-01
    10.10.10.10
        |
        | HTTP TCP/80
        v
    LAB-DMZ-01
    10.10.60.10

This path is used for:

- web-security testing
- Nginx monitoring
- Suricata detection validation
- Purple Team scenarios

The firewall explicitly allows:

`REDNET → LAB-DMZ-01 HTTP/80`

---

# REDNET → DMZ HTTP Validation

Test:

    curl -s -o /dev/null -w "%{http_code}\n" http://10.10.60.10/

Result:

`200`

Status:

**PASS**

This confirmed that Kali could reach the intended web target.

---

# REDNET → DMZ SSH Validation

Test:

    nc -zv -w 3 10.10.60.10 22

Result:

Connection timed out.

Status:

**PASS — Blocked as designed**

This confirms that Kali can test the web service without gaining SSH administration access.

---

# REDNET → CORPNET SMB Validation

Test:

    nc -zv -w 3 10.10.20.20 445

Result:

Connection timed out.

Status:

**PASS — Blocked**

This prevents direct SMB access to the Windows workstation.

---

# REDNET → ADNET LDAP Validation

Test:

    nc -zv -w 3 10.10.50.10 389

Result:

Connection timed out.

Status:

**PASS — Blocked**

This prevents direct LDAP access to the Domain Controller.

---

# Suricata Visibility

One of Kali’s most important final roles is generating traffic that can be inspected by Suricata.

Suricata runs on:

`LAB-FW-01`

Primary monitored interface:

`em2`

This interface corresponds to REDNET.

As a result, traffic generated by Kali can be inspected before it reaches target systems.

---

# Controlled Suricata Validation

A controlled test resource was created on the DMZ server containing:

    uid=0(root) gid=0(root) groups=0(root)

Kali requested:

`http://10.10.60.10/suricata-test.txt`

The response triggered:

- Suricata Signature ID `2100498`
- `GPL ATTACK_RESPONSE id check returned root`
- Category: `Potentially Bad Traffic`

Observed traffic included:

    Source:
    10.10.60.10:80

    Destination:
    10.10.10.10

    Direction:
    to_client

This confirmed that Kali-generated REDNET traffic was visible to the IDS.

---

# Suricata → Wazuh Validation

The Suricata event generated by the Kali test was forwarded to Wazuh.

Wazuh produced:

`Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

This validated the complete monitoring path:

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
    Suricata Alert
        |
        v
    LAB-SIEM-01
        |
        v
    Wazuh

---

# Nginx Monitoring Test

Kali was also used to validate host-level application monitoring on the DMZ.

A controlled request was generated:

`GET /wazuh-nginx-verify-01 HTTP/1.1`

Source:

`10.10.10.10`

Destination:

`10.10.60.10`

Result:

`404`

The request appeared in:

`/var/log/nginx/access.log`

Wazuh generated:

- Rule ID `31101`
- `Web server 400 error code.`
- Agent: `LAB-DMZ-01`
- Source IP: `10.10.10.10`

This demonstrated that Kali activity could be detected both:

- at the network layer through Suricata
- at the application layer through Nginx/Wazuh

---

# Kali as a Purple Team Source

By the end of the infrastructure phase, Kali’s role had matured significantly.

It is no longer used simply to run isolated attack commands.

It now functions as the controlled adversary in a larger detection ecosystem.

Attack path:

    Kali
      |
      v
    OPNsense Firewall
      |
      v
    Suricata IDS
      |
      v
    Target Host
      |
      v
    Endpoint / Application Logs
      |
      v
    Wazuh SIEM
      |
      v
    SOC Investigation

This makes `LAB-KALI-01` a central component of the lab’s Purple Team design.

---

# Kali and Active Directory

Kali does not receive normal access to ADNET.

This is deliberate.

The objective is to avoid creating an unrealistic attacker network where the offensive workstation begins with unrestricted identity-network access.

Future Active Directory attack simulations should therefore operate within clearly defined scenarios and approved paths rather than permanently opening broad firewall access.

---

# Kali and CORPNET

Kali is prevented from freely accessing the Windows workstation.

This creates a realistic distinction between:

- attacker network
- corporate endpoint network

Any future direct attack path toward CORPNET should be introduced intentionally for a specific scenario and removed or controlled afterward.

---

# Kali and SERVERNET

During earlier phases, Kali had more direct controlled interaction with the Ubuntu server.

This supported foundational exercises such as:

- SSH testing
- Hydra
- Nmap
- fail2ban

As the architecture matured, SERVERNET became more tightly protected.

This reflects the evolution from:

**learning individual tools**

to:

**operating within an enterprise security architecture**

---

# Internet Connectivity

Kali retains approved outbound Internet access through OPNsense.

This is required for:

- package updates
- security tools
- repositories
- research
- lab dependencies

The final architecture does not require a separate direct NAT adapter on Kali.

Traffic is routed through:

`LAB-FW-01`

---

# DNS

Kali uses approved DNS services through the firewall architecture.

This allows Internet and lab name resolution while maintaining central network control.

---

# Security Benefits of REDNET

Moving Kali into REDNET provides several benefits.

## Isolation

Kali cannot freely communicate with protected zones.

## Visibility

Traffic crosses OPNsense and can be logged.

## IDS Monitoring

Suricata can inspect Kali-generated traffic.

## Repeatability

Attack paths can be deliberately enabled for specific scenarios.

## Realism

The attacker system behaves more like an external or low-trust source rather than an ordinary internal workstation.

## Safety

Offensive testing is less likely to unintentionally affect unrelated lab systems.

---

# Final Validation Summary

Final Kali-related validation includes:

| Test | Result |
|---|---|
| REDNET → DMZ HTTP/80 | PASS |
| REDNET → DMZ SSH/22 blocked | PASS |
| REDNET → CORPNET SMB/445 blocked | PASS |
| REDNET → ADNET LDAP/389 blocked | PASS |
| Nginx request observed in Wazuh | PASS |
| Suricata REDNET-to-DMZ visibility | PASS |
| Suricata Rule 2100498 triggered | PASS |
| Wazuh Rule 86601 generated | PASS |

---

# Snapshot History

Snapshots were taken throughout the project before significant changes.

Kali previously retained development snapshots such as:

`REDNET Final Cutover Verified`

After the entire lab passed final validation, temporary snapshots were removed.

The remaining snapshot is:

`Final-Six-Zone-Validated-Baseline`

This represents the known-good final infrastructure state.

---

# Historical vs Current Kali State

This document intentionally records Kali from the earliest stage of the project.

Historical configuration includes:

- `192.168.56.10`
- NAT plus internal adapter
- direct attacker-to-Ubuntu testing
- early Nmap
- Hydra
- fail2ban validation

Current authoritative configuration is:

    Hostname: LAB-KALI-01
    IP: 10.10.10.10
    Zone: REDNET
    Gateway: 10.10.10.1
    Role: Offensive Security Workstation

The historical configuration is preserved because it shows how the project evolved.

It should not be interpreted as the current network state.

---

# Final Role

`LAB-KALI-01` now serves as the lab’s controlled adversarial platform.

Its primary responsibilities are:

- attack simulation
- reconnaissance
- controlled exploitation
- web testing
- network-security validation
- Suricata testing
- Purple Team exercises
- detection validation

The system is intentionally constrained by firewall policy so that offensive activity occurs within clearly defined security boundaries.

---

# Current Status

The Kali infrastructure phase is complete.

The system is:

- correctly segmented
- routed through OPNsense
- visible to Suricata
- able to reach the intended DMZ web target
- prevented from accessing unauthorized services
- ready for realistic SOC and Purple Team simulations

Future work should focus on operational attack scenarios rather than further infrastructure modification.
