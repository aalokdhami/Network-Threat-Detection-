# Adaptive Threat Detection System
## Complete Project Summary & Implementation Guide

**Author:** Aalok  
**Date:** April 2026  
**Version:** 1.0

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Component Overview](#component-overview)
4. [Installation Guide](#installation-guide)
5. [Configuration Files](#configuration-files)
6. [Source Code Documentation](#source-code-documentation)
7. [Testing Procedures](#testing-procedures)
8. [Attack Simulation Commands](#attack-simulation-commands)
9. [Troubleshooting](#troubleshooting)
10. [Appendix: Complete Code Listings](#appendix)

---

## 1. Executive Summary

The Adaptive Threat Detection System is an intelligent network security platform that combines:
- **Snort IDS** for signature-based detection
- **Custom Python parsers** for real-time log analysis and OpenVAS enrichment  
- **Behavioral correlation engine** for advanced pattern detection
- **Splunk SIEM** for visualization and alerting

### Key Capabilities
- Detects DoS/DDoS attacks (SYN flood, ICMP flood, HTTP flood)
- Identifies port scanning activity (NULL, FIN, XMAS, SYN scans)
- Recognizes brute force attacks (SSH, FTP, MySQL)
- Correlates multi-service targeting patterns
- Enriches alerts with vulnerability context from OpenVAS

---

## 2. System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Network Traffic (192.168.18.0/24)          │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
         ┌──────────────────────────────┐
         │   Snort IDS 2.9.20           │
         │   • 17 custom rules          │
         │   • Signature detection      │
         └──────────┬───────────────────┘
                    │ /var/log/snort/alert
                    ▼
         ┌──────────────────────────────┐
         │   parser.py                  │
         │   • Log parsing              │
         │   • OpenVAS enrichment       │
         └──────────┬───────────────────┘
                    │ parsed_alerts.json
                    ▼
         ┌──────────────────────────────┐
         │   behavior_engine.py         │
         │   • 6 correlation rules      │
         │   • Pattern detection        │
         └──────────┬───────────────────┘
                    │ behavior_alerts.json
                    ▼
         ┌──────────────────────────────┐
         │   Splunk Enterprise          │
         │   • Dashboard visualization  │
         └──────────────────────────────┘
```

---

## 3. Component Overview

### 3.1 Snort IDS
- **Version:** 2.9.20
- **Mode:** Fast alert mode
- **Interface:** enp0s3 (Bridged adapter)
- **Rules:** 17 custom detection rules

### 3.2 Parser (parser.py)
- **Language:** Python 3
- **Function:** Real-time Snort log parsing
- **Features:**
  - Auto-subnet detection
  - OpenVAS vulnerability enrichment
  - Risk score calculation
  - JSONL output format

### 3.3 Behavior Engine (behavior_engine.py)
- **Language:** Python 3
- **Function:** Behavioral correlation and pattern detection
- **Rules:**
  1. Multi-service targeting (3+ events, 2+ services in 60s)
  2. SSH brute force (10+ attempts in 120s)
  3. Port scan (5+ unique ports in 30s)
  4. FTP brute force (5+ attempts in 60s)
  5. MySQL brute force (5+ attempts in 60s)
  6. Flood/DoS detection (signature-based)

### 3.4 Splunk Dashboard
- **Version:** Enterprise 8.x
- **Port:** 8000
- **Features:** Real-time visualization, risk analytics, attack timelines

---

## 4. Installation Guide

### Step 1: System Preparation

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Snort IDS
sudo apt install snort -y

# Verify Snort installation
snort --version
```

### Step 2: Install Services (Attack Targets)

```bash
# Install FTP server
sudo apt install vsftpd -y

# Configure anonymous FTP access
sudo nano /etc/vsftpd.conf
# Set: anonymous_enable=YES

# Restart FTP
sudo systemctl restart vsftpd

# Install MySQL
sudo apt install mysql-server -y

# Configure MySQL to listen on all interfaces
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
# Set: bind-address = 0.0.0.0

# Restart MySQL
sudo systemctl restart mysql

# Install SSH (usually pre-installed)
sudo apt install openssh-server -y
```

### Step 3: Install Python Dependencies

```bash
# Python 3 is pre-installed on Ubuntu 24
python3 --version

# No external libraries needed (using standard library only)
```

### Step 4: Clone Project Repository

```bash
# Clone from GitHub
git clone https://github.com/yourusername/adaptive-threat-detection.git
cd adaptive-threat-detection

# Or create directory structure manually
sudo mkdir -p /home/aalok/snort_project
sudo mkdir -p /home/aalok/snort_project/openvas_input
```

### Step 5: Deploy Configuration Files

```bash
# Copy Snort rules
sudo cp config/snort/local.rules /etc/snort/rules/

# Edit Snort configuration to include custom rules
sudo nano /etc/snort/snort.conf
# Add at the end:
# include $RULE_PATH/local.rules

# Validate Snort configuration
sudo snort -T -c /etc/snort/snort.conf -S HOME_NET=192.168.18.0/24
```

### Step 6: Deploy Python Scripts

```bash
# Copy project files
sudo cp src/parser.py /home/aalok/snort_project/
sudo cp src/behavior_engine.py /home/aalok/snort_project/
sudo cp src/start.sh /home/aalok/snort_project/

# Copy OpenVAS database
sudo cp config/openvas/sample_openvas_report.csv /home/aalok/snort_project/openvas_input/

# Make scripts executable
sudo chmod +x /home/aalok/snort_project/start.sh
sudo chmod +x /home/aalok/snort_project/parser.py
sudo chmod +x /home/aalok/snort_project/behavior_engine.py
```

### Step 7: Install and Configure Splunk

```bash
# Download Splunk Enterprise
wget -O splunk.deb 'https://download.splunk.com/products/splunk/releases/...'

# Install
sudo dpkg -i splunk.deb

# Start Splunk
sudo /opt/splunk/bin/splunk start --accept-license

# Enable boot-start
sudo /opt/splunk/bin/splunk enable boot-start

# Access web interface
# URL: http://192.168.18.203:8000
# Default: admin/changeme
```

### Step 8: Configure Splunk Data Input

```bash
# Add data input for parsed_alerts.json
# In Splunk Web UI:
# Settings → Data Inputs → Files & Directories → New
# File path: /home/aalok/snort_project/parsed_alerts.json
# Source type: _json
# Index: main

# Add data input for behavior_alerts.json
# File path: /home/aalok/snort_project/behavior_alerts.json
# Source type: _json
# Index: main
```

---

## 5. Configuration Files

### 5.1 Snort Rules (`/etc/snort/rules/local.rules`)

#### DoS/DDoS Detection Rules

```bash
# SID 100011: HTTP Flood Detection  
alert tcp any any -> $HOME_NET 80 (msg:"HTTP FLOOD DETECTED"; flow:to_server,established; content:"GET"; http_method; detection_filter:track by_src,count 100,seconds 10; sid:100011; rev:1;)

# SID 100012: SYN Flood Detection
alert tcp any any -> $HOME_NET any (msg:"SYN FLOOD DETECTED"; flags:S; detection_filter:track by_src,count 200,seconds 5; sid:100012; rev:1;)

# SID 100013: ICMP Flood Detection
alert icmp any any -> $HOME_NET any (msg:"ICMP FLOOD DETECTED"; detection_filter:track by_src,count 50,seconds 5; sid:100013; rev:1;)
```

#### Port Scan Detection Rules

```bash
# SID 100014: Nmap NULL Scan (all flags = 0)
alert tcp any any -> $HOME_NET any (msg:"NMAP NULL SCAN DETECTED"; flags:0; sid:100014; rev:1;)

# SID 100015: Nmap FIN Scan (only FIN flag)
alert tcp any any -> $HOME_NET any (msg:"NMAP FIN SCAN DETECTED"; flags:F; sid:100015; rev:1;)

# SID 100016: Nmap XMAS Scan (FIN+PSH+URG flags)
alert tcp any any -> $HOME_NET any (msg:"NMAP XMAS SCAN DETECTED"; flags:FPU; sid:100016; rev:1;)

# SID 100017: Nmap SYN Scan (repeated SYN packets)
alert tcp any any -> $HOME_NET any (msg:"NMAP SYN SCAN DETECTED"; flags:S; detection_filter:track by_src,count 20,seconds 5; sid:100017; rev:1;)
```

#### Brute Force Detection Rules

```bash
# SID 100002: SSH Brute Force
alert tcp any any -> $HOME_NET 22 (msg:"REPEATED SSH CONNECTION ATTEMPTS"; flags:S; detection_filter:track by_src,count 5,seconds 60; sid:100002; rev:1;)

# SID 100007: FTP Brute Force
alert tcp any any -> $HOME_NET 21 (msg:"FTP BRUTE FORCE ATTEMPT"; flow:to_server,established; content:"USER"; nocase; detection_filter:track by_src,count 5,seconds 60; sid:100007; rev:1;)

# SID 100009: MySQL Brute Force
alert tcp any any -> $HOME_NET 3306 (msg:"MYSQL BRUTE FORCE ATTEMPT"; flow:to_server; detection_filter:track by_src,count 5,seconds 60; sid:100009; rev:1;)
```

### 5.2 OpenVAS Vulnerability Database

**File:** `/home/aalok/snort_project/openvas_input/sample_openvas_report.csv`

```csv
Host,Port,Service,Severity,Vulnerability
192.168.18.203,80,http,High,Outdated Apache Server
192.168.18.203,22,ssh,Medium,Weak SSH Configuration
192.168.18.203,21,ftp,High,Anonymous FTP Enabled
192.168.18.203,443,https,Low,Self Signed Certificate
192.168.18.203,3306,mysql,Critical,MySQL Exposed to Network
```

---

## 6. Source Code Documentation

### 6.1 parser.py - Alert Parser

**Purpose:** Parse Snort alerts in real-time and enrich with vulnerability context

**Key Functions:**

```python
def get_home_subnet(interface="enp0s3"):
    """Auto-detect network subnet from interface"""
    # Uses 'ip addr show' command
    # Returns: "192.168.18.0/24"
    
def load_openvas_vulnerabilities():
    """Load vulnerability database from CSV"""
    # Returns: dict{(host, port): {service, severity, description}}
    
def classify_attack_type(signature: str):
    """Classify attack based on signature"""
    # Returns: (attack_type, risk_score, severity)
    
def enrich_with_vulnerability(event: dict, vuln_db: dict):
    """Add vulnerability context to event"""
    # Adds vuln_context field to event
    
def parse_snort_alert(alert_text: str, vuln_db: dict):
    """Parse Snort alert into structured JSON"""
    # Extracts: timestamp, gid, sid, signature, IPs, ports, protocol
    # Classifies: attack_type, risk_score, severity
    # Enriches: vuln_context
```

**Execution:**

```bash
sudo python3 /home/aalok/snort_project/parser.py
```

**Output Format (JSONL):**

```json
{
  "event_time": "2026-04-16T23:35:49.132925",
  "gid": 1,
  "sid": 100017,
  "signature": "NMAP SYN SCAN DETECTED",
  "src_addr": "192.168.18.197",
  "dst_addr": "192.168.18.203",
  "dst_port": 8080,
  "protocol": "TCP",
  "attack_type": "port_scan",
  "risk_score": 60,
  "severity": "medium",
  "vuln_context": {
    "vuln_description": "MySQL Exposed to Network",
    "vuln_severity": "Critical",
    "vuln_service": "mysql"
  }
}
```

### 6.2 behavior_engine.py - Behavioral Correlation

**Purpose:** Detect advanced attack patterns through event correlation

**Detection Rules:**

```python
# Rule 1: Multi-Service Targeting
# Trigger: 3+ events across 2+ services in 60 seconds
# Risk Score: 85 (High)

# Rule 2: SSH Brute Force
# Trigger: 10+ SSH attempts in 120 seconds
# Risk Score: 90 (High)

# Rule 3: Port Scan
# Trigger: 5+ unique destination ports in 30 seconds
# Risk Score: 70 (Medium)

# Rule 4: FTP Brute Force
# Trigger: 5+ FTP attempts in 60 seconds
# Risk Score: 85 (High)

# Rule 5: MySQL Brute Force
# Trigger: 5+ MySQL connection attempts in 60 seconds
# Risk Score: 90 (High)

# Rule 6: Flood/DoS Detection
# Trigger: Snort signatures containing "FLOOD", "EXCESSIVE", "REPEATED"
# Risk Score: 95 (High)
```

**Key Functions:**

```python
class BehaviorEngine:
    def detect_multi_service_targeting(src_addr, current_time):
        """Detect attacks targeting multiple services"""
        
    def detect_ssh_brute_force(src_addr, current_time):
        """Detect SSH brute force attempts"""
        
    def detect_port_scan(src_addr, current_time):
        """Detect port scanning activity"""
        
    def detect_ftp_brute_force(src_addr, current_time):
        """Detect FTP brute force attempts"""
        
    def detect_mysql_brute_force(src_addr, current_time):
        """Detect MySQL brute force attempts"""
        
    def detect_flood_dos(event):
        """Detect flood/DoS attacks from signatures"""
```

**Execution:**

```bash
sudo python3 /home/aalok/snort_project/behavior_engine.py
```

**Output Format:**

```json
{
  "event_time": "2026-04-16T23:40:12.456789",
  "detection_type": "behavioral",
  "signature": "PORT SCAN DETECTED",
  "src_addr": "192.168.18.197",
  "target_host": "192.168.18.203",
  "services_targeted": ["ssh", "http", "mysql", "ftp"],
  "trigger_count": 15,
  "severity": "medium",
  "risk_score": 70,
  "trigger_condition": "5+ unique ports in 30s"
}
```

---

## 7. Testing Procedures

### 7.1 Unit Testing (TC-01 to TC-04)

#### TC-01: Snort Service Startup

```bash
# Start Snort
sudo snort -A fast -c /etc/snort/snort.conf -i enp0s3 -S HOME_NET=192.168.18.0/24 -D

# Verify running
ps aux | grep snort | grep -v grep

# Expected output:
# root  PID  ... snort -A fast -c /etc/snort/snort.conf -i enp0s3...
```

#### TC-02: Parser Script Initialization

```bash
# Start parser
cd /home/aalok/snort_project
sudo python3 parser.py &

# Verify running
ps aux | grep parser.py | grep -v grep

# Check output
tail -f parsed_alerts.json
```

#### TC-03: Behavior Engine Initialization

```bash
# Start behavior engine
sudo python3 behavior_engine.py &

# Verify running
ps aux | grep behavior_engine | grep -v grep

# Expected console output:
# Behavior engine initialized
# Detecting:
#   - Multi-service targeting (60s window)
#   - SSH brute force (10+ attempts in 120s)
#   ...
```

#### TC-04: OpenVAS Enrichment

```bash
# Generate test alert
ping -c 5 192.168.18.203  # From Kali

# Check parsed alert contains vuln_context
tail -1 parsed_alerts.json | python3 -m json.tool | grep vuln_context
```

### 7.2 Security Testing (TC-05 to TC-16)

#### TC-05: ICMP Flood Detection

**From Kali Linux:**

```bash
sudo hping3 -1 --flood 192.168.18.203
# Let run for 10 seconds, then Ctrl+C
```

**Verify on Ubuntu VM:**

```bash
# Check Snort log
sudo tail -20 /var/log/snort/alert | grep "ICMP FLOOD"

# Expected:
# [**] [1:100013:1] ICMP FLOOD DETECTED [**]
```

#### TC-06: SYN Flood Detection

**From Kali Linux:**

```bash
sudo hping3 -S -p 80 --flood 192.168.18.203
# Let run for 10 seconds, then Ctrl+C
```

**Verify on Ubuntu VM:**

```bash
sudo tail -20 /var/log/snort/alert | grep "SYN FLOOD"

# Expected:
# [**] [1:100012:1] SYN FLOOD DETECTED [**]
```

#### TC-07: HTTP Flood Detection

**Start HTTP server on Ubuntu:**

```bash
sudo python3 -m http.server 80
```

**From Kali Linux:**

```bash
ab -n 10000 -c 100 http://192.168.18.203/
```

**Verify on Ubuntu VM:**

```bash
sudo tail -20 /var/log/snort/alert | grep "HTTP FLOOD"
tail -1 behavior_alerts.json | python3 -m json.tool

# Expected behavioral alert:
# "signature": "EXCESSIVE HTTP REQUESTS"
```

#### TC-11: Nmap SYN Scan Detection

**From Kali Linux:**

```bash
sudo nmap -sS 192.168.18.203
```

**Verify on Ubuntu VM:**

```bash
# Snort detection
sudo tail -20 /var/log/snort/alert | grep "NMAP SYN SCAN"

# Behavioral detection
tail -1 behavior_alerts.json | python3 -m json.tool

# Expected behavioral alert:
# "signature": "PORT SCAN DETECTED"
```

#### TC-13: FTP Brute Force Detection

**Create wordlist on Ubuntu:**

```bash
cat > /home/aalok/snort_project/wordlist.txt << EOF
password
123456
admin
root
test
EOF
```

**From Kali Linux:**

```bash
hydra -l anonymous -P /home/aalok/snort_project/wordlist.txt ftp://192.168.18.203
```

**Verify on Ubuntu VM:**

```bash
sudo tail -20 /var/log/snort/alert | grep "FTP BRUTE"
tail -1 behavior_alerts.json | python3 -m json.tool

# Expected behavioral alert:
# "signature": "FTP BRUTE FORCE DETECTED"
```

#### TC-14: MySQL Brute Force Detection

**Create MySQL test user on Ubuntu:**

```bash
sudo mysql -e "CREATE USER IF NOT EXISTS 'testuser'@'%' IDENTIFIED BY 'testpass123';"
sudo mysql -e "FLUSH PRIVILEGES;"
```

**From Kali Linux:**

```bash
hydra -l testuser -P /home/aalok/snort_project/wordlist.txt mysql://192.168.18.203
```

**Verify on Ubuntu VM:**

```bash
sudo tail -20 /var/log/snort/alert | grep "MYSQL BRUTE"
tail -1 behavior_alerts.json | python3 -m json.tool

# Expected behavioral alert:
# "signature": "MYSQL BRUTE FORCE DETECTED"
```

#### TC-15: SSH Brute Force Detection

**From Kali Linux:**

```bash
hydra -l aalok -P /home/aalok/snort_project/wordlist.txt ssh://192.168.18.203
```

**Verify on Ubuntu VM:**

```bash
sudo tail -20 /var/log/snort/alert | grep "SSH"
tail -1 behavior_alerts.json | python3 -m json.tool

# Expected behavioral alert:
# "signature": "SSH BRUTE FORCE DETECTED"
```

---

## 8. Attack Simulation Commands

### Complete Attack Test Suite

```bash
# ===========================================
# DOS/DDOS ATTACKS
# ===========================================

# SYN Flood (from Kali)
sudo hping3 -S -p 80 --flood 192.168.18.203
# Run for 10 seconds, then Ctrl+C

# ICMP Flood (from Kali)
sudo hping3 -1 --flood 192.168.18.203
# Run for 10 seconds, then Ctrl+C

# HTTP Flood (from Kali)
# First, start HTTP server on Ubuntu:
# sudo python3 -m http.server 80
ab -n 10000 -c 100 http://192.168.18.203/

# ===========================================
# PORT SCANNING
# ===========================================

# NULL Scan (from Kali)
sudo nmap -sN 192.168.18.203

# FIN Scan (from Kali)
sudo nmap -sF 192.168.18.203

# XMAS Scan (from Kali)
sudo nmap -sX 192.168.18.203

# SYN Scan (from Kali)
sudo nmap -sS 192.168.18.203

# Comprehensive scan (from Kali)
sudo nmap -sS -sV -p- 192.168.18.203

# ===========================================
# BRUTE FORCE ATTACKS
# ===========================================

# FTP Brute Force (from Kali)
hydra -l anonymous -P wordlist.txt ftp://192.168.18.203

# MySQL Brute Force (from Kali)
hydra -l testuser -P wordlist.txt mysql://192.168.18.203

# SSH Brute Force (from Kali)
hydra -l aalok -P wordlist.txt ssh://192.168.18.203

# ===========================================
# MULTI-SERVICE TARGETING
# ===========================================

# Execute within 60 seconds (from Kali):
ftp 192.168.18.203        # Type 'quit' to exit
curl http://192.168.18.203/
mysql -h 192.168.18.203 -u testuser -p  # Enter password, then quit
```

---

## 9. Troubleshooting

### Issue 1: Snort Not Detecting Attacks

**Symptoms:**
- No alerts in `/var/log/snort/alert`
- Attacks executed but no detection

**Solutions:**

```bash
# 1. Verify Snort is running
ps aux | grep snort | grep -v grep

# 2. Check network interface
ip addr show enp0s3

# 3. Verify HOME_NET configuration
sudo snort -T -c /etc/snort/snort.conf -S HOME_NET=192.168.18.0/24

# 4. Test with simple traffic
ping 192.168.18.203  # From Kali

# 5. Check Snort log file permissions
ls -la /var/log/snort/alert

# 6. Restart Snort
sudo pkill -f snort
sudo snort -A fast -c /etc/snort/snort.conf -i enp0s3 -S HOME_NET=192.168.18.0/24 -D
```

### Issue 2: Parser Not Generating Alerts

**Symptoms:**
- `parsed_alerts.json` is empty
- Parser process not running

**Solutions:**

```bash
# 1. Check if parser is running
ps aux | grep parser.py | grep -v grep

# 2. Check for multiple parser instances
ps aux | grep parser.py

# If multiple instances, kill all and restart:
sudo pkill -f parser.py
cd /home/aalok/snort_project
sudo python3 parser.py &

# 3. Check parser logs for errors
cat /home/aalok/snort_project/parser.log

# 4. Manually test parser
sudo python3 parser.py
# Watch console output for errors

# 5. Verify OpenVAS file exists
ls -la /home/aalok/snort_project/openvas_input/sample_openvas_report.csv
```

### Issue 3: Behavioral Alerts Not Generating

**Symptoms:**
- `behavior_alerts.json` is empty or has errors
- "Expecting value" JSON errors

**Solutions:**

```bash
# 1. Check if behavior engine is running
ps aux | grep behavior_engine | grep -v grep

# 2. Clean JSON files and restart
cd /home/aalok/snort_project
sudo pkill -f behavior_engine.py
rm -f behavior_alerts.json
touch behavior_alerts.json
sudo python3 behavior_engine.py &

# 3. Verify parsed_alerts.json has valid content
tail -5 parsed_alerts.json | python3 -m json.tool

# 4. Check for errors in behavior engine log
cat /home/aalok/snort_project/behavior_engine.log

# 5. Test manually
sudo python3 behavior_engine.py
# Watch console output
```

### Issue 4: Attacks Not Reaching Target

**Symptoms:**
- Attack commands execute but target doesn't see traffic
- Snort doesn't detect anything

**Solutions:**

```bash
# CRITICAL: Verify network configuration

# 1. Check VirtualBox network adapter
# Must be: Bridged Adapter (NOT NAT)
# VirtualBox → VM Settings → Network → Adapter 1 → Bridged Adapter

# 2. Verify both machines are on same network
# On Ubuntu:
ip addr show enp0s3
# Should show: 192.168.18.203/24

# On Kali:
ip addr
# Should show: 192.168.18.197/24

# 3. Test connectivity
# From Kali:
ping 192.168.18.203
# Should get replies

# 4. NEVER attack from localhost
# Attacks must come FROM Kali TO Ubuntu
# Snort can't see loopback traffic!

# 5. Ensure target services are running
# On Ubuntu:
sudo systemctl status vsftpd
sudo systemctl status mysql
sudo systemctl status ssh
```

---

## 10. Appendix: Complete Code Listings

### A. parser.py (Complete Source)

See file: `src/parser.py` (548 lines)

**Key Sections:**
- Lines 1-30: Imports and configuration
- Lines 31-85: Helper functions (subnet detection, OpenVAS loading)
- Lines 86-200: Alert parsing logic
- Lines 201-300: Attack classification and enrichment
- Lines 301-400: Main processing loop

### B. behavior_engine.py (Complete Source)

See file: `src/behavior_engine.py` (402 lines)

**Key Sections:**
- Lines 1-40: Configuration and constants
- Lines 41-120: BehaviorEngine class initialization
- Lines 121-250: Detection rule methods
- Lines 251-350: Event processing logic
- Lines 351-402: Main loop

### C. Snort Rules (Complete)

See file: `config/snort/local.rules` (17 rules)

### D. start.sh (Complete)

See file: `src/start.sh` (System startup script)

---

## Quick Start Commands

```bash
# 1. Start entire system
cd /home/aalok/snort_project
sudo ./start.sh

# 2. Start HTTP server (in new terminal)
sudo python3 -m http.server 80

# 3. Monitor live alerts (in separate terminals)
sudo tail -f /var/log/snort/alert
tail -f /home/aalok/snort_project/parsed_alerts.json
tail -f /home/aalok/snort_project/behavior_alerts.json

# 4. From Kali: Run attack tests
sudo nmap -sS 192.168.18.203
sudo hping3 -S -p 80 --flood 192.168.18.203
ab -n 10000 -c 100 http://192.168.18.203/
hydra -l anonymous -P wordlist.txt ftp://192.168.18.203

# 5. View Splunk dashboard
# Browser: http://192.168.18.203:8000
```

---

**End of Document**

For questions, issues, or contributions:  
GitHub: https://github.com/aalokdhami/Network-Threat-Detection-
Email: iamaalok03@gmail.com
