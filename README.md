# Adaptive Threat Detection System

![Project Status](https://img.shields.io/badge/status-active-success.svg)
![Python Version](https://img.shields.io/badge/python-3.x-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

## 🎯 Project Overview

An intelligent network intrusion detection and behavioral analysis system that combines Snort IDS, custom Python parsers, behavioral correlation engines, and Splunk SIEM for comprehensive threat detection and visualization.

### Key Features

- **Real-time Network Monitoring**: Snort 2.9.20 IDS with 13 custom detection rules
- **Behavioral Analysis**: 6-rule correlation engine for advanced threat pattern detection
- **Vulnerability Enrichment**: OpenVAS integration for contextual threat intelligence
- **SIEM Integration**: Splunk Enterprise dashboard for visualization and alerting
- **Attack Detection**: DoS/DDoS, port scans, brute force, multi-service targeting

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Network Traffic                          │
│                        (enp0s3 - 192.168.18.0/24)              │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Snort IDS (2.9.20)                         │
│  • 13 Custom Rules (SID 100001-100017, 1001000-1001003)       │
│  • Signature-based detection                                   │
│  • Fast alert mode                                            │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼ /var/log/snort/alert
┌─────────────────────────────────────────────────────────────────┐
│                      parser.py                                  │
│  • Real-time log parsing                                       │
│  • OpenVAS vulnerability enrichment                            │
│  • Auto-subnet detection                                       │
│  • JSON output (JSONL format)                                  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼ parsed_alerts.json
┌─────────────────────────────────────────────────────────────────┐
│                  behavior_engine.py                             │
│  • Multi-service targeting detection                           │
│  • SSH/FTP/MySQL brute force detection                         │
│  • Port scan pattern recognition                               │
│  • DoS/DDoS correlation                                        │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼ behavior_alerts.json
┌─────────────────────────────────────────────────────────────────┐
│                   Splunk Enterprise                             │
│  • Dark-themed dashboard                                       │
│  • Real-time alerts visualization                              │
│  • Risk score analytics                                        │
│  • Attack timeline tracking                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📋 System Requirements

### Hardware
- **CPU**: 2+ cores
- **RAM**: 4GB minimum, 8GB recommended
- **Disk**: 20GB free space
- **Network**: Bridged adapter for packet capture

### Software
- **OS**: Ubuntu 24.04 LTS
- **Snort**: 2.9.20
- **Python**: 3.x
- **Splunk**: Enterprise 8.x+
- **Services**: vsftpd, MySQL 8.0, OpenSSH

### Attack Simulation Environment
- **Kali Linux**: 2024.x with tools: nmap, hping3, hydra, apache-bench

---

## 🚀 Installation Guide

### Step 1: Clone Repository

```bash
git clone https://github.com/yourusername/adaptive-threat-detection.git
cd adaptive-threat-detection
```

### Step 2: Install Dependencies

```bash
# Install Snort
sudo apt update
sudo apt install snort -y

# Install Python dependencies
pip3 install -r requirements.txt

# Install test services
sudo apt install vsftpd mysql-server openssh-server -y

# Install Splunk (download from splunk.com)
wget -O splunk.deb 'https://download.splunk.com/...'
sudo dpkg -i splunk.deb
```

### Step 3: Configure Snort

```bash
# Copy custom rules
sudo cp config/snort/local.rules /etc/snort/rules/

# Update Snort configuration
sudo nano /etc/snort/snort.conf
# Add: include $RULE_PATH/local.rules

# Validate configuration
sudo snort -T -c /etc/snort/snort.conf -S HOME_NET=192.168.18.0/24
```

### Step 4: Setup OpenVAS Vulnerability Database

```bash
# Copy vulnerability database
cp config/openvas/sample_openvas_report.csv openvas_input/
```

### Step 5: Deploy Project Files

```bash
# Copy all project files to working directory
sudo cp -r src/* /home/aalok/snort_project/
sudo chmod +x /home/aalok/snort_project/start.sh
```

### Step 6: Start the System

```bash
cd /home/aalok/snort_project
sudo ./start.sh
```

---

## 📁 Project Structure

```
adaptive-threat-detection/
├── README.md                          # This file
├── LICENSE                            # MIT License
├── requirements.txt                   # Python dependencies
├── docs/                             # Documentation
│   ├── PROJECT_SUMMARY.pdf           # Complete step-by-step guide
│   ├── TESTING_REPORT.pdf            # 20 test cases report
│   ├── ARCHITECTURE.md               # Detailed architecture
│   └── API_REFERENCE.md              # Code documentation
├── src/                              # Source code
│   ├── parser.py                     # Alert parser with OpenVAS enrichment
│   ├── behavior_engine.py            # Behavioral correlation engine
│   ├── start.sh                      # System startup script
│   └── utils/                        # Utility scripts
│       ├── test_runner.sh            # Automated testing
│       └── cleanup.sh                # System cleanup
├── config/                           # Configuration files
│   ├── snort/
│   │   └── local.rules               # Custom Snort rules (13 rules)
│   ├── openvas/
│   │   └── sample_openvas_report.csv # Vulnerability database
│   └── splunk/
│       └── dashboard.xml             # Splunk dashboard config
├── tests/                            # Test cases
│   ├── unit/                         # Unit tests (TC-01 to TC-04)
│   ├── security/                     # Security tests (TC-05 to TC-16)
│   ├── system/                       # System tests (TC-17 to TC-20)
│   └── attack_scripts/               # Attack simulation scripts
│       ├── dos_flood.sh              # DoS attack tests
│       ├── port_scan.sh              # Nmap scan tests
│       └── brute_force.sh            # Brute force tests
├── data/                             # Sample data
│   └── sample_logs/                  # Example alert logs
└── screenshots/                      # Project screenshots
    ├── splunk_dashboard.png
    ├── behavioral_alerts.png
    └── attack_detection.png
```

---

## 🔧 Configuration Files

### Snort Rules (`config/snort/local.rules`)

The system includes 13 custom Snort rules organized by attack type:

**DoS/DDoS Detection:**
- SID 100012: SYN Flood (200+ packets/5s)
- SID 100013: ICMP Flood (50+ packets/5s)
- SID 100011: HTTP Flood (100+ requests/10s)

**Port Scan Detection:**
- SID 100014: NULL Scan
- SID 100015: FIN Scan
- SID 100016: XMAS Scan
- SID 100017: SYN Scan (20+ packets/5s)

**Service-Specific Attacks:**
- SID 100002: SSH Brute Force (5+ attempts/60s)
- SID 100007: FTP Brute Force (5+ attempts/60s)
- SID 100009: MySQL Brute Force (5+ attempts/60s)

**Vulnerability-Based Rules:**
- SID 1001000-1001003: OpenVAS-enriched detection

### Behavioral Engine Rules (`behavior_engine.py`)

**6 Correlation Patterns:**
1. **Multi-Service Targeting**: 3+ events, 2+ services in 60s (Risk: 85)
2. **SSH Brute Force**: 10+ attempts in 120s (Risk: 90)
3. **Port Scan**: 5+ unique ports in 30s (Risk: 70)
4. **FTP Brute Force**: 5+ attempts in 60s (Risk: 85)
5. **MySQL Brute Force**: 5+ attempts in 60s (Risk: 90)
6. **Flood/DoS Detection**: Signature-based (Risk: 95)

---

## 🎮 Usage Examples

### Starting the System

```bash
# Start all components
cd /home/aalok/snort_project
sudo ./start.sh

# Start HTTP server (in separate terminal)
sudo python3 -m http.server 80
```

### Monitoring Live Alerts

```bash
# Watch Snort raw alerts
sudo tail -f /var/log/snort/alert

# Watch parsed events
tail -f /home/aalok/snort_project/parsed_alerts.json

# Watch behavioral alerts
tail -f /home/aalok/snort_project/behavior_alerts.json
```

### Running Attack Tests (From Kali Linux)

```bash
# SYN Flood
sudo hping3 -S -p 80 --flood 192.168.18.203

# ICMP Flood
sudo hping3 -1 --flood 192.168.18.203

# HTTP Flood
ab -n 10000 -c 100 http://192.168.18.203/

# Nmap Scans
sudo nmap -sN 192.168.18.203  # NULL scan
sudo nmap -sF 192.168.18.203  # FIN scan
sudo nmap -sX 192.168.18.203  # XMAS scan
sudo nmap -sS 192.168.18.203  # SYN scan

# Brute Force Attacks
hydra -l anonymous -P wordlist.txt ftp://192.168.18.203
hydra -l testuser -P wordlist.txt mysql://192.168.18.203
hydra -l aalok -P wordlist.txt ssh://192.168.18.203
```

### Checking Detection Results

```bash
# Latest parsed event (formatted)
tail -1 /home/aalok/snort_project/parsed_alerts.json | python3 -m json.tool

# Latest behavioral alert (formatted)
tail -1 /home/aalok/snort_project/behavior_alerts.json | python3 -m json.tool

# Check running processes
ps aux | grep -E "snort|parser|behavior" | grep -v grep
```

---

## 📊 Output Format

### Parsed Alert (JSONL)

```json
{
  "event_time": "2026-04-16T23:35:49.132925",
  "gid": 1,
  "sid": 100017,
  "rev": 1,
  "signature": "NMAP SYN SCAN DETECTED",
  "classification": "unknown",
  "priority": 0,
  "protocol": "TCP",
  "src_addr": "192.168.18.197",
  "src_port": 64000,
  "dst_addr": "192.168.18.203",
  "dst_port": 8080,
  "severity": "low",
  "attack_type": "general_alert",
  "risk_score": 25,
  "vuln_context": {
    "vuln_description": "MySQL Exposed to Network",
    "vuln_severity": "Critical",
    "vuln_service": "mysql"
  }
}
```

### Behavioral Alert

```json
{
  "event_time": "2026-04-16T23:40:12.456789",
  "detection_type": "behavioral",
  "signature": "PORT SCAN DETECTED",
  "src_addr": "192.168.18.197",
  "target_host": "192.168.18.203",
  "services_targeted": ["ssh", "http", "mysql", "ftp"],
  "trigger_count": 15,
  "signatures_seen": ["NMAP SYN SCAN DETECTED"],
  "severity": "medium",
  "risk_score": 70,
  "trigger_condition": "5+ unique ports in 30s",
  "reason": "Multiple ports targeted from 192.168.18.197"
}
```

---

## 🧪 Testing

The project includes a comprehensive test suite with 20 test cases:

### Run All Tests

```bash
cd tests
sudo ./run_all_tests.sh
```

### Test Categories

- **Unit Tests (4)**: Component initialization
- **Security Tests (9)**: Attack detection validation
- **System Tests (7)**: End-to-end integration

### Manual Testing

See `docs/TESTING_REPORT.pdf` for detailed test procedures.

---

## 📈 Performance Metrics

| Metric | Value |
|--------|-------|
| Average Detection Latency | < 2 seconds |
| Alert Processing Rate | 1000+ alerts/minute |
| False Positive Rate | < 5% |
| System Resource Usage | ~10% CPU, 512MB RAM |
| Snort Packet Drop Rate | < 0.1% |

---

## 🔒 Security Considerations

### Important Notes

1. **Testing Environment Only**: This system is designed for controlled lab environments
2. **Network Isolation**: Use isolated networks for attack simulation
3. **Credential Management**: Change default passwords in production
4. **Log Rotation**: Implement log rotation to prevent disk exhaustion
5. **Access Control**: Restrict Splunk dashboard access with authentication

### Ethical Use

⚠️ **WARNING**: Attack simulation tools (hping3, nmap, hydra) should ONLY be used:
- In controlled lab environments
- Against systems you own or have explicit permission to test
- For educational and security research purposes

Unauthorized network scanning or attacks may violate laws and regulations.

---

## 🐛 Troubleshooting

### Common Issues

**Issue**: Snort not detecting attacks
```bash
# Check if Snort is running
ps aux | grep snort

# Verify interface is correct
ip addr show enp0s3

# Check rules are loaded
sudo snort -T -c /etc/snort/snort.conf
```

**Issue**: Behavioral alerts not generating
```bash
# Check if behavior_engine.py is running
ps aux | grep behavior_engine

# Check parsed_alerts.json has data
tail -10 /home/aalok/snort_project/parsed_alerts.json

# Restart behavior engine
sudo pkill -f behavior_engine
cd /home/aalok/snort_project
sudo python3 behavior_engine.py &
```

**Issue**: Parser crashes
```bash
# Check for multiple parser processes
ps aux | grep parser.py

# Kill all parsers
sudo pkill -f parser.py

# Restart parser
sudo python3 /home/aalok/snort_project/parser.py &
```

---

## 📚 Documentation

- **[Project Summary PDF](docs/PROJECT_SUMMARY.pdf)**: Complete step-by-step guide with screenshots
- **[Testing Report](docs/TESTING_REPORT.pdf)**: All 20 test cases with results
- **[Architecture Guide](docs/ARCHITECTURE.md)**: Detailed system architecture
- **[API Reference](docs/API_REFERENCE.md)**: Code documentation

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👥 Authors

- Alok Bahadur Dhami - *Initial work* - [GitHub Profile]: https://github.com/aalokdhami 

---

## 🙏 Acknowledgments

- Snort IDS Project - https://www.snort.org/
- Splunk Inc. - https://www.splunk.com/
- OpenVAS/Greenbone Networks - https://www.openvas.org/
- MITRE ATT&CK Framework - https://attack.mitre.org/
- SANS Institute - https://www.sans.org/
- NIST Cybersecurity Framework - https://www.nist.gov/cyberframework

---

## 📧 Contact

For questions or support, please open an issue or contact: iamaalok03@gmail.com

---

## 🔗 Related Resources

- [Snort Documentation](https://www.snort.org/documents)
- [Splunk Documentation](https://docs.splunk.com/)
- [NIST SP 800-94: Guide to IDS/IPS](https://csrc.nist.gov/publications/detail/sp/800-94/rev-1/draft)
- [MITRE ATT&CK](https://attack.mitre.org/)

---

**Star ⭐ this repository if you find it helpful!**
