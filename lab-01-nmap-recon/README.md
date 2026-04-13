# 🛡️ SOC Lab 01: Nmap Reconnaissance & SIEM Engineering

## 📌 Project Overview
This project demonstrates the **Reconnaissance** phase of the Cyber Attack Lifecycle. The objective was to simulate an internal network scan using **Nmap** from a Kali Linux machine and analyze the resulting telemetry within a **Splunk SIEM**. 

Beyond basic detection, this lab highlights **SIEM Data Engineering** by implementing custom field extractions to transform unstructured Windows logs into actionable security intelligence.

---

## 🛠️ Lab Environment & Tools
- **Attacker Node:** Kali Linux (Nmap)
- **Target Node:** Windows 10 (Target IP: `192.168.56.102`)
- **SIEM Platform:** Splunk Enterprise on Ubuntu 22.04 LTS
- **Logging:** Splunk Universal Forwarder + Windows Filtering Platform (WFP)
- **Network:** Isolated VirtualBox Host-Only Adapter

---

## ⚙️ SIEM Engineering: Data Optimization
To transition from a basic analyst to an **SIEM Administrator**, I implemented **Search-Time Field Extractions (Knowledge Objects)**. This process normalizes unstructured raw logs into structured data fields, enabling high-speed analytics without the need for manual Regular Expressions (`rex`) in every search.

### 🧩 Custom Knowledge Objects Created:
| Field Name | Source Pattern | Purpose |
| :--- | :--- | :--- |
| `src_ip` | Source Address | Automatically identifies the attacker's origin IP. |
| `dest_port` | Destination Port | Extracts the targeted service port for scanning analysis. |
| `app_name` | Application Name | Identifies the specific Windows process (e.g., `svchost.exe`) targeted. |

### 📊 Automated Analytics & Extraction
![Splunk Structured Table](./Lab%2001%20nmap%20screenshots/11-splunk_automated_field_extraction_analytics.jpeg)
> **Figure 11:** Demonstrating the transition to professional SIEM administration. By automating the parsing of `src_ip`, `dest_port`, and `app_name`, the SOC can perform rapid, high-level analysis of blocked connections and the specific system processes being targeted.

---

## 🚀 Lab Activities

### 1. Attack Simulation
Executed a **SYN Stealth Scan (`-sS`)** and **Service Enumeration (`-sV`)** under two conditions:
- **Firewall OFF:** Proved full visibility of exposed services (SMB, RPC).
- **Firewall ON:** Resulted in "Filtered" states, demonstrating the firewall's silent-drop behavior.

### 2. SIEM Detection & Correlation
Ingested Windows Security logs into Splunk to identify reconnaissance signatures:
- **Event ID 5156:** Authorized/Allowed connections when security was relaxed.
- **Event ID 5152:** Blocked packet attempts, providing a "Blue Team" view of a stealth scan.
[Splunk Detection](./Lab%2001%20nmap%20screenshots/10-splunk-chart.jpeg)
> **Visualized Attack Spike:** A distinct spike in blocked connections (Event ID 5152) reveals the automated scanning activity that appeared "silent" to the attacker.

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
- **Data Engineering is Crucial:** Manual `rex` is slow; permanent field extractions make SOC operations scalable.
- **Filtered doesn't mean Hidden:** Even when a firewall blocks a scan, the activity is recorded in the Windows Filtering Platform logs.
- **Security Posture:** Monitoring for high-frequency **Event ID 5152** is a primary method for detecting internal reconnaissance.
