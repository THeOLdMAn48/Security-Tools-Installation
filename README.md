<div align="center">

# 🛠️ Security Tools Installation

**Step-by-step installation and usage guides for defensive and offensive security tools on Linux (Ubuntu).**

![Platform](https://img.shields.io/badge/Platform-Ubuntu_Linux-E95420?logo=ubuntu&logoColor=white)
![Focus](https://img.shields.io/badge/Focus-Blue_Team_%26_Red_Team-0A66C2)
![Level](https://img.shields.io/badge/Level-Beginner_Friendly-2BB673)
![Purpose](https://img.shields.io/badge/Use-Education_%26_Home_Lab-green)
![Status](https://img.shields.io/badge/Status-Growing-orange)

Companion repository for the **SEC W TOM** YouTube video series.
📺 [@TOmSec-OG](https://www.youtube.com/@TOmSec-OG) · 💻 [THeOLdMAn48](https://github.com/THeOLdMAn48/)

*Learn - Secure - Grow*

</div>

> ⚠️ **Legal and ethical notice.** Everything here is for **education and authorized testing only**. Use offensive tools **only on systems you own or have written permission to test** (for example your own home-lab VMs). Unauthorized scanning, access, or attacks are illegal. You are responsible for how you use this material.

---

## 📑 Table of Contents

1. [About this repository](#1-about-this-repository)
2. [Who is this for?](#2-who-is-this-for)
3. [Tools covered](#3-tools-covered)
4. [What each guide contains](#4-what-each-guide-contains)
5. [Repository structure](#5-repository-structure)
6. [Recommended home lab](#6-recommended-home-lab)
7. [How to use this repository](#7-how-to-use-this-repository)
8. [Learning path](#8-learning-path)
9. [Video series](#9-video-series)
10. [Contributing](#10-contributing)
11. [Disclaimer](#11-disclaimer)
12. [Author](#12-author)

---

## 1. About this repository

Installing security tools is often the hardest first step for beginners. Official docs are long, commands differ between versions, and small mistakes waste hours.

This repository fixes that with **short, clear, copy-paste guides** for installing and starting to use popular security tools on **Ubuntu**. It covers both sides of cybersecurity:

- 🔵 **Defensive (Blue Team):** detect, monitor, investigate, and respond to threats.
- 🔴 **Offensive (Red Team):** discover weaknesses and simulate attacks in a safe, authorized lab.

Each tool has its own folder with its own README, so you can jump straight to the tool you need.

---

## 2. Who is this for?

- Students starting a cybersecurity career (SOC analyst, pentester, security engineer)
- Learners building a **home lab** for practice
- Viewers following the **SEC W TOM** video tutorials
- Anyone who wants a clean, repeatable setup without digging through scattered docs

No advanced knowledge needed. Basic Linux terminal comfort is enough to start.

---

## 3. Tools covered

### 🔵 Defensive tools

| Tool | What it does | Status |
|---|---|---|
| [**Suricata**](./Suricata/) | Network IDS/IPS and network security monitoring engine. Detects and blocks threats using rules | ✅ Available |
| **Wazuh** | Open-source SIEM / XDR platform for log analysis, endpoint monitoring, and alerting | 🚧 Coming soon |
| **TheHive** | Security incident response and case management platform for SOC teams | 🚧 Coming soon |

### 🔴 Offensive tools

| Tool | What it does | Status |
|---|---|---|
| **Feroxbuster** | Fast web content and directory discovery tool | 🚧 Coming soon |
| **Sliver** | Command-and-control (C2) framework for adversary simulation | 🚧 Coming soon |
| **AutoPWN Suite** | Automated vulnerability scanning and exploitation helper | 🚧 Coming soon |

### ➕ More tools

More defensive and offensive tools will be added over time as the video series grows. Watch or star the repo to get updates.

**Legend:** ✅ guide is published · 🚧 planned for the series

---

## 4. What each guide contains

Every tool folder follows the same pattern so you always know where to look:

| Section | What you get |
|---|---|
| **Introduction** | What the tool is and why it matters |
| **Requirements** | What is mandatory and what is only recommended |
| **Installation** | Clear, tested, copy-paste steps for Ubuntu |
| **Configuration** | First-time setup and important files |
| **Usage** | Everyday commands and examples |
| **Lab scenarios** | Safe practice exercises in your own lab |
| **Troubleshooting** | Common errors and fixes |
| **Cheat sheet** | Quick command reference |

### Example: what the Suricata guide covers

The [Suricata guide](./Suricata/) is the first complete one. It includes:

- Install on Ubuntu (Ubuntu repo or official OISF PPA) and verification
- Configuration: `HOME_NET`, interface, rule loading
- Rule updates with `suricata-update`
- Reading alerts and logs (`fast.log`, `eve.json`, `jq` queries)
- **How rules work and how to write your own**
- **12 attack scenarios with detection rules** (port scans, SSH brute force, SQL injection, XSS, directory traversal, Log4Shell, reverse shells, DNS tunneling, and more)
- IPS (blocking) mode, tuning false positives, performance, and SIEM integration

---

## 5. Repository structure

```
Security-Tools-Installation/
├── README.md                ← you are here (overview of the whole series)
├── Suricata/
│   ├── README.md            ← Suricata install, usage, rules, attack scenarios
│   └── rules/
│       └── local.rules      ← lab detection rules
├── Wazuh/                   ← coming soon
├── TheHive/                 ← coming soon
├── Feroxbuster/             ← coming soon
├── Sliver/                  ← coming soon
└── AutoPWN-Suite/           ← coming soon
```

Folders marked *coming soon* will appear as each video and guide is published.

---

## 6. Recommended home lab

A simple, safe lab is enough for almost every guide here:

```
┌─────────────────┐        ┌──────────────────────────────┐
│  Attacker VM    │ ─────► │  Ubuntu Server VM            │
│  Kali Linux     │        │  (tool installed + target)   │
└─────────────────┘        └──────────────────────────────┘
        Host-only or NAT network (isolated from real networks)
```

**Suggested setup (recommendations, not strict requirements):**

| Item | Suggestion |
|---|---|
| Virtualization | VirtualBox, VMware, or Proxmox |
| Defensive VM | Ubuntu Server LTS, 2+ CPU cores, 4 GB+ RAM, 20 GB+ disk |
| Attacker VM | Kali Linux (for the offensive tools and attack scenarios) |
| Network | Host-only or NAT, **never** your real production network |
| Good habit | Take a VM snapshot before every installation |

Heavier tools (such as a full SIEM stack) may need more RAM and disk. Each tool's README states what it needs.

---

### Prerequisites

- A Linux machine or VM (Ubuntu recommended)
- A user with `sudo` access
- Internet access for package installs
- Basic terminal skills (`cd`, `ls`, `nano`, `sudo`)

---

## 7. How to use this repository

1. **Pick a tool** from the [table above](#3-tools-covered) and open its folder.
2. **Read the Requirements** section first.
3. **Follow the steps in order** and copy the commands into your terminal.
4. **Run the lab scenarios** only inside your own isolated lab.
5. **Use the Troubleshooting section** if something fails.
6. **Watch the matching video** on the SEC W TOM channel for a live walkthrough.

To download everything:

```bash
git clone https://github.com/THeOLdMAn48/Security-Tools-Installation.git
cd Security-Tools-Installation
```

---

## 8. Learning path

If you are new, this order works well:

| Step | Tool | Skill you build |
|---|---|---|
| 1 | **Suricata** | Network traffic, detection rules, alert analysis |
| 2 | **Wazuh** | Centralized logging, SIEM basics, endpoint monitoring |
| 3 | **TheHive** | Incident response, case handling, SOC workflow |
| 4 | **Feroxbuster** | Web reconnaissance and attack-surface discovery |
| 5 | **AutoPWN Suite** | Vulnerability scanning and assessment |
| 6 | **Sliver** | Adversary simulation and understanding attacker tradecraft |

Blue Team tools first helps you understand what attacks *look like* before you practice them.

---

## 9. Video series

Every guide matches a video on the **SEC W TOM** YouTube channel, explained in Hinglish with live demos.

📺 **Channel:** [youtube.com/@TOmSec-OG](https://www.youtube.com/@TOmSec-OG)

Subscribe so you do not miss new installation tutorials.

---

## 10. Contributing

Contributions are welcome.

- 🐞 Found a broken command or typo? Open an **Issue**.
- 💡 Have a better method or a new tool suggestion? Open an **Issue** or **Pull Request**.
- ✅ Please test commands on a clean Ubuntu VM before submitting.
- 📁 Keep the same folder and section structure for new tools.

---

## 11. Disclaimer

This repository is provided **for educational purposes and authorized security testing only**. The author is not responsible for misuse or damage caused by the information or tools described here. Always follow the law, get written permission before testing any system you do not own, and respect each tool's own license and terms.

---

## 12. Author

**Om Gohil**, cybersecurity student and creator of **SEC W TOM**.

- 💻 GitHub: [THeOLdMAn48](https://github.com/THeOLdMAn48/)
- 📺 YouTube: [@TOmSec-OG](https://www.youtube.com/@TOmSec-OG)

---

<div align="center">

### ⭐ If this repository helps you, star it and subscribe to **SEC W TOM**.

**Learn - Secure - Grow**

</div>
