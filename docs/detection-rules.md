# Detection Rules

## Overview
This file documents the detection logic used in the Elastic SIEM home lab.  
The lab is designed to identify suspicious network activity captured by Packetbeat and visualised in Kibana, with a focus on common SOC-relevant behaviours such as brute-force attempts, scanning activity, insecure protocol usage, and network reconnaissance.

The detections in this lab are based on traffic generated between the Kali attacker VM and the Ubuntu-based Elastic SIEM server.  
Packetbeat was configured to monitor protocols including SSH, FTP, Telnet, HTTP and DNS.

## Detection Cases
| Rule Name | Suspicious Behavior | Data Source | Example Query | Analyst Action |
|---|---|---|---|---|
| SSH brute force | Multiple failed SSH logins from one source | filebeat + packetbeat | event.action: "ssh_auth_failure" and event.outcome: "failure" | Check source IP, count failures, block if needed |
| Nmap scan | Many connection attempts across ports in a short time | packetbeat | where (destination.port >= 1 and destination.port <= 65535) also add where ports_scanned >= 10 | Confirm scan, isolate source |
| Telnet access | Telnet traffic to port 23 | packetbeat | destination.port == 23 or network.transport == "telnet" | Validate if Telnet is approved |
| FTP access | Legacy FTP traffic to port 21 | packetbeat | destination.port: 21 or destination.port: 20 and event.action: "ftp_login" and event.outcome: "success | Check if file transfer is expected |
| ICMP sweep | Repeated ICMP echo requests to multiple hosts | packetbeat | network.transport: "icmp" and icmp.type: 8 | Verify recon activity and what an attacker could be looking for |

## Rule Details

### 1. SSH Brute Force Attempts
**Threat description:**  
Repeated failed SSH login attempts may indicate brute-force activity against our Linux server.

**Why it matters:**  
SSH is a common remote administration protocol and a frequent target for attackers trying to gain initial access.

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
<br><br>


<br><br>
<img width="1851" height="926" alt="image" src="https://github.com/user-attachments/assets/2ab680ca-36e2-4560-baca-d93506ca2d25" />

### Analyst response:  
- Identify the source IP generating repeated attempts
- Check whether the activity was expected lab testing or unknown traffic
- Review surrounding activity from the same IP
- Consider blocking or isolating the source if this were a live environment

<br><br>
<img width="1834" height="776" alt="image" src="https://github.com/user-attachments/assets/3cf182f6-b711-4292-8801-c609e264bb65" />
<br><br>

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
<br><br>
<img width="1852" height="951" alt="image" src="https://github.com/user-attachments/assets/6a6b1cd3-028c-4b43-9598-5f8b9888225b" />

<br><br>

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

 

**Analyst response:**  
- Pivot on the source IP to review all related connections
- Check the volume and timing of port access attempts
- Compare activity against expected admin or scanner behaviour
- Escalate if scanning is unexpected or targets critical systems

**Kibana visualisation:**
- **Top Values**: source.ip by unique ports
- **Line chart**: SYN probes over time
- **Heatmap**: source.ip x destination.port matrix

**MITRE ATT&CK:** T1046 (Network Service Discovery) 

<br><br>

### FTP Usage port 20 (data) or 21 (control)

**Threat description:**  
FTP is another plaintext protocol that may expose usernames, passwords, and transferred data.

**Why it matters:**  
Unsecured file transfer protocols are high-risk and often violate modern security baselines.

**Relevant fields:**  
- `destination.port`
- `source.ip`
- `destination.ip`
- `network.transport`

**Kibana query:**  
```kql
destination.port : (20 or 21) or source.port : (20 or 21)
network.protocol : "ftp" or network.application : "ftp"
network.protocol : "ftp" and event.outcome : "failure"
```

**What to look for:**  
- Any FTP sessions between hosts
- Repeated file-transfer related traffic
- Systems using outdated transfer methods

**Analyst response:**  
- Validate whether the activity is expected
- Review communicating hosts and session frequency
- Recommend migration to SFTP or SCP
- Flag the activity as insecure protocol usage

<br><br>

### 3. Telnet Usage port 23

**Threat description:**  
Telnet is an insecure protocol that transmits data in plaintext.

**Why it matters:**  
Use of Telnet in a real environment may expose credentials and indicate legacy or weakly secured systems.

**Relevant fields:**  
- `destination.port`
- `event.dataset`
- `source.ip`
- `destination.ip`

**Kibana query:**  
```kql
destination.port : 23 or source.port : 23
network.protocol : "telnet" or network.application : "telnet"
```

**What to look for:**  
- Any Telnet session in the environment
- Repeated interactive connections
- Unexpected systems communicating over Telnet

**Analyst response:**  
- Verify whether the connection was part of a lab test
- Identify the source and destination systems
- Recommend disabling Telnet and replacing it with SSH
- Investigate whether credentials may have been exposed

<br><br>

### ICMP Sweep / Ping Activity
**Threat description:**  
ICMP traffic can be used for host discovery and environment mapping.

**Why it matters:**  
Ping sweeps are often part of early-stage reconnaissance.

**Relevant fields:**  
- `network.protocol`
- `source.ip`
- `destination.ip`

**Kibana query:**  
```kql
network.protocol : "icmp" and icmp.type : "echo-request"
```

**What to look for:**  
- One source sending ICMP requests to multiple hosts
- Repeated echo requests in a short period
- Reconnaissance patterns before other traffic appears

**Analyst response:**  
- Determine whether the ICMP traffic was expected testing
- Check whether the same source later initiated scans or login attempts
- Correlate with other Packetbeat events
- Escalate if recon activity appears linked to broader attack behaviour

---

## Notes
These detections are based on network telemetry collected by Packetbeat rather than endpoint logs.  
As the lab grows, future detections can include Elastic Defend, Sysmon, Sigma-based rules, and correlation across multiple log sources.

## Future Improvements
- Further screenshots detailing FTP, Telnet and ICMP detections
- Add threshold-based detections for repeated SSH attempts
- Create Kibana dashboards for each protocol
- Convert simple detections into Elastic Security rules
- Add Windows endpoint logs with Sysmon for host-based detections
- Map all use cases to MITRE ATT&CK techniques
