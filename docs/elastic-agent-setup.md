# Elastic Agent Setup (Ubuntu Target VM)

## Prerequisites
- Ubuntu Server 22.04 VM running and accessible via SSH
- Elastic Cloud account OR self-hosted Elasticsearch/Kibana
- **Fleet enrollment token** (Kibana → Fleet → Settings → Enrollment tokens)

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
