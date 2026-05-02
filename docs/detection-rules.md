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

<img width="1850" height="871" alt="image" src="https://github.com/user-attachments/assets/917c8ff1-993f-4314-aa7a-d430c6e65451" />

<img width="1851" height="926" alt="image" src="https://github.com/user-attachments/assets/2ab680ca-36e2-4560-baca-d93506ca2d25" />


**Analyst response:**  
- Identify the source IP generating repeated attempts
- Check whether the activity was expected lab testing or unknown traffic
- Review surrounding activity from the same IP
- Consider blocking or isolating the source if this were a live environment

<img width="1834" height="776" alt="image" src="https://github.com/user-attachments/assets/3cf182f6-b711-4292-8801-c609e264bb65" />


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

**Below is an example of a successful login for refrence**

<img width="1852" height="951" alt="image" src="https://github.com/user-attachments/assets/6a6b1cd3-028c-4b43-9598-5f8b9888225b" />


## Nmap Scan Detection (Network Reconnaissance)

**What Packetbeat captures:**
- TCP SYN/ACK scans (half-open connections)
- Multiple port probes from single source
- High connection rates to target host
- Stealth scan patterns (low/rare ports)

**Why it matters:**  
Port scanning is typically the first step in reconnaissance (MITRE ATT&CK T1046: Network Service Discovery). Detects attackers mapping your network.

**Key fields in Kibana:**
| Field | Description | Example |
|-------|-------------|---------|
| `source.ip` | Scanner IP (Kali) | 10.0.1.5 |
| `destination.ip` | Target host | 10.0.1.7 |
| `destination.port` | Probed ports | 22,80,443,... |
| `network.transport` | Scan type | tcp |
| `tcp.flags.syn` | SYN scan | true |
| `event.dataset` | Packetbeat protocol | tcp |

**Kibana query:**  
```kql
# 1. All TCP SYN scans (Nmap -sS default)
network.transport: tcp AND tcp.flags.syn: true AND tcp.flags.ack: false

# 2. Single IP scanning many ports (>10 unique ports)
network.transport: tcp AND tcp.flags.syn: true | stats unique_ports = unique(destination.port) by source.ip | where unique_ports > 10

# 3. Rapid port probes (20+ connections in 60s)
network.transport: tcp AND tcp.flags.syn: true AND @timestamp > now-60s | stats count() by source.ip | where count >= 20

# 4. SYN scan on common ports (22,80,443,3389)
network.transport: tcp AND tcp.flags.syn: true AND destination.port in (22,80,443,3389,445,1433)
```

**What to look for:**
- 1 source → many destination ports (not single service)
- Short-duration flows (scanner doesn't complete handshake)
- SYN-only traffic (no full TCP 3-way)

<img width="1846" height="926" alt="image" src="https://github.com/user-attachments/assets/1feb6985-97eb-471a-aafd-3309890637e9" />

 

**Analyst playbook:**
1. Filter `source.ip` → check port count and timing
2. Correlate with SSH/ICMP for attack chain
3. Review `flow.final: false` (aborted connections)
4. Response: block IP, asset inventory check (what is running that the attacker may be scanning for)?

**Kibana visualisation:**
- **Top Values**: source.ip by unique ports
- **Line chart**: SYN probes over time
- **Heatmap**: source.ip x destination.port matrix

**MITRE ATT&CK:** T1046 (Network Service Discovery) 

# FTP
destination.port: 21

# Telnet
destination.port: 23

# ICMP
network.protocol: "icmp"
