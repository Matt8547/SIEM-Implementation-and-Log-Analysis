# Elastic Agent Setup (Ubuntu Target VM)

## Prerequisites
- Ubuntu Server 22.04 VM running and accessible via SSH
- Elastic Cloud account OR self-hosted Elasticsearch/Kibana

-<img width="1204" height="640" alt="Image" src="https://github.com/user-attachments/assets/9310da48-6cb7-4314-891a-337101d99874" />


##Fleet enrollment(Kibana → Fleet → Add Agent → Enrollment tokens)

-<img width="1589" height="268" alt="Image" src="https://github.com/user-attachments/assets/f710b00d-f2c4-470e-80ee-6033b1ec5e7a" />

## Installation Steps

### 1. Download & Extract Agent
```bash
cd ~
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.15.0-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.15.0-linux-x86_64.tar.gz
cd elastic-agent-8.15.0-linux-x86_64/
```

### 2. Enroll Agent with Fleet (Recommended)
```bash
sudo ./elastic-agent install \
  --url=https://fleet.YOUR_ELASTIC_CLOUD_URL \
  --enrollment-token=YOUR_FLEET_TOKEN_HERE \
  --non-interactive
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
- [ ] Logs appear in **Discover** (`host.name:ubuntu-target`)
- [ ] Metrics dashboard shows **CPU/Memory usage**
- [ ] Packetbeat flows visible (`network.protocol: tcp`)

## Troubleshooting
| Issue | Fix |
|-------|-----|
| `Agent offline` | `sudo systemctl restart elastic-agent` |
| `Enrollment failed` | Verify token hasn't expired (Kibana → Fleet → Settings) |
| `No logs in Kibana` | Check `sudo journalctl -u elastic-agent` for errors |

## Next Steps
- [Packetbeat-specific config](packetbeat-setup.md)
- [Generate test traffic](detection-rules.md)

**Screenshot example**: Add `![Agent healthy in Fleet](screenshots/agent-healthy.png)`
