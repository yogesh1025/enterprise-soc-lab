# 🛡️ Enterprise-SOC-Lab

> A hands-on Security Operations Centre (SOC) home lab built to simulate real-world enterprise threat detection, incident response, and security monitoring — using industry-standard tools including Wazuh SIEM, Suricata IDS/IPS, and Kali Linux.

**Built as a portfolio project to demonstrate practical SOC analyst skills for cybersecurity employment.**

---

## 📋 Project Overview

This project builds a fully functional mini enterprise SOC environment from scratch. Each phase simulates real attack scenarios, generates security alerts, and produces professional incident reports — mirroring the day-to-day work of a Tier 1/2 SOC Analyst.

### What This Lab Demonstrates
- SIEM-based monitoring, event analysis, and incident triage
- Security incident response workflows
- Threat detection and alert tuning
- Vulnerability assessment and threat hunting
- MITRE ATT&CK framework mapping
- Professional security report writing

---

## 🏗️ Lab Architecture

```
┌─────────────────────┐         ┌─────────────────────┐
│   Kali Linux        │─attacks▶│   Windows 11 Pro VM │
│   (Attacker)        │         │   (Target/Victim)   │
│   MacBook           │         │   UTM on MacBook     │
└────────┬────────────┘         └──────────┬──────────┘
         │                                 │
         │ Wazuh Agent                     │ Wazuh Agent
         │                                 │
         └──────────────┬──────────────────┘
                        ▼
             ┌─────────────────────┐
             │   Wazuh SIEM        │
             │   (VPS Server)      │
             │   Manager +         │
             │   Dashboard         │
             └─────────────────────┘
```

### Hardware & Infrastructure

| Component | Spec | Role |
|---|---|---|
| MacBook (Kali Linux) | Apple Silicon | Attacker Machine |
| MacBook (macOS + UTM) | Apple Silicon | Host for Windows VM |
| Windows 11 Pro VM | UTM / Apple Silicon ARM | Target / Victim Machine |
| VPS Server | Cloud-hosted | Wazuh SIEM Manager + Dashboard |

---

## 🔧 Tools & Technologies

| Tool | Purpose |
|---|---|
| **Wazuh** | SIEM + EDR — log ingestion, alerting, agent monitoring |
| **Suricata** | IDS/IPS — network traffic analysis |
| **Kali Linux** | Offensive security / attack simulation |
| **Nmap** | Network reconnaissance and port scanning |
| **Hydra** | Credential brute force attacks |
| **Metasploit** | Exploitation framework |
| **Mimikatz** | Credential dumping simulation |
| **Netcat** | C2 beaconing simulation |
| **UTM** | Virtualisation on Apple Silicon |

---

## 📁 Project Phases

| Phase | Title | Status |
|---|---|---|
| **Phase 1** | Environment Setup — Windows 11 VM | ✅ Complete |
| **Phase 2** | Wazuh Agent Deployment | ✅ Complete |
| **Phase 3** | Deploy Suricata IDS/IPS and Integrate with Wazuh | ✅ Complete |
| **Phase 4** | Attack Scenario 1 — Nmap Reconnaissance | 🔜 Coming Soon |
| **Phase 5** | Attack Scenario 2 — SSH Brute Force | 🔜 Coming Soon |
| **Phase 6** | Attack Scenario 3 — Metasploit Exploitation | 🔜 Coming Soon |
| **Phase 7** | Attack Scenario 4 — Mimikatz Credential Dump | 🔜 Coming Soon |
| **Phase 8** | Attack Scenario 5 — C2 Beaconing Simulation | 🔜 Coming Soon |
| **Phase 9** | Detection Tuning & Custom Rules | 🔜 Coming Soon |
| **Phase 10** | Threat Hunt Report | 🔜 Coming Soon |
| **Phase 11** | Full Incident Report Portfolio | 🔜 Coming Soon |

---

## 📄 Incident Reports
*(Will be added as attack scenarios are completed)*

---

## 🎯 MITRE ATT&CK Coverage
*(Will be updated as attack scenarios are completed)*

---

## 👤 Author
**Yogesh Khandelwal**
Cybersecurity enthusiast | Aspiring SOC Analyst
- LinkedIn: [https://www.linkedin.com/in/yogesh1025/]


---

## ⚠️ Disclaimer
This lab is built entirely on isolated virtual machines and a private VPS for educational purposes only. All attack simulations are conducted in a controlled environment against systems I own. This project is intended to demonstrate cybersecurity skills for employment purposes.
