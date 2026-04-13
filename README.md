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
| **Phase 2** | Wazuh Agent Deployment | 🔜 Coming Soon |
| **Phase 3** | Attack Scenario 1 — Nmap Reconnaissance | 🔜 Coming Soon |
| **Phase 4** | Attack Scenario 2 — SSH Brute Force | 🔜 Coming Soon |
| **Phase 5** | Attack Scenario 3 — Metasploit Exploitation | 🔜 Coming Soon |
| **Phase 6** | Attack Scenario 4 — Mimikatz Credential Dump | 🔜 Coming Soon |
| **Phase 7** | Attack Scenario 5 — C2 Beaconing Simulation | 🔜 Coming Soon |
| **Phase 8** | Detection Tuning & Custom Rules | 🔜 Coming Soon |
| **Phase 9** | Threat Hunt Report | 🔜 Coming Soon |
| **Phase 10** | Full Incident Report Portfolio | 🔜 Coming Soon |

---

## ✅ Phase 1 — Environment Setup: Windows 11 VM

### Objective
Set up a Windows 11 Pro virtual machine on Apple Silicon MacBook using UTM, to serve as the target/victim machine in all attack scenarios.

### Prerequisites
- MacBook with Apple Silicon (M1/M2/M3)
- [UTM](https://utm.app) installed (free virtualisation app for macOS)
- Approximately 10GB free disk space for the VM
- Internet connection for ISO download

### Step 1 — Install UTM
1. Download UTM from [utm.app](https://utm.app)
2. Drag UTM to your Applications folder and open it
3. Click **Create a New Virtual Machine**
4. Select **Virtualise**

### Step 2 — Download Windows 11 ARM ISO via CrystalFetch
1. In UTM's OS selection, choose **Windows**
2. On the Image File Type screen, click **"Fetch latest Windows installer..."**
   - This opens **CrystalFetch** — a built-in ISO downloader
3. Set the following options:
   - **Version:** Windows 11
   - **Build:** Latest
   - **Architecture:** Apple Silicon
   - **Language:** English (United States)
   - **Edition:** Windows 11
4. Click **Download** and save the ISO (approx. 5–6 GB)
5. Keep **"Install drivers and SPICE tools"** checkbox ticked
6. Once downloaded, click **Browse** and select the ISO file
7. Click **Continue**

> **Why Apple Silicon?** Standard Windows 10/11 x86 ISOs do not run on Apple Silicon. CrystalFetch automatically downloads the correct Windows 11 ARM build.

### Step 3 — Configure VM Hardware
Recommended settings for the VM:

| Setting | Recommended Value |
|---|---|
| RAM | 4–6 GB (half your total MacBook RAM) |
| CPU Cores | 2–4 cores |
| Storage | 64 GB virtual disk |
| Network | Shared Network (NAT) |

### Step 4 — Shared Directory
- Skip the Shared Directory screen — click **Continue** without selecting a path
- Not required for this lab environment

### Step 5 — Install Windows 11
1. Start the VM — it boots to the UEFI shell initially
2. If the UEFI shell appears, type the following commands:
   ```
   FS1:
   ls
   ```
3. Navigate to the boot directory and run:
   ```
   Bootaa64.efi
   ```
4. When prompted **"Press any key to boot from CD or DVD..."** — press a key immediately
5. The Windows 11 Setup screen will load

### Step 6 — Windows 11 Setup Wizard
Follow these selections through the installer:

| Screen | Selection |
|---|---|
| Select setup option | Install Windows 11 |
| Product Key | Click "I don't have a product key" |
| Select Image | **Windows 11 Pro** |
| License Agreement | Accept |
| Installation type | Custom Install |
| Select location | Disk 0 Unallocated Space (64 GB) → Next |

> **Why Windows 11 Pro?** Pro includes Group Policy, Remote Desktop, and BitLocker — features present in real enterprise environments that SOC analysts monitor.

> **No product key needed.** Windows runs fully functional in an unactivated state for lab purposes. A small "Activate Windows" watermark appears on the desktop but does not affect any functionality.

### Step 7 — Complete Installation
- The installation takes approximately 15–20 minutes
- The VM will reboot 2–3 times automatically — this is normal
- After reboots, complete the Windows OOBE (Out of Box Experience) setup:
  - Region: Australia
  - Keyboard: US
  - Account: Create a **local account** (skip Microsoft account — click "Sign-in options" → "Offline account")
  - Set a username and password you will remember

### ✅ Phase 1 Complete
Your Windows 11 Pro VM is now running inside UTM on your MacBook. In the next phase, we will install the Wazuh agent on this machine and connect it to the Wazuh SIEM on the VPS.

---

## 📄 Incident Reports
*(Will be added as attack scenarios are completed)*

---

## 🎯 MITRE ATT&CK Coverage
*(Will be updated as attack scenarios are completed)*

---

## 👤 Author
**Yogesh Khadnelwal**
Cybersecurity enthusiast | Aspiring SOC Analyst
- LinkedIn: [https://www.linkedin.com/in/yogesh1025/]


---

## ⚠️ Disclaimer
This lab is built entirely on isolated virtual machines and a private VPS for educational purposes only. All attack simulations are conducted in a controlled environment against systems I own. This project is intended to demonstrate cybersecurity skills for employment purposes.
