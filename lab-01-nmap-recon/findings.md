# 🛡️ SOC Lab: Network Reconnaissance Analysis

## 📌 Project Overview
This document details a controlled network reconnaissance activity performed in a virtual lab environment. The objective was to analyze the relationship between **attacker visibility** (Nmap) and **defender telemetry** (Windows Event Logs & Splunk SIEM) under different security configurations.

---

## 🛠️ Lab Environment
- **Attacker Node:** Kali Linux
- **Target Node:** Windows 10/11 (Target IP: `192.168.56.102`)
- **SIEM Platform:** Splunk Enterprise (Hosted on Ubuntu)
- **Network:** Isolated VirtualBox Host-Only Adapter

---

## 🔍 Observed Behavior

### 🔴 Scenario 1: Firewall DISABLED (Unrestricted Exposure)
In this state, the host-based firewall was disabled to simulate a misconfigured or unprotected internal asset.

| Port | Service | State | Analysis |
| :--- | :--- | :--- | :--- |
| **135/tcp** | RPC | Open | Endpoint mapper accessible |
| **139/tcp** | NetBIOS | Open | Session service active |
| **445/tcp** | SMB | Open | Direct host-to-host communication allowed |

**Result:** The system responded directly to scan probes, allowing Nmap to clearly identify open services and the operating system.

---

### 🔵 Scenario 2: Firewall ENABLED (Filtered/Stealth)
In this state, Windows Defender Firewall was active. The scanning behavior changed significantly.

- **Nmap Result:** `1000 scanned ports in ignored states (filtered)`.
- **Packet Behavior:** The firewall utilized a **"Silent Drop"** strategy, ignoring incoming SYN packets rather than rejecting them with a RST packet.
- **Impact:** The attacker cannot confirm if the host is alive or what services are running without more advanced "firewall-walking" techniques.

---

## 📊 SIEM Detection (Splunk Analysis)

Even when the firewall blocked the scan, the Windows Filtering Platform (WFP) generated high-fidelity logs that were ingested into Splunk.

### Key Event IDs Tracked:
* **Event ID 5156 (Connection Allowed):** Observed during Scenario 1. Confirmed successful TCP handshakes between the Kali IP and Windows services.
* **Event ID 5152 (Packet Blocked):** Observed during Scenario 2. **Crucial Finding:** Splunk recorded the source IP, destination port, and protocol for packets that the Windows Firewall dropped.

> **Analyst Note:** This proves that "Filtered" does not mean "Invisible." A SOC Analyst can detect a port scan even when the firewall is working correctly by monitoring for blocked packet spikes.

---

## 🛡️ Security Interpretation & Framework Mapping

This activity aligns with the **Reconnaissance** phase of the Cyber Attack Lifecycle.

| Framework | Identifier | Description |
| :--- | :--- | :--- |
| **MITRE ATT&CK** | **T1595.001** | Active Scanning: IP Addresses |
| **MITRE ATT&CK** | **T1046** | Network Service Scanning |
| **Cyber Kill Chain**| Phase 1 | Reconnaissance |

---

## 💡 Conclusion & Recommendations

The lab successfully demonstrated that network reconnaissance is a "loud" activity that leaves significant footprints in endpoint logs regardless of the firewall state.

### **Recommendations for SOC Hardening:**
1.  **Detection Rule:** Implement a Splunk alert for high-frequency **Event ID 5152** triggers (e.g., >100 blocks from one source in 60 seconds).
2.  **Attack Surface Reduction:** Use GPO to restrict SMB (445) and RPC (135) access to only authorized administrative subnets.
3.  **Logging Policy:** Ensure "Audit Filtering Platform Connection" is enabled in Advanced Audit Policy to maintain visibility into blocked traffic.

---
