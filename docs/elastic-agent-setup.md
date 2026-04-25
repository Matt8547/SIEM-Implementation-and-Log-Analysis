# Elastic Agent Setup (Ubuntu Target VM)

## Prerequisites
- Ubuntu Server 22.04 VM running and accessible via SSH
- Elastic Cloud account OR self-hosted Elasticsearch/Kibana

-<img width="1204" height="640" alt="Image" src="https://github.com/user-attachments/assets/9310da48-6cb7-4314-891a-337101d99874" />

<div>
  
### Fleet enrollment
Kibana → Fleet → Add Agent → Select Architecture → Copy Command and Paste into Terminal of Target machine

-<img width="1589" height="268" alt="Image" src="https://github.com/user-attachments/assets/f710b00d-f2c4-470e-80ee-6033b1ec5e7a" />

<img width="766" height="720" alt="Image" src="https://github.com/user-attachments/assets/811a2329-5bf6-4a00-8d5c-267a77fe0273" />

## Installation Steps

### 1. Download & Extract Agent
```bash
cd ~
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.3.3+build202604082258-linux-x86_64.tar.gz 
tar xzvf elastic-agent-9.3.3+build202604082258-linux-x86_64.tar.gz
cd elastic-agent-9.3.3+build202604082258-linux-x86_64

```

### 2. Enroll Agent with Fleet (Recommended)
```bash
sudo ./elastic-agent install \
  --url=https://fleet.YOUR_ELASTIC_SERVER_URL 
  --enrollment-token=YOUR_FLEET_TOKEN_HERE 
  --insecure (if using a self signed certificate) 
```

### 3. Verify Agent Status
```bash
sudo systemctl status elastic-agent
sudo systemctl enable elastic-agent
sudo journalctl -u elastic-agent -f
```

### 4. Configure Integrations (Kibana UI)


## Expected Data Streams
| Integration | Data Collected |
|-------------|----------------|
| **System** | `/var/log/*.log`, CPU, memory, disk |
| **Packetbeat** | Network flows, HTTP requests, DNS |

## Verification Checklist
- [ ] Agent shows **Healthy** in Fleet → Agents
- [ ] Logs appear in **Discover** (`host.name:####`)
- [ ] Metrics dashboard shows **CPU/Memory usage**
- [ ] Packetbeat flows visible (`network.protocol: tcp`)

## Troubleshooting
| Issue | Fix |
|-------|-----|
| `Agent offline` | `sudo systemctl restart elastic-agent` |
| `Enrollment failed` | Verify token hasn't expired (Kibana → Fleet → Settings) |
| `No logs in Kibana` | Check `sudo journalctl -u elastic-agent` for errors |


**Screenshot example**: Add `![Agent healthy in Fleet](screenshots/agent-healthy.png)`


## Next Steps
- [Packetbeat-specific config](packetbeat-setup.md)
- [Generate test traffic](detection-rules.md)


