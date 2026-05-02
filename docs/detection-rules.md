# Detection Rules

## Overview
This file documents the detection logic used in the Elastic SIEM home lab.  
The lab is designed to identify suspicious network activity captured by Packetbeat and visualised in Kibana, with a focus on common SOC-relevant behaviours such as brute-force attempts, scanning activity, insecure protocol usage, and network reconnaissance.

The detections in this lab are based on traffic generated between the Kali attacker VM and the Ubuntu-based Elastic SIEM server.  
Packetbeat was configured to monitor protocols including SSH, FTP, Telnet, HTTP and DNS.

## Detection Cases
| Rule Name | Suspicious Behavior | Data Source | Example Query | Analyst Action |
|---|---|---|---|---|
| SSH brute force | Multiple failed SSH logins from one source | filebeat + packetbeat | ... | Check source IP, count failures, block if needed |
| Nmap scan | Many connection attempts across ports in a short time | packetbeat | ... | Confirm scan, isolate source |
| Telnet access | Telnet traffic to port 23 | packetbeat | ... | Validate if Telnet is approved |
| FTP access | Legacy FTP traffic to port 21 | packetbeat | ... | Check if file transfer is expected |
| ICMP sweep | Repeated ICMP echo requests to multiple hosts | packetbeat | ... | Verify recon activity |

## Rule Details

### 1. SSH Brute Force Attempts
**Threat description:**  
Repeated failed SSH login attempts may indicate brute-force activity against our Linux server.

**Why it matters:**  
SSH is a common remote administration protocol and a frequent target for attackers trying to gain initial access.
...
# SSH brute force
event.dataset: "system.auth" and message: ("Failed password" or "authentication failure")
**Relevant fields:**  
- `event.dataset`
- `source.ip`
- `destination.ip`
- `destination.port`
- `network.transport`

**Kibana query:**  
```kql
# 1. All SSH traffic (baseline)
destination.port: 22 AND agent.type: packetbeat

# 2. Failed SSH logins only (brute force hunting)
destination.port: 22 AND ssh.auth_success: false

# 3. Failed logins from single IP (>5 attempts)
destination.port: 22 AND ssh.auth_success: false 
| stats count() by source.ip | where count > 5

# 4. Successful SSH logins (legit access)
destination.port: 22 AND ssh.auth_success: true

```
**What to look for:**  
- Multiple connection attempts from the same source IP
- Repeated failures over a short time window
- Connections targeting the SIEM or monitored Linux host

**Analyst response:**  
- Identify the source IP generating repeated attempts
- Check whether the activity was expected lab testing or unknown traffic
- Review surrounding activity from the same IP
- Consider blocking or isolating the source if this were a live environment
- 
### SSH Traffic Monitoring (Port 22)

**What Packetbeat captures:**
- Connection attempts (source/destination IP, ports)
- Authentication status (success/fail)
- Flow duration, packet/byte counts
- Bi-directional traffic stats

**Key fields in Kibana:**
| Field | Description | Example |
|-------|-------------|---------|
| `destination.port` | SSH service port | 22 |
| `source.ip` | Attacker IP (Kali VM) | 10.0.1.5 |
| `destination.ip` | Target host | 10.0.1.7 |
| `network.transport` | Protocol layer | tcp |
| `ssh.method` | Auth method | password |
| `ssh.auth_success` | Login result | false |
| `flow.bytes` | Total bytes transferred | 1234 |
| `flow.packets` | Packets sent/received | 15/12 |

# NMAP Scan

# FTP
destination.port: 21

# Telnet
destination.port: 23

# ICMP
network.protocol: "icmp"
