# Detailed Findings — testphp.vulnweb.com Vulnerability Assessment

**Assessment Date:** April 2025  
**Methodology:** OWASP Web Security Testing Guide v4.2 (Passive Subset)  
**Total Findings:** 10 (1 Critical · 3 High · 4 Medium · 2 Low)

---

## F-01 · Open Telnet Port (Port 23)

| Field | Value |
|-------|-------|
| **Risk Level** | 🔴 CRITICAL |
| **CVSS Score** | 9.1 |
| **Category** | Attack Surface — Open Ports |
| **Evidence File** | `evidence/nmap/nmap-scan-output.txt` |

**What is it?**  
Telnet is a legacy remote access protocol that sends all data — including usernames, passwords, and commands — in plain text with zero encryption. Port 23 was found open and actively accepting connections from the public internet.

**Why does it matter?**  
This is the equivalent of leaving an unlocked back door to the server. Any attacker on the internet can attempt to connect, and any credentials typed are transmitted in cleartext — readable by anyone with network access between the user and the server.

**Technical Evidence**
```
23/tcp   open   telnet   Linux telnetd
```

**Remediation**
```bash
sudo systemctl disable telnet --now
sudo ufw deny 23/tcp
# Verify closure:
nmap -p 23 testphp.vulnweb.com
# Expected: 23/tcp closed telnet
```

---

## F-02 · Missing Content-Security-Policy Header

| Field | Value |
|-------|-------|
| **Risk Level** | 🟠 HIGH |
| **CVSS Score** | 7.4 |
| **Category** | HTTP Security Headers |
| **Evidence File** | `evidence/headers/curl-headers-http.txt` |

**What is it?**  
A Content Security Policy is a browser-enforced security policy that restricts which scripts, stylesheets, and media the page is permitted to load. The server returns no CSP header.

**Why does it matter?**  
Without CSP, any malicious script injected into the page (via XSS, a compromised CDN, or a third-party plugin) runs without restriction. This is the primary defence against Cross-Site Scripting attacks, which can steal session tokens, redirect users, or log keystrokes.

**Technical Evidence**
```
curl -I http://testphp.vulnweb.com
→ Content-Security-Policy: [ABSENT]
```

**Remediation**
```apache
# In Apache .htaccess or httpd.conf:
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none'"
```

---

## F-03 · Outdated TLS Protocol (v1.0 and v1.1 Accepted)

| Field | Value |
|-------|-------|
| **Risk Level** | 🟠 HIGH |
| **CVSS Score** | 7.5 |
| **Category** | Transport Security |
| **Evidence File** | `evidence/ssl/ssllabs-report.txt` |

**What is it?**  
TLS 1.0 and TLS 1.1 are outdated encryption protocol versions deprecated by the IETF in RFC 8996 (March 2021). The server continues to accept connections using both deprecated versions.

**Why does it matter?**  
Attackers on the same network as a visitor (e.g., shared Wi-Fi) can use known weaknesses in these protocol versions — including the BEAST and POODLE vulnerabilities — to downgrade the connection and intercept traffic that appears to be encrypted.

**Technical Evidence**
```
SSL Labs Overall Grade : C
TLS 1.0               : YES (DEPRECATED)
TLS 1.1               : YES (DEPRECATED)
RC4-SHA cipher        : Offered (BROKEN)
```

**Remediation**
```apache
# In Apache SSL configuration (/etc/apache2/sites-available/default-ssl.conf):
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite HIGH:!aNULL:!MD5:!RC4:!3DES
SSLHonorCipherOrder on
```

---

## F-04 · Server Version Disclosed in Response Headers

| Field | Value |
|-------|-------|
| **Risk Level** | 🟠 HIGH |
| **CVSS Score** | 6.5 |
| **Category** | Information Disclosure |
| **Evidence File** | `evidence/headers/curl-headers-http.txt` |

**What is it?**  
The HTTP response headers disclose the exact Apache version (2.4.7), operating system (Ubuntu), and PHP version (5.6.40). PHP 5.6.40 reached end-of-life in December 2018.

**Why does it matter?**  
Version disclosure provides an attacker with a precise target for CVE lookups. PHP 5.6 has over 20 known, unpatched public vulnerabilities. Combined with the exact version string, an attacker can immediately identify and select a working exploit.

**Technical Evidence**
```
Server: Apache/2.4.7 (Ubuntu)
X-Powered-By: PHP/5.6.40
```

**Remediation**
```apache
# Suppress version in httpd.conf:
ServerTokens Prod
ServerSignature Off
Header unset X-Powered-By
# CRITICAL: Upgrade PHP from 5.6 to 8.2 minimum.
```

---

## F-05 · Missing HTTP Strict-Transport-Security (HSTS) Header

| Field | Value |
|-------|-------|
| **Risk Level** | 🟡 MEDIUM |
| **CVSS Score** | 6.1 |
| **Category** | HTTP Security Headers |
| **Evidence File** | `evidence/headers/curl-headers-https.txt` |

**What is it?**  
HSTS instructs browsers to always use HTTPS for the domain and to reject plain HTTP connections. The header is absent on both HTTP and HTTPS responses.

**Remediation**
```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
```

---

## F-06 · Missing X-Frame-Options Header

| Field | Value |
|-------|-------|
| **Risk Level** | 🟡 MEDIUM |
| **CVSS Score** | 5.4 |
| **Category** | HTTP Security Headers |
| **Evidence File** | `evidence/zap/zap-passive-alerts.txt` |

**What is it?**  
Without X-Frame-Options, the site can be embedded inside an `<iframe>` on any external domain, enabling clickjacking attacks where users are tricked into clicking elements on the real site while believing they are interacting with a different page.

**Remediation**
```apache
Header always set X-Frame-Options "DENY"
```

---

## F-07 · Session Cookies Missing HttpOnly and Secure Flags

| Field | Value |
|-------|-------|
| **Risk Level** | 🟡 MEDIUM |
| **CVSS Score** | 5.9 |
| **Category** | Session Management |
| **Evidence File** | `evidence/cookies/cookie-analysis.txt` |

**What is it?**  
The PHPSESSID session cookie is set without the HttpOnly flag (allowing JavaScript to read it) and without the Secure flag (allowing transmission over HTTP).

**Remediation**
```ini
; In php.ini:
session.cookie_httponly = 1
session.cookie_secure   = 1
session.cookie_samesite = Strict
```

---

## F-08 · Directory Listing Enabled on /images/

| Field | Value |
|-------|-------|
| **Risk Level** | 🟡 MEDIUM |
| **CVSS Score** | 5.3 |
| **Category** | Information Disclosure |

**What is it?**  
Navigating to `/images/` returns an Apache-generated directory index listing all files in the folder rather than returning a 403 Forbidden response.

**Remediation**
```apache
# In .htaccess or httpd.conf:
Options -Indexes
```

---

## F-09 · Missing Referrer-Policy Header

| Field | Value |
|-------|-------|
| **Risk Level** | 🟢 LOW |
| **CVSS Score** | 3.1 |
| **Category** | Privacy / HTTP Headers |

**What is it?**  
Without a Referrer-Policy, the browser sends the full URL of the referring page (including query parameters) to third-party sites linked from this page. This can expose user paths and search queries to analytics providers and CDNs.

**Remediation**
```apache
Header set Referrer-Policy "strict-origin-when-cross-origin"
```

---

## F-10 · Verbose PHP Error Messages

| Field | Value |
|-------|-------|
| **Risk Level** | 🟢 LOW |
| **CVSS Score** | 3.5 |
| **Category** | Information Disclosure |
| **Evidence File** | `evidence/zap/zap-passive-alerts.txt` |

**What is it?**  
PHP error reporting is enabled in production, displaying full error messages including server-side file paths in HTTP responses when errors occur.

**Technical Evidence**
```
Warning: mysql_fetch_array() expects parameter 1 to be resource
in /var/www/html/artists.php on line 62
```

**Remediation**
```ini
; In php.ini:
display_errors = Off
log_errors     = On
error_log      = /var/log/php_errors.log
```
