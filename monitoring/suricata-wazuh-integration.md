# Suricata IDS → Wazuh SIEM Integration

## 1. Overview

This document describes the implementation and validation of Suricata IDS monitoring within the Purple Team CyberLab.

Suricata runs on the OPNsense firewall and monitors REDNET traffic. Suricata generates EVE JSON security events, which are forwarded to LAB-SIEM-01 using a dedicated syslog collection path.

The events are then ingested by Wazuh for centralized detection, investigation, and threat hunting.

---

## 2. Architecture

The final event pipeline is:

LAB-KALI-01 / REDNET
        |
        | Network traffic
        v
OPNsense Firewall
        |
        | Suricata IDS
        | EVE JSON
        v
UDP/5514
        |
        v
LAB-SIEM-01
        |
        | rsyslog
        v
/var/log/suricata/eve.json
        |
        | Wazuh Logcollector
        v
JSON Decoder
        |
        v
Wazuh Suricata Rules
        |
        v
Wazuh Alerts / Threat Hunting Dashboard

---

## 3. Relevant Network Information

### REDNET

Network:

10.10.10.0/24

Test endpoint:

LAB-KALI-01
10.10.10.10

Purpose:

Controlled offensive security and attack simulation network.

### SOCNET

Network:

10.10.40.0/24

OPNsense SOCNET interface:

10.10.40.1

Wazuh SIEM:

LAB-SIEM-01
10.10.40.10

---

## 4. Suricata Configuration

Suricata IDS is installed on OPNsense.

The monitored network/interface corresponds to REDNET.

ET Open rules are installed and enabled.

Suricata EVE JSON output is enabled.

Remote Suricata logging is configured with:

Application:

suricata

Severity:

info

Transport:

UDP

Destination:

10.10.40.10

Port:

5514

UDP/5514 is dedicated specifically to Suricata telemetry.

Existing OPNsense firewall logging remains independent from the Suricata collection path.

---

## 5. LAB-SIEM-01 Remote Collector

LAB-SIEM-01 uses rsyslog to receive Suricata events on UDP/5514.

Collector configuration:

/etc/rsyslog.d/30-suricata-remote.conf

Configuration:

module(load="imudp")

template(name="SuricataEVE" type="string" string="%msg%\n")

ruleset(name="suricata5514") {
    action(
        type="omfile"
        file="/var/log/suricata/eve.json"
        template="SuricataEVE"
        fileOwner="syslog"
        fileGroup="wazuh"
        fileCreateMode="0640"
    )
    stop
}

input(
    type="imudp"
    port="5514"
    ruleset="suricata5514"
)

The configuration was validated using:

sudo rsyslogd -N1

UDP/5514 listener verification:

sudo ss -lunp | grep ':5514'

rsyslog successfully listens on:

0.0.0.0:5514

and

[::]:5514

---

## 6. Wazuh Collection

Wazuh monitors the Suricata EVE JSON file directly.

The following localfile configuration was added to:

/var/ossec/etc/ossec.conf

Configuration:

<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>

Configuration validation was performed using:

sudo /var/ossec/bin/wazuh-logcollector -t

The configuration passed validation successfully.

The Wazuh manager was then restarted.

---

## 7. Detection Validation

A controlled Suricata detection test was performed from LAB-KALI-01 using:

curl http://testmyids.com

The HTTP response contained:

uid=0(root) gid=0(root) groups=0(root)

Suricata generated the expected alert:

Signature ID:

2100498

Signature:

GPL ATTACK_RESPONSE id check returned root

Category:

Potentially Bad Traffic

Severity:

2

Example event characteristics:

event_type:

alert

Source:

217.160.0.187:80

Destination:

10.10.10.10

Protocol:

TCP

Application protocol:

HTTP

Direction:

to_client

Monitored interface:

em2

---

## 8. Wazuh Detection

The EVE JSON event was successfully decoded using Wazuh's built-in JSON decoder.

The event matched the built-in Suricata rule chain.

Generated Wazuh alert:

Rule ID:

86601

Level:

3

Description:

Suricata: Alert - GPL ATTACK_RESPONSE id check returned root

Groups:

ids
suricata

Decoder:

json

Event source:

/var/log/suricata/eve.json

The alert was successfully written to:

/var/ossec/logs/alerts/alerts.json

---

## 9. Dashboard Verification

The alert was successfully verified in the Wazuh Threat Hunting interface.

Search:

rule.id:86601

The dashboard displayed multiple controlled test detections.

Relevant fields included:

agent.name = lab-siem-01

data.event_type = alert

data.alert.signature_id = 2100498

data.alert.signature = GPL ATTACK_RESPONSE id check returned root

data.alert.category = Potentially Bad Traffic

data.dest_ip = 10.10.10.10

data.proto = TCP

The dashboard therefore confirmed successful end-to-end indexing and visualization.

---

## 10. Detection Pipeline Validation

The following stages were individually verified:

1. Kali generated controlled HTTP traffic.

2. Suricata inspected REDNET traffic.

3. Suricata detected SID 2100498.

4. EVE JSON was generated locally on OPNsense.

5. OPNsense transmitted Suricata events to:

   10.10.40.10:5514/UDP

6. LAB-SIEM-01 received UDP/5514 traffic.

7. rsyslog wrote the event to:

   /var/log/suricata/eve.json

8. Wazuh Logcollector read the JSON file.

9. Wazuh JSON decoder parsed the Suricata fields.

10. Built-in Wazuh rule 86601 generated an alert.

11. The event appeared in alerts.json.

12. The event appeared in the Wazuh Threat Hunting dashboard.

The Suricata → Wazuh integration is therefore considered operational.

---

## 11. Troubleshooting Lessons Learned

An initial architecture attempted to send Suricata EVE JSON directly to Wazuh's remote syslog listener on UDP/514.

Network packet captures proved that the Suricata syslog packets reached LAB-SIEM-01.

However, the events arrived with a traditional syslog wrapper surrounding the EVE JSON.

Example structure:

<PRI>timestamp hostname suricata[PID]: {EVE JSON}

Wazuh's JSON decoder successfully decoded the raw EVE JSON but did not automatically decode the JSON when it was embedded inside this syslog wrapper.

Custom decoder testing confirmed that the underlying JSON and built-in Suricata rule set were functioning correctly.

A dedicated rsyslog collection architecture was therefore implemented.

The final architecture provides several advantages:

- Preserves raw IDS telemetry.
- Separates Suricata IDS logging from general firewall logging.
- Simplifies troubleshooting.
- Allows log replay and forensic review.
- Allows Wazuh to consume native JSON directly.
- Avoids unnecessary custom decoding rules.
- Uses Wazuh's existing Suricata detection rules.

A temporary Suricata syslog wrapper decoder created during troubleshooting was retired after the final architecture was validated.

The final environment therefore relies on Wazuh's built-in JSON decoder and Suricata rules rather than unnecessary custom parsing.

---

## 12. Suricata Tuning Review

An initial alert-frequency review was performed using:

sudo grep -E '^[[:space:]]*\{' /var/log/suricata/eve.json | \
jq -r 'select(.event_type=="alert") |
"\(.alert.signature_id)\t\(.alert.signature)"' | \
sort | uniq -c | sort -nr | head -20

Observed result:

6 2100498 GPL ATTACK_RESPONSE id check returned root

All observed alerts were generated intentionally during integration testing.

No unrelated noisy signatures were observed.

Therefore:

No suppression rules were implemented.

No thresholding was implemented.

No Suricata signatures were disabled.

This decision preserves detection capability until sufficient real telemetry exists to justify tuning.

Future IDS tuning will be performed during Purple Team simulations using evidence such as alert frequency, false-positive rate, asset relevance, and detection value.

---

## 13. Operational Status

Suricata IDS:

OPERATIONAL

EVE JSON:

OPERATIONAL

OPNsense → SIEM forwarding:

OPERATIONAL

rsyslog collector:

OPERATIONAL

Wazuh JSON ingestion:

OPERATIONAL

Wazuh Suricata rules:

OPERATIONAL

Threat Hunting dashboard:

OPERATIONAL

Test SID 2100498:

VERIFIED

Suricata → Wazuh integration:

COMPLETE

---

## 14. Next Phase

The next CyberLab implementation phase is Linux endpoint monitoring.

Planned work includes:

- Linux Wazuh agent deployment
- SSH monitoring
- Authentication monitoring
- sudo activity
- Auditd
- File Integrity Monitoring
- Service and system monitoring
- Linux attack simulation
- Detection verification
- Dashboard investigation
- Documentation

After Linux monitoring, the infrastructure roadmap continues with:

1. Active Directory

   AD-NET
   10.10.50.0/24

2. DMZ

   10.10.60.0/24

3. Final infrastructure validation

4. Transition from build tutorials to realistic SOC and Purple Team operational simulations
