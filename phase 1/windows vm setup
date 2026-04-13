# Phase 1 — Windows 11 Pro VM Setup on Apple Silicon Mac (UTM)

## Overview

This document covers the complete setup of a **Windows 11 Pro ARM virtual machine** on an Apple Silicon MacBook using **UTM**, which serves as the **target/victim machine** in all attack scenarios throughout this SOC lab project.

---

## Why Windows 11 Pro as the Target Machine?

In real enterprise environments, the vast majority of endpoints that SOC analysts defend run Windows. The most critical and commonly seen attacks in a SOC — credential dumping, lateral movement, registry-based persistence, and Windows Event Log analysis — all occur in Windows environments.

Using Windows 11 Pro specifically gives us access to:
- **Group Policy** — mirrors enterprise domain-joined machines
- **Windows Event Logging** — rich log source for Wazuh to ingest
- **Remote Desktop Protocol (RDP)** — target for brute force scenarios
- **BitLocker and security features** — realistic enterprise hardening to work around
- **Windows Defender** — real EDR to test detections against

This makes the lab look and behave like a real corporate environment rather than a personal hobbyist setup.

---

## Why UTM on Apple Silicon?

**UTM** is a free, open-source virtualisation app for macOS built on QEMU. It is the recommended virtualisation solution for Apple Silicon Macs (M1/M2/M3) because:

- VirtualBox and VMware do not fully support Apple Silicon ARM architecture
- UTM natively supports ARM-based virtual machines which is required for Windows on Apple Silicon
- It is completely free unlike VMware Fusion Pro (paid)
- It supports SPICE guest tools for clipboard sharing and dynamic display resolution

> **Important:** Standard Windows 10/11 x86/x64 ISO files will NOT work on Apple Silicon. Apple Silicon uses ARM architecture, so we need a Windows 11 ARM build — which UTM handles automatically via CrystalFetch.

---

## Environment Details

| Component | Details |
|---|---|
| **Host Machine** | MacBook with Apple Silicon (M1/M2/M3) |
| **Virtualisation Software** | UTM (latest version) — [utm.app](https://utm.app) |
| **Guest OS** | Windows 11 Pro ARM64 |
| **Windows Build** | Latest (downloaded via CrystalFetch) |
| **Architecture** | ARM64 (Apple Silicon native) |
| **VM Storage** | 64 GB virtual disk |
| **Role in Lab** | Target / Victim machine |

---

## Prerequisites

Before starting, ensure you have the following:

- MacBook with Apple Silicon chip (M1, M2, or M3)
- At least **80 GB free disk space** (64 GB for VM + overhead)
- At least **8 GB RAM** on your MacBook
- Stable internet connection (ISO download is ~5–6 GB)
- UTM installed from [utm.app](https://utm.app)

---

## Step-by-Step Installation Guide

### Step 1 — Install UTM

1. Open your browser and go to [utm.app](https://utm.app)
2. Click **Download** and open the downloaded `.dmg` file
3. Drag the UTM app into your **Applications** folder
4. Open UTM from Applications
5. Click **"Create a New Virtual Machine"**
6. On the next screen select **"Virtualise"**

> Virtualise uses your Mac's hardware directly for near-native performance. The alternative "Emulate" is much slower and is for running non-ARM operating systems.

---

### Step 2 — Select Windows as the Operating System

1. In the OS selection screen, click **Windows**
2. You will see the **Image File Type** screen with these options:
   - ☑️ Install Windows 10 or higher
   - 🔗 Fetch latest Windows installer...
   - 🔗 Windows Install Guide
   - ☑️ Install drivers and SPICE tools

3. Make sure both checkboxes are ticked:
   - **"Install Windows 10 or higher"** ✅
   - **"Install drivers and SPICE tools"** ✅

> **SPICE tools** are guest drivers that enable clipboard sharing between your Mac and the VM, and allow the VM display to resize dynamically. Always keep this ticked.

---

### Step 3 — Download Windows 11 ARM via CrystalFetch

Instead of manually finding and downloading a Windows ISO, UTM includes **CrystalFetch** — a built-in tool that automatically downloads the correct ARM build of Windows for Apple Silicon.

1. Click **"Fetch latest Windows installer..."**
2. The CrystalFetch window opens with these settings — configure as follows:

| Setting | Value |
|---|---|
| Version | Windows 11 |
| Build | Latest |
| Architecture | **Apple Silicon** |
| Language | English (United States) |
| Edition | Windows 11 |

3. At the bottom of the window you will see a filename generating — this confirms CrystalFetch is fetching the correct build metadata
4. Click **"Download..."** and choose a save location (Desktop is fine)
5. The download will take **15–30 minutes** depending on your internet speed — the file is approximately **5–6 GB**

> **Why Apple Silicon architecture?** Selecting Intel x64 here would download an x86 ISO that cannot run on Apple Silicon hardware. Always select Apple Silicon on M1/M2/M3 Macs.

---

### Step 4 — Attach the ISO to UTM

1. Once the download completes, go back to the UTM Windows setup screen
2. Under **Boot ISO Image**, click **Browse**
3. Navigate to where you saved the ISO and select it
4. Click **Continue**

---

### Step 5 — Skip Shared Directory

The next screen asks you to configure a **Shared Directory** — a folder on your Mac that the VM can access.

- **Skip this** — click **Continue** without selecting any path
- A shared directory is not needed for this security lab and keeping the VM isolated is better practice

---

### Step 6 — Configure VM Hardware

On the hardware configuration screen, set the following:

| Setting | Recommended Value | Notes |
|---|---|---|
| RAM | 4096 MB (4 GB) minimum | Use 6144 MB (6 GB) if your Mac has 16 GB+ RAM |
| CPU Cores | 2 minimum | Use 4 if available |
| Storage | 64 GB | Default — sufficient for Windows + tools |

> **Rule of thumb:** Never allocate more than half your total Mac RAM to the VM. If you have 8 GB total, give the VM 4 GB. If you have 16 GB, give it 6–8 GB.

Click **Save** to create the VM.

---

### Step 7 — First Boot and UEFI Shell

1. Click the **Play** button in UTM to start the VM
2. The VM may boot into the **UEFI Interactive Shell** — this is normal on first boot
3. You will see a screen showing a mapping table with entries like `FS0:`, `FS1:`, `BLK0:` etc.

If this happens, follow these steps to manually boot the installer:

**Type the following commands one at a time, pressing Enter after each:**

```
FS1:
```
```
ls
```

Look at the output. If you see an `EFI` folder listed, continue:

```
cd EFI\Boot
```
```
Bootaa64.efi
```

4. You will see the message: **"Press any key to boot from CD or DVD..."**
5. **Press any key immediately** — you have approximately 2–3 seconds before this times out
6. The Windows 11 Setup screen will load

> **Tip:** Have your finger on the keyboard ready *before* you run `Bootaa64.efi` so you can press a key the instant the prompt appears.

---

### Step 8 — Windows 11 Setup Wizard

Work through the installer screens as follows:

**Screen 1 — Select Setup Option**
- Select: **Install Windows 11**
- Tick the checkbox: **"I agree everything will be deleted..."**
- Click **Next**

**Screen 2 — Product Key**
- Click **"I don't have a product key"**

> Windows will install and run fully without a product key. A small "Activate Windows" watermark appears in the corner of the desktop but has zero impact on functionality for lab use.

**Screen 3 — Select Image**
- Select: **Windows 11 Pro**
- Click **Next**

> Choose Pro over Home. Windows 11 Pro includes Group Policy Editor, Remote Desktop, and additional security features that mirror real enterprise environments.

**Screen 4 — License Agreement**
- Accept the terms
- Click **Next**

**Screen 5 — Installation Type**
- Select: **Custom: Install Windows only (advanced)**

**Screen 6 — Select Location to Install Windows**
- You will see: **Disk 0 Unallocated Space — 64.0 GB**
- Select it and click **Next**

> Windows will automatically create the necessary partitions. No manual partitioning is needed.

---

### Step 9 — Installation and Reboots

- Windows will now begin copying and installing files
- This process takes approximately **15–20 minutes**
- The VM will **reboot 2–3 times** automatically during installation — this is completely normal
- Do not interrupt the process

---

### Step 10 — Windows Out of Box Experience (OOBE)

After installation completes, Windows will walk you through initial setup:

| Screen | What to Do |
|---|---|
| Country/Region | Select **Australia** |
| Keyboard Layout | Select **US** |
| Second Keyboard Layout | Skip |
| Network | Click **"I don't have internet"** → **"Continue with limited setup"** |
| Name your PC | Enter a name e.g. `SOC-Target` |
| Microsoft Account | Click **"Sign-in options"** → **"Offline account"** → **"Limited experience"** |
| Username | Set a username e.g. `labuser` |
| Password | Set a password you will remember |
| Privacy Settings | Turn everything **Off** → Accept |

> **Why offline account?** Using a local account instead of a Microsoft account keeps the VM self-contained and avoids any sync or privacy issues in a lab environment.

---

### Step 11 — Install SPICE Guest Tools (UTM Drivers)

After Windows boots to the desktop:

1. In UTM's toolbar, click the **CD/DVD icon**
2. Select **"Install SPICE Guest Tools and QEMU Agent"**
3. Inside the Windows VM, open **File Explorer** → **This PC**
4. You will see a CD Drive — open it and run the installer
5. Follow the prompts and restart the VM when prompted

> SPICE tools enable clipboard sharing between your Mac and the VM, and allow the display to resize when you resize the UTM window. Install this before doing anything else in the VM.

---

## ✅ Phase 1 Complete — Verification Checklist

Confirm the following before moving to Phase 2:

- [ ] UTM installed on MacBook
- [ ] Windows 11 Pro ARM VM created and running
- [ ] Windows desktop accessible
- [ ] SPICE Guest Tools installed
- [ ] VM network connectivity working (open Edge and test a website)
- [ ] Local user account created (no Microsoft account)
- [ ] VM name set to `SOC-Target`

---

## What's Next — Phase 2

In Phase 2 we will:
1. Install the **Wazuh Agent** on this Windows 11 VM
2. Install the **Wazuh Agent** on the Kali Linux machine
3. Connect both agents to the **Wazuh SIEM on the VPS**
4. Verify that both machines are visible and reporting in the Wazuh dashboard

Once both agents are connected, the detection infrastructure is live and we can begin running attack scenarios.

---

## Troubleshooting

**VM boots to UEFI shell every time:**
Manually run `Bootaa64.efi` as described in Step 7. After Windows is installed this will no longer happen.

**"Press any key" prompt disappears too fast:**
Run `Bootaa64.efi` again and immediately press a key — you need to react within 2–3 seconds.

**Windows setup is very slow:**
This is normal on first boot. Allocating more RAM and CPU cores in UTM settings will help.

**Black screen after reboot:**
Wait 2–3 minutes — Windows is still initialising. If it persists, stop and restart the VM in UTM.

---

*Documentation for Enterprise-SOC-Lab | Phase 1 of 10*
