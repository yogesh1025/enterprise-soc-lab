# Phase 2 — Wazuh Agent Deployment on Windows 11 VM

## Overview

In this phase we connect the **Windows 11 Pro VM** (our target machine) to the **Wazuh SIEM running on our VPS** by installing and enrolling a Wazuh agent. Once connected, every security event, file change, login attempt, and process execution on the Windows VM will be forwarded to Wazuh for detection and analysis.

This is the step that turns our VM from a standalone machine into a **monitored endpoint** — exactly how enterprise SOC environments work.

---

## Why Wazuh Agents Matter in a SOC

In a real enterprise environment, Wazuh agents are installed on every endpoint — servers, workstations, and cloud instances. The agent acts as a sensor, collecting:

- **Windows Event Logs** — login events, process creation, privilege escalation
- **File Integrity Monitoring (FIM)** — alerts when critical files are created, modified or deleted
- **Vulnerability Detection** — identifies unpatched CVEs on the host
- **MITRE ATT&CK Mapping** — automatically maps activity to adversary techniques
- **Configuration Assessment** — checks for security misconfigurations

Without agents, Wazuh has no visibility into what is happening on endpoints. With agents, it becomes a fully operational SIEM.

---

## Lab State at Start of Phase 2

Before this phase began, the Wazuh dashboard showed the following agent status:

- **Active (1)** — M2 MacBook (already enrolled from a previous setup)
- **Disconnected (1)** — our target machine which is offline right now.

The Windows VM had not yet been enrolled. The goal of this phase is to enrol it as a new active agent.

![Step 1 - Wazuh Dashboard Overview](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%201.png)

---

## Prerequisites

Before starting this phase ensure:

- [ ] Windows 11 VM is installed and running in UTM (Phase 1 complete)
- [ ] Windows VM has internet access (test by opening Microsoft Edge)
- [ ] You have access to the Wazuh dashboard on your VPS
- [ ] You know your Wazuh VPS IP address or hostname

---

## Step-by-Step Guide

### Step 1 — Navigate to the Endpoints Page in Wazuh

1. Log into your Wazuh dashboard
2. In the left sidebar, click **Endpoints**
3. You will see the current agent list — at this point only the M2 MacBook is active
4. Click the **"Deploy new agent"** button (highlighted in red in the screenshot below)

![Step 2 - Endpoints Page with Deploy New Agent Button](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%202.png)

---

### Step 2 — Configure the New Agent Deployment

The **Deploy new agent** wizard will walk you through generating the installation command. Configure it as follows:

**Section 1 — Select the package:**
- Select **WINDOWS**
- Under Windows, select **MSI 32/64 bits**

> We select Windows MSI because our target machine is Windows 11 Pro. Even though the VM runs on Apple Silicon ARM, Windows 11 ARM still uses the standard Windows MSI Wazuh agent package.

**Section 2 — Server address:**
- Enter your **Wazuh VPS IP address** (or FQDN) in the "Assign a server address" field
- This tells the agent where to send its data
- Enable **"Remember server address"** toggle

**Section 3 — Optional settings:**
- In the **"Assign an agent name"** field, enter: `WindowsVM`

> **Important:** The agent name must be unique and cannot be changed after enrollment. Choose a descriptive name that identifies the machine.

![Step 3 - Deploy New Agent Configuration](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%203.png)

---

### Step 3 — Copy the Installation Command

Once all fields are filled in, Wazuh automatically generates a **PowerShell installation command** in Section 4.

The command looks like this (with your actual server IP):

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.4-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='YOUR_VPS_IP' WAZUH_AGENT_NAME='WindowsVM'
```

**What this command does:**
- Downloads the Wazuh agent MSI installer directly from Wazuh's official package server
- Silently installs it (`/q` flag = quiet/no UI)
- Automatically configures it to point to your Wazuh manager IP
- Assigns the agent name `WindowsVM`

Also note the **Start the agent** command in Section 5:

```powershell
NET START Wazuh
```

**Copy both commands** — you will run them inside the Windows VM.

![Step 4 - Installation Commands Generated](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%204.png)

---

### Step 4 — Open PowerShell as Administrator in the Windows VM

Switch to your **Windows VM** in UTM. The agent installation requires Administrator privileges.

1. Click the **Start menu** (Windows icon in the taskbar)
2. Type `power` in the search bar
3. **Windows PowerShell** will appear as the best match
4. On the right panel, click **"Run as administrator"**

> Do NOT open the regular PowerShell — the installation will fail without administrator privileges.

![Step 5 - Opening PowerShell as Administrator](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%205.png)

---

### Step 5 — Accept the UAC Prompt

Windows will display a **User Account Control (UAC)** prompt asking:

> *"Do you want to allow this app to make changes to your device?"*
> Windows PowerShell — Verified publisher: Microsoft Windows

Click **Yes** to allow PowerShell to run with administrator privileges.

> UAC is a Windows security feature that prevents unauthorised changes to the system. In a real SOC environment, this type of prompt being triggered without user action would itself generate a security alert — worth noting for future detection scenarios.

![Step 6 - UAC Prompt](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%206.png)

---

### Step 6 — Run the Installation Command

In the **Administrator: Windows PowerShell** window:

1. Paste the full installation command copied from the Wazuh dashboard
2. Press **Enter**

The command will:
- Download the Wazuh agent MSI from the internet (~50 MB)
- Silently install it in the background
- Configure it with your Wazuh server address and agent name

> The download may take 1–3 minutes depending on your internet speed. The PowerShell prompt will return once installation is complete.

![Step 7 - Running Installation Command in PowerShell](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%207.png)

---

### Step 7 — Start the Wazuh Agent Service

Once the installation command completes, run the second command to start the Wazuh service:

```powershell
NET START Wazuh
```

Press **Enter**.

This starts the Wazuh agent as a **Windows service**, meaning it will run in the background automatically — even after reboots.

> You should see: `The Wazuh service was started successfully.`

![Step 8 - Starting the Wazuh Service](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%208.png)

---

### Step 8 — Verify Agent is Active in Wazuh Dashboard

Switch back to your **Wazuh dashboard** in the browser.

Navigate to **Endpoints** → click on **WindowsVM** to open the agent detail view.

You should see:

| Field | Value |
|---|---|
| Status | ● active |
| Operating System | Microsoft Windows 11 Pro 10.0.26100.4349 |
| Wazuh Version | v4.14.4 |
| Group | default |
| Cluster Node | node01 |
| Registration Date | Today's date |

The agent dashboard will already show initial data including:
- **Events count evolution** graph
- **MITRE ATT&CK** top tactics (Defense Evasion already detected from normal Windows activity)
- **Vulnerability Detection** panel
- **FIM: Recent Events** (File Integrity Monitoring)

> The fact that MITRE ATT&CK already shows "Defense Evasion" activity from a freshly installed Windows machine is a great example of how sensitive Wazuh is — and why tuning detections is an important SOC skill.

![Step 9 - WindowsVM Agent Active in Wazuh](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%209.png)

---

### Step 9 — Verify All Three Agents Are Active

Navigate back to the main **Endpoints** page to confirm the full lab infrastructure is live.

You should now see **3 active agents**:

| ID | Name | OS | Role |
|---|---|---|---|
| 001 | M2 | macOS 26.4.1 | Host MacBook |
| 003 | 2015 | Kali GNU/Linux 2026.1 | **Attacking Machine** |
| 004 | WindowsVM | Microsoft Windows 11 Pro | **Target Machine** |

All three agents show **● active** status — meaning the Wazuh SIEM on the VPS is receiving telemetry from all machines in real time.

The **Top 5 OS** chart now shows: darwin (1), kali (1), windows (1) — exactly our lab architecture.

![Step 10 - All Three Agents Active](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/Phase%202/images/Step%2010.png)

---

## ✅ Phase 2 Complete — Verification Checklist

- [ ] Wazuh agent installed on Windows 11 VM via PowerShell
- [ ] Wazuh service started and running (`NET START Wazuh`)
- [ ] WindowsVM agent shows **active** status in Wazuh dashboard
- [ ] Agent correctly identified as **Microsoft Windows 11 Pro**
- [ ] All 3 agents active: M2 (macOS), 2015 (Kali), WindowsVM (Windows)
- [ ] Initial telemetry flowing — events visible in WindowsVM agent dashboard

---

## What the Wazuh Dashboard Now Shows

With all agents connected, the Wazuh SIEM is now receiving live telemetry from:

- **M2 MacBook** — host machine activity
- **Kali Linux** — attacker machine (all attack commands will be logged here)
- **Windows 11 VM** — target machine (all attack impacts will be detected here)

This mirrors a real enterprise SOC setup where the SIEM has visibility across multiple endpoints simultaneously. In the next phase, we begin running actual attack scenarios from Kali against the Windows VM and observe how Wazuh detects them.

---

## Key Concepts Demonstrated

**Agent-based monitoring** — Unlike agentless monitoring (which only captures network traffic), agent-based monitoring provides deep endpoint visibility including process execution, file changes, and registry modifications.

**Centralised SIEM** — All three machines report to a single Wazuh manager on the VPS. This is how enterprise SOCs work — a central platform ingesting data from hundreds or thousands of endpoints.

**Automatic MITRE ATT&CK mapping** — Wazuh automatically tagged initial Windows activity as "Defense Evasion" without any manual configuration, demonstrating built-in threat intelligence.

---

## What's Next — Phase 3

In Phase 3 we run **Attack Scenario 1: Nmap Reconnaissance** from the Kali Linux machine against the Windows VM, and observe how Wazuh detects and alerts on network scanning activity.

---

*Documentation for Enterprise-SOC-Lab | Phase 2 of 10*
