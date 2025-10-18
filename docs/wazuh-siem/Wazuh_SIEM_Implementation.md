# Wazuh SIEM - Complete Implementation Summary

## Document Information

| Attribute | Value |
|-----------|-------|
| **Implementation Date** | October 17-18, 2025 |
| **Platform** | Hyper-V on Windows Server |
| **Wazuh Version** | 4.13.1 |
| **Status** | ✅ Production-Ready |
| **Document Version** | 1.0 |
| **Last Updated** | October 18, 2025 |
| **Next Review Date** | November 18, 2025 |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [Access Credentials](#3-access-credentials)
4. [VM Roles & Configurations](#4-vm-roles--configurations)
5. [Monitoring Capabilities](#5-monitoring-capabilities)
6. [Custom Security Rules](#6-custom-security-rules)
7. [What Was Accomplished](#7-what-was-accomplished)
8. [Health Check Procedures](#8-health-check-procedures)
9. [Backup & Recovery](#9-backup--recovery)
10. [Next Steps & Lab Ideas](#10-next-steps--lab-ideas)
11. [Troubleshooting Guide](#11-troubleshooting-guide)
12. [Performance Metrics](#12-performance-metrics)
13. [Reference Documentation](#13-reference-documentation)
14. [Important Notes](#14-important-notes)
15. [Quick Reference](#15-quick-reference)

---

## 1. Executive Summary

### What We Built

A complete Security Information and Event Management (SIEM) system monitoring 3 servers across your homelab network, providing:

- Real-time security monitoring of all systems
- Compliance scanning against CIS Debian 13 benchmarks
- Container security monitoring for 5 Docker services
- Intrusion detection via Fail2Ban integration
- File integrity monitoring with realtime alerts
- Centralized logging and alert correlation

### Key Achievements

| Achievement | Status |
|-------------|--------|
| Migrated HQ Wazuh from Proxmox to Hyper-V | ✅ Complete |
| Upgraded entire stack from v4.12.0 → v4.13.1 | ✅ Complete |
| Configured 2 active agents monitoring production systems | ✅ Complete |
| Implemented custom detection rules for Docker, Pi-hole, Fail2Ban | ✅ Complete |
| Enabled CIS compliance scanning (200+ security checks) | ✅ Complete |
| Established network segregation (VLAN 10 for VMs, VLAN 20 for host) | ✅ Complete |
| All services auto-start on boot and survive reboots | ✅ Complete |

---

## 2. Architecture Overview

### Network Topology Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Hyper-V Host (VLAN 20)                   │
│                   192.168.20.x Network                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Virtual Switch
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼────────┐    ┌──────▼──────┐    ┌────────▼────────┐
│  HQ Wazuh SIEM │    │ Ingest VM   │    │  AMD64 Server   │
│ 192.168.10.156 │    │192.168.10.189│   │ 192.168.10.26   │
│                │    │              │    │                 │
│ • Manager      │◄───┤ • Agent      │    │ • Agent         │
│ • Indexer      │    │   (v4.12.0)  │    │   (v4.13.1)     │
│ • Dashboard    │◄───┤              │    │ • Pi-hole       │
│   (v4.13.1)    │    │              │    │ • Portainer     │
│                │    │              │    │ • Netdata       │
│ 8 vCPU         │    │ 2 vCPU       │    │ • Uptime Kuma   │
│ 16GB RAM       │    │ 2GB RAM      │    │ • Dozzle        │
└────────────────┘    └──────────────┘    └─────────────────┘
     VLAN 10              VLAN 10              VLAN 10
```

### Network Configuration

| Component | IP Address | VLAN | Gateway | Role |
|-----------|------------|------|---------|------|
| **HQ Wazuh SIEM** | 192.168.10.156 | 10 | 192.168.10.1 | Central Security Manager |
| **Ingest VM** | 192.168.10.189 | 10 | 192.168.10.1 | Monitored Agent |
| **AMD64 Server** | 192.168.10.26 | 10 | 192.168.10.1 | Production Server |
| **Hyper-V Host** | 192.168.20.x | 20 | 192.168.20.1 | Hypervisor |

**Network Services:**
- **Gateway:** Alta Labs Route 10
- **DNS:** Pi-hole (192.168.10.26)

---

## 3. Access Credentials

### Wazuh Dashboard

| Parameter | Value |
|-----------|-------|
| **URL** | https://192.168.10.156 |
| **Username** | admin |
| **Password** |  |

### AMD64 Server Services

| Service | URL | Username | Password |
|---------|-----|----------|----------|
| **Pi-hole** | http://192.168.10.26/admin | admin |  |
| **Portainer** | http://192.168.10.26:9000 | admin |  |
| **Netdata** | http://192.168.10.26:19999 | - | (no auth) |
| **Uptime Kuma** | http://192.168.10.26:3001 | admin |  |
| **Dozzle** | http://192.168.10.26:8888 | - | (no auth) |

### SSH Access

| System | Command | Notes |
|--------|---------|-------|
| **HQ Wazuh** | `ssh leonel@192.168.10.156` | - |
| **Ingest VM** | `ssh leonel@192.168.10.189` | - |
| **AMD64** | `ssh leonel@192.168.10.26` | Use `su -` for root access |

---

## 4. VM Roles & Configurations

### 4.1 HQ Wazuh SIEM (192.168.10.156)

#### System Specifications

| Parameter | Value |
|-----------|-------|
| **Operating System** | Ubuntu 22.04.5 LTS |
| **Resources** | 8 vCPU, 16GB RAM |
| **Wazuh Version** | 4.13.1 |

#### Components

| Component | Description | Status |
|-----------|-------------|--------|
| **Wazuh Manager** | • Processes security events from all agents<br>• Applies detection rules (built-in + custom)<br>• Manages agent configurations<br>• Performs threat correlation | Active, auto-starts on boot |
| **Wazuh Indexer** | • Stores all security data<br>• Indexes logs for fast searching<br>• Retains historical data | Active, auto-starts on boot |
| **Wazuh Dashboard** | • Web UI for security monitoring<br>• Real-time alert visualization<br>• Compliance reporting<br>• Agent management interface | Active, accessible via HTTPS |

#### Key Configuration Files

| File Path | Purpose |
|-----------|---------|
| `/var/ossec/etc/ossec.conf` | Manager configuration |
| `/var/ossec/etc/rules/local_rules.xml` | Custom rules |
| `/var/ossec/logs/` | Logs directory |
| `/var/ossec/logs/alerts/alerts.log` | Security alerts |

#### Backups Created

| Backup File | Purpose |
|-------------|---------|
| `/var/ossec/etc/ossec.conf.backup_20251018` | Configuration backup |
| `/var/ossec/etc/rules/local_rules.xml.backup_20251018` | Custom rules backup |

---

### 4.2 Wazuh Ingest VM (192.168.10.189)

#### System Specifications

| Parameter | Value |
|-----------|-------|
| **Operating System** | Ubuntu 22.04.5 LTS |
| **Resources** | 2 vCPU, 2GB RAM |
| **Wazuh Agent Version** | 4.12.0 |
| **Purpose** | Dedicated log collection and forwarding agent |

#### Current Monitoring

| Feature | Status |
|---------|--------|
| System logs (journald) | ✅ Enabled |
| Package management (dpkg) | ✅ Enabled |
| Active responses | ✅ Enabled |
| File Integrity Monitoring | ❌ Disabled |
| Rootcheck | ❌ Disabled |

#### Configuration Status

- Agent connected and active
- Sending logs to HQ every 20 seconds
- FIM/Rootcheck can be enabled later if needed

#### Backups Created

| Backup File | Purpose |
|-------------|---------|
| `/var/ossec/etc/ossec.conf.backup_20251018` | Configuration backup |

---

### 4.3 AMD64 Server "debian" (192.168.10.26)

#### System Specifications

| Parameter | Value |
|-----------|-------|
| **Operating System** | Debian GNU/Linux 13 (Trixie) |
| **Resources** | Custom hardware server |
| **Wazuh Agent Version** | 4.13.1 |
| **Purpose** | Production server running network services with comprehensive security monitoring |

#### Services Running

**Docker Containers (5):**

| Container | Purpose |
|-----------|---------|
| **Pi-hole** | Network-wide DNS and ad blocking |
| **Portainer** | Docker container management |
| **Netdata** | Real-time system monitoring |
| **Uptime Kuma** | Service uptime monitoring |
| **Dozzle** | Docker log viewer |

**Security Services:**

| Service | Status | Description |
|---------|--------|-------------|
| **UFW Firewall** | Active | Logging enabled |
| **Fail2Ban** | Active | Protecting SSH from brute force |
| **Unattended-upgrades** | Active | Automatic security patches |

#### Comprehensive Monitoring Enabled

**1. File Integrity Monitoring (FIM)**

| Path | Type | Description |
|------|------|-------------|
| `/etc` | Watch | System configuration files |
| `/usr/bin`, `/usr/sbin` | Watch | System binaries |
| `/bin`, `/sbin` | Watch | Core binaries |
| `/boot` | Watch | Boot files |
| `/root` | Watch | Root home directory |
| `/root/lab` | **REALTIME** | Docker configs (real-time monitoring) |

**Detection Capabilities:**
- File modifications
- Permission changes
- Ownership changes
- File creation/deletion
- Content changes (with diff)

**2. Docker Container Monitoring**

- All 5 container JSON logs being analyzed
- Container status monitoring every 5 minutes
- Error detection in container logs
- Container restart/crash detection

**3. Log Collection**

| Source | Path/Method | Format |
|--------|-------------|--------|
| **System Logs** | journald | All systemd logs |
| **Package Management** | `/var/log/dpkg.log` | Text |
| **Fail2Ban** | `/var/log/fail2ban.log` | Text |
| **Security Updates** | `/var/log/unattended-upgrades/` | Text |
| **Docker Containers** | All 5 container logs | JSON format |

**4. System Commands (every 6 minutes)**

- Disk usage (`df -P`)
- Network connections (`netstat`)
- User logins (`last -n 20`)
- Docker container status

**5. Rootcheck (Rootkit Detection)**

- Scans every 12 hours
- Checks for trojans, rootkits, hidden processes
- Port scan detection
- System anomaly detection

**6. Syscollector (System Inventory)**

- Hardware inventory (CPU, RAM, disk)
- OS information
- Network interfaces
- Installed packages
- Open ports
- Running processes
- Updates hourly

**7. Security Configuration Assessment (SCA)**

| Parameter | Value |
|-----------|-------|
| **Policy** | CIS Debian 13 Benchmark |
| **Checks** | ~200 security configuration tests |
| **Frequency** | Every 12 hours |

**Compliance Standards:**
- CIS Benchmarks
- PCI-DSS
- HIPAA
- NIST SP 800-53
- ISO 27001

**Example SCA Checks:**
- Is SSH root login disabled?
- Are passwords strong enough?
- Is firewall enabled and configured?
- Are critical patches installed?
- File permission checks
- Service hardening verification

#### Configuration Files

| File Path | Purpose |
|-----------|---------|
| `/var/ossec/etc/ossec.conf` | Agent config |
| `/var/ossec/ruleset/sca/cis_debian13.yml` | SCA policy |
| `/var/ossec/logs/ossec.log` | Agent logs |

#### Backups Created

| Backup File | Purpose |
|-------------|---------|
| `/var/ossec/etc/ossec.conf.backup` | Original configuration |
| `/var/ossec/etc/ossec.conf.backup_20251018` | Latest backup |
| `/var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak` | SCA working config |

---

## 5. Monitoring Capabilities

### Real-Time Monitoring Matrix

| Capability | AMD64 | Ingest VM | HQ |
|------------|-------|-----------|-----|
| **File Integrity Monitoring** | ✅ Realtime on /root/lab | ❌ Disabled | ✅ Self-monitoring |
| **Rootkit Detection** | ✅ Every 12h | ❌ Disabled | ✅ Every 12h |
| **Log Collection** | ✅ Full | ✅ Basic | ✅ Self |
| **Docker Monitoring** | ✅ All 5 containers | N/A | N/A |
| **Security Compliance** | ✅ CIS Debian 13 | ❌ Not configured | ✅ Self |
| **Intrusion Detection** | ✅ Fail2Ban | N/A | N/A |
| **System Inventory** | ✅ Hourly | ✅ Hourly | ✅ Hourly |

### Alert Levels

| Level | Description | Example |
|-------|-------------|---------|
| **3** | Low - Informational | Docker container started, user login |
| **5** | Medium - Notable event | Security update installed |
| **7** | High - Attention needed | File modified, service error |
| **10** | Critical - Immediate action | Docker config changed, critical file modified |
| **12** | Critical - Service down | Pi-hole container crashed |

---

## 6. Custom Security Rules

### Overview

Custom rules are located in: `/var/ossec/etc/rules/local_rules.xml` on HQ Wazuh

### Docker Monitoring Rules

| Rule ID | Level | Description |
|---------|-------|-------------|
| **100001** | 3 | Docker container event detected |
| **100005** | 7 | Error detected in container logs |

### Fail2Ban Rules

| Rule ID | Level | Description |
|---------|-------|-------------|
| **100010** | 8 | IP address banned by Fail2Ban |
| **100011** | 3 | IP address unbanned |
| **100012** | 3 | Fail2Ban jail started |

### Pi-hole Rules

| Rule ID | Level | Description |
|---------|-------|-------------|
| **100021** | 7 | Pi-hole FTL error or warning |

### File Integrity Monitoring (FIM) Rules

| Rule ID | Level | Description |
|---------|-------|-------------|
| **100031** | 7 | File modification in /root/lab |
| **100032** | 10 | Critical: Docker compose file modified |
| **100033** | 7 | File permissions or ownership changed |
| **100034** | 7 | New file created in monitored directory |

### Complete Custom Rules XML

```xml
<group name="custom_rules,">
  
  <!-- Docker Container Events -->
  <rule id="100001" level="3">
    <decoded_as>json</decoded_as>
    <field name="log">\.+</field>
    <description>Docker container event detected</description>
    <group>docker,</group>
  </rule>

  <rule id="100005" level="7">
    <if_sid>100001</if_sid>
    <field name="log">error</field>
    <description>Error detected in container logs</description>
    <group>docker,errors,</group>
  </rule>

  <!-- Fail2Ban Events -->
  <rule id="100010" level="8">
    <decoded_as>fail2ban</decoded_as>
    <match>Ban</match>
    <description>IP address banned by Fail2Ban</description>
    <group>fail2ban,intrusion_detection,</group>
  </rule>

  <rule id="100011" level="3">
    <decoded_as>fail2ban</decoded_as>
    <match>Unban</match>
    <description>IP address unbanned by Fail2Ban</description>
    <group>fail2ban,</group>
  </rule>

  <rule id="100012" level="3">
    <decoded_as>fail2ban</decoded_as>
    <match>Jail.*started</match>
    <description>Fail2Ban jail started</description>
    <group>fail2ban,</group>
  </rule>

  <!-- Pi-hole FTL Events -->
  <rule id="100021" level="7">
    <decoded_as>json</decoded_as>
    <field name="log">WARN|ERROR</field>
    <match>pihole-FTL</match>
    <description>Pi-hole FTL error or warning</description>
    <group>pihole,dns,</group>
  </rule>

  <!-- File Integrity Monitoring - Docker Configs -->
  <rule id="100031" level="7">
    <if_sid>550</if_sid>
    <match>/root/lab</match>
    <description>File modification detected in Docker configuration directory</description>
    <group>syscheck,docker,pci_dss_11.5,gdpr_II_5.1.f,</group>
  </rule>

  <rule id="100032" level="10">
    <if_sid>100031</if_sid>
    <match>docker-compose.yml</match>
    <description>Critical: Docker compose file modified</description>
    <group>syscheck,docker,pci_dss_11.5,gdpr_II_5.1.f,</group>
  </rule>

  <rule id="100033" level="7">
    <if_sid>552</if_sid>
    <description>File permissions or ownership changed</description>
    <group>syscheck,pci_dss_11.5,</group>
  </rule>

  <rule id="100034" level="7">
    <if_sid>554</if_sid>
    <description>New file created in monitored directory</description>
    <group>syscheck,pci_dss_11.5,</group>
  </rule>

</group>
```

---

## 7. What Was Accomplished

### Migration & Upgrade

| Task | Status |
|------|--------|
| Migrated HQ Wazuh from Proxmox to Hyper-V | ✅ Complete |
| Created new Ubuntu 22.04.5 LTS VM | ✅ Complete |
| Installed Wazuh 4.13.1 (all-in-one) | ✅ Complete |
| Configured proper network settings (VLAN 10) | ✅ Complete |
| Verified all services start on boot | ✅ Complete |
| Upgraded all components from 4.12.0 → 4.13.1 | ✅ Complete |
| Maintained backward compatibility for Ingest VM (4.12.0) | ✅ Complete |

### Security Monitoring Implementation

| Component | Status |
|-----------|--------|
| Installed and configured Wazuh agent 4.13.1 on AMD64 | ✅ Complete |
| Enabled comprehensive FIM on critical paths | ✅ Complete |
| Configured realtime monitoring on /root/lab | ✅ Complete |
| Set up Docker container log analysis (all 5 containers) | ✅ Complete |
| Enabled CIS Debian 13 compliance scanning | ✅ Complete |
| Configured Rootcheck for rootkit detection | ✅ Complete |
| Enabled Syscollector for system inventory | ✅ Complete |
| Integrated Fail2Ban log monitoring | ✅ Complete |
| Set up system command monitoring | ✅ Complete |
| Created configuration backups | ✅ Complete |
| Configured Ingest VM basic monitoring | ✅ Complete |

### Custom Detection Rules

| Task | Status |
|------|--------|
| Created 10 custom rules for Docker, Fail2Ban, Pi-hole, FIM | ✅ Complete |
| Tested and verified all rules work | ✅ Complete |
| Rules auto-load on manager restart | ✅ Complete |
| Backed up local_rules.xml | ✅ Complete |

### Integration Testing

| Test | Status |
|------|--------|
| FIM alerts for test files | ✅ Pass |
| Docker compose modification alerts | ✅ Pass |
| Container event detection | ✅ Pass |
| Fail2Ban integration | ✅ Pass |
| SCA compliance scans | ✅ Pass |
| Agent connectivity verification | ✅ Pass |
| HQ auto-start after reboot | ✅ Pass |
| Agent auto-reconnect after reboot | ✅ Pass |
| Dashboard accessibility | ✅ Pass |

---

## 8. Health Check Procedures

### Daily Health Checks (5 minutes)

#### 1. Dashboard Quick Check

- Access: https://192.168.10.156
- Login with admin credentials
- Check "Security Events" dashboard
- Look for any level 10+ alerts (critical)

**Expected Results:**
- ✅ Dashboard loads successfully
- ✅ All agents showing "Active" status
- ✅ No critical alerts (or only expected ones)
- ✅ Event count is reasonable

#### 2. Agent Status Check

```bash
ssh leonel@192.168.10.156
sudo /var/ossec/bin/agent_control -l
```

**Expected Output:**
```
Wazuh agent_control. List of available agents:
   ID: 000, Name: HQ (server), IP: 127.0.0.1, Active/Local
   ID: 001, Name: leonel, IP: 192.168.10.189, Active
   ID: 002, Name: debian, IP: 192.168.10.26, Active
```

### Weekly Health Checks (15 minutes)

#### 1. Compliance Score Review

- Dashboard → Modules → Security Configuration Assessment
- Check CIS Debian 13 score for AMD64
- Review any failed checks

**Expected Results:**
- ✅ Score > 75%
- ✅ No new critical failures

#### 2. Alert Review

```bash
ssh leonel@192.168.10.156

# Review last week's critical alerts
sudo grep "level.*10\|level.*12" /var/ossec/logs/alerts/alerts.log | tail -50

# Check for repeated patterns
sudo tail -1000 /var/ossec/logs/alerts/alerts.log | grep -o "rule: [0-9]*" | sort | uniq -c | sort -rn | head -10
```

**Expected Results:**
- ✅ No unexpected critical alerts
- ✅ No alert flooding

#### 3. System Resource Check

```bash
ssh leonel@192.168.10.156
free -h
df -h /var/ossec
```

**Expected Results:**
- ✅ Memory < 80% usage
- ✅ Disk < 70% usage

#### 4. Log Rotation Check

```bash
ssh leonel@192.168.10.156
ls -lh /var/ossec/logs/alerts/
```

**Expected Results:**
- ✅ Logs are rotating
- ✅ Old logs exist

### Monthly Health Checks (30 minutes)

#### 1. Full Configuration Backup

```bash
# HQ Wazuh
ssh leonel@192.168.10.156
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup_$(date +%Y%m%d)
sudo cp /var/ossec/etc/rules/local_rules.xml /var/ossec/etc/rules/local_rules.xml.backup_$(date +%Y%m%d)

# AMD64
ssh leonel@192.168.10.26
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup_$(date +%Y%m%d)

# Ingest VM
ssh leonel@192.168.10.189
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup_$(date +%Y%m%d)
```

#### 2. Index Management

- Dashboard → Stack Management → Index Management
- Check index sizes
- Delete old indices if needed (older than 30 days)

#### 3. Rule Effectiveness Review

- Dashboard → Management → Rules
- Review custom rules (100xxx series)
- Check if they're firing as expected

#### 4. System Updates

```bash
# HQ Wazuh
ssh leonel@192.168.10.156
sudo apt update && sudo apt upgrade -y

# AMD64
ssh leonel@192.168.10.26
su -
apt update && apt upgrade -y

# Ingest VM
ssh leonel@192.168.10.189
sudo apt update && sudo apt upgrade -y
```

#### 5. VM Snapshot Creation

- Take snapshot of HQ Wazuh in Hyper-V
- Name: "HQ_Wazuh_Working_[DATE]"
- Take snapshot of AMD64 if virtual
- Take snapshot of Ingest VM

---

## 9. Backup & Recovery

### Critical Backup Locations

#### HQ Wazuh (192.168.10.156)

| File/Directory | Purpose | Restore Priority |
|----------------|---------|------------------|
| `/var/ossec/etc/ossec.conf` | Manager configuration | **CRITICAL** |
| `/var/ossec/etc/rules/local_rules.xml` | Custom detection rules | **CRITICAL** |
| `/var/ossec/etc/shared/` | Custom agent configurations | HIGH |
| `/var/ossec/logs/alerts/` | Historical alerts | MEDIUM |
| `/var/ossec/stats/` | Performance statistics | LOW |

#### AMD64 Server (192.168.10.26)

| File/Directory | Purpose | Restore Priority |
|----------------|---------|------------------|
| `/var/ossec/etc/ossec.conf` | Agent configuration | **CRITICAL** |
| `/var/ossec/etc/ossec.conf.backup_*` | Config backups | **CRITICAL** |
| `/root/lab/` | Docker compose files | HIGH |

#### Ingest VM (192.168.10.189)

| File/Directory | Purpose | Restore Priority |
|----------------|---------|------------------|
| `/var/ossec/etc/ossec.conf` | Agent configuration | **CRITICAL** |
| `/var/ossec/etc/ossec.conf.backup_*` | Config backups | **CRITICAL** |

### Current Backups (as of Oct 18, 2025)

**HQ Wazuh:**
- `/var/ossec/etc/ossec.conf.backup_20251018`
- `/var/ossec/etc/rules/local_rules.xml.backup_20251018`

**AMD64:**
- `/var/ossec/etc/ossec.conf.backup`
- `/var/ossec/etc/ossec.conf.backup_20251018`
- `/var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak`

**Ingest VM:**
- `/var/ossec/etc/ossec.conf.backup_20251018`

### Recovery Procedures

#### Scenario 1: HQ Wazuh Manager Configuration Corruption

**Symptoms:**
- Manager won't start
- Errors in `/var/ossec/logs/ossec.log`

**Recovery Steps:**

```bash
ssh leonel@192.168.10.156

# Stop the manager
sudo systemctl stop wazuh-manager

# Restore last known good config
sudo cp /var/ossec/etc/ossec.conf.backup_20251018 /var/ossec/etc/ossec.conf

# Verify syntax
sudo /var/ossec/bin/wazuh-control start

# Restart normally
sudo systemctl start wazuh-manager
sudo systemctl status wazuh-manager
```

#### Scenario 2: Custom Rules Lost/Corrupted

**Recovery Steps:**

```bash
ssh leonel@192.168.10.156

# Restore custom rules
sudo cp /var/ossec/etc/rules/local_rules.xml.backup_20251018 /var/ossec/etc/rules/local_rules.xml

# Restart manager to reload rules
sudo systemctl restart wazuh-manager

# Verify rules loaded
sudo grep "local_rules" /var/ossec/logs/ossec.log
```

#### Scenario 3: AMD64 Agent Configuration Lost

**Recovery Steps:**

```bash
ssh leonel@192.168.10.26
su -

# Stop agent
systemctl stop wazuh-agent

# Restore config (use the SCA working backup!)
cp /var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak /var/ossec/etc/ossec.conf

# Start agent
systemctl start wazuh-agent
systemctl status wazuh-agent

# Verify connection
tail -f /var/ossec/logs/ossec.log
```

#### Scenario 4: Complete HQ Wazuh Failure

**Method 1: Restore from VM Snapshot**
- Use Hyper-V to restore latest snapshot
- Boot VM
- Verify services start
- Check agents reconnect automatically

**Method 2: Fresh Install + Config Restore**

```bash
# Install fresh Ubuntu 22.04.5 LTS
# Run Wazuh all-in-one installer
curl -sO https://packages.wazuh.com/4.13/wazuh-install.sh
sudo bash ./wazuh-install.sh -a

# Restore configuration files
sudo systemctl stop wazuh-manager
# Copy ossec.conf and local_rules.xml from backups
sudo systemctl start wazuh-manager

# Re-enroll agents if needed
```

---

## 10. Next Steps & Lab Ideas

### Immediate Next Steps (This Week)

#### 1. Enable Email Alerts (Optional)

Configure Wazuh to send email alerts for critical events:

```bash
ssh leonel@192.168.10.156
sudo nano /var/ossec/etc/ossec.conf
```

Add email configuration:

```xml
<global>
  <email_notification>yes</email_notification>
  <smtp_server>smtp.gmail.com</smtp_server>
  <email_from>your-email@gmail.com</email_from>
  <email_to>your-alert-email@gmail.com</email_to>
</global>

<alerts>
  <email_alert_level>10</email_alert_level>
</alerts>
```

Then restart:

```bash
sudo systemctl restart wazuh-manager
```

#### 2. Create Hyper-V Snapshots

- HQ Wazuh VM: "Baseline_Production_Oct2025"
- Ingest VM: "Baseline_Production_Oct2025"
- AMD64: (if virtual) "Baseline_Production_Oct2025"

#### 3. Review First Week of Alerts

- Check most common alerts
- Tune rules if needed
- Identify any security issues to address

### Short-Term Improvements (Next Month)

| Improvement | Description |
|-------------|-------------|
| **Add Windows Server Monitoring** | Install Wazuh agent on Windows Server host, monitor Hyper-V events |
| **Expand Compliance Scanning** | Add PCI-DSS scanning, enable HIPAA compliance checks |
| **Implement Active Response** | Auto-block IPs, quarantine files, restart containers |

### Lab Ideas - Hands-On Practice

#### Beginner Labs

**Lab 1: Trigger FIM Alerts**

```bash
ssh leonel@192.168.10.26
su -

# Create a test file
echo "test" > /root/lab/test.txt

# Modify it
echo "modified" > /root/lab/test.txt

# Check Dashboard for FIM alerts
```

**Lab 2: Docker Container Lifecycle**

```bash
ssh leonel@192.168.10.26
su -

# Stop a container
docker stop pihole

# Check Dashboard for alert

# Start it back
docker start pihole
```

**Lab 3: Simulate Brute Force Attack**

```bash
# From another machine on network
ssh fake-user@192.168.10.26
# Try wrong password 5+ times

# Check Dashboard for Fail2Ban ban alert
```

#### Intermediate Labs

**Lab 4: Create Custom Rule**

```bash
ssh leonel@192.168.10.156
sudo nano /var/ossec/etc/rules/local_rules.xml
```

Add new rule:

```xml
<rule id="100050" level="5">
  <decoded_as>json</decoded_as>
  <field name="log">test-string</field>
  <description>Test rule - custom log pattern detected</description>
  <group>custom_test,</group>
</rule>
```

Test it:

```bash
ssh leonel@192.168.10.26
logger "test-string in my log"
# Check Dashboard for alert
```

**Lab 5: Compliance Report Generation**

- Dashboard → Modules → Security Configuration Assessment
- Select AMD64 agent
- Review failed checks
- Remediate 3 failed checks
- Re-scan and verify improvement

**Lab 6: Log Analysis with Logtest**

```bash
ssh leonel@192.168.10.156
sudo /var/ossec/bin/wazuh-logtest
# Paste sample log lines
# See what rules match
```

#### Advanced Labs

**Lab 7: Build VirusTotal Integration**

- Sign up for VirusTotal API key
- Configure Wazuh to check file hashes
- Test with known malware hash

**Lab 8: Create Alert Correlation Rule**

Create rule to detect if same IP triggers Fail2Ban ban AND has FIM alert

**Lab 9: Active Response Configuration**

```bash
ssh leonel@192.168.10.156
sudo nano /var/ossec/etc/ossec.conf
```

Add active response:

```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100010</rules_id>
  <timeout>300</timeout>
</active-response>
```

**Lab 10: GeoIP Investigation**

- Enable GeoIP in Dashboard
- Generate map of authentication attempts
- Identify suspicious countries

---

## 11. Troubleshooting Guide

### Issue: Agent Shows "Disconnected" Status

**Symptoms:**
- Agent shows red in Dashboard
- Last seen time is old
- No new events from agent

**Diagnosis:**

```bash
# On HQ
ssh leonel@192.168.10.156
sudo /var/ossec/bin/agent_control -i 002
sudo grep "192.168.10.26" /var/ossec/logs/ossec.log

# On AMD64
ssh leonel@192.168.10.26
sudo systemctl status wazuh-agent
sudo tail -50 /var/ossec/logs/ossec.log
```

**Solutions:**

```bash
# On agent
sudo systemctl restart wazuh-agent

# Check connectivity
ping 192.168.10.156

# Verify firewall
sudo ufw status

# Re-register if needed
sudo /var/ossec/bin/agent-auth -m 192.168.10.156
```

---

### Issue: SCA Scans Not Running on AMD64

**Symptoms:**
- No compliance data in Dashboard
- Last scan time never updates

**Diagnosis:**

```bash
# On AMD64
ssh leonel@192.168.10.26
sudo grep -i "sca" /var/ossec/logs/ossec.log | tail -20
ls -la /var/ossec/ruleset/sca/cis_debian*.yml
sudo grep -A 20 "<sca>" /var/ossec/etc/ossec.conf
```

**Solutions:**

```bash
# Verify correct configuration
sudo grep "cis_debian13.yml" /var/ossec/etc/ossec.conf

# Restore working config
sudo cp /var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-agent

# Check for errors
sudo grep -i "error\|warning" /var/ossec/logs/ossec.log | grep -i sca
```

---

### Issue: Custom Rules Not Triggering

**Symptoms:**
- Events happening but no alerts
- Rule IDs 100xxx not appearing

**Diagnosis:**

```bash
# On HQ Wazuh
sudo tail -100 /var/ossec/logs/alerts/alerts.log
sudo grep "100001\|100010\|100040" /var/ossec/logs/alerts/alerts.log
```

**Solutions:**

```bash
# Check if rules loaded
sudo grep "local_rules" /var/ossec/logs/ossec.log

# Verify rule file syntax
sudo cat /var/ossec/etc/rules/local_rules.xml

# Restart manager
sudo systemctl restart wazuh-manager

# Check manager logs for errors
sudo tail -50 /var/ossec/logs/ossec.log | grep -i error
```

---

### Issue: High Memory Usage on HQ

**Symptoms:**
- System slow
- Dashboard unresponsive

**Diagnosis:**

```bash
ssh leonel@192.168.10.156
free -h
sudo systemctl status wazuh-indexer
```

**Solutions:**

```bash
# Check Indexer heap size
sudo grep -i "Xms\|Xmx" /etc/wazuh-indexer/jvm.options

# Reduce retention if needed
# Delete old indices via Dashboard

# Restart services
sudo systemctl restart wazuh-indexer
```

---

### Issue: Docker Logs Not Being Collected

**Symptoms:**
- No Docker alerts
- Container logs not in Dashboard

**Diagnosis:**

```bash
# On AMD64
sudo grep "docker" /var/ossec/logs/ossec.log
sudo grep "Analyzing.*json.log" /var/ossec/logs/ossec.log
```

**Solutions:**

```bash
# Check log file permissions
ls -la /var/lib/docker/containers/*/*-json.log

# Fix permissions if needed
sudo chmod 644 /var/lib/docker/containers/*/*-json.log

# Verify configuration
sudo grep -A 5 "docker.*json.log" /var/ossec/etc/ossec.conf

# Restart agent
sudo systemctl restart wazuh-agent
```

---

### Issue: Dashboard Shows "503 Service Unavailable"

**Symptoms:**
- Cannot access https://192.168.10.156
- 503 error in browser

**Diagnosis:**

```bash
ssh leonel@192.168.10.156
sudo systemctl status wazuh-dashboard
sudo systemctl status wazuh-indexer
```

**Solutions:**

```bash
# Wait 2-3 minutes after boot (Indexer startup time)

# Restart services in order
sudo systemctl restart wazuh-indexer
sleep 30
sudo systemctl restart wazuh-manager
sleep 10
sudo systemctl restart wazuh-dashboard

# Check logs
sudo journalctl -u wazuh-dashboard -f
```

---

### Issue: FIM Not Detecting Changes

**Symptoms:**
- File changes not generating alerts
- No syscheck alerts

**Diagnosis:**

```bash
# On AMD64
sudo grep "syscheck" /var/ossec/logs/ossec.log | tail -20
sudo grep "Monitoring path" /var/ossec/logs/ossec.log
```

**Solutions:**

```bash
# Verify FIM is enabled
sudo grep -A 10 "<syscheck>" /var/ossec/etc/ossec.conf

# Check if path is monitored
sudo grep "/root/lab" /var/ossec/etc/ossec.conf

# Force scan (run from HQ)
sudo /var/ossec/bin/agent_control -r -u 002

# Check scan completion
sudo grep "scan ended" /var/ossec/logs/ossec.log
```

---

## 12. Performance Metrics

### Current Resource Usage

#### HQ Wazuh (192.168.10.156)

| Resource | Usage |
|----------|-------|
| **CPU** | ~15-20% average |
| **Memory** | ~5GB / 16GB (32%) |
| **Disk** | 20GB / 96GB (21%) |
| **Network** | <1 Mbps |

#### AMD64 (192.168.10.26)

| Resource | Usage |
|----------|-------|
| **CPU** | <10% average |
| **Memory** | 1GB / 7.7GB (13%) |
| **Disk** | 7.9GB / 291GB (3%) |
| **Wazuh Agent** | ~40MB RAM |

#### Ingest VM (192.168.10.189)

| Resource | Usage |
|----------|-------|
| **CPU** | <5% average |
| **Memory** | <500MB / 2GB |
| **Wazuh Agent** | ~65MB RAM |

### Expected Growth

With current configuration:

| Metric | Value |
|--------|-------|
| **Log retention** | ~30 days |
| **Daily log volume** | ~500MB |
| **Monthly storage** | ~15GB |
| **Indexer disk usage growth** | ~1GB/week |

---

## 13. Reference Documentation

### Official Wazuh Documentation

| Resource | URL |
|----------|-----|
| **Main Docs** | https://documentation.wazuh.com |
| **API Reference** | https://documentation.wazuh.com/current/user-manual/api/ |
| **Rule Syntax** | https://documentation.wazuh.com/current/user-manual/ruleset/ |

### CIS Benchmarks

| Resource | URL |
|----------|-----|
| **CIS Debian** | https://www.cisecurity.org/benchmark/debian_linux |

### Compliance Frameworks

| Framework | URL |
|-----------|-----|
| **PCI-DSS** | https://www.pcisecuritystandards.org |
| **HIPAA** | https://www.hhs.gov/hipaa |
| **NIST 800-53** | https://nvd.nist.gov/800-53 |

### MITRE ATT&CK

| Resource | URL |
|----------|-----|
| **Framework** | https://attack.mitre.org |
| **Techniques** | https://attack.mitre.org/techniques/ |

---

## 14. Important Notes

### Security Considerations

| Consideration | Recommendation |
|---------------|----------------|
| **⚠️ Change Default Passwords** | All service passwords listed in this document should be changed in a production environment |
| **⚠️ Network Exposure** | For internet exposure, implement: Strong authentication (2FA), SSL/TLS certificates, VPN access, IP whitelisting |
| **⚠️ Log Retention** | Current retention is ~30 days. Adjust based on compliance requirements |
| **⚠️ Backup Strategy** | Implement automated backups of: Wazuh configurations, Custom rules, Indexer data, VM snapshots |

### Maintenance Schedule

| Frequency | Tasks |
|-----------|-------|
| **Daily** | Monitor Dashboard for critical alerts |
| **Weekly** | Review compliance scores, check agent status |
| **Monthly** | Clean old logs, verify backups, update systems |
| **Quarterly** | Review and tune rules, update policies |

---

## 15. Quick Reference

### Common Commands

**Check agent status (HQ):**

```bash
sudo /var/ossec/bin/agent_control -l
sudo /var/ossec/bin/agent_control -i 002
```

**Restart services:**

```bash
# On HQ
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard

# On AMD64/Ingest
sudo systemctl restart wazuh-agent
```

**View logs:**

```bash
# Manager logs
sudo tail -f /var/ossec/logs/ossec.log

# Alerts
sudo tail -f /var/ossec/logs/alerts/alerts.log

# Agent logs
sudo tail -f /var/ossec/logs/ossec.log
```

**Test rules:**

```bash
# On HQ
sudo /var/ossec/bin/wazuh-logtest
```

---

## Conclusion

You now have a production-ready SIEM monitoring your homelab infrastructure with:

- ✅ Real-time threat detection
- ✅ Compliance monitoring (CIS Debian 13)
- ✅ Container security
- ✅ File integrity monitoring
- ✅ Intrusion detection
- ✅ Centralized logging
- ✅ Custom detection rules
- ✅ Automated security scanning

### This Infrastructure Provides:

- Hands-on experience with enterprise SIEM tools
- Real security monitoring for your homelab
- Foundation for advanced security labs
- Compliance and audit capabilities
- Incident response readiness

### You're Ready To:

- Detect and respond to security incidents
- Build advanced detection use cases
- Expand monitoring to additional systems
- Develop custom security automations
- Practice threat hunting and forensics

---

**Document Version:** 1.0  
**Last Updated:** October 18, 2025  
**System Status:** ✅ Fully Operational  
**Next Review Date:** November 18, 2025

---

**End of Document**
