# Lab Overview

## Project Goal
Build a realistic SOC monitoring environment in VirtualBox to demonstrate Elastic SIEM deployment, network traffic analysis, and threat detection using real-world tools and scenarios.

## Architecture
Kali Linux → Ubuntu Server → Elastic SIEM
(Attacker) (Monitored) (Cloud/Local)

Nmap scans ←→  Elastic Agent ←→  Dashboards

SSH brute force - Packetbeat - Alerts

HTTP traffic - System logs - Investigations


## Components

| Role | VM/OS | Tools | Purpose |
|------|-------|-------|---------|
| **Attacker** | Kali Linux | Nmap, SSH clients | Generate detectable traffic |
| **Target** | Ubuntu 22.04 | Elastic Agent, Packetbeat | Generate telemetry |
| **SIEM** | Elastic | Kibana, Detection Rules | Collect, analyze, alert |


</div>

<img width="835" height="727" alt="Image" src="https://github.com/user-attachments/assets/bdee1a2c-9226-4467-8421-8bf1123f6039" />

## Data Sources
- **Host telemetry**: System logs, process activity, authentication events
- **Network flows**: Packetbeat capturing protocols, ports, and traffic volume
- **Test scenarios**: Nmap scans, failed logins, application traffic

## Detection Scenarios
- **Port scanning** - Nmap SYN scan detection via Packetbeat flows
- **Auth failures** - Multiple SSH login attempts in host logs
- **Traffic anomalies** - Unusual port/protocol combinations

## Learning Outcomes
- Deploy production-grade SIEM with multiple data sources
- Correlate host and network events during investigations
- Build custom dashboards for SOC monitoring
- Document analyst workflows for common alerts

## Why This Lab?
I decided to use this lab as it simulates a real SOC analyst workflow: **collect → detect → investigate → document**. Uses enterprise tools (Elastic Stack) in a controlled environment to prove practical blue-team skills.
