# Wazuh SIEM Deployment

## 1. Overview

Wazuh is deployed as the central Security Information and Event Management (SIEM) and endpoint monitoring platform in the cybersecurity homelab.

The SIEM server is located inside a dedicated SOC network and receives security telemetry from systems located in other security zones through controlled OPNsense firewall rules.

This deployment currently monitors:

- LAB-WIN-01
- LAB-UBUNTU-01

The environment is designed to support future blue-team activities including:

- Endpoint monitoring
- Security event analysis
- Vulnerability detection
- Security Configuration Assessment
- Threat detection
- Incident investigation
- Centralized log collection
- Attack-and-detection exercises

---

## 2. SIEM Server

Hostname: `lab-siem-01`

Virtual machine name: `LAB-SIEM-01`

Operating system: Ubuntu Server 26.04.1 LTS

Network zone: `SOCNET`

IP address: `10.10.40.10/24`

Default gateway: `10.10.40.1`

DNS server: `10.10.40.1`

Virtual network:

`SOC-NET`

Approximate allocated memory:

`8 GB`

Root filesystem capacity after LVM configuration:

Approximately `53 GB`

---

## 3. Wazuh Deployment

Wazuh version:

`4.14.7`

The Wazuh all-in-one deployment includes:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

The deployment was performed using the official Wazuh installation script.

The installer produced an operating-system compatibility warning because Ubuntu 26.04 was not included in the listed recommended Ubuntu releases.

Because this is an isolated cybersecurity laboratory environment, installation was continued and the deployment was tested before proceeding.

All major Wazuh components were confirmed operational.

---

## 4. Wazuh Network Architecture

The Wazuh server resides on:

`SOCNET - 10.10.40.0/24`

The SIEM server address is:

`10.10.40.10`

Monitored systems communicate with Wazuh through OPNsense.

Current architecture:

`LAB-WIN-01 -> CORPNET -> OPNsense -> SOCNET -> LAB-SIEM-01`

`LAB-UBUNTU-01 -> SERVERNET -> OPNsense -> SOCNET -> LAB-SIEM-01`

The monitoring network remains separated from the endpoint and server networks.

---

## 5. Wazuh Communication Ports

The following Wazuh communication paths are currently used:

- TCP 1514 - Wazuh agent communication
- TCP 1515 - Wazuh agent enrollment/registration
- TCP 443 - Wazuh Dashboard HTTPS
- TCP 9200 - Local Wazuh Indexer/OpenSearch service

TCP 9200 is used internally by Wazuh components and was verified listening locally on the SIEM server.

---

## 6. OPNsense Integration

OPNsense controls communication between the monitored security zones and SOCNET.

Firewall aliases were created for the Wazuh infrastructure, including aliases representing:

- Wazuh server
- Wazuh agent communication ports
- Wazuh Dashboard service

Controlled rules permit authorized monitoring traffic while maintaining network segmentation.

Examples include:

`LAB-WIN-01 -> LAB-SIEM-01 TCP 1514/1515`

`LAB-UBUNTU-01 -> LAB-SIEM-01 TCP 1514/1515`

`ADMIN_WORKSTATION -> LAB-SIEM-01 TCP 443`

Broad unrestricted communication between the security zones is not required for endpoint monitoring.

---

## 7. Windows Endpoint

Agent name:

`LAB-WIN-01`

IP address:

`10.10.20.20`

Security zone:

`CORPNET`

Operating system:

Microsoft Windows 11 Enterprise

Wazuh agent version:

`4.14.7`

The Wazuh agent was installed and configured to communicate with:

`10.10.40.10`

The Windows service was verified running.

Connectivity from the Windows endpoint to the Wazuh Dashboard was tested using PowerShell:

`Test-NetConnection 10.10.40.10 -Port 443`

Result:

`TcpTestSucceeded : True`

The endpoint was verified in the Wazuh Dashboard with status:

`Active`

Wazuh telemetry from the Windows endpoint includes information such as:

- System inventory
- Security Configuration Assessment
- Vulnerability information
- Security events

---

## 8. Ubuntu Endpoint

Agent name:

`LAB-UBUNTU-01`

IP address:

`10.10.30.30`

Security zone:

`SERVERNET`

Operating system:

Ubuntu 26.04.1 LTS

Wazuh agent version:

`4.14.7`

The Ubuntu Wazuh agent was configured to communicate with:

`10.10.40.10`

The agent successfully completed registration with the Wazuh manager.

Agent logs confirmed:

- Authentication key request
- Authentication key received
- Connection attempt to `10.10.40.10:1514`
- Successful TCP connection to the Wazuh server

The service was enabled for automatic startup.

After a complete shutdown and restart of the lab, the following command was used:

`sudo systemctl is-active wazuh-agent`

Result:

`active`

The endpoint was subsequently verified in the Wazuh Dashboard with status:

`Active`

---

## 9. SERVERNET Firewall Troubleshooting

During Ubuntu agent deployment, communication from:

`10.10.30.30`

to:

`10.10.40.10`

on TCP ports `1514` and `1515` initially failed.

OPNsense Live View showed the traffic being matched by an unintended blocking rule.

The firewall policy was reviewed and corrected.

Connectivity was then retested using:

`nc -zv -w 5 10.10.40.10 1514`

and:

`nc -zv -w 5 10.10.40.10 1515`

Both tests succeeded after the firewall correction.

This demonstrated the value of firewall logging when troubleshooting segmented network environments.

---

## 10. Ubuntu Repository Troubleshooting

During installation of the Ubuntu Wazuh agent, several repository-related issues were encountered.

These included:

- Initial Wazuh GPG key import failure
- Intermittent DNS resolution problems
- Incorrect keyring path caused by a typing error
- Difficulty retrieving the Wazuh repository key

IPv4 connectivity was tested explicitly.

The Wazuh signing key was successfully retrieved and verified.

The repository was configured using:

`/usr/share/keyrings/wazuh.gpg`

The configured repository was:

`deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main`

APT repository access was subsequently verified successfully.

---

## 11. Host Disk-Full Incident

During operation of the cybersecurity lab, the physical VirtualBox host reached zero available disk space.

VirtualBox suspended running virtual machines and reported:

`DrvVD_DISKFULL`

The affected virtual machines included the SIEM and Windows systems.

Investigation of the host identified a large approximately 45.7 GB archive in the Downloads directory.

The unnecessary archive was removed.

Host free space increased to approximately:

`45.7 GB`

VirtualBox virtual disk files, snapshot files and VM configuration files were not manually deleted.

---

## 12. Wazuh Indexer Recovery

Following the VirtualBox disk-full incident, the Wazuh Dashboard displayed:

`Wazuh dashboard server is not ready yet`

Dashboard logs showed connection failures to:

`127.0.0.1:9200`

The Wazuh Indexer was not initially listening on TCP 9200.

Service investigation showed that the indexer had previously experienced startup timeout behavior while OpenSearch was still initializing.

Indexer logs were reviewed.

The logs showed normal OpenSearch module and plugin initialization activity and did not show clear evidence of index corruption.

No Wazuh data directories were deleted.

No index data was manually modified.

No systemd timeout override was applied.

The lab was cleanly shut down and restarted.

After the clean restart:

`wazuh-indexer.service`

reported:

`active (running)`

TCP port 9200 was verified using:

`sudo ss -lntp | grep ':9200'`

The output confirmed that the Java/OpenSearch process was listening on:

`127.0.0.1:9200`

The Wazuh Dashboard service was also verified:

`wazuh-dashboard.service`

Status:

`active (running)`

The dashboard then became accessible again through HTTPS.

---

## 13. Final Agent Verification

After recovery, both monitored endpoints were checked from the Wazuh Dashboard.

Verified endpoints:

| Agent | IP Address | Zone | Status |
| --- | --- | --- | --- |
| LAB-WIN-01 | 10.10.20.20 | CORPNET | Active |
| LAB-UBUNTU-01 | 10.10.30.30 | SERVERNET | Active |

Dashboard summary:

- Active agents: 2
- Disconnected agents: 0
- Pending agents: 0

This confirms successful communication across:

`CORPNET -> OPNsense -> SOCNET`

and:

`SERVERNET -> OPNsense -> SOCNET`

---

## 14. Recovery and Change Management

Snapshots were used before significant configuration and recovery activities.

Important recovery snapshots include:

`LAB-SIEM-01 - Wazuh + Windows Agent Verified`

`SIEM - Pre Indexer Timeout Recovery`

`SIEM - Wazuh Windows Ubuntu Agents Verified`

Snapshots provide rollback points before significant changes are introduced.

Sensitive configuration files, credentials, private keys and OPNsense configuration backups are not stored publicly in the GitHub repository.

---

## 15. Current SIEM Status

Current status:

- Wazuh Manager: Operational
- Wazuh Indexer: Operational
- Wazuh Dashboard: Operational
- Filebeat: Operational
- Windows agent: Active
- Ubuntu agent: Active
- CORPNET monitoring path: Verified
- SERVERNET monitoring path: Verified
- SOCNET segmentation: Operational
- Dashboard HTTPS access: Verified
- Wazuh agent enrollment: Verified

The Wazuh SIEM deployment is now operational and integrated into the segmented cybersecurity homelab.

---

## 16. Planned Improvements

Future defensive-security improvements include:

- Microsoft Sysmon deployment
- Enhanced Windows event collection
- Linux auditing
- OPNsense log forwarding to Wazuh
- Suricata IDS deployment
- Network intrusion monitoring
- Active Directory monitoring
- Vulnerability laboratory monitoring
- Custom Wazuh detection rules
- MITRE ATT&CK mapping
- Attack simulation and detection validation
- Incident-response exercises
- SOC investigation documentation

The objective is to evolve the environment from basic endpoint monitoring into a complete attack, detection, investigation and response laboratory.
