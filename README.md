# Wazuh SOC Simulation Lab — Threat Detection & Incident Monitoring

## Overview
This project is a research-based Security Operations Center (SOC) 
simulation built to explore real-world threat detection using Wazuh 
SIEM/XDR in a controlled virtualized lab environment.

The lab consists of three virtual machines:
- **Kali Linux** — Simulated attacker machine
- **Ubuntu 2** — Target system running Wazuh Agent, Apache, 
  MariaDB, and DVWA
- **Ubuntu 1** — Monitoring server running Wazuh Manager, 
  OpenSearch, and Dashboard

## Scenarios Covered
| # | Scenario | Category |
|---|----------|----------|
| 1 | Operation StaleBase — DB Exfiltration | Data Theft |
| 2 | CloudShadow — Credential File Exposure | Cloud Security |
| 3 | GhostEnum — Insider Enumeration & Lateral Movement | Insider Threat |

## Attack Techniques Simulated
- SSH & MySQL brute force
- SQL Injection via DVWA
- Remote Code Execution (web shell upload)
- Database dump via `INTO OUTFILE`
- Sensitive credential file discovery
- Internal network enumeration
- Persistence via crontab

## Wazuh Detection Modules Used
- File Integrity Monitoring (FIM)
- Log Data Analysis (Apache, Auth, MySQL)
- Auditd Integration
- Custom Correlation Rules
- Active Response

## Purpose
This project is purely for educational and research purposes, 
demonstrating defensive security techniques, SOC workflows, 
and the effectiveness of open-source SIEM tooling in detecting 
common attack patterns mapped to MITRE ATT&CK.

## Tools & Technologies
`Wazuh` `OpenSearch` `DVWA` `MariaDB` `Apache2` 
`Kali Linux` `Ubuntu 22.04` `Hydra` `Nmap` `Auditd`
