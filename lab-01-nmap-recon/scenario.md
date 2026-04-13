# 🏗️ Lab Scenario: Detecting Internal Reconnaissance

## 📌 Project Overview
This scenario simulates a common real-world threat: **Internal Network Discovery**. In this lab, an unauthorized device (Kali Linux) is introduced to a corporate environment to "map" a Windows workstation. The goal is to determine if the SOC (Security Operations Center) can detect this activity using **Splunk SIEM** and **Windows Security Logs**, even when the target's firewall is actively blocking the attacker.

---

## 🎯 Objectives
1. **Network Isolation:** Establish a secure, host-only virtual environment.
2. **Telemetry Pipeline:** Configure log forwarding from a Windows endpoint to a central Splunk instance.
3. **Attack Simulation:** Execute Nmap scans to identify open services (SMB, RPC, NetBIOS).
4. **Detection Validation:** Correlate Nmap "Filtered" results with Windows Event ID **5152** (Packet Drops) in Splunk.

---

## 💻 Infrastructure & Topology
The environment consists of three virtual machines communicating over a private **Host-Only** network:

| Machine | Role | Operating System | IP Address |
| :--- | :--- | :--- | :--- |
| **Attacker** | Rogue Host / Pentester | Kali Linux | `192.168.56.103` |
| **Target** | Corporate Workstation | Windows 10 | `192.168.56.102` |
| **SIEM Server** | Log Collector / Analysis | Ubuntu (Splunk) | `192.168.56.101` |

---

## 🛡️ The "Story" (The Attack Lifecycle)

### Phase 1: The Initial Entry
An attacker gains access to a physical Ethernet port in a corporate office. They connect their Kali Linux machine and receive a local IP address. Their first goal is to find other machines in the same subnet.

### Phase 2: Host & Service Discovery
The attacker uses **Nmap** to scan the `192.168.56.0/24` range. They identify the Windows workstation and begin a detailed port scan (`-sS`) to find entry points like SMB (Port 445) for potential lateral movement or ransomware deployment.

### Phase 3: Defensive Monitoring
Unbeknownst to the attacker, the Windows machine is running a **Splunk Universal Forwarder**. Every time the Windows Firewall drops an Nmap probe, a log is generated. These logs are sent in real-time to the Ubuntu SIEM server.

---

## 🛠️ Tools Used
- **Nmap:** For network discovery and security auditing.
- **Splunk Enterprise:** For log aggregation, searching, and dashboarding.
- **Windows Filtering Platform (WFP):** The engine generating the raw security events.
- **VirtualBox:** For creating the isolated lab network.

---

## 📜 Framework Mapping
This lab demonstrates techniques defined in the **MITRE ATT&CK®** framework:
- **Tactic:** Reconnaissance ([TA0043](https://attack.mitre.org/tactics/TA0043/))
- **Technique:** Active Scanning ([T1595](https://attack.mitre.org/techniques/T1595/))
- **Technique:** Network Service Scanning ([T1046](https://attack.mitre.org/techniques/T1046/))

---
