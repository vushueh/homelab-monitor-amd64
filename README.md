# 🛡️ Homelab Monitor - AMD64 SIEM & Defense Platform

## 📋 Project Overview

This repository documents the **Defender/Monitor** component of my three-PC cybersecurity homelab. The AMD64 system serves as an always-on Security Information and Event Management (SIEM) platform that monitors, detects, and analyzes security events across the lab environment.

**Part of a Three-Tier Security Lab:**
- **AMD64 (This Repo)**: 🛡️ Defender - SIEM, monitoring, and log analysis
- **i5 System**: 🎯 Victim - Intentionally vulnerable applications for testing
- **i3 System**: 🔴 Attacker - Offensive security tools (Kali Linux, Metasploit)

---

## 🎯 Purpose & Learning Objectives

This project demonstrates hands-on experience with:

### Security Operations (SOC)
- ✅ SIEM deployment and configuration (Wazuh)
- ✅ Real-time security event monitoring
- ✅ Log aggregation and correlation
- ✅ Intrusion detection system (Suricata)
- ✅ Alert creation and incident response workflows

### Infrastructure Management
- ✅ Container orchestration with Docker Compose
- ✅ Resource management and optimization
- ✅ Service monitoring and health checks
- ✅ Network segmentation and isolation

### Blue Team Operations
- ✅ Defensive security posture
- ✅ Attack detection and analysis
- ✅ Security baseline monitoring
- ✅ Forensic log analysis

---

## 🏗️ Architecture

```
                    Attack Flow
                    
[i3 - Attacker] ──────────> [i5 - Victim] 
   Kali Linux              DVWA/Juice Shop
   Metasploit              Vulnerable Apps
   Nmap Scans              Wazuh Agent
        │                        │
        │                        │
        └────────> Logs ─────────┘
                     │
                     ▼
            [AMD64 - Monitor/SIEM]
            ┌─────────────────────┐
            │  Wazuh Dashboard    │ ← Web UI
            │  Wazuh Manager      │ ← SIEM Core
            │  Wazuh Indexer      │ ← Log Storage
            │  Suricata IDS       │ ← Intrusion Detection
            │  Netdata            │ ← System Metrics
            │  Portainer          │ ← Container Mgmt
            │  Uptime Kuma        │ ← Service Health
            └─────────────────────┘
                     │
                     ▼
              SOC Analyst
           (Detection & Response)
```

---

## 🖥️ Hardware Specifications

**System:** Dell AMD64 PC  
**OS:** Debian 13.1 (Trixie)  
**RAM:** 8GB (recommended) / 2GB + 4GB swap (minimum)  
**Storage:** SSD for Docker volumes  
**Network:** Connected to Admin VLAN (192.168.10.x)  
**IP Address:** 192.168.10.26

---

## 📦 Technology Stack

### Core Components

| Service | Purpose | Resource Usage |
|---------|---------|----------------|
| **Wazuh Manager** | SIEM core engine, log processing | 2GB RAM |
| **Wazuh Indexer** | OpenSearch-based log storage | 2GB RAM |
| **Wazuh Dashboard** | Web-based SIEM interface | 1GB RAM |
| **Suricata** | Network intrusion detection | 1GB RAM |
| **Portainer** | Docker container management | 512MB RAM |
| **Netdata** | Real-time system monitoring | 512MB RAM |
| **Uptime Kuma** | Service health monitoring | 512MB RAM |
| **Filebeat** | Log forwarder and collector | 256MB RAM |

### Deployment Platform
- **Docker Engine** - Container runtime
- **Docker Compose** - Multi-container orchestration
- **Docker Networks** - Isolated container networking

---

## 🚀 Deployment Guide

### Prerequisites

```bash
# System requirements check
free -h          # Verify RAM (8GB recommended)
docker --version # Docker 20.10+
docker compose version # Compose V2
```

### Phase 1: Lightweight Services (2GB RAM)

Deploy management and monitoring tools that work with limited resources:

```bash
# Clone repository
git clone https://github.com/vushueh/homelab-monitor-amd64.git
cd homelab-monitor-amd64

# Deploy Phase 1 services
docker compose up -d portainer netdata uptime-kuma filebeat

# Verify deployment
docker compose ps
docker compose logs -f portainer
```

**Access Services via SSH Tunnel:**

```bash
# From your workstation
ssh -L 9000:localhost:9000 \
    -L 19999:localhost:19999 \
    -L 3001:localhost:3001 \
    leonel@192.168.10.26
```

- **Portainer:** http://localhost:9000 (Docker GUI)
- **Netdata:** http://localhost:19999 (System metrics)
- **Uptime Kuma:** http://localhost:3001 (Service health)

### Phase 2: Full SIEM Stack (8GB RAM Required)

Deploy the complete security monitoring platform:

```bash
# Deploy all services
docker compose --profile full-stack up -d

# Wait 2-3 minutes for initialization
docker compose logs -f wazuh-manager

# Check all services
docker compose ps
```

**Access Wazuh Dashboard:**
- URL: https://192.168.10.26
- Default User: `admin`
- Default Pass: `SecurePassword123!`

⚠️ **Change default password immediately after first login!**

### Phase 3: Intrusion Detection (Optional)

```bash
# Deploy Suricata IDS
docker compose --profile full-stack up -d suricata

# Monitor IDS alerts
docker compose logs -f suricata
tail -f /var/log/suricata/fast.log
```

---

## 🔧 Configuration

### Network Configuration

**Docker Networks:**
- `monitor_net` (172.20.0.0/24) - Internal container communication

**Exposed Services:**
- `1514/udp` - Wazuh agent communication
- `1515/tcp` - Wazuh agent enrollment
- `443/tcp` - Wazuh dashboard (HTTPS)
- Management UIs bound to `127.0.0.1` (SSH tunnel required)

### Security Hardening

**Firewall Rules (UFW):**

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow Wazuh agent communication (Admin VLAN only)
sudo ufw allow from 192.168.10.0/24 to any port 1514 proto udp
sudo ufw allow from 192.168.10.0/24 to any port 1515 proto tcp

# Allow Wazuh dashboard (Admin VLAN only)
sudo ufw allow from 192.168.10.0/24 to any port 443 proto tcp

# Enable firewall
sudo ufw enable
sudo ufw status
```

### Wazuh Agent Configuration (for i5 Victim)

**On i5 system, install Wazuh agent:**

```bash
# Download agent
curl -so wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.2-1_amd64.deb

# Install and configure
WAZUH_MANAGER='192.168.10.26' dpkg -i ./wazuh-agent.deb

# Start agent
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

---

## 📊 Monitoring & Operations

### Health Checks

```bash
# View all service status
docker compose ps

# Check resource usage
docker stats

# View logs
docker compose logs -f wazuh-manager
docker compose logs -f suricata

# System metrics
free -h
df -h
```

### Common Operations

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# Restart a service
docker compose restart wazuh-manager

# Update containers
docker compose pull
docker compose up -d

# View Wazuh alerts
docker compose exec wazuh-manager tail -f /var/ossec/logs/alerts/alerts.log
```

---

## 🎓 Attack Detection Scenarios

### Scenario 1: Port Scan Detection

**Attack (from i3):**
```bash
nmap -sS 192.168.10.XX
```

**Detection:**
- Suricata IDS flags multiple SYN packets
- Wazuh correlates port scan pattern
- Alert generated in dashboard

### Scenario 2: Web Application Attack

**Attack (from i3):**
```bash
sqlmap -u "http://192.168.10.XX/vulnerabilities/sqli/?id=1&Submit=Submit"
```

**Detection:**
- Wazuh agent on i5 detects SQL injection attempts
- Web server logs analyzed
- Alert with attack signature

### Scenario 3: Brute Force Detection

**Attack (from i3):**
```bash
hydra -l admin -P passwords.txt ssh://192.168.10.XX
```

**Detection:**
- Multiple failed SSH authentication attempts
- Wazuh triggers brute force rule
- Source IP flagged

---

## 📈 Skills Demonstrated

### Technical Skills
- **SIEM Administration**: Wazuh deployment, rule creation, alert tuning
- **Container Orchestration**: Docker Compose, multi-service management
- **Network Security**: IDS/IPS, log analysis, network monitoring
- **Linux Administration**: Debian server management, systemd services
- **Infrastructure as Code**: Declarative service definitions
- **Security Operations**: Incident detection, log correlation, threat hunting

### Soft Skills
- **Documentation**: Clear technical writing and architecture diagrams
- **Problem Solving**: Resource optimization, troubleshooting
- **Process Thinking**: Phased deployment, security hardening
- **Continuous Learning**: Self-directed cybersecurity lab

---

## 🔒 Security Considerations

### Passwords
- ⚠️ Default passwords shown in config files are **EXAMPLES ONLY**
- 🔐 Change all default credentials immediately
- 🔑 Use strong, unique passwords for each service
- 📝 Store credentials in password manager

### Network Isolation
- ✅ Vulnerable apps (i5) isolated from production networks
- ✅ Attack traffic (i3) contained to lab environment
- ✅ SIEM (AMD64) accessible only from Admin VLAN
- ✅ No direct internet exposure

### Data Protection
- 🛡️ All management interfaces behind SSH tunnels
- 🔒 HTTPS for web dashboards
- 📊 Logs contain sensitive data - access control required

---

## 📚 Resources & References

### Documentation
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Suricata User Guide](https://suricata.readthedocs.io/)
- [Docker Compose Reference](https://docs.docker.com/compose/)

### Related Projects
- [i5 Victim System](https://github.com/vushueh/homelab-victim-i5) - Vulnerable applications
- [i3 Attacker System](https://github.com/vushueh/homelab-attacker-i3) - Offensive tools

### Learning Path
- CompTIA Security+
- Blue Team Level 1 (BTL1)
- CCNA CyberOps
- Splunk Certified User

---

## 🤝 Contributing

This is a personal learning project, but suggestions are welcome!

- 🐛 **Issues**: Report bugs or suggest improvements
- 💡 **Ideas**: Share alternative approaches or tools
- 📖 **Documentation**: Help improve clarity

---

## 📝 Project Status

- ✅ Phase 1: Lightweight monitoring - **COMPLETE**
- 🔄 Phase 2: Full SIEM stack - **IN PROGRESS**
- ⏳ Phase 3: Advanced detection rules - **PLANNED**
- ⏳ Integration with i5/i3 systems - **PLANNED**

---
## 📸 Screenshots

### Netdata - Real-time System Monitoring
![Netdata Dashboard](screenshots/Netdata.JPG)

### Portainer - Docker Container Management  
![Portainer Interface](screenshots/Portainer.JPG)

### Uptime Kuma - Service Health Monitoring
![Uptime Kuma Dashboard](screenshots/Uptime%20Kuma.JPG)
---

## 📧 Contact

**GitHub**: [@vushueh](https://github.com/vushueh)  
**Project Link**: [homelab-monitor-amd64](https://github.com/vushueh/homelab-monitor-amd64)

---

## 📄 License

This project is for educational purposes. Configurations and scripts are provided as-is.

---

## 🙏 Acknowledgments

- Wazuh Team for the excellent open-source SIEM
- Suricata Project for robust IDS capabilities
- Docker Community for containerization best practices
- r/homelab for inspiration and guidance

---

**Built with 💙 for cybersecurity learning and hands-on skill development**
