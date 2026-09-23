# NETWORKWALKS-ZUHAIR_HUSSAIN-B083-WK2-CYBERSECURITY-LAB

# Footprinting & Network Scanning — Cybersecurity Lab Report

**Intern:** Zuhair Hussain
**Program:** Networkwalks Cybersecurity Internship 
**Batch:** B083 — Week 2
**Date:** 23/9/2026
**Target (authorized):** networkwalks.com + own local network

> ⚠️ All testing below was performed only against systems I was explicitly authorized to test, under a signed Letter of Authorization. No unauthorized systems were scanned.

---

## 📌 Overview

This report covers two parts of the Week 2 project:

1. **Part 1 – Footprinting with Kali Linux tools** (whois, whatweb, nslookup, curl, wafw00f, dnsrecon) on `networkwalks.com`
2. **Part 2 – Network scanning with Zenmap** on my own local network (LAN)

The goal was to learn how attackers gather public information about a target *before* ever touching it — and how defenders can use the same tools to see what they're exposing.

---

## Part 1: Footprinting with Multiple Kali Tools

**Tool used:** Kali Linux (VirtualBox VM)
**Target:** `networkwalks.com` (authorized)

### Task 1 — whois
```
whois networkwalks.com
```
**Purpose:** Find domain owner, registrar, registration/expiry dates, and name servers.

<img width="1366" height="662" alt="task1_whois" src="https://github.com/user-attachments/assets/fedf7b58-6018-4282-9c34-75b1e0e0740e" />

**Key finding:** Domain is registered via GoDaddy, hosted on HostGator name servers.

---

### Task 2 — whatweb
```
whatweb networkwalks.com
```
**Purpose:** Fingerprint the web server, CMS, and plugin versions running on the site.

<img width="1366" height="662" alt="task2_whatweb" src="https://github.com/user-attachments/assets/f0ca9929-0fd0-4539-ba83-ea3b893038ba" />

**Key finding:** Site runs Apache + WordPress with visible plugin versions — useful for an attacker to look up known vulnerabilities for that exact version.

---

### Task 3 — nslookup
```
nslookup networkwalks.com
```
**Purpose:** Resolve the domain name to its real IP address.

<img width="439" height="217" alt="task3_nslookup" src="https://github.com/user-attachments/assets/7c490b56-a9ad-49dd-b2a8-1bf74bd914bf" />

**Key finding:** Domain resolves to `192.232.216.135`.

---

### Task 4 — curl -I
```
curl -I https://networkwalks.com
```
**Purpose:** Read raw HTTP response headers (server type, cookies, redirects).

<img width="1351" height="288" alt="task4_curl" src="https://github.com/user-attachments/assets/a3d21e46-b23d-4344-b997-e472969d71e7" />

**Key finding:** Headers reveal the Apache server banner and hint at a WordPress REST API endpoint (`/wp-json/`).

---

### Task 5 — wafw00f
```
wafw00f networkwalks.com
```
**Purpose:** Detect whether a Web Application Firewall (WAF) is protecting the site.

<img width="681" height="302" alt="task5_wafw00f" src="https://github.com/user-attachments/assets/45ade77c-90da-416c-96fc-941341ee01e5" />

**Key finding:** Site is protected by a WAF (ModSecurity/SpiderLabs) — meaning naive attacks would be blocked/logged.

---

### Task 6 — dnsrecon
```
dnsrecon -d networkwalks.com > task6_dnsrecon.txt 2>&1
```
**Purpose:** Enumerate all DNS records (NS, MX, SOA, TXT, SRV).

<img width="1105" height="487" alt="task6_dnsrecon" src="https://github.com/user-attachments/assets/ba7dac0c-e0e7-45f3-b005-219ccb27c046" />

**Key findings:**
| Record Type | Value |
|---|---|
| SOA / NS | ns6135.hostgator.com, ns6136.hostgator.com |
| MX | mail.networkwalks.com → 192.232.216.135 |
| A | networkwalks.com → 192.232.216.135 |
| TXT (SPF) | `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` |
| SRV | 8 autodiscover records pointing to cpanelemaildiscovery.cpanel.net |

**Why it matters:** DNS records reveal the mail server setup, hosting provider, and email infrastructure — all useful footholds for an attacker planning further recon.

---

## Part 2: Network Scanning with Zenmap

**Tool used:** Zenmap (Nmap GUI) — installed on Windows PC
**Target:** My own local LAN (authorized — own devices only)

### Task 1 — Install Zenmap
Downloaded and installed from the official site: `nmap.org/download.html` (includes Npcap driver).

---

### Task 2 — Find local IP & subnet
```
ipconfig
```
**Result:**
- IPv4 Address: `192.168.1.111`
- Subnet Mask: `255.255.255.0`
- → Subnet to scan: `192.168.1.0/24`
<img width="1282" height="726" alt="ipconfig_cmd" src="https://github.com/user-attachments/assets/115c6f3e-e205-4ea6-93f7-7c4521fe39fa" />

---

### Task 3 — Ping scan for live hosts
```
nmap -sn 192.168.1.0/24
```
<img width="740" height="490" alt="Zenmap_Scan" src="https://github.com/user-attachments/assets/eeebe562-3217-42d9-af80-e98c11fdb0b2" />

---

### Task 4 — How many hosts are live?
**Answer:** 3 hosts

---

### Task 5 — Live host IP addresses
| IP Address | Notes |
|---|---|
| 192.168.1.1 | Router/Gateway |
| 192.168.1.107 | Unknown device on network |
| 192.168.1.111 | My own laptop |

---

### Task 6 — Live host MAC addresses
| IP | MAC Address | Vendor |
|---|---|---|
| 192.168.1.1 | E8:65:D4:D6:BA:18 | Tenda Technology (router) |
| 192.168.1.107 | 42:BA:E3:F0:B0:FD | Unknown |
| 192.168.1.111 | *(from `ipconfig /all`)* | My laptop |
<img width="770" height="719" alt="ipconfigall_scan" src="https://github.com/user-attachments/assets/2e5778ee-8ad5-449b-84a8-b0acb0b6db47" />

---

### Task 7 — Topology diagram
Saved as PDF via Zenmap → Topology tab → Save Graphic → PDF.

[Topology.pdf](https://github.com/user-attachments/files/32579942/Topology.pdf)
---

## 🎓 Lessons Learned

- Footprinting/recon tools **never touch the target directly** — they only read information that is already public. This makes passive recon powerful and very hard to detect.
- Small pieces of public info (DNS records, HTTP headers, software versions) combine into a full profile an attacker can use to plan further attacks.
- The same tools used offensively are used by defenders to see what their own organization is leaking — and reduce it.
- On a local network, even a simple ping scan instantly reveals every connected device, its IP, and its MAC/vendor — a reminder to secure home/office networks (strong Wi-Fi passwords, MAC filtering, updated router firmware).

---

## ⚖️ Disclaimer

This work was carried out strictly for educational purposes as part of the Networkwalks Cybersecurity Internship, under a signed Letter of Authorization, and only against systems I own or was explicitly permitted to test. Unauthorized scanning or testing of any system is illegal.

---

*Report prepared by Zuhair Hussain — Networkwalks Cybersecurity & Ethical Hacking Internship, Batch B083*
