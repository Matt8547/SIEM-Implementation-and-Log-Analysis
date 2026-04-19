# VirtualBox Setup

## Prerequisites
- **VirtualBox 7.1.** (latest)
- **Host machine**: 8GB+ RAM, 50GB free disk
- **Network**: Host-only adapter for VM-to-VM communication

## VM Specifications

| VM Name | OS | CPU | RAM | Disk | Network |
|---------|----|-----|-----|------|---------|
| **kali-attacker** | Kali Linux  | 2 cores | 4GB | 40GB VDI | Host-only Adapter 1 |
| **ubuntu-target** | Ubuntu Server 22.04 LTS | 2 cores | 4GB | 30GB VDI | Host-only Adapter 1 |


## Step-by-Step Setup

### 1. Create Host-Only Network

-VirtualBox → File → Host Network Manager → Create
Name: vboxnet0
IPv4: 192.168.56.1/24 (DHCP server enabled)



### 2. Download OS Images

-Kali Linux: https://www.kali.org/get-kali/#kali-virtual-machines
-Ubuntu Server: https://ubuntu.com/download/server



### 3. Kali Linux VM (Attacker)

-New → Name: kali-attacker → Type: Linux → Version: Debian 64-bit

-Memory: 4096MB → Processors: 2

-Create virtual hard disk → VDI → Dynamically allocated → 40GB

-Settings → Network → Adapter 1 → Enable → Host-only Adapter → vboxnet0

-Mount Kali ISO → Start → Install (username: kali, password: kali)



### 4. Ubuntu Server VM (Target)

-New → Name: ubuntu-target → Type: Linux → Version: Ubuntu 64-bit

-Memory: 4096MB → Processors: 2

-Create virtual hard disk → VDI → Dynamically allocated → 30GB

-Settings → Network → Adapter 1 → Enable → Host-only Adapter → vboxnet0

-Mount Ubuntu Server ISO → Start → Install (minimal server)



### 5. Post-Install Configuration

**Kali Linux:**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nmap ssh -y
ip addr show  # Note IP address (update ip)
```

**Ubuntu Server:**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install openssh-server ufw -y
sudo ufw allow ssh
sudo ufw enable
ip addr show  # Note IP address (TBU)
```

### 6. Verify Connectivity

From Kali to Ubuntu:
```bash
nmap -sn 192.168.56.0/24    # Discover Ubuntu IP
ssh ubuntu@192.168.56.102   # Test SSH access
```

### 7. Screenshots
