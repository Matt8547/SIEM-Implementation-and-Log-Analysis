# Packetbeat Setup

## Overview
Packetbeat was installed on the Ubuntu Server target VM to capture network traffic and send flow and protocol data to Elastic. This allowed the lab to monitor activity such as Nmap scans, SSH access attempts, HTTP requests, and DNS traffic in Kibana. 

## Purpose
The goal of Packetbeat in this project was to provide network visibility alongside host telemetry from Elastic Agent. Packetbeat captures packets from the local interface, creates network flow records, and can decode supported protocols such as HTTP and DNS for investigation and dashboarding. 

## Installation
Packetbeat was installed on the Ubuntu Server VM using the Elastic package repository and configured to send data to Elastic Cloud / Elasticsearch. Elastic documents both quick-start installation and direct configuration through the `packetbeat.yml` file. 

### Install Packetbeat
```bash
sudo apt update
sudo apt install packetbeat -y
```

### Enable the service
```bash
sudo systemctl enable packetbeat
sudo systemctl start packetbeat
sudo systemctl status packetbeat
```
  <img width="799" height="340" alt="image" src="https://github.com/user-attachments/assets/86ab6214-8247-4075-b558-edbe279bcd56" />


## Configuration
Packetbeat was configured in `/etc/packetbeat/packetbeat.yml` to monitor the main VM interface and collect traffic flow data. Elastic’s flow documentation explains that the `packetbeat.flows` section is used to collect statistics on flows, and that if this section is missing, flow monitoring is disabled. 

### Example configuration
```yaml
packetbeat.interfaces.device: any

packetbeat.flows:
  timeout: 30s
  period: 10s

packetbeat.protocols:
  - type: dns
    ports: [53]
  - type: http
    ports: [80, 443, 8080]

output.elasticsearch:
  hosts: ["localhost:9200"]
  username: "elastic"
  password: "Your Password "

setup.kibana:
  host: "localhost:5601"
```

## Setup Tasks
After editing the configuration file, Packetbeat was tested and dashboards were loaded into Kibana. Elastic and other installation references show Packetbeat setup and dashboard loading as part of the normal deployment workflow. 

### Test configuration
```bash
sudo packetbeat test config -e
```

### Test output connection
```bash
sudo packetbeat test output
```

<img width="800" height="653" alt="Image" src="https://github.com/user-attachments/assets/90b1ff2f-34c8-4ad9-8c89-046f2747f1a3" />

### Load dashboards
```bash
sudo packetbeat setup -e
```

### Restart service
```bash
sudo systemctl restart packetbeat
```




## Traffic Generation
To validate the setup, traffic was generated from the Kali Linux VM toward the Ubuntu Server VM. Packetbeat was then used to observe flows and protocol activity related to scanning and service access. 

### Example test activity
```bash
nmap -sS Target IP
curl http://Target IP
nslookup google.com
ssh user@Target IP
```

## Verification in Kibana
Verification was completed in Kibana by checking for Packetbeat indices, network flow records, and dashboard activity. Packetbeat sends JSON documents into Elasticsearch for each observed transaction or flow, which can then be explored in Kibana. 

<img width="1847" height="920" alt="image" src="https://github.com/user-attachments/assets/f370471a-3b8f-46d1-ad55-e3ff445864c8" />


<img width="1843" height="924" alt="image" src="https://github.com/user-attachments/assets/90d2836d-0af3-4a26-8327-47eea6b1ae11" />


<img width="1846" height="927" alt="image" src="https://github.com/user-attachments/assets/62193f6e-1819-48c6-a52b-d335f963a49e" />


<img width="1847" height="962" alt="image" src="https://github.com/user-attachments/assets/e3f38b0d-8e4b-40b9-a331-64c6afb967e1" />


<img width="522" height="845" alt="image" src="https://github.com/user-attachments/assets/9fd8ecd9-79bc-462a-ab31-5f96236cf283" />


### Useful checks
- Confirm Packetbeat service is running on Ubuntu.
- Search Discover for `agent.type : "packetbeat"`.
- Filter on `source.ip`, `destination.ip`, or `network.protocol`.
- Review dashboards for top talkers, protocols, and flow volume. 

## Example KQL Queries
```text
agent.type : "packetbeat"
network.protocol : "dns"
source.ip : "192.168.56.10"
destination.ip : "192.168.56.20"
```

## Findings
Packetbeat successfully captured flow-level visibility between the Kali and Ubuntu VMs and made it possible to identify scans, service access, and basic network behavior in Kibana. This complemented the host-level logs from Elastic Agent and improved investigation context. 

## Troubleshooting
| Issue | Fix |
|---|---|
| No traffic visible | Confirm correct interface with `ip a` and update `packetbeat.interfaces.device` |
| Config errors | Run `sudo packetbeat test config -e` |
| No data in Kibana | Check Elasticsearch output settings and service status |
| Missing dashboards | Run `sudo packetbeat setup -e` again |


## Next Steps

Planned future improvements include adding more servers and clients to simulate a more realistic enterprise environment, refining dashboards, and testing detections using larger traffic sets and suspicious behavior simulations.

https://github.com/Matt8547/SIEM-Implementation-and-Log-Analysis/blob/main/docs/detection-rules.md
