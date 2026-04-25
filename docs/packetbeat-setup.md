# Packetbeat Setup

## Overview
Packetbeat was installed on the Ubuntu Server target VM to capture network traffic and send flow and protocol data to Elastic. This allowed the lab to monitor activity such as Nmap scans, SSH access attempts, HTTP requests, and DNS traffic in Kibana. [cite:20][cite:23]

## Purpose
The goal of Packetbeat in this project was to provide network visibility alongside host telemetry from Elastic Agent. Packetbeat captures packets from the local interface, creates network flow records, and can decode supported protocols such as HTTP and DNS for investigation and dashboarding. [cite:20][cite:17]

## Installation
Packetbeat was installed on the Ubuntu Server VM using the Elastic package repository and configured to send data to Elastic Cloud / Elasticsearch. Elastic documents both quick-start installation and direct configuration through the `packetbeat.yml` file. [cite:135][cite:136]

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
nmap -sS 192.168.56.20
curl http://192.168.56.20
nslookup google.com
ssh user@192.168.56.20
```

## Verification in Kibana
Verification was completed in Kibana by checking for Packetbeat indices, network flow records, and dashboard activity. Packetbeat sends JSON documents into Elasticsearch for each observed transaction or flow, which can then be explored in Kibana. 

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


## Screenshots
Add screenshots such as:
- `screenshots/packetbeat-service-status.png`
- `screenshots/packetbeat-discover.png`
- `screenshots/packetbeat-dashboard.png`

## Next Steps
Planned future improvements include adding more protocols, refining dashboards, and testing detections using larger traffic sets and suspicious behavior simulations.

