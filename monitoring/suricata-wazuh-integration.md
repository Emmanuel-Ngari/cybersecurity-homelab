# Suricata to Wazuh Integration

## Overview

This document records the complete evolution of Suricata network IDS integration in the Cybersecurity Home Lab, from the period when the lab had no dedicated IDS through the final validated Suricata-to-Wazuh detection pipeline.

Suricata was introduced only after the core lab had already evolved beyond the original flat environment.

The project began with:

- Kali Linux
- Windows
- Ubuntu Server

on:

`192.168.56.0/24`

At that stage, security testing relied mainly on:

- endpoint logs
- Nmap
- SSH
- Hydra
- fail2ban
- manual network observation

There was no dedicated network IDS.

As the lab matured, OPNsense was introduced, segmentation was added, Wazuh became the central SIEM, and network-level detection became the next major capability.

Suricata was deployed on:

`LAB-FW-01`

and integrated with:

`LAB-SIEM-01`

The final pipeline uses:

- Suricata on OPNsense
- remote logging
- UDP/5514
- rsyslog
- `/var/log/suricata/eve.json`
- Wazuh JSON ingestion
- built-in Suricata rules
- centralized detection in Wazuh

---

# Phase 1 — Before Suricata

The original flat lab had no IDS.

Security activity was observed mainly at the host level.

Examples included:

- Nmap output on Kali
- SSH logs on Ubuntu
- fail2ban bans
- Windows local logs

This provided useful endpoint visibility but little network-layer detection.

The environment could answer:

- Did the service respond?
- Was the attacker blocked?
- Did the endpoint log the activity?

But it could not yet answer:

- Did the network traffic match a known IDS signature?
- Could malicious traffic be detected before or independently of endpoint logs?
- Could the SOC correlate network and endpoint events?

This created the need for Suricata.

---

# OPNsense as the IDS Platform

Suricata was installed on:

`LAB-FW-01`

rather than on a separate sensor.

This allowed the firewall to provide both:

- traffic enforcement
- network detection

The design simplified the lab while still providing realistic IDS visibility.

---

# Why Suricata Was Added to OPNsense

The objectives included:

- detect known attack patterns
- inspect REDNET traffic
- generate EVE JSON
- centralize IDS telemetry in Wazuh
- support Purple Team validation
- correlate network alerts with endpoint events

This expanded the defensive architecture beyond firewall allow/block decisions.

---

# Monitored Interface

Suricata was configured on:

`em2`

This interface corresponds to:

`REDNET`

Network:

`10.10.10.0/24`

Primary attacker host:

`LAB-KALI-01 — 10.10.10.10`

This placement was intentional because REDNET generates controlled offensive traffic.

---

# Why REDNET Was Selected

Monitoring REDNET provided high-value visibility into:

- reconnaissance
- attack simulation
- web testing
- suspicious responses
- controlled Purple Team traffic

It also avoided unnecessary collection from every interface during the initial IDS deployment.

---

# Ruleset

Suricata used:

`ET Open`

rules.

This provided a broad community IDS ruleset for detecting known suspicious patterns.

---

# Initial Suricata Validation

A controlled test generated a known alert.

The event contained:

    Signature ID: 2100498
    Signature: GPL ATTACK_RESPONSE id check returned root
    Category: Potentially Bad Traffic
    Severity: 2

This confirmed that Suricata itself was functioning.

The next challenge was integrating that alert into Wazuh correctly.

---

# First Wazuh Ingestion Attempt

Suricata telemetry was forwarded toward the SIEM.

The event reached Wazuh, but decoding was initially incorrect.

The event fell through to:

`Rule 1002`

This indicated that Wazuh had received data, but the event structure was not being parsed in the intended way.

---

# Why Rule 1002 Was Important

The fallback rule was actually useful for troubleshooting.

It showed that:

- traffic reached the SIEM
- Wazuh saw the event
- transport was functioning

The problem was therefore not complete network failure.

The issue was:

- decoding
- message format
- parsing

This helped narrow the investigation.

---

# Initial Decoder Troubleshooting

The project investigated:

- Suricata EVE output
- OPNsense remote logging format
- Wazuh decoders
- syslog wrapping
- JSON fields
- test events

The objective was to determine how the Suricata event actually arrived at the manager.

---

# Custom Decoder Phase

A custom Suricata decoder was introduced during troubleshooting.

One observed decoder name was:

`suricata-eve-syslog`

This allowed the event to be matched more specifically.

The custom decoder helped prove that the event structure could be recognized.

However, this was not the final design.

---

# Why the Custom Decoder Was Not Retained

The custom wrapper introduced additional complexity.

Once it became clear that the Suricata payload could be handled as structured JSON, maintaining a custom wrapper was unnecessary.

A simpler solution was preferred.

This followed the project principle:

**Use built-in, maintainable functionality where it works reliably.**

---

# Transition to JSON Parsing

The Suricata event path was simplified so Wazuh could use the built-in:

`json`

decoder.

This improved:

- maintainability
- field extraction
- rule compatibility
- troubleshooting

The effective decoder later observed was:

`json`

---

# Built-In Suricata Rules

Once the event was parsed as JSON, Wazuh's built-in Suricata rules could process the alert.

Relevant ruleset file:

`0475-suricata_rules.xml`

The controlled test generated:

`Rule 86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

This confirmed the built-in ruleset worked.

---

# Transport Architecture Evolution

The Suricata integration went through several stages.

The final transport architecture uses a dedicated syslog port.

Protocol:

`UDP`

Port:

`5514`

Destination:

`LAB-SIEM-01 — 10.10.40.10`

This separated Suricata traffic from other syslog sources.

---

# Why UDP/5514 Was Used

A dedicated port simplified:

- traffic identification
- rsyslog filtering
- troubleshooting
- log separation

It also avoided mixing IDS telemetry with the existing firewall syslog path.

---

# SIEM rsyslog Listener

The SIEM uses rsyslog to receive Suricata events.

Configuration file:

`/etc/rsyslog.d/30-suricata-remote.conf`

The configuration uses:

`imudp`

on port:

`5514`

---

# Suricata Log Destination

Incoming Suricata events are written to:

`/var/log/suricata/eve.json`

This provides a dedicated local file for Wazuh ingestion.

---

# Directory Creation

The directory:

`/var/log/suricata`

was created before events were written.

Final ownership:

`syslog:wazuh`

Permissions:

`750`

This design ensures:

- rsyslog can write
- Wazuh can read
- unnecessary users cannot access the directory

---

# rsyslog Listener Validation

The SIEM was verified listening on:

`0.0.0.0:5514`

and:

`[::]:5514`

This proved that the network receiver was active.

---

# Why Listener Validation Came First

Before troubleshooting Wazuh decoding, the project verified the lower layers.

The sequence was:

    Is port 5514 listening?
          |
          v
    Is traffic arriving?
          |
          v
    Is the file being written?
          |
          v
    Is Wazuh reading the file?
          |
          v
    Is decoding correct?
          |
          v
    Does the rule fire?

This layered approach reduced unnecessary changes.

---

# OPNsense Remote Logging

OPNsense was configured to forward Suricata events to:

`10.10.40.10`

using:

`UDP/5514`

This created the network path:

    LAB-FW-01
        |
        v
    LAB-SIEM-01

---

# Suricata Event Reception

Once forwarding was enabled, events began reaching:

`/var/log/suricata/eve.json`

This confirmed the transport path.

At this point, the remaining work focused on Wazuh ingestion and decoding.

---

# Wazuh Localfile Configuration

Wazuh was configured to monitor:

`/var/log/suricata/eve.json`

using:

`log_format`

set to:

`json`

This allowed the logcollector to process the file as structured JSON.

---

# Logcollector Validation

Wazuh logcollector reported that it was analyzing:

`/var/log/suricata/eve.json`

This confirmed that the file was successfully registered for ingestion.

---

# Decoder Validation

After the localfile configuration was corrected, Wazuh successfully parsed the Suricata event.

The effective decoder became:

`json`

This allowed fields such as:

- signature ID
- source IP
- destination IP
- category
- severity

to be available to Wazuh rules.

---

# Controlled Signature Validation

The known controlled alert:

`SID 2100498`

was used repeatedly because it provided a predictable validation condition.

Signature:

`GPL ATTACK_RESPONSE id check returned root`

Category:

`Potentially Bad Traffic`

Severity:

`2`

This became the primary test for confirming the complete pipeline.

---

# REDNET to DMZ Test

The final validation used:

`LAB-KALI-01`

in:

`REDNET`

and:

`LAB-DMZ-01`

in:

`DMZ`

A temporary file was created on the DMZ web server containing:

    uid=0(root) gid=0(root) groups=0(root)

Kali requested the file over HTTP.

---

# Why This Test Was Chosen

The response matched the known Suricata signature.

The test was:

- controlled
- harmless
- repeatable
- easy to remove afterward

It allowed the entire IDS pipeline to be tested without relying on actual malware.

---

# Suricata Alert Details

The final observed Suricata event included:

    Source IP: 10.10.60.10
    Source Port: 80

    Destination IP: 10.10.10.10

    Signature ID: 2100498

    Signature:
    GPL ATTACK_RESPONSE id check returned root

    Category:
    Potentially Bad Traffic

    Severity:
    2

    Direction:
    to_client

Interface:

`em2`

---

# Understanding the Traffic Direction

Although Kali initiated the HTTP request, the signature matched the response from the DMZ server.

This explains why the alert showed:

    Source:
    10.10.60.10:80

    Destination:
    10.10.10.10

Direction:

`to_client`

This is an important analytical detail.

The alert source does not always represent the original initiator of the session.

---

# Wazuh Detection

The same event reached Wazuh.

Rule:

`86601`

Description:

`Suricata: Alert - GPL ATTACK_RESPONSE id check returned root`

Status:

**PASS**

This confirmed that the final integration worked end to end.

---

# Final Detection Path

The final Suricata detection pipeline is:

    LAB-KALI-01
    10.10.10.10
          |
          v
      LAB-FW-01
       OPNsense
          |
          v
       Suricata
          |
          v
     LAB-DMZ-01
    10.10.60.10
          |
          v
    Suricata Alert
          |
          v
    OPNsense Remote Logging
          |
          | UDP/5514
          v
     LAB-SIEM-01
          |
          v
        rsyslog
          |
          v
    /var/log/suricata/eve.json
          |
          v
     Wazuh Logcollector
          |
          v
       JSON Decoder
          |
          v
      Rule 86601

---

# full_log Behaviour

During investigation, the Suricata Wazuh event showed:

`full_log: null`

This initially looked unusual.

However, the structured JSON fields were still present.

The event could be searched and investigated using:

- rule ID
- location
- structured fields

This showed that:

**An empty full_log field does not automatically mean the event is invalid.**

---

# Better Suricata Searches

For structured Suricata events, useful search criteria include:

- `rule.id:86601`
- location
- source IP
- destination IP
- signature ID
- signature text

These can be more reliable than searching only the raw log field.

---

# Live File jq Parsing Issue

Direct use of:

`jq`

against the actively written:

`/var/log/suricata/eve.json`

occasionally produced parse errors.

This could occur because:

- the file was being written live
- a partial record was read
- a malformed/partial line existed

The presence of valid Wazuh alerts proved the full pipeline was still working.

---

# Why the Pipeline Was Not Rebuilt

The project avoided unnecessary reconfiguration because:

- rsyslog was receiving data
- the file was being written
- Wazuh was analyzing the file
- Rule 86601 was firing

A single `jq` parse error therefore did not justify rebuilding a working architecture.

This reinforced:

**Use end-to-end evidence before making major changes.**

---

# Decoder Testing Lessons

The Suricata integration produced several important lessons.

## Arrival Does Not Mean Parsing

Rule 1002 proved that an event can arrive without being decoded correctly.

## Decoder Matching Matters

Wazuh needs the correct structure before rules can work reliably.

## Structured JSON Is Preferable

Where available, JSON simplifies field extraction.

## Built-In Rules Should Be Preferred

Custom wrappers should not be retained without a need.

---

# Transport Troubleshooting Lessons

The final troubleshooting model became:

    1. Confirm OPNsense generated an alert
    2. Confirm remote logging target
    3. Confirm UDP/5514 listener
    4. Confirm rsyslog receives traffic
    5. Confirm eve.json grows
    6. Confirm Wazuh logcollector monitors the file
    7. Confirm decoder
    8. Confirm rule
    9. Generate fresh test traffic

This provides a repeatable method for future integrations.

---

# Suricata and Firewall Relationship

Suricata and OPNsense firewall rules serve different purposes.

## Firewall

Determines whether traffic is allowed or blocked.

## Suricata

Inspects traffic and identifies suspicious patterns.

This means allowed traffic can still generate an alert.

The REDNET-to-DMZ HTTP path is a good example.

The firewall allowed:

`TCP/80`

while Suricata still detected suspicious content.

---

# Suricata and Nginx Correlation

The same REDNET-to-DMZ path is also monitored at the application layer.

This allows multiple views of one interaction.

Example:

    Kali HTTP Request
          |
          v
    OPNsense Firewall
          |
          v
    Suricata
          |
          v
    Nginx
          |
          v
    Wazuh Agent

This supports richer Purple Team investigations.

---

# Network and Application Evidence

For a web scenario, Wazuh may eventually contain:

- firewall event
- Suricata event
- Nginx access event
- Linux host event

This multi-layer visibility is more useful than relying on a single sensor.

---

# Current Suricata Role

Suricata now functions as the lab's network intrusion detection layer.

Its role includes:

- REDNET traffic visibility
- signature detection
- EVE JSON generation
- Wazuh integration
- attack-response validation
- future Purple Team network detections

---

# Final Validation Summary

| Validation | Result |
|---|---|
| Suricata installed on OPNsense | PASS |
| Monitored interface `em2` | PASS |
| ET Open rules active | PASS |
| SID 2100498 generated | PASS |
| Initial Rule 1002 fallback observed | PASS |
| Custom decoder testing completed | PASS |
| JSON decoder working | PASS |
| UDP/5514 listener active | PASS |
| rsyslog receiving Suricata events | PASS |
| `/var/log/suricata/eve.json` writing | PASS |
| Wazuh localfile configured | PASS |
| Wazuh logcollector analyzing file | PASS |
| Rule 86601 generated | PASS |
| REDNET-to-DMZ visibility | PASS |
| Wazuh alert correlation | PASS |

---

# Historical vs Current State

This document intentionally preserves the complete Suricata integration journey.

Historical stages include:

- no IDS
- endpoint-only monitoring
- Suricata installation
- first local alert
- initial Wazuh fallback Rule 1002
- custom decoder experimentation
- syslog wrapper parsing
- transition to JSON
- dedicated UDP/5514
- rsyslog file ingestion
- final Wazuh built-in rule processing

The authoritative current architecture is:

    Sensor:
    LAB-FW-01 / Suricata

    Interface:
    em2 / REDNET

    Remote Destination:
    10.10.40.10

    Transport:
    UDP/5514

    Receiver:
    rsyslog

    File:
    /var/log/suricata/eve.json

    Wazuh Format:
    json

    Validated Wazuh Rule:
    86601

---

# Evolution Summary

The Suricata integration evolved through:

    No IDS
       |
       v
    OPNsense Suricata Installation
       |
       v
    REDNET Monitoring
       |
       v
    First Suricata Alert
       |
       v
    Wazuh Rule 1002 Fallback
       |
       v
    Decoder Troubleshooting
       |
       v
    Custom Decoder
       |
       v
    JSON Parsing
       |
       v
    Dedicated UDP/5514
       |
       v
    rsyslog
       |
       v
    eve.json
       |
       v
    Wazuh Rule 86601
       |
       v
    Final REDNET-to-DMZ IDS Pipeline

---

# Final Status

The Suricata-to-Wazuh integration is complete.

The final environment provides:

- network IDS visibility
- REDNET monitoring
- EVE JSON telemetry
- dedicated Suricata transport
- centralized SIEM ingestion
- structured JSON parsing
- built-in Wazuh Suricata detection
- validated REDNET-to-DMZ visibility

The integration is now considered stable baseline infrastructure.

Future work should focus on:

- controlled attack simulations
- additional IDS signatures
- alert triage
- false-positive tuning
- correlation with endpoint logs
- MITRE ATT&CK mapping
- Purple Team detection validation

rather than rebuilding the underlying Suricata transport or parser architecture without a specific operational reason.
