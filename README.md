# Week 2 – Footprinting & Network Scanning | Networkwalks Cybersecurity Internship

**Author:** Emmanuel Ufuah
**Program:** B083-Networkwalks
**Modules:** W2-PM1 (Multiple Kali Tools) · W2-PM5 (Zenmap Scanning)
**Date:** 17 September 2026
**Target:** `networkwalks.com` (written permission secured) + my own local LAN

> ⚠️ **Disclaimer:** All activity here was performed only against systems I own or had explicit written permission to test. This repo is for educational purposes as part of a cybersecurity internship lab. Do not use anything here to access systems you don't own or have authorization to test — unauthorized access is illegal in most jurisdictions.

---

## 📋 Overview

This write-up documents Week 2 of my Cybersecurity & Ethical Hacking internship: a footprinting pass against `networkwalks.com` using multiple Kali Linux tools, and a network discovery scan of my local subnet using Zenmap. Together they show the progression from open-source information gathering to live host discovery.

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux (VirtualBox VM) | OS environment for all activities |
| `whois` | Domain registration details |
| `whatweb` | Web technology fingerprinting |
| `nslookup` | DNS resolution |
| `curl -i` | HTTP response header inspection |
| `wafw00f` | WAF detection |
| `theHarvester` | OSINT enumeration (used in place of `dnsrecon`, which wasn't installed) |
| Zenmap (Nmap GUI) | Local subnet host discovery |

---

## 🔎 Phase 1: Footprinting & Reconnaissance

### WHOIS
```
$ whois networkwalks.com
Registrar: GoDaddy.com, LLC
Creation Date: 2019-11-06
Registry Expiry Date: 2027-11-06
Name Server: NS6135.HOSTGATOR.COM
Name Server: NS6136.HOSTGATOR.COM
Registrant: Registration Private (Domains By Proxy, LLC)
```
Domain is privacy-protected and locked (client transfer/update/renew/delete prohibited).



### WhatWeb
```
$ whatweb networkwalks.com
https://networkwalks.com [200 OK] Apache, Bootstrap[7.1], IP[192.232.216.135],
JQuery[3.7.1], MetaGenerator[WordPress 7.1, WordPress Download Manager 3.3.58],
Title[Networkwalks Academy]
```
Identified **WordPress 7.1** and **WP Download Manager 3.3.58**.



### Nslookup
```
$ nslookup networkwalks.com
Server:  8.8.8.8
Address: 192.232.216.135
```



### curl -i
```
$ curl -i networkwalks.com
HTTP/2 200
server: Apache
x-redirect-by: WordPress - Really Simple Security
link: <https://networkwalks.com/wp-json/>; rel="https://api.w.org/"
```
Exposed the WordPress REST API endpoint `/wp-json/`.



### Wafw00f
```
$ wafw00f networkwalks.com
[+] The site https://networkwalks.com is behind ModSecurity (SpiderLabs) WAF.
```



### DNSRecon → theHarvester (fallback)
`dnsrecon` wasn't installed on this build, so I pivoted to `theHarvester`:

```
$ dnsrecon -d networkwalks.com
Command 'dnsrecon' not found
```



```
$ theHarvester -d networkwalks.com -l 50 -b all
[*] ASNS found: AS13335, AS31898, AS46606
[*] IPs found: 192.232.216.135, 172.67.198.228
[*] Emails found: info@networkwalks.com
[*] Hosts found: 20 (cpanel, webmail, webdisk, mail, ftp, autodiscover...)
```
Surfaced 20 subdomains, 3 ASNs (AS13335 = Cloudflare), and a contact email.



---

## 🌐 Phase 2: Network Scanning with Zenmap

```
$ nmap -sn 10.0.0.0/24
Nmap scan report for 10.0.0.1
Host is up. MAC Address: 52:55:0A:00:00:01 (Unknown)
Nmap scan report for 10.0.0.2 — Host is up.
Nmap scan report for 10.0.0.3 — Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 7.09 seconds
```

- `10.0.0.1` — likely gateway (MAC address exposed)
- `10.0.0.2` — live host
- `10.0.0.3` — my own device



---

## ⚠️ Risk Summary

| # | Finding | Risk |
|---|---|---|
| 1 | WordPress + plugin version exposed | 🟠 Medium |
| 2 | Server IP identifiable | 🟢 Low |
| 3 | HTTP headers / `/wp-json/` exposed | 🟢 Low |
| 4 | WAF (ModSecurity) identifiable | 🟢 Low |
| 5 | 20 subdomains + ASN info exposed | 🟠 Medium |
| 6 | 3 live hosts + MAC address on local network | 🟠 Medium |

These are **observations, not confirmed vulnerabilities** — no exploitation was attempted in this module.

## ✅ Recommendations

1. Regularly review publicly exposed CMS/plugin technology info
2. Keep WordPress and plugins updated
3. Review HTTP headers for unnecessary disclosure
4. Audit and restrict exposed admin/mail subdomains
5. Keep the WAF tuned and monitored
6. Run periodic internal network discovery scans
7. Investigate unrecognized devices/MAC addresses
8. Maintain up-to-date network documentation
9. Verify tooling availability before an engagement
10. Only test systems/networks with proper authorization

## 🧠 Takeaways

Information gathering is a critical first step before any exploitation attempt — a surprising amount can be learned about a target just from public records, HTTP headers, and OSINT tools. I also learned to adapt when a planned tool (`dnsrecon`) isn't available, substituting `theHarvester` to still meet the enumeration goal. All of this was performed strictly within an authorized, educational lab scope.

---

*Part of my ongoing Cybersecurity & Ethical Hacking internship at Networkwalks. Full formatted report (`.docx`) available on request.*
