🛡️ Wazuh SIEM - Complete Implementation Summary
Implementation Date: October 17-18, 2025
Platform: Hyper-V on Windows Server
Wazuh Version: 4.13.1
Status: ✅ Production-Ready

📋 Table of Contents

Executive Summary
Architecture Overview
Access Credentials
VM Roles & Configurations
Monitoring Capabilities
Custom Security Rules
What Was Accomplished
Health Check Procedures
Backup & Recovery
Next Steps & Lab Ideas
Troubleshooting Guide


🎯 Executive Summary
What We Built
A complete Security Information and Event Management (SIEM) system monitoring 3 servers across your homelab network, providing:

Real-time security monitoring of all systems
Compliance scanning against CIS Debian 13 benchmarks
Container security monitoring for 5 Docker services
Intrusion detection via Fail2Ban integration
File integrity monitoring with realtime alerts
Centralized logging and alert correlation

Key Achievements
✅ Migrated HQ Wazuh from Proxmox to Hyper-V
✅ Upgraded entire stack from v4.12.0 → v4.13.1
✅ Configured 2 active agents monitoring production systems
✅ Implemented custom detection rules for Docker, Pi-hole, Fail2Ban
✅ Enabled CIS compliance scanning (200+ security checks)
✅ Established network segregation (VLAN 10 for VMs, VLAN 20 for host)
✅ All services auto-start on boot and survive reboots

🏗️ Architecture Overview
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
Network Configuration
ComponentIP AddressVLANGatewayRoleHQ Wazuh SIEM192.168.10.15610192.168.10.1Central Security ManagerIngest VM192.168.10.18910192.168.10.1Monitored AgentAMD64 Server192.168.10.2610192.168.10.1Production ServerHyper-V Host192.168.20.x20192.168.20.1Hypervisor
Gateway: Alta Labs Route 10
DNS: Pi-hole (192.168.10.26)

🔐 Access Credentials
Wazuh Dashboard

URL: https://192.168.10.156
Username: admin
Password: wR3SO3NcSmytxjK1jRPPNJzn.?rPmI44

AMD64 Server Services
ServiceURLUsernamePasswordPi-holehttp://192.168.10.26/adminadminMichelle3@91984Portainerhttp://192.168.10.26:9000adminMichelle3@91984Netdatahttp://192.168.10.26:19999-(no auth)Uptime Kumahttp://192.168.10.26:3001adminMichelle3@91984Dozzlehttp://192.168.10.26:8888-(no auth)
SSH Access

HQ Wazuh: ssh leonel@192.168.10.156
Ingest VM: ssh leonel@192.168.10.189
AMD64: ssh leonel@192.168.10.26 (then su - for root)


🖥️ VM Roles & Configurations
1. HQ Wazuh SIEM (192.168.10.156)
Operating System: Ubuntu 22.04.5 LTS
Resources: 8 vCPU, 16GB RAM
Wazuh Version: 4.13.1
Components:
Wazuh Manager

Processes security events from all agents
Applies detection rules (built-in + custom)
Manages agent configurations
Performs threat correlation
Status: Active, auto-starts on boot

Wazuh Indexer

Stores all security data
Indexes logs for fast searching
Retains historical data
Status: Active, auto-starts on boot

Wazuh Dashboard

Web UI for security monitoring
Real-time alert visualization
Compliance reporting
Agent management interface
Status: Active, accessible via HTTPS

Key Configuration Files:

Manager config: /var/ossec/etc/ossec.conf
Custom rules: /var/ossec/etc/rules/local_rules.xml
Logs: /var/ossec/logs/
Alerts: /var/ossec/logs/alerts/alerts.log

Backups Created:

/var/ossec/etc/ossec.conf.backup_20251018
/var/ossec/etc/rules/local_rules.xml.backup_20251018


2. Wazuh Ingest VM (192.168.10.189)
Operating System: Ubuntu 22.04.5 LTS
Resources: 2 vCPU, 2GB RAM
Wazuh Agent Version: 4.12.0
Purpose: Dedicated log collection and forwarding agent
Current Monitoring:

✅ System logs (journald)
✅ Package management (dpkg)
✅ Active responses
❌ File Integrity Monitoring (disabled)
❌ Rootcheck (disabled)

Configuration Status:

Agent connected and active
Sending logs to HQ every 20 seconds
FIM/Rootcheck can be enabled later if needed

Backups Created:

/var/ossec/etc/ossec.conf.backup_20251018


3. AMD64 Server "debian" (192.168.10.26)
Operating System: Debian GNU/Linux 13 (Trixie)
Resources: Custom hardware server
Wazuh Agent Version: 4.13.1
Purpose: Production server running network services with comprehensive security monitoring
Services Running
Docker Containers (5):

Pi-hole - Network-wide DNS and ad blocking
Portainer - Docker container management
Netdata - Real-time system monitoring
Uptime Kuma - Service uptime monitoring
Dozzle - Docker log viewer

Security Services:

UFW Firewall - Active, logging enabled
Fail2Ban - Protecting SSH from brute force
Unattended-upgrades - Automatic security patches

Comprehensive Monitoring Enabled
1. File Integrity Monitoring (FIM)

Paths Monitored:

/etc - System configuration files
/usr/bin, /usr/sbin - System binaries
/bin, /sbin - Core binaries
/boot - Boot files
/root - Root home directory
/root/lab - Docker configs (REALTIME monitoring)


Detection Capabilities:

File modifications
Permission changes
Ownership changes
File creation/deletion
Content changes (with diff)



2. Docker Container Monitoring

All 5 container JSON logs being analyzed
Container status monitoring every 5 minutes
Error detection in container logs
Container restart/crash detection

3. Log Collection

System Logs: journald (all systemd logs)
Package Management: /var/log/dpkg.log
Fail2Ban: /var/log/fail2ban.log
Security Updates: /var/log/unattended-upgrades/
Docker Containers: All 5 container logs (JSON format)

4. System Commands (every 6 minutes)

Disk usage (df -P)
Network connections (netstat)
User logins (last -n 20)
Docker container status

5. Rootcheck (Rootkit Detection)

Scans every 12 hours
Checks for trojans, rootkits, hidden processes
Port scan detection
System anomaly detection

6. Syscollector (System Inventory)

Hardware inventory (CPU, RAM, disk)
OS information
Network interfaces
Installed packages
Open ports
Running processes
Updates hourly

7. Security Configuration Assessment (SCA)

Policy: CIS Debian 13 Benchmark
Checks: ~200 security configuration tests
Frequency: Every 12 hours
Compliance Standards:

CIS Benchmarks
PCI-DSS
HIPAA
NIST SP 800-53
ISO 27001



Example SCA Checks:

Is SSH root login disabled?
Are passwords strong enough?
Is firewall enabled and configured?
Are critical patches installed?
File permission checks
Service hardening verification

Configuration Files:

Agent config: /var/ossec/etc/ossec.conf
SCA policy: /var/ossec/ruleset/sca/cis_debian13.yml
Logs: /var/ossec/logs/ossec.log

Backups Created:

/var/ossec/etc/ossec.conf.backup (original)
/var/ossec/etc/ossec.conf.backup_20251018 (latest)
/var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak (SCA working)


🔍 Monitoring Capabilities
Real-Time Monitoring
CapabilityAMD64Ingest VMHQFile Integrity Monitoring✅ Realtime on /root/lab❌ Disabled✅ Self-monitoringRootkit Detection✅ Every 12h❌ Disabled✅ Every 12hLog Collection✅ Full✅ Basic✅ SelfDocker Monitoring✅ All 5 containersN/AN/ASecurity Compliance✅ CIS Debian 13❌ Not configured✅ SelfIntrusion Detection✅ Fail2BanN/AN/ASystem Inventory✅ Hourly✅ Hourly✅ Hourly
Alert Levels
LevelDescriptionExample3Low - InformationalDocker container started, user login5Medium - Notable eventSecurity update installed7High - Attention neededFile modified, service error10Critical - Immediate actionDocker config changed, critical file modified12Critical - Service downPi-hole container crashed

�� Custom Security Rules
Custom Rules Created (HQ Wazuh)
Location: /var/ossec/etc/rules/local_rules.xml
Docker Monitoring Rules
Rule IDLevelDescription1000013Docker container event detected1000057Error detected in container logs
Fail2Ban Rules
Rule IDLevelDescription1000108IP address banned by Fail2Ban1000113IP address unbanned1000123Fail2Ban jail started
Pi-hole Rules
Rule IDLevelDescription1000217Pi-hole FTL error or warning
Security Updates Rules
Rule IDLevelDescription1000303Security packages being upgraded1000315All upgrades completed successfully
Critical Configuration Rules
Rule IDLevelDescription10004010CRITICAL: Docker compose config modified1000413Pi-hole database updated (normal)
Rule Triggers
What triggers alerts:

Any file change in /root/lab/ (Docker configs)
Docker container errors in logs
Fail2Ban banning an IP
Security updates being installed
Pi-hole FTL errors


✅ What Was Accomplished
Infrastructure

VM Migration

Migrated HQ Wazuh from Proxmox to Hyper-V
Configured static MAC addresses
Disabled cloud-init network management
Established VLAN segregation


Upgrade & Modernization

Upgraded Wazuh Manager: 4.12.0 → 4.13.1
Upgraded Wazuh Indexer: 4.12.0 → 4.13.1
Upgraded Wazuh Dashboard: 4.12.0 → 4.13.1
Installed latest agent on AMD64 (4.13.1)


Network Configuration

VLAN 10: All VMs (192.168.10.x)
VLAN 20: Hyper-V host (192.168.20.x)
Static IPs survive reboots
All services auto-start on boot



Security Monitoring

AMD64 Comprehensive Setup

Configured FIM on critical directories
Enabled realtime monitoring on Docker configs
Set up Docker container log collection (all 5 containers)
Integrated Fail2Ban logs
Added unattended-upgrades monitoring
Enabled rootcheck for rootkit detection
Configured syscollector for inventory


Compliance Scanning

Created CIS Debian 13 policy from Debian 11 template
Enabled SCA module on AMD64
Running 200+ security checks every 12 hours
Fixed policy validation errors
Scans completing successfully


Custom Detection Rules

Created 10 custom rules for Docker, Fail2Ban, Pi-hole
Implemented critical file change alerting
Set up multi-level alert severity
Tested and verified all rules working



Testing & Validation

Alert Testing

Generated test events (Docker restart, file changes, Fail2Ban)
Verified alerts appearing in HQ Dashboard
Confirmed custom rules triggering correctly
Validated agent connectivity


Health Checks

All Docker containers running and healthy
Pi-hole DNS active and blocking ads
Wazuh agents connected and sending data
FIM scans completing every 12 hours
SCA scans completing successfully
System resources healthy (1GB/7.7GB RAM, 3% disk)




🔧 Health Check Procedures
Daily Checks
Quick Status Check (AMD64)
bash# Check all services
docker ps
sudo systemctl status wazuh-agent

# Check Wazuh monitoring
sudo tail -20 /var/ossec/logs/ossec.log
Quick Dashboard Check (HQ)

Open https://192.168.10.156
Login with admin credentials
Check Security Events → Filter by today
Verify agents are Active in Agents tab

Weekly Checks
Detailed Agent Check (HQ)
bashssh leonel@192.168.10.156

# Check agent statistics
sudo /var/ossec/bin/agent_control -i 002  # AMD64
sudo /var/ossec/bin/agent_control -i 001  # Ingest VM

# Check recent alerts
sudo tail -100 /var/ossec/logs/alerts/alerts.log
Compliance Check (Dashboard)

Navigate to Security Configuration Assessment
Select agent debian (002)
Review CIS Debian 13 compliance score
Address any failed checks

Monthly Checks
System Health
bash# On AMD64
free -h  # Memory usage
df -h    # Disk usage
docker stats --no-stream  # Container resources

# On HQ Wazuh
sudo du -sh /var/ossec/logs/  # Log size
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard
Backup Verification
bash# Verify backups exist
ls -lh /var/ossec/etc/*.backup*

💾 Backup & Recovery
Current Backups
HQ Wazuh (192.168.10.156)

/var/ossec/etc/ossec.conf.backup_20251018
/var/ossec/etc/rules/local_rules.xml.backup_20251018

AMD64 (192.168.10.26)

/var/ossec/etc/ossec.conf.backup
/var/ossec/etc/ossec.conf.backup_20251018
/var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak

Ingest VM (192.168.10.189)

/var/ossec/etc/ossec.conf.backup_20251018

Recovery Procedures
Restore AMD64 Wazuh Config
bash# If agent fails to start or misconfigured
sudo cp /var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
Restore HQ Custom Rules
bash# If rules cause manager to fail
sudo cp /var/ossec/etc/rules/local_rules.xml.backup_20251018 /var/ossec/etc/rules/local_rules.xml
sudo systemctl restart wazuh-manager
Restore HQ Manager Config
bashsudo cp /var/ossec/etc/ossec.conf.backup_20251018 /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-manager
VM Snapshots
Recommended: Take Hyper-V snapshots of all 3 VMs before major changes:

Before upgrades
Before configuration changes
Before adding new agents
Monthly baseline snapshots


🚀 Next Steps & Lab Ideas
Immediate Enhancements
1. Enable Monitoring on Ingest VM (15 min)
bashssh leonel@192.168.10.189
sudo nano /var/ossec/etc/ossec.conf

# Enable FIM and Rootcheck
# Change <disabled>yes</disabled> to <disabled>no</disabled>

sudo systemctl restart wazuh-agent
2. Configure Email Alerts (30 min)
Add SMTP configuration to HQ Wazuh for critical alerts:

Docker container crashes
Critical file modifications
Fail2Ban bans
SCA compliance failures

3. Create Uptime Kuma Monitors (20 min)
Add monitors in Uptime Kuma for:

Wazuh Dashboard (https://192.168.10.156)
Pi-hole
All Docker containers
Wazuh agents

Beginner Labs
Lab 1: Brute Force Detection
Objective: Trigger Fail2Ban and observe Wazuh alerts
bash# From another machine, SSH to AMD64 with wrong passwords 5 times
ssh wronguser@192.168.10.26

# Check Fail2Ban response
ssh leonel@192.168.10.26
sudo fail2ban-client status sshd

# Check Wazuh alert in Dashboard
# Security Events → Filter: rule.id:100010
Expected Result: IP banned, alert in Wazuh with rule 100010
Lab 2: File Tampering Detection
Objective: Trigger FIM realtime alert
bash# SSH to AMD64
ssh leonel@192.168.10.26
su -

# Modify Docker config
echo "# test comment" >> /root/lab/pihole/docker-compose.yml

# Check alert (should appear within 10 seconds)
# Dashboard → Security Events → Filter: rule.id:100040
Expected Result: CRITICAL alert (level 10) appears immediately
Lab 3: Container Monitoring
Objective: Monitor Docker container lifecycle
bash# Stop Pi-hole container
sudo docker stop pihole

# Wait 30 seconds, check alerts
# Restart Pi-hole
sudo docker start pihole

# Check Dashboard for:
# - Container stopped event
# - Container started event
# - Any errors in logs
Intermediate Labs
Lab 4: Custom Rule Development
Objective: Create a rule to detect specific Pi-hole queries
On HQ Wazuh:
bashsudo nano /var/ossec/etc/rules/local_rules.xml

# Add new rule:
<rule id="100050" level="5">
  <decoded_as>json</decoded_as>
  <match>malware|phishing|suspicious</match>
  <description>Pi-hole: Suspicious domain query detected</description>
  <group>pihole,dns,threat,</group>
</rule>

sudo systemctl restart wazuh-manager
Lab 5: Compliance Remediation
Objective: Fix SCA compliance failures

Check SCA results in Dashboard
Identify failed checks
Follow remediation steps
Re-run scan
Verify compliance improved

Lab 6: Active Response Setup
Objective: Auto-block IPs when Fail2Ban triggers
Configuration needed:

Create active response script on AMD64
Add UFW block command
Configure Wazuh to trigger on rule 100010
Test by triggering Fail2Ban

Advanced Labs
Lab 7: Multi-Agent Correlation
Objective: Correlate events across multiple agents
Scenario: Detect attack pattern:

Failed SSH on AMD64
Failed SSH on Ingest VM
Same source IP
Within 5 minutes

Create correlation rule to trigger high-priority alert
Lab 8: MITRE ATT&CK Mapping
Objective: Tag alerts with MITRE techniques
Example mappings:

Fail2Ban bans → T1110 (Brute Force)
Docker config changes → T1036 (Masquerading)
File modifications → T1565 (Data Manipulation)

Lab 9: Threat Intelligence Integration
Objective: Integrate IP reputation feeds

Add AlienVault OTX or AbuseIPDB feeds
Create rules to check IPs against feeds
Auto-block known malicious IPs
Track threat actor activity

Lab 10: Container Security Scanning
Objective: Scan Docker images for vulnerabilities

Install Trivy or Clair
Scan all Docker images
Send results to Wazuh
Create alerts for critical CVEs

Lab 11: Log Enrichment
Objective: Add GeoIP and threat intel context

Configure GeoIP lookups for source IPs
Add IP reputation scoring
Enrich alerts with context
Create geographic attack maps

Lab 12: Custom Dashboard Creation
Objective: Build executive security dashboard
Include widgets for:

Active threats map
Top attacked services
Compliance score trend
Container health status
Failed login attempts by country

Expert Labs
Lab 13: Kubernetes Cluster Monitoring
Objective: Monitor K8s cluster with Wazuh

Deploy Wazuh agent as DaemonSet
Monitor pod logs
Track container creation/deletion
Detect privilege escalation
Monitor API server access

Lab 14: SOAR Integration
Objective: Automate incident response

Install TheHive + Cortex
Forward high-priority Wazuh alerts
Auto-create cases
Run automated analyzers
Track incident lifecycle

Lab 15: Deception Technology
Objective: Deploy honeypots and detect intrusions

Deploy Cowrie SSH honeypot
Forward honeypot logs to Wazuh
Create high-priority alerts for honeypot access
Track attacker behavior
Correlate with other alerts


🔬 Learning Outcomes
Skills Developed
SIEM Operations:

Centralized log collection and management
Security event correlation
Alert tuning and prioritization
Incident detection and investigation

Detection Engineering:

Writing custom detection rules
Understanding rule logic and syntax
Testing and validating detections
Reducing false positives

Container Security:

Monitoring Docker environments
Detecting container anomalies
Securing container configurations
Log aggregation from containers

Compliance:

CIS benchmark implementation
Security configuration assessment
Compliance reporting
Remediation tracking

Incident Response:

Alert investigation workflows
Evidence collection from logs
Threat hunting techniques
Forensic analysis preparation


🐛 Troubleshooting Guide
Common Issues & Solutions
Issue: Wazuh Agent Not Connecting
Symptoms:

Agent shows "Disconnected" in Dashboard
No recent keepalive

Diagnosis:
bash# On agent (AMD64 or Ingest VM)
sudo systemctl status wazuh-agent
sudo tail -50 /var/ossec/logs/ossec.log | grep -i connect
Solutions:
bash# Check network connectivity
ping 192.168.10.156

# Test Wazuh manager port
telnet 192.168.10.156 1514

# Restart agent
sudo systemctl restart wazuh-agent

# Check manager address in config
sudo grep "<address>" /var/ossec/etc/ossec.conf

Issue: SCA Scan Not Running
Symptoms:

No SCA results in Dashboard
"Skipping policy" messages in logs

Diagnosis:
bash# On AMD64
sudo grep -i sca /var/ossec/logs/ossec.log | tail -20
Solutions:
bash# Verify policy file exists
ls -la /var/ossec/ruleset/sca/cis_debian13.yml

# Check for errors
sudo grep -i "error\|warning" /var/ossec/logs/ossec.log | grep -i sca

# Restore working config
sudo cp /var/ossec/etc/ossec.conf.final_good_config_20251017_221530.bak /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-agent

Issue: Custom Rules Not Triggering
Symptoms:

Events happening but no alerts
Rule IDs 100xxx not appearing

Diagnosis:
bash# On HQ Wazuh
sudo tail -100 /var/ossec/logs/alerts/alerts.log
sudo grep "100001\|100010\|100040" /var/ossec/logs/alerts/alerts.log
Solutions:
bash# Check if rules loaded
sudo grep "local_rules" /var/ossec/logs/ossec.log

# Verify rule file syntax
sudo cat /var/ossec/etc/rules/local_rules.xml

# Restart manager
sudo systemctl restart wazuh-manager

# Check manager logs for errors
sudo tail -50 /var/ossec/logs/ossec.log | grep -i error

Issue: High Memory Usage on HQ
Symptoms:

System slow
Dashboard unresponsive

Diagnosis:
bashssh leonel@192.168.10.156
free -h
sudo systemctl status wazuh-indexer
Solutions:
bash# Check Indexer heap size
sudo grep -i "Xms\|Xmx" /etc/wazuh-indexer/jvm.options

# Reduce retention if needed
# Delete old indices via Dashboard

# Restart services
sudo systemctl restart wazuh-indexer

Issue: Docker Logs Not Being Collected
Symptoms:

No Docker alerts
Container logs not in Dashboard

Diagnosis:
bash# On AMD64
sudo grep "docker" /var/ossec/logs/ossec.log
sudo grep "Analyzing.*json.log" /var/ossec/logs/ossec.log
Solutions:
bash# Check log file permissions
ls -la /var/lib/docker/containers/*/*-json.log

# Fix permissions if needed
sudo chmod 644 /var/lib/docker/containers/*/*-json.log

# Verify configuration
sudo grep -A 5 "docker.*json.log" /var/ossec/etc/ossec.conf

# Restart agent
sudo systemctl restart wazuh-agent

Issue: Dashboard Shows "503 Service Unavailable"
Symptoms:

Cannot access https://192.168.10.156
503 error in browser

Diagnosis:
bashssh leonel@192.168.10.156
sudo systemctl status wazuh-dashboard
sudo systemctl status wazuh-indexer
Solutions:
bash# Wait 2-3 minutes after boot (Indexer startup time)

# Restart services in order
sudo systemctl restart wazuh-indexer
sleep 30
sudo systemctl restart wazuh-manager
sleep 10
sudo systemctl restart wazuh-dashboard

# Check logs
sudo journalctl -u wazuh-dashboard -f

Issue: FIM Not Detecting Changes
Symptoms:

File changes not generating alerts
No syscheck alerts

Diagnosis:
bash# On AMD64
sudo grep "syscheck" /var/ossec/logs/ossec.log | tail -20
sudo grep "Monitoring path" /var/ossec/logs/ossec.log
Solutions:
bash# Verify FIM is enabled
sudo grep -A 10 "<syscheck>" /var/ossec/etc/ossec.conf

# Check if path is monitored
sudo grep "/root/lab" /var/ossec/etc/ossec.conf

# Force scan
sudo /var/ossec/bin/agent_control -r -u 002  # (run from HQ)

# Check scan completion
sudo grep "scan ended" /var/ossec/logs/ossec.log

📊 Performance Metrics
Current Resource Usage
HQ Wazuh (192.168.10.156)

CPU: ~15-20% average
Memory: ~5GB / 16GB (32%)
Disk: 20GB / 96GB (21%)
Network: <1 Mbps

AMD64 (192.168.10.26)

CPU: <10% average
Memory: 1GB / 7.7GB (13%)
Disk: 7.9GB / 291GB (3%)
Wazuh Agent: ~40MB RAM

Ingest VM (192.168.10.189)

CPU: <5% average
Memory: <500MB / 2GB
Wazuh Agent: ~65MB RAM

Expected Growth
With current configuration:

Log retention: ~30 days
Daily log volume: ~500MB
Monthly storage: ~15GB
Indexer disk usage growth: ~1GB/week


📚 Reference Documentation
Official Wazuh Documentation

Main Docs: https://documentation.wazuh.com
API Reference: https://documentation.wazuh.com/current/user-manual/api/
Rule Syntax: https://documentation.wazuh.com/current/user-manual/ruleset/

CIS Benchmarks

CIS Debian: https://www.cisecurity.org/benchmark/debian_linux

Compliance Frameworks

PCI-DSS: https://www.pcisecuritystandards.org
HIPAA: https://www.hhs.gov/hipaa
NIST 800-53: https://nvd.nist.gov/800-53

MITRE ATT&CK

Framework: https://attack.mitre.org
Techniques: https://attack.mitre.org/techniques/


📝 Important Notes
Security Considerations
⚠️ Change Default Passwords: All service passwords listed in this document should be changed in a production environment.
⚠️ Network Exposure: Wazuh Dashboard is currently accessible on the local network. For internet exposure, implement:

Strong authentication (2FA)
SSL/TLS certificates
VPN access
IP whitelisting

⚠️ Log Retention: Current retention is ~30 days. Adjust based on compliance requirements.
⚠️ Backup Strategy: Implement automated backups of:

Wazuh configurations
Custom rules
Indexer data
VM snapshots

Maintenance Schedule
Daily: Monitor Dashboard for critical alerts
Weekly: Review compliance scores, check agent status
Monthly: Clean old logs, verify backups, update systems
Quarterly: Review and tune rules, update policies

🎓 Conclusion
You now have a production-ready SIEM monitoring your homelab infrastructure with:
✅ Real-time threat detection
✅ Compliance monitoring (CIS Debian 13)
✅ Container security
✅ File integrity monitoring
✅ Intrusion detection
✅ Centralized logging
✅ Custom detection rules
✅ Automated security scanning
This infrastructure provides:

Hands-on experience with enterprise SIEM tools
Real security monitoring for your homelab
Foundation for advanced security labs
Compliance and audit capabilities
Incident response readiness

You're ready to:

Detect and respond to security incidents
Build advanced detection use cases
Expand monitoring to additional systems
Develop custom security automations
Practice threat hunting and forensics


📞 Quick Reference
Common Commands
Check agent status (HQ):
bashsudo /var/ossec/bin/agent_control -l
sudo /var/ossec/bin/agent_control -i 002
Restart services:
bash# On HQ
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard

# On AMD64/Ingest
sudo systemctl restart wazuh-agent
View logs:
bash# Manager logs
sudo tail -f /var/ossec/logs/ossec.log

# Alerts
sudo tail -f /var/ossec/logs/alerts/alerts.log

# Agent logs
sudo tail -f /var/ossec/logs/ossec.log
Test rules:
bash# On HQ
sudo /var/ossec/bin/wazuh-logtest
