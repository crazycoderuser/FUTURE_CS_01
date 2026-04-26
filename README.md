# Website-Vulnerability-and-Security-Audit-
# 🛡️ Vulnerability Assessment — testphp.vulnweb.com

<div align="center">

![Security Audit](https://img.shields.io/badge/Assessment%20Type-Passive%20%2F%20Read--Only-blue?style=for-the-badge&logo=shield)
![Target](https://img.shields.io/badge/Target-testphp.vulnweb.com-navy?style=for-the-badge)
![Risk Level](https://img.shields.io/badge/Overall%20Risk-CRITICAL%20%E2%80%94%20Grade%20F-red?style=for-the-badge)
![Findings](https://img.shields.io/badge/Total%20Findings-10-orange?style=for-the-badge)
![License](https://img.shields.io/badge/Use-Educational%20%2F%20Portfolio-green?style=for-the-badge)

**A professional, passive vulnerability assessment conducted as part of a B.Tech Computer Science & Engineering cybersecurity project.**

</div>

---

## 📋 Table of Contents

- [About This Project](#about-this-project)
- [Website Tested](#website-tested)
- [Scope & Ethics](#scope--ethics)
- [Tools Used](#tools-used)
- [Findings Summary](#findings-summary)
- [Repository Structure](#repository-structure)
- [How to Reproduce](#how-to-reproduce)
- [Remediation Summary](#remediation-summary)
- [Disclaimer](#disclaimer)

---

## About This Project

This repository documents a **read-only, passive vulnerability assessment** of a publicly available, intentionally vulnerable web application. The assessment was completed as **Cybersecurity Task 1** of a B.Tech CSE final-year security project, following the OWASP Web Security Testing Guide v4.2 methodology.

The objective was to simulate the work of a professional security consultant — identifying real weaknesses, classifying them by severity, explaining them in business-friendly language, and producing actionable remediation guidance — without performing any active exploitation.

**10 vulnerabilities were identified**, spanning critical infrastructure exposure (open Telnet), missing HTTP security headers, outdated SSL/TLS configuration, insecure cookie handling, and server information disclosure.

---

## Website Tested

| Field | Details |
|-------|---------|
| **Target URL** | `http://testphp.vulnweb.com` |
| **IP Address** | `44.228.249.3` (Amazon EC2, us-west-2) |
| **Server Technology** | Apache/2.4.7 (Ubuntu) |
| **Backend Language** | PHP 5.6.40 |
| **Site Purpose** | Intentionally vulnerable demo application by Acunetix |
| **Assessment Date** | April 2025 |
| **Tester** | B.Tech CSE Student — Cybersecurity Track |

### Why This Target?

`testphp.vulnweb.com` is maintained by **Acunetix** specifically for security testing practice. It is:

- **Publicly accessible** with no login required for the assessment scope
- **Intentionally vulnerable** — designed to demonstrate real-world security weaknesses
- **Legally safe to test** — Acunetix explicitly permits passive security scanning for educational purposes
- **Realistic** — runs real server software (Apache, PHP, MySQL) with real misconfigurations

---

## Scope & Ethics

### ✅ In Scope (Permitted)

- Public-facing HTTP/HTTPS pages accessible without authentication
- HTTP response header inspection (passive, read-only)
- SSL/TLS certificate and protocol analysis
- Open port enumeration on common ports (top 20)
- Technology fingerprinting via passive observation
- Cookie attribute inspection via browser DevTools
- Automated passive scanning (OWASP ZAP — passive mode only)

### ❌ Out of Scope (Strictly Prohibited)

- Login bypass or authentication circumvention
- Active exploitation of any vulnerability
- Brute-force or credential stuffing attacks
- Denial-of-Service (DoS) or any traffic flooding
- SQL injection or command injection testing
- Accessing any data beyond what is publicly visible
- Social engineering

### Ethical Statement

> *This assessment was conducted strictly within a passive, read-only scope. No credentials were tested, no exploits were executed, no data was accessed beyond what is publicly visible, and no harm was caused to the target system. All activity complied with the Acunetix permitted use policy for testphp.vulnweb.com.*

Under Indian law, unauthorized access to computer systems is punishable under **Section 66 of the IT Act, 2000**. All testing was conducted on an explicitly authorized target within a defined, non-destructive scope.

---

## Tools Used

| Tool | Version | Purpose | Documentation |
|------|---------|---------|---------------|
| **Nmap** | 7.94 | Port scanning and service/version detection | [nmap.org](https://nmap.org/docs.html) |
| **curl** | 8.x | HTTP header extraction (passive, HEAD requests only) | [curl.se/docs](https://curl.se/docs/) |
| **OWASP ZAP** | 2.14 | Passive spider scan — no active attack modules | [zaproxy.org](https://www.zaproxy.org/docs/) |
| **securityheaders.com** | Online | HTTP security header grading (A–F scale) | [securityheaders.com](https://securityheaders.com) |
| **SSL Labs** | Online | Deep SSL/TLS certificate and cipher suite analysis | [ssllabs.com/ssltest](https://www.ssllabs.com/ssltest/) |
| **WhatWeb / whatweb.net** | Online | Passive technology and CMS fingerprinting | [whatweb.net](https://whatweb.net) |
| **Browser DevTools** | Chrome 124 | Cookie flag inspection, response header review | Built-in |

### Nmap Command Used

```bash
# Service and version detection on common ports (passive-safe)
nmap -sV -p 21,22,23,25,80,443,3306,8080,8443 testphp.vulnweb.com

# OS detection (read-only)
nmap -O testphp.vulnweb.com

# Default script scan (non-intrusive)
nmap -sC testphp.vulnweb.com
```

### HTTP Header Extraction

```bash
# Fetch response headers only — no page content downloaded
curl -I -L https://testphp.vulnweb.com

# Check specific security headers
curl -I http://testphp.vulnweb.com | grep -i "content-security\|strict-transport\|x-frame\|x-content-type"
```

---

## Findings Summary

| ID | Vulnerability | Risk Level | CVSS Score |
|----|--------------|------------|------------|
| F-01 | Open Telnet Port (Port 23) | 🔴 **CRITICAL** | 9.1 |
| F-02 | Missing Content-Security-Policy Header | 🟠 **HIGH** | 7.4 |
| F-03 | Outdated TLS — v1.0 and v1.1 Accepted | 🟠 **HIGH** | 7.5 |
| F-04 | Apache & PHP Version Disclosed in Headers | 🟠 **HIGH** | 6.5 |
| F-05 | Missing HTTP Strict-Transport-Security (HSTS) | 🟡 **MEDIUM** | 6.1 |
| F-06 | Missing X-Frame-Options Header | 🟡 **MEDIUM** | 5.4 |
| F-07 | Session Cookies Missing HttpOnly and Secure Flags | 🟡 **MEDIUM** | 5.9 |
| F-08 | Directory Listing Enabled on /images/ | 🟡 **MEDIUM** | 5.3 |
| F-09 | Missing Referrer-Policy Header | 🟢 **LOW** | 3.1 |
| F-10 | Verbose PHP Error Messages Exposed | 🟢 **LOW** | 3.5 |

**Overall Security Grade: F (Critical)**

Detailed findings with plain-English explanations, evidence, and remediation steps are in [`docs/findings-detailed.md`](docs/findings-detailed.md).

---

## Repository Structure

```
vuln-assessment/
│
├── README.md                          ← You are here
├── LICENSE
│
├── evidence/
│   ├── nmap/
│   │   ├── nmap-scan-output.txt       ← Raw Nmap scan results
│   │   └── nmap-script-output.txt     ← Nmap default scripts output
│   ├── headers/
│   │   ├── curl-headers-http.txt      ← HTTP header dump (port 80)
│   │   ├── curl-headers-https.txt     ← HTTPS header dump (port 443)
│   │   └── securityheaders-report.txt ← securityheaders.com output
│   ├── ssl/
│   │   └── ssllabs-report.txt         ← SSL Labs scan summary
│   ├── cookies/
│   │   └── cookie-analysis.txt        ← Cookie flag inspection output
│   └── zap/
│       └── zap-passive-alerts.txt     ← OWASP ZAP passive scan alerts
│
├── docs/
│   ├── findings-detailed.md           ← All 10 findings with full detail
│   ├── remediation-guide.md           ← Prioritized fix guide
│   ├── methodology.md                 ← Assessment methodology & process
│   └── scope.md                       ← Full scope definition
│
└── .github/
    └── ISSUE_TEMPLATE/
        └── finding-report.md          ← Template for new finding reports
```

---

## How to Reproduce

All commands below are passive and safe to run against testphp.vulnweb.com as permitted by Acunetix.

### 1. Port Scan

```bash
nmap -sV -p 21,22,23,25,53,80,443,3306,5432,8080 testphp.vulnweb.com -oN evidence/nmap/nmap-scan-output.txt
```

### 2. HTTP Header Analysis

```bash
curl -I http://testphp.vulnweb.com  > evidence/headers/curl-headers-http.txt
curl -I https://testphp.vulnweb.com > evidence/headers/curl-headers-https.txt
```

### 3. SSL/TLS Test

Visit: `https://www.ssllabs.com/ssltest/analyze.html?d=testphp.vulnweb.com`

### 4. Security Header Grade

Visit: `https://securityheaders.com/?q=testphp.vulnweb.com`

### 5. OWASP ZAP Passive Scan

1. Open OWASP ZAP → Automated Scan
2. Enter: `http://testphp.vulnweb.com`
3. Run **Spider only** (do NOT run Active Scan)
4. Export alerts from: Report → Generate HTML Report

---

## Remediation Summary

| Priority | Finding | Effort | Impact |
|----------|---------|--------|--------|
| 🔴 Immediate | Close Telnet Port 23 | 1 hour | Critical |
| 🔴 Immediate | Add Content-Security-Policy header | 2 hours | High |
| 🔴 Immediate | Disable TLS 1.0 / 1.1 | 1 hour | High |
| 🔴 Immediate | Suppress server version headers + upgrade PHP | 4 hours | High |
| 🟠 This Month | Enable HSTS, X-Frame-Options, fix cookie flags | 1–2 hours | Medium |
| 🟡 Scheduled | Disable directory listing, add Referrer-Policy, fix error display | 1 hour | Low |

Full remediation instructions with exact configuration commands: [`docs/remediation-guide.md`](docs/remediation-guide.md)

---

## Disclaimer

> This repository was created for **educational and portfolio purposes** as part of a B.Tech Computer Science & Engineering cybersecurity project.
>
> `testphp.vulnweb.com` is an **intentionally vulnerable** demonstration application maintained by Acunetix. All testing was **passive and non-destructive**. No exploitation, data access, or harmful activity was performed.
>
> **Do not** replicate any technique in this repository against systems you do not own or have explicit written permission to test. Unauthorized computer access is illegal in India under the IT Act 2000 and in most jurisdictions worldwide.

---

<div align="center">

**B.Tech CSE — Cybersecurity Track**  
Shambhunath Institute of Engineering & Technology (AKTU) · April 2025  
*Part of the Cyberecurity Internship Project *

</div>
