# 🛡️ SOC Lab: Network Reconnaissance Analysis

## 📌 Project Overview
This document details a controlled network reconnaissance activity performed in a virtual lab environment. The objective was to analyze the relationship between **attacker visibility** (Nmap) and **defender telemetry** (Windows Event Logs & Splunk SIEM) under different security configurations.

---

## 🛠️ Lab Environment
- **Attacker Node:** Kali Linux
- **Target Node:** Windows 10/11 (Target IP: `192.168.56.102`)
- **SIEM Platform:** Splunk Enterprise (Hosted on Ubuntu)
- **Network:** Isolated VirtualBox Host-Only Adapter

### 1. Target Verification
![Windows IP](Lab%2001%20nmap%20screenshots/01-windows-ip.jpeg)
> **Extended Analysis:** Verification of the target machine's network configuration using `ipconfig`. This identifies the IPv4 address within the VirtualBox Host-Only subnet, establishing the specific target baseline for the reconnaissance exercise.

### 2. Connectivity Test
![Kali Ping](Lab%2001%20nmap%20screenshots/02-kali-ping-windows.jpeg)
> **Extended Analysis:** ICMP Echo Request (Ping) from the Kali Linux attacker node to the Windows target. This confirms active Layer 3 connectivity and ensures that the virtual network is correctly routed before moving to port-level scanning.

---

## 🔍 Observed Behavior

### 🔴 Scenario 1: Firewall DISABLED (Unrestricted Exposure)
In this state, the host-based firewall was disabled to simulate a misconfigured or unprotected internal asset.

| Port | Service | State | Analysis |
| :--- | :--- | :--- | :--- |
| **135/tcp** | RPC | Open | Endpoint mapper accessible |
| **139/tcp** | NetBIOS | Open | Session service active |
| **445/tcp** | SMB | Open | Direct host-to-host communication allowed |

### 3. Attack: Firewall Disabled
![Nmap Open Ports](Lab%2001%20nmap%20screenshots/03-nmap-firewall-off.jpeg)
> **Extended Analysis:** Scenario 1 (Firewall Disabled). The Nmap scan successfully identifies open high-value ports including **135 (RPC)** and **445 (SMB)**. This demonstrates the "unrestricted" view an attacker gains when host-based defenses are absent.

---

### 🔵 Scenario 2: Firewall ENABLED (Filtered/Stealth)
In this state, Windows Defender Firewall was active. The scanning behavior changed significantly.

- **Nmap Result:** `1000 scanned ports in ignored states (filtered)`.
- **Packet Behavior:** The firewall utilized a **"Silent Drop"** strategy, ignoring incoming SYN packets rather than rejecting them.
- **Impact:** The attacker cannot confirm if the host is alive or what services are running.

### 4. Defense: Firewall Enabled
![Nmap Filtered](Lab%2001%20nmap%20screenshots/04-nmap-firewall-on.jpeg)
> **Extended Analysis:** Scenario 2 (Firewall Enabled). Nmap reports ports as **"Filtered."** This visualizes the impact of the Windows Defender Firewall's "Silent Drop" strategy, which hides the system's true service state from the attacker.

### 5. Defensive Posture
![Firewall Status](Lab%2001%20nmap%20screenshots/05-firewall-status.jpeg)
> **Extended Analysis:** Evidence of the defensive posture. This shows the Windows Defender Firewall settings in the "On" state for Domain, Private, and Public profiles, validating the conditions for the stealth reconnaissance detection.

---

## 📊 SIEM Detection (Splunk Analysis)

Even when the firewall blocked the scan, the Windows Filtering Platform (WFP) generated high-fidelity logs that were ingested into Splunk.

### Key Event IDs Tracked:
* **Event ID 5156 (Connection Allowed):** Observed during Scenario 1. Confirmed successful TCP handshakes.
* **Event ID 5152 (Packet Blocked):** Observed during Scenario 2. **Crucial Finding:** Splunk recorded the source IP and destination port for packets that the Firewall dropped.

### 6. Data Ingestion
![Splunk Events](Lab%2001%20nmap%20screenshots/06-splunk-events.jpeg)
> **Extended Analysis:** Initial telemetry ingestion in Splunk. The search results show a massive ingestion of raw Windows Security logs, confirming that the Universal Forwarder (UF) is successfully transmitting Event IDs to the SIEM.

### 7. Analyst Investigation
![Detection Query](Lab%2001%20nmap%20screenshots/07-detection-query.jpeg)
> **Extended Analysis:** Advanced investigation using Search Processing Language (SPL). This query utilizes **Regular Expressions (rex)** to extract source IPs and destination ports from unstructured raw logs, transforming them into a structured investigative table.

### 8. Blocked Event Detail
![Event 5152 Details](Lab%2001%20nmap%20screenshots/08-event-5152-details.jpeg)
> **Extended Analysis:** Deep-dive into **Event ID 5152 (Packet Blocked)**. This log proves that even though the attacker saw "Filtered" in Nmap, the SIEM captured the specific intent, including the attacker's IP and the targeted port (445/SMB).

### 9. Allowed Event Detail
![Event 5156 Details](Lab%2001%20nmap%20screenshots/09-event-5156-details.jpeg)
> **Extended Analysis:** Deep-dive into **Event ID 5156 (Connection Allowed)**. This is used for comparison analysis to show traffic that successfully bypassed the firewall, helping to distinguish between a scan and a successful connection.

### 11. Automated Field Extraction & Structured Analytics
![Splunk Structured Table](Lab%2001%20nmap%20screenshots/11-splunk-structured-data.jpg)
> **Extended Analysis:** Transitioning to professional SIEM administration. This view shows the results of implementing permanent **Field Extractions**. By automating the parsing of `src_ip`, `dest_port`, and `app_name`, the SOC can perform rapid, high-level analysis of blocked connections and the specific system processes (like `svchost.exe`) being targeted.

---

## 🛡️ Security Interpretation & Framework Mapping

| Framework | Identifier | Description |
| :--- | :--- | :--- |
| **MITRE ATT&CK** | **T1595.001** | Active Scanning: IP Addresses |
| **MITRE ATT&CK** | **T1046** | Network Service Scanning |
| **Cyber Kill Chain**| Phase 1 | Reconnaissance |

---

## 💡 Conclusion & Recommendations

The lab successfully demonstrated that network reconnaissance is a "loud" activity that leaves significant footprints in endpoint logs regardless of the firewall state.

### 10. Visualized Attack Spike
![Splunk Chart](Lab%2001%20nmap%20screenshots/10-splunk-chart.jpeg)
> **Extended Analysis:** **Attack Pattern Visualization.** An area chart showing a distinct spike in blocked connections. This high-volume "noise" is a classic Indicator of Attack (IoA) for automated scanning tools like Nmap.

### **Recommendations for SOC Hardening:**
1. **Detection Rule:** Implement a Splunk alert for high-frequency **Event ID 5152** triggers (e.g., >100 blocks from one source in 60 seconds).
2. **Attack Surface Reduction:** Use GPO to restrict SMB (445) and RPC (135) access to authorized subnets only.
3. **Logging Policy:** Ensure "Audit Filtering Platform Connection" is enabled in Advanced Audit Policy to maintain visibility into blocked traffic.
