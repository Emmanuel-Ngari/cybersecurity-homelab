# Purple Team Scenario Template

## Scenario Information

**Scenario ID:** `SCN-XXX`

**Scenario Title:**  
`<Short descriptive title>`

**Date:**  
`YYYY-MM-DD`

**Status:**  
`Planned / In Progress / Completed`

**Author:**  
`<Name or analyst identifier>`

---

# Objective

Describe the purpose of the scenario.

Explain:

- what behaviour is being simulated
- why the behaviour matters
- what defensive capability is being tested

Example objective:

Evaluate whether suspicious PowerShell reconnaissance performed from a domain workstation generates sufficient telemetry for SOC detection and investigation.

---

# Attack Hypothesis

State the assumption being tested.

Example:

If an attacker performs PowerShell-based reconnaissance from a compromised Windows workstation, then Sysmon, PowerShell logging, Windows Security auditing, and Wazuh should provide enough evidence to identify and investigate the activity.

---

# Environment

## Source System

**Hostname:**  
`<Source Host>`

**IP Address:**  
`<Source IP>`

**Zone:**  
`<Source Zone>`

**Role:**  
`Attacker / User / Administrator / Server`

---

## Target System

**Hostname:**  
`<Target Host>`

**IP Address:**  
`<Target IP>`

**Zone:**  
`<Target Zone>`

**Role:**  
`<Target Role>`

---

# MITRE ATT&CK Mapping

**Tactic:**  
`<Tactic>`

**Technique:**  
`<Technique ID> — <Technique Name>`

**Sub-Technique:**  
`<If applicable>`

Reference the most appropriate ATT&CK technique for the simulated behaviour.

Avoid forcing a technique mapping when the activity does not clearly match one.

---

# Scenario Scope

This scenario is authorized only within the Cybersecurity Home Lab.

In-scope systems:

- `<System 1>`
- `<System 2>`

Out-of-scope systems:

- public infrastructure
- third-party systems
- systems outside the controlled lab

---

# Prerequisites

List the conditions that must be true before execution.

Examples:

- required VMs powered on
- correct network interfaces active
- Wazuh agents connected
- Suricata operational
- required firewall paths validated
- target service running
- baseline snapshots available

---

# Snapshot / Change Control

Before significant changes:

1. Confirm the current known-good state.
2. Create a VirtualBox snapshot if the scenario requires configuration changes.
3. Record the snapshot name.
4. Proceed with the test.
5. Validate the final state.
6. Remove temporary changes when appropriate.

**Snapshot Name:**  
`<Snapshot name or N/A>`

---

# Baseline State

Document the environment before generating attack activity.

Confirm items such as:

- relevant services are active
- Wazuh agents are Active
- no matching alerts already exist in the test window
- target connectivity behaves as expected
- logging sources are functioning

The baseline should make it possible to distinguish scenario activity from earlier events.

---

# Attack Execution

## Step 1

**Action:**

Describe the controlled action.

**Source:**

`<Host>`

**Target:**

`<Host>`

**Command / Procedure:**

    <Command or action>

**Expected Result:**

Describe expected attacker-side behaviour.

---

## Step 2

**Action:**

`<Describe next action>`

**Command / Procedure:**

    <Command or action>

**Expected Result:**

`<Expected result>`

---

# Expected Telemetry

Before reviewing alerts, document what telemetry should exist if monitoring is working correctly.

Possible sources:

- Windows Security
- Sysmon
- PowerShell
- Defender
- Windows Firewall
- Linux authentication logs
- auditd
- OPNsense firewall logs
- Suricata
- Nginx
- Wazuh FIM

Expected telemetry:

| Source | Expected Event | Reason |
|---|---|---|
| `<Telemetry Source>` | `<Expected event>` | `<Why it should appear>` |
| `<Telemetry Source>` | `<Expected event>` | `<Why it should appear>` |

---

# Observed Telemetry

Record what actually appeared.

| Source | Observed Event | Result |
|---|---|---|
| `<Telemetry Source>` | `<Observed event>` | PASS / FAIL |
| `<Telemetry Source>` | `<Observed event>` | PASS / FAIL |

Do not mark a source as failed until timestamps, ingestion status, and collection paths have been checked.

---

# Detection Results

## Wazuh

**Rule ID:**  
`<Rule ID or N/A>`

**Rule Description:**  
`<Description>`

**Agent:**  
`<Agent Name>`

**Severity / Level:**  
`<Level if relevant>`

**Result:**  
`PASS / FAIL / PARTIAL`

---

## Suricata

**Signature ID:**  
`<SID or N/A>`

**Signature:**  
`<Signature>`

**Category:**  
`<Category>`

**Severity:**  
`<Severity>`

**Result:**  
`PASS / FAIL / N/A`

---

## Firewall

**Rule / Policy:**  
`<Firewall rule>`

**Observed Action:**  
`ALLOW / BLOCK / N/A`

**Result:**  
`PASS / FAIL`

---

# SOC Investigation

Treat this section as if the alert was received during real SOC operations.

Investigate:

- who initiated the activity
- source IP
- destination IP
- affected host
- affected user
- process
- command line
- parent process
- network destination
- timestamp
- related alerts
- surrounding events
- whether the activity appears legitimate or suspicious

---

# Analyst Timeline

| Time | Event | Source |
|---|---|---|
| `<Time>` | `<Event>` | `<Log source>` |
| `<Time>` | `<Event>` | `<Log source>` |
| `<Time>` | `<Event>` | `<Log source>` |

Build the timeline from actual telemetry rather than from the execution steps alone.

---

# Investigation Findings

Summarize what the analyst determined.

Include:

- initial activity
- host involved
- account involved
- relevant processes
- network activity
- security-control response
- whether further activity occurred

---

# Analyst Assessment

**Classification:**  
`Benign Test / Suspicious / Malicious Simulation`

**Confidence:**  
`Low / Medium / High`

**Impact:**  
`<Assessment>`

**Scope:**  
`<Single host / Multiple hosts / Network / Identity / Application>`

---

# Detection Gap Analysis

Determine whether the existing controls provided enough visibility.

Questions to answer:

- Did the expected event exist?
- Was it ingested?
- Was it decoded correctly?
- Did an alert fire?
- Was the alert specific enough?
- Was important context missing?
- Was there too much noise?
- Could an analyst understand what happened from the available telemetry?

---

# Identified Gaps

Document each gap separately.

## Gap 1

**Description:**  
`<Detection or visibility gap>`

**Impact:**  
`<Why it matters>`

**Root Cause:**  
`<Logging / ingestion / decoding / rule / tuning / firewall / other>`

---

# Detection Improvement

Document changes made as a result of the scenario.

Possible improvements include:

- new Wazuh rule
- rule tuning
- severity adjustment
- additional event collection
- Sysmon tuning
- auditd rule
- PowerShell logging change
- Suricata tuning
- firewall logging change
- dashboard query
- correlation logic

Do not make unnecessary infrastructure changes if the existing controls already provide sufficient visibility.

---

# Retest

After any improvement, repeat the relevant activity.

Record:

**Retest Date:**  
`YYYY-MM-DD`

**Expected Result:**  
`<Expected result after improvement>`

**Observed Result:**  
`<Observed result>`

**Status:**  
`PASS / FAIL`

---

# Evidence

Only include evidence that directly supports the scenario conclusions.

Recommended evidence categories:

- attacker-side activity
- firewall decision
- endpoint telemetry
- SIEM event
- IDS alert
- investigation query
- final detection

Example naming:

    SCN-001-01-attack-execution.png
    SCN-001-02-sysmon-event.png
    SCN-001-03-wazuh-alert.png
    SCN-001-04-investigation.png

---

# Evidence Register

| Evidence ID | Description | File |
|---|---|---|
| `E01` | `<Description>` | `<Filename>` |
| `E02` | `<Description>` | `<Filename>` |

---

# Final Result

**Scenario Outcome:**  
`PASS / PARTIAL / FAIL`

Summarize whether the lab successfully:

- generated the intended activity
- collected the expected telemetry
- detected the behaviour
- supported analyst investigation
- identified any gaps
- improved the defensive control
- passed the retest

---

# Lessons Learned

Document the main technical and operational lessons.

Focus on:

- what worked
- what failed
- what was misleading
- how troubleshooting was performed
- what would be done differently in a real environment

---

# Purple Team Summary

Summarize the scenario using the Purple Team cycle:

    Attack
       |
       v
    Telemetry
       |
       v
    Detection
       |
       v
    Investigation
       |
       v
    Gap Analysis
       |
       v
    Improvement
       |
       v
    Retest

---

# Scenario Status

**Final Status:**  
`Completed / Requires Follow-Up`

**Next Action:**  
`<Next step or N/A>`
