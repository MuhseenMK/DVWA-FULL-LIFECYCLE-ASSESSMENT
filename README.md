# 🔐 DVWA Full Lifecycle Assessment

**Web Application Penetration Test — From Exploitation to Remediation**

---

**Author:** Muhammad Muhsin Khamis  
**Program:** BSc Cybersecurity  
**Project Type:** Self-Directed Portfolio Project  
**Target:** DVWA (Damn Vulnerable Web Application) v1.9  
**Environment:** Self-hosted in Kali Linux via Docker  
**Assessment Type:** Full lifecycle web application security assessment  
**Date:** October 2026  

**Overall Assessment Risk:** 🔴 **CRITICAL**

---

## Key Achievements

- ✅ **9 vulnerabilities identified and documented**
- ✅ **CVSS v3.1 scoring with vector justification for each finding**
- ✅ **Full lifecycle coverage — recon → exploitation → reporting → remediation → re-test**
- ✅ **All 9 exploits verified as blocked at DVWA Impossible security level**
- ✅ **Professional 30-page pentest report included (PDF)**
- ✅ **Full evidence trail with screenshots for every confirmed finding**

---

## Table of Contents

1. [Why I Built This](#why-i-built-this)
2. [What This Project Covers](#what-this-project-covers)
3. [Scope & Limitations](#scope--limitations)
4. [Tools & Versions](#tools--versions)
5. [Reconnaissance](#reconnaissance)
6. [Vulnerabilities Identified](#vulnerabilities-identified)
7. [The Attack Chain](#the-attack-chain)
8. [Confirmed Findings & Evidence](#confirmed-findings--evidence)
9. [Remediation Verification](#remediation-verification)
10. [Executive Summary](#executive-summary)
11. [Lessons Learned](#lessons-learned)
12. [Full Report](#full-report)
13. [Ethical Statement](#ethical-statement)
14. [References](#references)
15. [Author](#author)

---

## Why I Built This

I wanted a project that would cover the full penetration testing lifecycle — 
not just finding a vulnerability, but exploiting it, documenting it, and 
then verifying the fix works.

DVWA was my target. This repository is the complete result.

---

## What This Project Covers

DVWA is a deliberately vulnerable PHP/MySQL application designed for 
security training. I set it up in my own isolated lab and assessed it 
end-to-end.

The project has five phases:

1. **Reconnaissance** — Identify services, technologies, and attack surface
2. **Exploitation** — Find and exploit 9 vulnerabilities at security level = Low
3. **Reporting** — Produce a professional pentest report
4. **Remediation** — Verify the built-in fixes at security level = Impossible
5. **Re-Test** — Confirm the exploits are blocked

---

## Scope & Limitations

### Testing Approach

- **Reconnaissance and enumeration** followed a black-box methodology — 
  only externally observable behaviour was used.
- **Exploitation** targeted known DVWA vulnerabilities documented in the 
  application itself.
- **No source code review** was performed to discover findings. Source 
  code was only inspected *after* exploitation to understand why the 
  vulnerability existed.

### What This Project Is

A **beginner-to-intermediate** portfolio project demonstrating the full 
penetration testing workflow against a deliberately vulnerable training 
application.

### What This Project Is Not

- Not a production pentest — DVWA is intentionally insecure
- Not a claim of advanced exploitation skills
- Not an independent remediation — the fixes are DVWA's own built-in 
  "Impossible" level implementations, not code I wrote myself

### Known Limitations

- Only Low security level exploits were demonstrated in this version
- Medium and High security levels were not tested
- Remote File Inclusion (RFI) was not exploited
- The full attack chain was not executed end-to-end — each step was 
  verified independently

---

## Tools & Versions

| Tool | Version | Purpose |
|---|---|---|
| Kali Linux | 2026.2 | Testing environment |
| Docker | 28.5.2 | Lab deployment |
| DVWA | v1.9 | Target application |
| Apache | 2.4.7 (Ubuntu) | Web server (from recon) |
| PHP | 5.5.9 | Backend language (from recon) |
| Nmap | 7.99 | Port scanning |
| WhatWeb | Latest | Technology fingerprinting |
| Nikto | 2.6.0 | Web server scanning |
| Gobuster | 3.8.2 | Directory enumeration |
| Firefox | 140.0 | Manual application testing |
| curl | 8.x | HTTP request analysis |
| John the Ripper | Latest | Password hash cracking |
| rockyou.txt | N/A | Dictionary wordlist (14M entries) |

---

## Lab Setup

DVWA was deployed as a Docker container on Kali Linux:

```bash
sudo docker run -d --name dvwa -p 80:80 vulnerables/web-dvwa
```

screenshots/06-dvwa-login.png

Access: http://localhost — default credentials admin / password

---

Reconnaissance

Port Scan

```bash
nmap -sV -p- localhost
```

Result: Port 80/tcp open — Apache httpd 2.4.7 (Ubuntu). No other ports exposed.

screenshots/01-nmap.png

Technology Fingerprint

```bash
whatweb http://localhost
```

Key findings:

· Server: Apache 2.4.7 (Ubuntu) — outdated
· Backend: PHP 5.5.9 — significantly outdated (2014 release)
· X-Powered-By: PHP/5.5.9-1ubuntu4.25 header exposed
· Session cookies: PHPSESSID, security

Web Server Misconfigurations

```bash
nikto -h http://localhost
```

Selected findings:

· Missing security headers (X-Frame-Options, X-Content-Type-Options, 
  Strict-Transport-Security, Referrer-Policy, Content-Security-Policy)
· Directory listing enabled on /docs/ and /config/
· .git directory exposed
· phpinfo.php accessible

Directory Enumeration

```bash
gobuster dir -u http://localhost -w /usr/share/wordlists/dirb/common.txt
```

Notable paths discovered:

· /vulnerabilities/ — main vulnerability modules
· /config/ — configuration directory with directory listing
· /docs/ — documentation directory
· /hackable/uploads/ — file upload directory
· /php.ini — PHP configuration file exposed

Reconnaissance deliverable: Complete attack surface mapped before 
any exploitation attempt.

---

Vulnerabilities Identified

CVSS scores use CVSS v3.1 Base Score metrics only (no temporal or 
environmental adjustments). Severity ratings are aligned with FIRST.org 
guidance.

# Vulnerability CWE Severity CVSS Vector
1 Brute Force — No rate limiting CWE-307 🟠 High 7.5 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
2 Command Injection CWE-78 🔴 Critical 9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
3 CSRF — Password change CWE-352 🟠 High 7.1 AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N
4 File Inclusion — LFI CWE-98 🟠 High 8.6 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:L
5 File Upload — Unrestricted PHP CWE-434 🔴 Critical 9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
6 SQL Injection CWE-89 🔴 Critical 9.8 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
7 SQL Injection (Blind) CWE-89 🟠 High 7.5 AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
8 XSS (Reflected) CWE-79 🟡 Medium 6.1 AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N
9 XSS (Stored) CWE-79 🟠 High 8.2 AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:H/A:N

Total: 3 Critical, 5 High, 1 Medium

Note on Overall Risk: The "CRITICAL" overall rating reflects the 
combined impact of these findings if chained together in a real 
application — not the sum of individual scores.

---

The Attack Chain

Each vulnerability is serious on its own. But the real impact comes from 
what happens when they chain together:

```
Reconnaissance
    ↓
Information Disclosure (X-Powered-By, outdated software)
    ↓
SQL Injection → Database extraction
    ↓
Password Cracking → Full authentication bypass
    ↓
File Upload → Remote code execution
    ↓
Command Injection → Server compromise
    ↓
File Inclusion → Source code extraction
```

Important: This attack chain is a theoretical scenario based on 
confirmed findings. Each step was independently verified at security 
level = Low, but the full chain was not executed end-to-end as a single 
attack in this version.

---

Confirmed Findings & Evidence

Each confirmed finding below follows this structure:

· Endpoint / parameter
· Reproduction steps
· Payload used
· Observed result
· Impact
· Remediation

Finding 2 — Command Injection

Endpoint: /vulnerabilities/exec/
Vulnerable parameter: ip (POST)
Prerequisites: Authenticated session, security level = Low

Reproduction:

1. Navigate to Command Injection module
2. Submit payload: 127.0.0.1; whoami
3. Observe returned output

Result: OS command executed — returned www-data

screenshots/02-command-injection-whoami.png

Impact: Remote code execution as the Apache web server user. In this 
Docker container, impact was limited to the container. In a production 
deployment with network access, this typically escalates to full server 
compromise.

Remediation: Never pass user input to OS commands. Use language-native 
APIs instead of shell. Whitelist input (regex for IP addresses only).

---

Finding 4 — File Inclusion (Local)

Endpoint: /vulnerabilities/fi/
Vulnerable parameter: page (GET)
Prerequisites: Authenticated session, security level = Low

Reproduction:

1. Navigate to File Inclusion module
2. Change URL to: ?page=/etc/passwd
3. Observe returned file contents

Result: Full /etc/passwd file returned through the browser

screenshots/04-file-inclusion-passwd.png

Impact: Local File Inclusion — arbitrary file read from the server 
file system (limited to files readable by the web server user).

Remediation: Use a whitelist of allowed pages. Never pass user input 
directly to include() / require(). Store sensitive files outside 
the web root.

---

Finding 6 — SQL Injection

Endpoint: /vulnerabilities/sqli/
Vulnerable parameter: id (GET)
Prerequisites: Authenticated session, security level = Low

Reproduction:

1. Navigate to SQL Injection module
2. Submit payload: 1' UNION SELECT user, password FROM users-- -
3. Observe returned user table

Result: All usernames and MD5 password hashes returned

screenshots/06-sql-injection-users.png

Extracted users and cracked passwords:

Username MD5 Hash Cracked Password
admin 5f4dcc3b5aa765d61d8327deb882cf99 password
gordonb e99a18c428cb38d5f260853678922e03 abc123
1337 8d3533d75ae2c3966d7e0d4fcc69216b charley
pablo 0d107d09f5bbe40cade3de5c71e9e9b7 letmein
smithy 5f4dcc3b5aa765d61d8327deb882cf99 password

Cracking method: John the Ripper + rockyou.txt dictionary attack. 
All 5 hashes cracked in under 30 seconds.

screenshots/06-sql-injection-cracked.png

Why MD5 is unsuitable for passwords:

· Extremely fast to compute (billions of hashes/sec on modern GPU)
· No salt — identical passwords produce identical hashes
· Pre-computed lookup tables exist for common passwords
· Modern best practice: Argon2id or bcrypt — both salted, adaptive

Remediation: Use prepared statements with parameterised queries. 
Never concatenate user input into SQL strings. Migrate to Argon2id or 
bcrypt for password hashing.

---

Remediation Verification

After exploitation, I switched DVWA to security level = Impossible — 
DVWA's own built-in secure implementation — and re-tested each exploit.

What this demonstrates: That the exploits are blocked when secure 
coding practices are in place.

What this does NOT demonstrate: That I independently implemented 
each fix. DVWA's Impossible level ships with the fixes already applied. 
This phase validates that the standard mitigations work against these 
attack classes.

# Vulnerability Standard Mitigation Re-Test Result
1 Brute Force Rate limiting ✅ Blocked
2 Command Injection Input whitelist ✅ Blocked
3 CSRF CSRF tokens ✅ Blocked
4 File Inclusion Page whitelist ✅ Blocked
5 File Upload Extension + content validation ✅ Blocked
6 SQL Injection Prepared statements ✅ Blocked
7 SQL Injection (Blind) Prepared statements ✅ Blocked
8 XSS (Reflected) Output encoding ✅ Blocked
9 XSS (Stored) Output encoding ✅ Blocked

All 9 exploits confirmed blocked during re-testing.

Proof — SQL Injection blocked at Impossible level:

screenshots/07-sql-injection-blocked.png

Proof — CSRF blocked with token validation:

screenshots/04-csrf-blocked.png

---

Executive Summary

This assessment of a deliberately vulnerable web application identified 
9 vulnerabilities — 3 Critical, 5 High, and 1 Medium severity.

The most significant findings:

· SQL Injection — full database access including user credentials
· Command Injection — remote code execution as the web server user
· File Upload — upload and execution of malicious PHP scripts

If this were a production application handling real data, these 
vulnerabilities would enable complete compromise of the system and 
any data it stores.

Prioritized recommendations:

1. Immediate (this week): Fix SQL Injection, Command Injection, and 
   File Upload — all three are Critical and directly enable RCE or 
   database compromise.
2. High priority (this month): Fix File Inclusion, CSRF, and 
   Stored XSS.
3. Medium priority (this quarter): Add rate limiting to login 
   (Brute Force), fix Reflected XSS.
4. Foundational: Migrate from MD5 to Argon2id or bcrypt for 
   password hashing.

---

Lessons Learned

1. Vulnerabilities compound. A single SQL injection is critical. But 
when it leads to hashes, which lead to cracked passwords, which lead to 
authenticated access — the impact explodes.

2. Defense in depth is real. No single fix would have protected this 
application. It took prepared statements AND output encoding AND rate 
limiting AND input validation to close every finding.

3. Documentation is a skill. Finding a vulnerability is only half the 
work. Explaining it clearly — in a way a non-technical reader can 
understand — is just as important.

4. Re-testing proves the work. Anyone can claim they fixed something. 
Proving it with the same exploit that worked before separates a real 
assessment from a guess.

5. Understanding the fix matters. It's not enough to know a 
vulnerability exists. Knowing which specific mitigation applies — and 
why — is what makes findings actionable.

6. Reproducibility matters. A finding without a clear endpoint, 
parameter, payload, and observed result is not a finding — it's a 
rumour.

---

Full Report

The complete penetration testing report — executive summary, all 9 
findings with CVSS vectors, reproduction steps, impact analysis, and 
remediation guidance — is available as a PDF:

📄 DVWA-Penetration-Test-Report.pdf

---

Ethical Statement

This assessment was conducted against DVWA — a deliberately vulnerable 
application hosted in an isolated personal lab environment.

· ✅ No real systems were targeted
· ✅ No unauthorised access occurred
· ✅ All testing was against a self-owned lab
· ✅ All findings are documented for educational purposes

All techniques documented here are for education only. Using them 
against systems you do not own or do not have explicit written 
authorisation to test is illegal in most countries.

If you are learning cybersecurity, do what I did — build your own lab.

---

References

· OWASP Top 10
· DVWA Official
· PTES
· OWASP Testing Guide
· CVSS v3.1 Specification
· MITRE CWE
· CrackStation

---

Author

Muhammad Muhsin Khamis
Cybersecurity Student

· 🐙 GitHub: @MuhseenMK
· 💼 LinkedIn: Muhammad Muhsin Khamis
· 📧 Email: khamismuhammadmuhsin@gmail.com

---

Project Metadata

Item Value
Project Type Self-Directed Lab
Category Web Application Penetration Testing
Difficulty Beginner → Intermediate
Duration 5–7 days
Report Reference DVWA-PT-2026-01
Classification Educational / Internal

---

The best way to learn to defend a system is to learn how to break it.

```

