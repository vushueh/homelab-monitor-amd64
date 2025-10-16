# 🛡️ AMD64 Homelab Server - Network Services & Monitoring Platform

## 📋 Project Overview

This repository documents the **AMD64 Network Services & Monitoring** server in my cybersecurity homelab. The system serves as an always-on platform providing network-wide DNS filtering, ad blocking, system monitoring, and Docker container management.

**Part of a Multi-Tier Security Lab:**
- **AMD64 (This Repo)**: 🛡️ Network Services - DNS filtering, monitoring, and infrastructure management
- **Proxmox Server**: 🖥️ Virtualization - SIEM, vulnerable VMs, and security testing environments
- **Windows Server**: 💼 Enterprise Services - Active Directory, file services
- **Kali Linux**: 🔴 Security Testing - Offensive security tools and penetration testing

---

## 🎯 Purpose & Learning Objectives

This project demonstrates hands-on experience with:

### Infrastructure & Operations
- ✅ DNS filtering and network-wide ad blocking (Pi-hole)
- ✅ Container orchestration with Docker and Docker Compose
- ✅ Real-time system and service monitoring
- ✅ Infrastructure hardening and security best practices
- ✅ Automated backup and disaster recovery
- ✅ Network segmentation with VLANs

### Security Operations
- ✅ Firewall configuration and management (UFW)
- ✅ Intrusion prevention (Fail2Ban)
- ✅ Automatic security patch management
- ✅ Service health monitoring and alerting
- ✅ Log aggregation and analysis (ready for SIEM integration)

### DevOps Practices
- ✅ Infrastructure as Code principles
- ✅ Container lifecycle management
- ✅ Service reliability engineering
- ✅ Documentation and knowledge management

---

## 🗺️ Network Architecture

```
                    Alta Labs Route 10 Router
                    (192.168.x.x)
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
    VLAN 1           VLAN 10 (Admin)        VLAN 20
  (General)         192.168.x.0/24      (Windows Server)
                            │
                    Cisco 2960 Switch
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   [AMD64 Server]      [Kali Linux]        [Other Devices]
   192.168.x.x       192.168.x.x
        │
        ├─ Pi-hole (DNS + Ad Blocking)
        ├─ Portainer (Docker Management)
        ├─ Netdata (System Monitoring)
        ├─ Uptime Kuma (Service Health)
        └─ Dozzle (Log Viewer)

DNS Flow:
Client → Router DHCP → DNS: 192.168.x.x → Pi-hole
    → Blocks Ads (271,552 domains) → Upstream: 1.1.1.1
```

---

## 🖥️ Hardware & System Specifications

**Hardware:** AMD64 PC  
**Operating System:** Debian GNU/Linux (Trixie/Sid)  
**Hostname:** `debian`  
**IP Address:** `192.168.x.x`  
**VLAN:** 10 (Admin/Management Network)  
**Gateway:** `192.168.10.1` (Alta Labs Route 10)  
**RAM:** 7.7GB  
**Storage:** 291GB SSD (3% utilized)  
**Network Interface:** `enp0s7` (Gigabit Ethernet)

---

## 📦 Deployed Services

### Core Services

| Service | Purpose | Port | Status | Resources |
|---------|---------|------|--------|-----------|
| **Pi-hole** | DNS filtering & ad blocking | 53, 80 | ✅ Healthy | ~200MB RAM |
| **Portainer** | Docker management UI | 9000 | ✅ Running | ~100MB RAM |
| **Netdata** | Real-time system monitoring | 19999 | ✅ Healthy | ~300MB RAM |
| **Uptime Kuma** | Service uptime monitoring | 3001 | ✅ Healthy | ~150MB RAM |
| **Dozzle** | Docker log viewer | 8888 | ✅ Running | ~50MB RAM |

### Service Details

#### Pi-hole (DNS & Ad Blocking)
- **Container:** `pihole/pihole:latest`
- **Network Mode:** Host
- **Web Interface:** http://192.168.10.26/admin
- **DNS Port:** 53 (TCP/UDP)
- **Blocked Domains:** 271,552
- **Active Clients:** 9+
- **Block Rate:** ~20% of total queries
- **Upstream DNS:** Cloudflare (1.1.1.1, 1.0.0.1)
- **Features:** Query logging, DHCP integration, conditional forwarding

#### Portainer (Container Management)
- **Container:** `portainer/portainer-ce:latest`
- **Web Interface:** http://192.168.10.26:9000
- **Features:** Container control, stack deployment, image management, volume management

#### Netdata (System Monitoring)
- **Container:** `netdata/netdata:latest`
- **Web Interface:** http://192.168.10.26:19999
- **Metrics:** CPU, RAM, disk I/O, network, Docker containers
- **Update Interval:** Real-time (1-second granularity)

#### Uptime Kuma (Service Health)
- **Container:** `louislam/uptime-kuma:1`
- **Web Interface:** http://192.168.10.26:3001
- **Monitors:** HTTP(S), DNS, Ping, TCP ports
- **Notifications:** Email, Discord, Slack, Telegram (configurable)

#### Dozzle (Log Viewer)
- **Container:** `amir20/dozzle:latest`
- **Web Interface:** http://192.168.10.26:8888
- **Features:** Real-time logs, multi-container view, search functionality

---

## 🚀 Deployment Guide

### Prerequisites

```bash
# Verify system requirements
free -h                    # Check available RAM
df -h                      # Check disk space
docker --version           # Docker 20.10+
docker compose version     # Docker Compose V2
```

### Quick Start Deployment

```bash
# Clone repository
git clone https://github.com/vushueh/homelab-amd64-server.git
cd homelab-amd64-server

# Deploy all services
cd lab/pihole
docker compose up -d

# Verify deployment
docker ps
docker compose logs -f pihole
```

### Individual Service Deployment

```bash
# Deploy Pi-hole only
cd lab/pihole
docker compose up -d pihole

# Deploy monitoring stack
docker compose up -d portainer netdata uptime-kuma dozzle

# Check status
docker compose ps
```

### Access Services

All services are accessible via web browser:

| Service | URL | Credentials |
|---------|-----|-------------|
| Pi-hole | http://192.168.10.26/admin | admin / @91984 |
| Portainer | http://192.168.10.26:9000 | admin / @91984 |
| Netdata | http://192.168.10.26:19999 | No authentication |
| Uptime Kuma | http://192.168.10.26:3001 | admin / @91984 |
| Dozzle | http://192.168.10.26:8888 | No authentication |

⚠️ **Security Notice:** Change default passwords immediately in production!

---

## 🔐 Security Configuration

### Firewall (UFW)

**Status:** Active and enabled on system boot

**Configured Rules:**

| Port | Service | Protocol | Source | Purpose |
|------|---------|----------|--------|---------|
| 22 | SSH | TCP | Any | Remote management |
| 53 | DNS | TCP/UDP | Any | Pi-hole DNS queries |
| 80 | HTTP | TCP | Any | Pi-hole web interface |
| 9000 | Portainer | TCP | Any | Docker management |
| 19999 | Netdata | TCP | Any | System monitoring |
| 3001 | Uptime Kuma | TCP | Any | Service health |
| 8888 | Dozzle | TCP | Any | Log viewer |

**Management Commands:**
```bash
# View firewall status
sudo ufw status numbered

# Add new rule
sudo ufw allow <port>/<protocol> comment 'Service Name'

# Delete rule
sudo ufw delete <rule_number>

# View logs
sudo tail -f /var/log/ufw.log
```

### Fail2Ban (Intrusion Prevention)

**Status:** Active and monitoring SSH

**Protected Services:**
- SSH (Port 22) - Automatically bans IPs after failed login attempts

**Configuration:**
- **Max Retries:** 5 attempts
- **Ban Time:** 10 minutes (default)
- **Find Time:** 10 minutes

**Management Commands:**
```bash
# Check Fail2Ban status
sudo fail2ban-client status

# Check SSH jail
sudo fail2ban-client status sshd

# View banned IPs
sudo fail2ban-client status sshd | grep "Banned IP"

# Unban an IP
sudo fail2ban-client set sshd unbanip <IP_ADDRESS>
```

### Automatic Security Updates

**Status:** Enabled via `unattended-upgrades`

**Configuration:**
- Automatic installation of security updates
- Daily update check
- Automatic cleanup of old packages

**Logs:** `/var/log/unattended-upgrades/`

---

## 💾 Backup & Disaster Recovery

### Automated Backup System

**Script Location:** `~/backups/backup-homelab.sh`

**Backup Contents:**
- Docker Compose configurations
- Pi-hole configuration (teleporter export)
- Network configuration
- Container list and images
- System DNS settings

**Backup Storage:** `~/backups/`  
**Retention Policy:** Last 7 backups kept automatically  
**Latest Backup:** 24MB compressed archive

### Manual Backup

```bash
# Run backup manually
~/backups/backup-homelab.sh

# List existing backups
ls -lh ~/backups/

# Extract backup
cd ~/backups
tar -xzf homelab-backup-YYYYMMDD_HHMMSS.tar.gz
```

### Restore Procedure

```bash
# Extract backup
cd ~/backups
tar -xzf homelab-backup-YYYYMMDD_HHMMSS.tar.gz
cd homelab-backup-YYYYMMDD_HHMMSS

# Restore Docker Compose configurations
cp -r lab/ ~/

# Restore Pi-hole
# (Import via web interface: Settings → Teleporter → Restore)

# Recreate containers
cd ~/lab/pihole
docker compose up -d
```

### Scheduled Backups (Optional)

```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /root/backups/backup-homelab.sh >> /var/log/homelab-backup.log 2>&1
```

---

## 📊 Monitoring & Operations

### System Status Check

**Quick Status Command:**
```bash
homelab-status
```

**Output includes:**
- System uptime and hostname
- Disk usage (current: 3% of 291GB)
- Memory usage (current: ~900MB of 7.7GB)
- Docker container status
- Firewall status
- Fail2Ban status
- Network interfaces
- Security configuration summary

### Container Management

```bash
# View all containers
docker ps

# View container stats (CPU/Memory)
docker stats

# View logs
docker logs -f pihole
docker logs -f portainer
docker logs --tail 100 netdata

# Restart container
docker restart pihole

# Stop/Start container
docker stop pihole
docker start pihole
```

### Service Health Checks

```bash
# Check Pi-hole status
docker exec pihole pihole status

# Test DNS resolution
nslookup google.com 192.168.10.26

# Test ad blocking
nslookup doubleclick.net 192.168.10.26
# Should return: Address: 0.0.0.0

# Check Docker service
sudo systemctl status docker

# View system resources
free -h
df -h
uptime
```

### Updating Services

```bash
# Navigate to service directory
cd ~/lab/pihole

# Pull latest images
docker compose pull

# Recreate containers with new images
docker compose up -d

# Remove old images
docker image prune -a
```

---

## 🌐 Network Configuration

### DNS Setup

**Pi-hole Configuration:**
- **Interface:** eth0 (192.168.10.26)
- **Listening Mode:** LOCAL (VLAN 10 only)
- **Upstream DNS Servers:**
  - Primary: 1.1.1.1 (Cloudflare)
  - Secondary: 1.0.0.1 (Cloudflare)

**Router Configuration (Alta Labs Route 10):**
- **VLAN 10 DNS Servers:** 192.168.10.26, 1.1.1.1
- **DHCP Range:** 192.168.10.10 - 192.168.10.252

### Pi-hole Statistics

**Current Metrics:**
- **Total Queries:** Thousands daily
- **Queries Blocked:** ~20% average
- **Active Clients:** 9+ devices
- **Blocklists:** 6 active lists
- **Total Blocked Domains:** 271,552

**Blocklist Sources:**
1. StevenBlack's Unified Hosts
2. AdGuard DNS Filter
3. EasyList
4. Malware Domain List
5. Phishing Army
6. Additional privacy lists

### VLAN Configuration

**Active VLANs on Route 10:**
- **VLAN 1:** 192.168.1.0/24 (General)
- **VLAN 10:** 192.168.10.0/24 (Admin/Management) ← AMD64 Location
- **VLAN 20:** 192.168.20.0/24 (Windows Server)
- **VLAN 30:** 10.30.0.0/24 (Proxmox Virtualization)
- **VLAN 50:** 192.168.50.0/24 (IoT/WiFi)
- **VLAN 250:** 192.168.250.0/24 (Attack Lab)

**Current DNS Coverage:** VLAN 10 (expandable to other VLANs)

---

## 🔄 Auto-Start & Resilience

### Docker Container Restart Policies

All containers configured with `restart: unless-stopped`

**This ensures:**
- ✅ Containers automatically start after system reboot
- ✅ Containers restart if they crash
- ✅ Containers stay stopped only if manually stopped

**Verify restart policies:**
```bash
docker inspect pihole portainer netdata uptime-kuma dozzle \
  --format='{{.Name}}: {{.HostConfig.RestartPolicy.Name}}'
```

### System Services (Auto-Start on Boot)

| Service | Status | Purpose |
|---------|--------|---------|
| Docker | ✅ Enabled | Container runtime |
| UFW | ✅ Enabled | Firewall |
| Fail2Ban | ✅ Enabled | Intrusion prevention |
| Unattended-Upgrades | ✅ Enabled | Automatic security updates |

**Verify service status:**
```bash
# Check all auto-start services
sudo systemctl is-enabled docker ufw fail2ban unattended-upgrades

# Check service status
sudo systemctl status docker
```

### Reboot Survival Test

```bash
# Test system resilience
sudo reboot

# After reboot (2-3 minutes), verify:
docker ps                  # All containers running
homelab-status            # All services operational
sudo ufw status           # Firewall active
```

---

## 🛠️ Troubleshooting Guide

### Pi-hole Issues

**Problem:** Ads not being blocked
```bash
# Check Pi-hole is running
docker ps | grep pihole

# Verify DNS is being used
nslookup google.com
# Should show: Server: 192.168.10.26

# Test ad blocking
nslookup doubleclick.net
# Should return: 0.0.0.0

# Update blocklists
docker exec pihole pihole -g

# Check Pi-hole logs
docker logs pihole --tail 100
```

**Problem:** Cannot access Pi-hole admin
```bash
# Check container status
docker ps | grep pihole

# Check port 80 is listening
sudo ss -tulnp | grep :80

# Restart Pi-hole
docker restart pihole

# Check firewall
sudo ufw status | grep 80
```

### Container Issues

**Problem:** Container won't start
```bash
# View container logs
docker logs <container_name>

# Check for port conflicts
sudo ss -tulnp | grep <port>

# Check disk space
df -h

# Restart Docker service
sudo systemctl restart docker

# Recreate container
docker compose up -d --force-recreate <service_name>
```

**Problem:** High resource usage
```bash
# Check resource consumption
docker stats

# View system resources
homelab-status
htop

# Restart resource-heavy container
docker restart <container_name>
```

### Network Issues

**Problem:** Cannot reach services
```bash
# Check network interface
ip addr show enp0s7

# Check gateway
ip route show

# Test connectivity
ping 192.168.10.1  # Router
ping 8.8.8.8       # Internet

# Check firewall rules
sudo ufw status numbered

# Check DNS resolution
cat /etc/resolv.conf
nslookup google.com
```

### System Performance

**Problem:** System running slow
```bash
# Check system resources
homelab-status
free -h
df -h

# Check Docker disk usage
docker system df

# Clean up Docker
docker system prune -a  # WARNING: Removes unused images

# Check running processes
htop
ps aux --sort=-%mem | head -10  # Top memory users
ps aux --sort=-%cpu | head -10  # Top CPU users
```

---

## 📁 Important File Locations

| Path | Contents |
|------|----------|
| `~/lab/pihole/` | Pi-hole Docker Compose configuration |
| `~/lab/pihole/etc-pihole/` | Pi-hole configuration files |
| `~/lab/pihole/etc-dnsmasq.d/` | DNS configuration |
| `~/backups/` | System backups |
| `~/backups/backup-homelab.sh` | Backup script |
| `~/check-system.sh` | System status script |
| `/etc/pihole/pihole.toml` | Pi-hole main configuration |
| `/var/log/pihole/` | Pi-hole logs |
| `/var/log/ufw.log` | Firewall logs |
| `/var/log/fail2ban.log` | Fail2Ban logs |
| `/var/log/unattended-upgrades/` | Update logs |

---

## 🎓 Skills Demonstrated

### Technical Competencies

**Infrastructure Management**
- Docker containerization and orchestration
- Multi-service deployment with Docker Compose
- Network architecture and VLAN segmentation
- DNS server configuration and management

**System Administration**
- Linux server administration (Debian)
- Service monitoring and health checks
- Log management and analysis
- Resource optimization and capacity planning

**Security Operations**
- Firewall configuration and management (UFW)
- Intrusion prevention systems (Fail2Ban)
- Security patch management
- Network traffic filtering (Pi-hole)
- Backup and disaster recovery planning

**DevOps Practices**
- Infrastructure as Code principles
- Automated deployment procedures
- Service reliability engineering
- Documentation and knowledge management
- Monitoring and alerting strategies

### Soft Skills
- Technical documentation and communication
- Problem-solving and troubleshooting
- Process improvement and optimization
- Self-directed learning and research
- Project planning and execution

---

## 🔮 Future Enhancements

### Planned Implementations

**Phase 1: Enhanced Monitoring (In Progress)**
- [ ] Configure Uptime Kuma monitors for all services
- [ ] Set up notification channels (Email, Discord)
- [ ] Create custom Pi-hole statistics dashboard
- [ ] Implement automated health checks

**Phase 2: SIEM Integration (Planned)**
- [ ] Deploy Wazuh SIEM on Proxmox VM
- [ ] Install Wazuh agent on AMD64
- [ ] Configure log forwarding from all services
- [ ] Create custom detection rules
- [ ] Set up security event correlation

**Phase 3: Network Expansion (Planned)**
- [ ] Extend Pi-hole DNS to VLAN 1, 20, 50
- [ ] Configure inter-VLAN routing for DNS
- [ ] Implement network-wide ad blocking
- [ ] Add Pi-hole redundancy (secondary DNS)

**Phase 4: Advanced Features (Future)**
- [ ] Grafana dashboard for metrics visualization
- [ ] Prometheus for advanced monitoring
- [ ] Automated certificate management (Let's Encrypt)
- [ ] VPN server integration
- [ ] Network intrusion detection (Suricata)

---

## 📚 Resources & Documentation

### Official Documentation
- **Pi-hole:** https://docs.pi-hole.net/
- **Docker:** https://docs.docker.com/
- **Docker Compose:** https://docs.docker.com/compose/
- **Portainer:** https://docs.portainer.io/
- **Netdata:** https://learn.netdata.cloud/
- **Uptime Kuma:** https://github.com/louislam/uptime-kuma/wiki
- **Debian:** https://www.debian.org/doc/

### Learning Resources
- **Docker Mastery Course:** Udemy - Bret Fisher
- **Linux System Administration:** Linux Academy
- **Network Security Fundamentals:** Cybrary
- **r/homelab:** Reddit community for homelab enthusiasts
- **r/selfhosted:** Self-hosting community and resources

### Related Projects
- **Proxmox Virtualization Server** - SIEM VMs, vulnerable systems
- **Windows Server 2022** - Enterprise services integration
- **Kali Linux Security Testing** - Offensive security tools

---

## 🤝 Contributing & Feedback

This is a personal learning project documenting my homelab journey.

**Feedback Welcome:**
- 🐛 Bug reports or issues
- 💡 Suggestions for improvements
- 📖 Documentation clarity
- 🔧 Alternative approaches

**Not Accepting:**
- Pull requests (personal learning project)
- Feature requests beyond learning scope

---

## 📸 Screenshots

### Pi-hole Dashboard
![Pi-hole Dashboard](screenshots/pihole-dashboard.png)
*Network-wide ad blocking statistics and query logs*

### Portainer Container Management
![Portainer Interface](screenshots/portainer-dashboard.png)
*Docker container management and monitoring*

### Netdata System Monitoring
![Netdata Dashboard](screenshots/netdata-dashboard.png)
*Real-time system metrics and performance*

### Uptime Kuma Service Health
![Uptime Kuma Dashboard](screenshots/uptime-kuma-dashboard.png)
*Service availability monitoring and alerts*

### Dozzle Log Viewer
![Dozzle Interface](screenshots/dozzle-dashboard.png)
*Real-time Docker container logs*

---

## 📊 Project Statistics

**Deployment Date:** October 15, 2025  
**System Uptime:** 3+ hours (as of last check)  
**Total Containers:** 5  
**Total Queries Processed:** 1000s daily  
**Ad Block Rate:** ~20%  
**Active Clients:** 9+  
**Disk Usage:** 7.6GB / 291GB (3%)  
**Memory Usage:** ~900MB / 7.7GB (11%)  
**Services Status:** ✅ All Operational  

---

## 🏆 Achievements

- ✅ Deployed production-ready DNS filtering solution
- ✅ Implemented comprehensive monitoring stack
- ✅ Configured enterprise-grade security hardening
- ✅ Automated backup and recovery procedures
- ✅ Documented entire infrastructure professionally
- ✅ Achieved 100% service availability since deployment
- ✅ Blocked 20% of network traffic (ads/trackers)
- ✅ Zero security incidents since hardening

---

## ⚠️ Important Notices

### Security Disclaimers
- ⚠️ **Default passwords shown are EXAMPLES** - Change immediately
- 🔒 **Credentials stored securely** in password manager
- 🛡️ **Network isolated** from production/work networks
- 📊 **Logs may contain sensitive data** - access control required

### Usage Notes
- 📝 Configuration validated and tested October 15, 2025
- 🔄 System designed for 24/7 operation
- 💾 Regular backups recommended (automated script provided)
- 🔧 Update containers regularly for security patches
- 📡 VLAN 10 access required for management interfaces

---

## 📞 Contact & Links

**GitHub Profile:** [@vushueh](https://github.com/vushueh)  
**Project Repository:** [homelab-amd64-server](https://github.com/vushueh/homelab-amd64-server)  
**LinkedIn:** [Your LinkedIn Profile]  
**Portfolio:** [Your Portfolio Website]

---

## 📄 License

This project is for **educational and personal use only**.

- Configurations provided as-is without warranty
- Use at your own risk
- Not intended for commercial deployment
- Learning and skill development purposes

---

## 🙏 Acknowledgments

**Open Source Communities:**
- Pi-hole team for excellent DNS filtering solution
- Docker community for containerization platform
- Portainer team for intuitive container management
- Netdata team for comprehensive monitoring
- Louis Lam (Uptime Kuma creator)
- Amir Raminfar (Dozzle creator)

**Learning Resources:**
- r/homelab community for inspiration
- r/selfhosted for self-hosting guidance
- Docker documentation and tutorials
- Debian community support

**Special Thanks:**
- Alta Labs for affordable enterprise networking
- Cisco for reliable switching equipment
- Open source maintainers worldwide

---

## 🎯 Project Goals Met

- ✅ **Hands-on experience** with production infrastructure
- ✅ **Real-world problem solving** - DNS, monitoring, security
- ✅ **Documentation skills** - Professional technical writing
- ✅ **System administration** - Linux server management
- ✅ **Network security** - Firewall, IPS, DNS filtering
- ✅ **DevOps practices** - Container orchestration, automation
- ✅ **Portfolio development** - Demonstrable technical skills

---

**Last Updated:** October 15, 2025  
**Document Version:** 2.0  
**System Status:** ✅ Fully Operational  
**Next Milestone:** Wazuh SIEM Integration (Proxmox VM)

---

**Built with 💙 for learning, growing, and sharing knowledge in cybersecurity and infrastructure management**

---

*This README is a living document and will be updated as the homelab evolves and new services are deployed.*
