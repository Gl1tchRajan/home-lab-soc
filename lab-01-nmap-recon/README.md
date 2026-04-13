# 🛡️ SOC Lab 01: Nmap Reconnaissance & SIEM Detection

## 📌 Project Overview
This project demonstrates the **Reconnaissance** phase of the Cyber Attack Lifecycle. The goal was to simulate an internal network scan using **Nmap** from a Kali Linux machine and analyze the resulting telemetry within a **Splunk SIEM**. 

This lab highlights the critical relationship between host-based security controls (Windows Defender Firewall) and the visibility provided to SOC Analysts through Windows Event Logs.

---

## 🛠️ Lab Environment & Tools
- **Attacker Node:** Kali Linux (Nmap 7.9x)
- **Target Node:** Windows 10/11 (Target IP: `192.168.56.102`)
- **SIEM Platform:** Splunk Enterprise on Ubuntu 22.04 LTS
- **Logging:** Splunk Universal Forwarder + Windows Filtering Platform (WFP)
- **Network:** Isolated VirtualBox Host-Only Adapter

---

## 🚀 Lab Activities

### 1. Attack Simulation
Executed a **SYN Stealth Scan (`-sS`)** and **Service Enumeration (`-sV`)** to identify open ports (135, 139, 445). The scan was performed under two conditions:
- **Firewall OFF:** Proved full visibility of exposed services.
- **Firewall ON:** Resulted in "Filtered" states, demonstrating the firewall's silent-drop behavior.

### 2. SIEM Detection & Correlation
Ingested Windows Security logs into Splunk to identify reconnaissance signatures:
- **Event ID 5156:** Authorized/Allowed connections when security was relaxed.
- **Event ID 5152:** Blocked packet attempts, providing a "Blue Team" view of a stealth scan.

---

## 📂 Repository Structure
| File | Description |
| :--- | :--- |
| [**README.md**](README.md) | Main project overview and landing page. |
| [**scenario.md**](scenario.md) | Detailed lab setup, "story" context, and network topology. |
| [**findings.md**](findings.md) | Technical analysis of Nmap results and log correlation. |
| [**splunk-queries.spl**](splunk-queries.spl) | The exact SPL searches used to investigate the attack. |

---

## 🛡️ Framework Mapping
This lab demonstrates techniques defined in the **MITRE ATT&CK** framework:
- **Tactic:** Reconnaissance ([TA0043](https://attack.mitre.org/tactics/TA0043/))
- **Technique:** Active Scanning ([T1595.001](https://attack.mitre.org/techniques/T1595/001/))
- **Technique:** Network Service Scanning ([T1046](https://attack.mitre.org/techniques/T1046/))

---

## 💡 Key Takeaways
- **Filtered doesn't mean Hidden:** Even when a firewall blocks a scan, the activity is recorded in the Windows Filtering Platform logs.
- **SOC Monitoring:** Monitoring for high-frequency **Event ID 5152** is a primary method for detecting internal reconnaissance.
- **Security Posture:** Host-based firewalls are essential for reducing the attack surface, but centralized logging is essential for visibility.

---
