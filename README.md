# SIEM-Implementation-and-Log-Analysis
Built a VirtualBox-based SIEM lab using Elastic to collect host and network telemetry, monitor traffic, and detect suspicious activity such as Nmap scans and failed logins. The project shows practical skills in log ingestion, dashboarding, alert review, and basic incident triage.

## Objective

To build and document a VirtualBox-based Elastic SIEM lab that collects host and network telemetry, monitors traffic, and demonstrates basic threat detection and investigation skills through practical alerts and dashboarding. 

## Skills

Hands-on VirtualBox SIEM lab project demonstrating Elastic-based log collection, network traffic monitoring, alert review, and threat detection. Showcases practical blue team skills in SIEM configuration, traffic analysis, and technical documentation.

## Tools
<div>
    <img src="https://img.shields.io/badge/-VirtualBox-183A61?&style=for-the-badge&logo=virtualbox&logoColor=white" />
    <img src="https://img.shields.io/badge/-Elastic-005571?&style=for-the-badge&logo=Elastic&logoColor=white" />
    <img src="https://img.shields.io/badge/-Wireshark-1679A7?&style=for-the-badge&logo=Wireshark&logoColor=white" />
    <img src="https://img.shields.io/badge/-Nmap-0078D7?&style=for-the-badge&logo=nmap&logoColor=white" />
    <img src="https://img.shields.io/badge/-pfSense-212121?&style=for-the-badge&logo=pfsense&logoColor=white" />
    <img src="https://img.shields.io/badge/-Kali%20Linux-268BFF?style=for-the-badge&logo=kali-linux&logoColor=white" />
</div>

## Lab Setup
<div>
- Kali VM used as traffic generator
- Target VM used as monitored endpoint
- Elastic used for log ingestion, dashboards, and alerts

## Data Collected
- Host logs
- Authentication events
- Network flow / packet-derived events

## Detection Use Cases
- Nmap scan detection
- Failed login monitoring
- Unusual outbound traffic review

- ## Key Outcomes
- Ingested telemetry from endpoint and network sources
- Built dashboards for traffic visibility
- Triggered and reviewed alerts
- Documented investigation workflow

## Screenshots
(Add screenshots here)

## Lessons Learned
- Packet and host data together give better context
- Clean documentation matters as much as the setup
- Small, repeatable detections are best for a first portfolio project
