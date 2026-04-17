# Phase 3 — Deploy Suricata IDS/IPS and Integrate with Wazuh

## Overview

In this phase we install **Suricata** — an open source Intrusion Detection and Prevention System (IDS/IPS) — directly on the Wazuh VPS server and integrate it with Wazuh so that network alerts automatically appear in your SIEM dashboard.

By the end of this phase your detection pipeline looks like this:

```
[Attack Traffic] → [Suricata detects it] → [eve.json log] → [Wazuh ingests] → [Dashboard Alert]
```

**What Suricata covers for this project:**

| Attack Scenario | Detection |
|---|---|
| Nmap port scan | Suricata ET SCAN rules |
| Hydra SSH brute force | Suricata + Wazuh |
| Metasploit exploitation | Suricata + Wazuh |
| Simulated C2 beaconing | Suricata |

---

## Prerequisites

- Wazuh server running on Ubuntu 24.04 VPS ✅
- SSH access to your VPS ✅
- Windows 10 VM enrolled as Wazuh agent ✅

---

## Step 1 — SSH Into Your VPS

Open Terminal on your Mac and connect to your VPS:

```bash
ssh root@<your-vps-ip>
```

Enter your password when prompted.

![Step 1 - SSH Login](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%201.png?raw=true)

---

## Step 2 — Add the Official Suricata Repository

This adds the OISF (Open Information Security Foundation) PPA so you get the latest stable version of Suricata, not an outdated package:

```bash
sudo add-apt-repository ppa:oisf/suricata-stable -y
```

You will see a long description of Suricata's features scroll by. Wait for `Done` at the bottom.

![Step 2 - Add Suricata Repository](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%202.png?raw=true)

---

## Step 3 — Update Package Lists

Refresh Ubuntu's package index so it knows about the new Suricata source:

```bash
sudo apt update
```

![Step 3 - apt update](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%203.png?raw=true)

---

## Step 4 — Install Suricata

```bash
sudo apt install suricata -y
```

This downloads and installs Suricata and all its dependencies. It will take 1–2 minutes.

![Step 4 - Installing Suricata](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%204.png?raw=true)

---

## Step 5 — Installation Complete

When installation finishes you will see Suricata's service being set up and a note about a pending kernel upgrade. This is normal — do not reboot yet.

![Step 5 - Installation Complete](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%205.png?raw=true)

---

## Step 6 — Verify the Installation

Confirm Suricata installed correctly and check the version:

```bash
suricata -V
```

You should see **Suricata 8.0.4** (or later). The output also shows you the full list of command options available.

> **Note:** Use `-V` (capital V) for version. The `--version` flag is not supported and will show the help menu instead.

![Step 6 - Verify Suricata Version](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%206.png?raw=true)

---

## Step 7 — Download Detection Rules

Run `suricata-update` to download the **Emerging Threats Open** ruleset. This is a free, community-maintained ruleset with ~50,000 rules covering port scans, brute force, exploits, malware C2 beaconing, and more:

```bash
sudo suricata-update
```

You will see it fetching rules from `rules.emergingthreats.net`. This takes 1–3 minutes depending on your connection. At the end you will see a line like:

```
Writing rules to /var/lib/suricata/rules/suricata.rules: total: 65458; enabled: 49593
```

![Step 7 - Download Rules](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%207.png?raw=true)

---

## Step 8 — View Available Rule Sources

List all available rulesets you can optionally enable later:

```bash
sudo suricata-update list-sources
```

The free sources relevant to this project include:
- **et/open** — Emerging Threats Open (main ruleset, already downloaded)
- **abuse.ch/feodotracker** — Botnet C2 IP blocklist
- **aleksibovellan/nmap** — Dedicated Nmap detection rules

![Step 8 - List Rule Sources](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%208.png?raw=true)

---

## Step 9 — Enable the Emerging Threats Open Ruleset

Explicitly enable the `et/open` source so it is included in all future `suricata-update` runs:

```bash
sudo suricata-update enable-source et/open
```

You should see: `Source et/open enabled`

![Step 9 - Enable et/open](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%209.png?raw=true)

---

## Step 10 — Re-run suricata-update

Run the update again now that `et/open` is explicitly enabled. Since the rules were just downloaded, it will say `No changes detected` — this is expected:

```bash
sudo suricata-update
```

From now on, run `sudo suricata-update` weekly to keep your rules fresh.

![Step 10 - Re-run suricata-update](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%2010.png?raw=true)

---

## Step 11 — Start Suricata and Verify

Enable Suricata to start automatically on boot, then start it now:

```bash
sudo systemctl enable suricata
sudo systemctl start suricata
sudo systemctl status suricata
```

You are looking for **`active (running)`** in green. You will also notice a warning about `no rules were loaded` — this is because Suricata started before the rules were downloaded. The next step fixes this.

![Step 11 - Start Suricata](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%2011.png?raw=true)

---

## Step 12 — Reload Suricata With Rules

Restart Suricata so it picks up the downloaded rules, then verify rules loaded correctly:

```bash
sudo systemctl restart suricata
sudo grep -i "rules loaded" /var/log/suricata/suricata.log
```

You should see a line confirming rules loaded:

```
49598 signatures processed. 1278 are IP-only rules, 4484 are inspecting packet payload...
```

Then open the Wazuh config file ready for the next step:

```bash
nano /var/ossec/etc/ossec.conf
```

![Step 12 - Reload Suricata With Rules](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%2012.png?raw=true)

---

## Step 13 — Integrate Suricata With Wazuh

This is the critical integration step. Wazuh needs to know where Suricata writes its alerts so it can ingest them into the SIEM.

In the `ossec.conf` file that is now open, scroll to the **very bottom** and add the following block **before** the final `</ossec_config>` closing tag:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Your file bottom should look like this after adding:

```xml
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/dpkg.log</location>
  </localfile>

  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>

</ossec_config>
```

Save and exit: `Ctrl+X` → `Y` → `Enter`

> **Why this works:** Suricata writes all alerts to `/var/log/suricata/eve.json` in JSON format. By adding this `<localfile>` block, Wazuh's log collector monitors that file in real time. Every time Suricata fires an alert, Wazuh reads it, processes it through its ruleset, and surfaces it as a security event in the dashboard.

![Step 13 - Edit ossec.conf](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%2013.png?raw=true)

---

## Step 14 — Restart Wazuh Manager

Apply the configuration change by restarting Wazuh:

```bash
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager
```

Confirm you see **`active (running)`**. The Wazuh manager is now watching Suricata's alert log.

![Step 14 - Restart Wazuh Manager](https://github.com/yogesh1025/enterprise-soc-lab/blob/main/phase%203/images/Step%2014.png?raw=true)

---

## Verification Checklist

Run these commands to confirm everything is working end-to-end:

```bash
# 1. Suricata is running
sudo systemctl status suricata

# 2. Rules are loaded (should show ~49,000+)
sudo grep -i "signatures processed" /var/log/suricata/suricata.log

# 3. Alert log exists and is being written to
sudo ls -lh /var/log/suricata/eve.json

# 4. Wazuh is running
sudo systemctl status wazuh-manager
```

| Check | Expected Result |
|---|---|
| Suricata status | `active (running)` |
| Rules loaded | `49,000+` signatures |
| eve.json | File exists and has recent timestamps |
| Wazuh status | `active (running)` |

---

## What's Been Built

```
┌─────────────────────────────────────────────┐
│              Wazuh VPS Server               │
│                                             │
│  ┌─────────────┐      ┌──────────────────┐  │
│  │  Suricata   │─────▶│   eve.json log   │  │
│  │  IDS/IPS    │      └────────┬─────────┘  │
│  │  49k rules  │               │            │
│  └─────────────┘               ▼            │
│       │                ┌──────────────────┐  │
│       │ monitors       │  Wazuh Manager   │  │
│       ▼ eth0           │  (reads alerts)  │  │
│  [Network Traffic]     └──────────────────┘  │
└─────────────────────────────────────────────┘
```

**SOC capabilities now active:**

- ✅ SIEM platform experience (Wazuh)
- ✅ IDS/IPS experience (Suricata)
- ✅ Network traffic monitoring on eth0
- ✅ 49,593 active detection signatures
- ✅ Real-time alert pipeline to SIEM

---

## Next Steps — Phase 4

With Suricata integrated, the next phase is running live attack scenarios and generating real alerts:

1. **Use Kali Linux** as the attacker
2. **Run Nmap port scan** against the VPS and observe Suricata fire `ET SCAN` alerts
3. **Run Hydra SSH brute force** and observe alerts in Wazuh dashboard
4. **Write your first incident report** documenting what you detected

---

## MITRE ATT&CK Mapping

| Technique | ID | Detected By |
|---|---|---|
| Network Service Discovery | T1046 | Suricata (ET SCAN rules) |
| Brute Force: Password Spraying | T1110.003 | Suricata + Wazuh |
| Exploit Public-Facing Application | T1190 | Suricata + Wazuh |
| Command and Scripting Interpreter | T1059 | Wazuh EDR |
| Command and Control Beaconing | T1071 | Suricata |
