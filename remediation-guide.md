# Prioritized Remediation Guide — testphp.vulnweb.com

**Assessment Date:** April 2025  
**Total Fixes Required:** 10  
**Estimated Total Remediation Time:** 10–14 hours

This guide organises all findings into three action tiers. Work through them in order — Tier 1 delivers the highest security improvement per hour of effort.

---

## Tier 1 — Fix This Week (Critical & High)

All four items below are configuration changes. None requires application code changes. Each can be completed in under four hours by a developer or sysadmin.

---

### Fix 1 · Close Telnet Port 23 `[F-01 · CVSS 9.1]`

**Effort:** 1 hour  
**Why now:** An open Telnet port is an unencrypted back door accessible from the public internet. This is the single most dangerous finding in this report.

```bash
# Disable and stop the Telnet service
sudo systemctl disable telnet --now

# Block the port at the firewall
sudo ufw deny 23/tcp
sudo ufw reload

# Also evaluate and disable anonymous FTP (Port 21):
sudo nano /etc/vsftpd.conf
# Set: anonymous_enable=NO
sudo systemctl restart vsftpd

# Verify both ports are closed:
nmap -p 21,23 testphp.vulnweb.com
# Both should show: STATE = closed or filtered
```

---

### Fix 2 · Add Content-Security-Policy Header `[F-02 · CVSS 7.4]`

**Effort:** 2 hours (including testing in report-only mode)  
**Why now:** This is the most effective single defence against XSS attacks — the most common class of web vulnerability.

```apache
# Step 1: Enable report-only mode first (no impact on users)
# Add to Apache config or .htaccess:
Header set Content-Security-Policy-Report-Only "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; object-src 'none'"

# Step 2: Monitor for violations for 24 hours, then switch to enforcement:
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; object-src 'none'; frame-ancestors 'none'"

# Step 3: Verify using securityheaders.com — target grade A or B
```

---

### Fix 3 · Disable TLS 1.0 and TLS 1.1 `[F-03 · CVSS 7.5]`

**Effort:** 1 hour  
**Why now:** Deprecated protocol versions expose visitors to network-level interception attacks. This is a one-line configuration change.

```apache
# In /etc/apache2/sites-available/default-ssl.conf
# Replace the existing SSLProtocol line with:
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1

# Also remove weak ciphers:
SSLCipherSuite HIGH:!aNULL:!MD5:!RC4:!3DES
SSLHonorCipherOrder on

# Enable OCSP stapling while you are here:
SSLUseStapling on
SSLStaplingCache shmcb:/var/run/ocsp(128000)

# Restart Apache:
sudo systemctl restart apache2

# Verify: re-run ssllabs.com — expected grade: A
```

---

### Fix 4 · Suppress Version Headers and Upgrade PHP `[F-04 · CVSS 6.5]`

**Effort:** 30 minutes (headers) + 3–4 hours (PHP upgrade — requires testing)

**Headers (30 minutes):**
```apache
# In httpd.conf or apache2.conf:
ServerTokens Prod
ServerSignature Off

# Remove PHP version header:
Header unset X-Powered-By

# Restart Apache:
sudo systemctl restart apache2
```

**PHP Upgrade (critical — do not defer):**  
PHP 5.6.40 is six years past end-of-life and has over 20 known unpatched CVEs. This is not a configuration change — it requires application testing.

```bash
# Install PHP 8.2 (Ubuntu 20.04+):
sudo apt update
sudo apt install php8.2 php8.2-mysql php8.2-curl php8.2-mbstring

# Test the application on a staging environment first.
# Common breaking changes from PHP 5.6 to 8.2:
#   - mysql_* functions removed (use mysqli_* or PDO)
#   - Deprecated features become errors
#   - Case sensitivity changes in class names
```

---

## Tier 2 — Fix This Month (Medium)

All four items below are single-line configuration changes. They should be batched and completed together in a single maintenance window.

```apache
# Add all four to your Apache config or .htaccess at once:

# F-05: Force HTTPS in future — HSTS
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

# F-06: Prevent clickjacking
Header always set X-Frame-Options "DENY"

# Bonus: Prevent MIME-type sniffing
Header always set X-Content-Type-Options "nosniff"
```

```ini
# F-07: Fix session cookie flags in php.ini:
session.cookie_httponly = 1
session.cookie_secure   = 1
session.cookie_samesite = Strict
```

```apache
# F-08: Disable directory listing
# In .htaccess or httpd.conf:
Options -Indexes
```

---

## Tier 3 — Next Maintenance Window (Low)

```apache
# F-09: Referrer-Policy (15 minutes)
Header set Referrer-Policy "strict-origin-when-cross-origin"
```

```ini
# F-10: Disable PHP error display (15 minutes)
display_errors = Off
log_errors     = On
error_log      = /var/log/php_errors.log
```

---

## Post-Remediation Verification

After completing all tiers, re-run the following checks to confirm the security posture has improved:

| Check | Tool | Target Grade |
|-------|------|--------------|
| Security headers | securityheaders.com | A or B |
| SSL/TLS | ssllabs.com/ssltest | A |
| Open ports | nmap -sV testphp.vulnweb.com | Telnet/FTP closed |
| Cookie flags | Browser DevTools | HttpOnly + Secure visible |

Document the before/after screenshots as evidence for your project portfolio.
