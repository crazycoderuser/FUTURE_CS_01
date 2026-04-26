# Assessment Methodology — testphp.vulnweb.com

**Standard Applied:** OWASP Web Security Testing Guide v4.2 (Passive Subset)  
**Assessment Type:** Passive / Read-Only Black-Box Audit  
**Date:** April 2025

---

## Overview

This assessment follows a structured five-phase methodology adapted from the OWASP Web Security Testing Guide. All five phases were completed using passive, non-destructive techniques only. No exploitation, authentication testing, or active probing beyond standard HTTP requests was performed.

---

## Phase 1 — Planning and Scoping

Before any technical activity, the scope was formally defined:

- **Target:** testphp.vulnweb.com (Acunetix permitted testing environment)
- **Assessment boundaries:** Public-facing HTTP/HTTPS pages only; no authenticated content
- **Exclusions:** Login pages, administrative panels, active exploitation of any kind
- **Authorization basis:** Acunetix explicitly permits passive security scanning of testphp.vulnweb.com for educational purposes
- **Legal context:** All activity complies with the IT Act 2000 (India) — testing was confined to a permitted target within a defined, non-destructive scope

---

## Phase 2 — Information Gathering (Passive Reconnaissance)

The goal of this phase was to collect publicly observable information about the target without triggering any server-side alerts or causing any load.

**Activities performed:**
- DNS resolution to obtain the server IP address
- HTTP HEAD requests to retrieve response headers without downloading page content
- WhatWeb passive fingerprinting to identify server technology and CMS
- robots.txt review to identify disclosed paths
- SSL certificate inspection via SSL Labs

**Key findings from this phase:**
- Server: Apache/2.4.7 running on Ubuntu Linux
- Backend language: PHP/5.6.40 (end-of-life)
- IP: 44.228.249.3 (Amazon EC2, us-west-2)
- robots.txt disclosed: /admin/

---

## Phase 3 — Vulnerability Analysis

This phase systematically checks each category of common security weakness.

| Check Category | Tool Used | Method |
|---------------|-----------|--------|
| HTTP security headers | curl, securityheaders.com | HEAD request analysis |
| SSL/TLS configuration | Qualys SSL Labs | Passive certificate and protocol test |
| Open port exposure | Nmap | SYN scan, version detection |
| Cookie security | Browser DevTools | Response header inspection |
| Passive web scan | OWASP ZAP | Spider + passive analysis, no active attacks |
| Technology fingerprinting | WhatWeb | Passive HTTP analysis |

---

## Phase 4 — Risk Classification

Each finding was classified using the industry-standard CVSS v3.1 scoring system. Risk levels were assigned as follows:

| CVSS Score | Risk Level |
|------------|------------|
| 9.0 – 10.0 | Critical |
| 7.0 – 8.9  | High |
| 4.0 – 6.9  | Medium |
| 0.1 – 3.9  | Low |

Business impact was assessed separately from technical severity to ensure the remediation guidance would be meaningful to non-technical stakeholders.

---

## Phase 5 — Reporting

All findings were documented with five mandatory fields: the technical name, a plain-language explanation, technical evidence, the assessed business impact, and specific remediation steps with configuration examples. Findings were then prioritized into three action tiers based on risk level and remediation effort.

---

## What This Assessment Does NOT Cover

The following were explicitly excluded from scope for this assessment:

- **Active exploitation:** No vulnerability was exploited or demonstrated through a proof-of-concept attack
- **Authentication testing:** No login credentials were tested, guessed, or bypassed
- **Application logic testing:** Business logic flaws require authenticated access and active testing
- **Source code review:** No access to server-side code was obtained or attempted
- **Social engineering:** No human targets were contacted

A full penetration test — which would involve active exploitation, authenticated testing, and application logic review — would require broader authorization and is beyond the scope of this educational exercise.
