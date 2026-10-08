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

### Scope Clarification

**This was not a true black-box assessment.** DVWA is a well-known, 
deliberately vulnerable application. The "black-box" methodology applies 
to the reconnaissance and enumeration phases — where only externally 
observable behaviour was used. Exploitation relied on known DVWA 
vulnerabilities from its documentation. No source code review was 
performed to discover findings.

### Limitations

- Testing was limited to a deliberately vulnerable training application
- Remediation was validated against DVWA's built-in "Impossible" security 
  level — this demonstrates the fixes exist, but I did not independently 
  implement them
- Only Low security level exploits were demonstrated; Medium and High 
  levels were not tested in this version

---

## Tools I Used

| Tool | Purpose |
|---|---|
| Kali Linux | Testing environment |
| Docker | Lab deployment (DVWA container) |
| Nmap | Port scanning and service detection |
| WhatWeb | Web technology fingerprinting |
| Nikto | Web server misconfiguration scanning |
| Gobuster | Directory and file enumeration |
| Firefox | Manual application testing |
| curl | HTTP request analysis |
| John the Ripper | Password hash cracking |
| rockyou.txt | Dictionary wordlist |
| CrackStation / Hashes.com | Online hash lookup |

---

## Lab Setup

DVWA was deployed as a Docker container on Kali Linux.

![DVWA running](screenshots/06-dvwa-login.png)

---

## Reconnaissance

Initial reconnaissance identified the attack surface. Only port 80 was 
exposed, running Apache 2.4.7 and PHP 5.5.9 — both significantly outdated.

![Nmap scan](screenshots/01-nmap.png)

---

## Vulnerabilities Identified

The table below lists the 9 findings identified. CVSS scores were assigned 
using **CVSS v3.1** with the **Base Score** metrics only (no environmental 
or temporal modifications). Scores reflect the **technical impact** of 
each vulnerability in isolation.

| # | Vulnerability | Severity | CVSS | Vector |
|---|---|---|---|---|
| 1 | Brute Force — No rate limiting | 🟠 High | 7.5 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N |
| 2 | Command Injection — OS command execution | 🔴 Critical | 9.8 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |
| 3 | CSRF — Password change via GET | 🟠 High | 7.1 | AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N |
| 4 | File Inclusion — Local file read | 🟠 High | 8.6 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:L |
| 5 | File Upload — Unrestricted PHP upload | 🔴 Critical | 9.8 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |
| 6 | SQL Injection — Database extraction | 🔴 Critical | 9.8 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |
| 7 | SQL Injection (Blind) — Boolean inference | 🟠 High | 7.5 | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N |
| 8 | XSS (Reflected) — JavaScript injection | 🟡 Medium | 6.1 | AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N |
| 9 | XSS (Stored) — Persistent JavaScript | 🟠 High | 8.2 | AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:H/A:N |

**Total:** 3 Critical, 5 High, 1 Medium

**Note on Overall Risk:** The "CRITICAL" overall rating reflects the 
combined impact of these findings if they were exploited together in a 
real application — not the sum of individual CVSS scores. Each finding 
is rated individually above; the overall assessment considers their 
combined attack-chain potential.

---

## The Attack Chain

Each vulnerability is serious on its own. But the real lesson is what 
happens when they chain together:

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

**Important distinction:** The attack chain above is a **theoretical 
scenario** based on the confirmed findings. Each step was independently 
verified at security level = Low, but the full chain was not executed 
end-to-end as a single attack.

---

## Finding 6 — SQL Injection (Confirmed)

**Affected endpoint:** `/vulnerabilities/sqli/`  
**Vulnerable parameter:** `id` (GET)  
**Payload used:** `1' UNION SELECT user, password FROM users-- -`  
**Observed result:** All usernames and MD5 password hashes returned

![SQL Injection — all users dumped](screenshots/06-sql-injection-users.png)

**Extracted users and cracked passwords:**

| Username | MD5 Hash | Cracked Password |
|---|---|---|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordonb | e99a18c428cb38d5f260853678922e03 | abc123 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99 | password |

**Why MD5 matters:** MD5 is a fast, unsalted hashing algorithm. It is 
unsuitable for password storage because:
- It computes billions of hashes per second on modern hardware
- Pre-computed lookup tables (rainbow tables) exist for common passwords
- Identical passwords produce identical hashes, revealing reuse

Modern best practice is **Argon2id** or **bcrypt** — both are salted and 
intentionally slow, making offline cracking impractical.

![Cracked hashes with John](screenshots/06-sql-injection-cracked.png)

---

## Finding 2 — Command Injection (Confirmed)

**Affected endpoint:** `/vulnerabilities/exec/`  
**Vulnerable parameter:** `ip` (POST)  
**Payload used:** `127.0.0.1; whoami`  
**Observed result:** OS command executed — returned `www-data`

![Command Injection — www-data](screenshots/02-command-injection-whoami.png)

**What this means:** The `www-data` user is the Apache service account. 
This proves arbitrary OS command execution. The conditions required for 
broader compromise:
- The web server user's file system permissions
- Available binaries in the container
- Network access from the container to other systems

In this Docker environment, the impact was limited to the container. In 
a real deployment with network access, this typically escalates to full 
server compromise.

---

## Finding 4 — File Inclusion (Confirmed)

**Affected endpoint:** `/vulnerabilities/fi/`  
**Vulnerable parameter:** `page` (GET)  
**Payload used:** `?page=/etc/passwd`  
**Observed result:** The full `/etc/passwd` file was returned

![File Inclusion — /etc/passwd](screenshots/04-file-inclusion-passwd.png)

**What was specifically tested:** Local File Inclusion (LFI) via path 
traversal. I confirmed reading a system file outside the web root. Remote 
File Inclusion (RFI) was not exploited. The "arbitrary file read" 
description refers specifically to local files readable by the web 
server user.

---

## Verification of Remediation

After exploitation, I switched DVWA to security level = **Impossible** — 
DVWA's own built-in secure implementation — and re-tested each exploit.

**What this demonstrates:** That the exploits are blocked when secure 
coding practices are in place.

**What this does NOT demonstrate:** That I independently implemented 
each fix. DVWA's Impossible level ships with the fixes already applied. 
This phase validates the effectiveness of the standard mitigations.

| # | Vulnerability | Standard Fix | Re-Test Result |
|---|---|---|---|
| 1 | Brute Force | Rate limiting | ✅ Blocked |
| 2 | Command Injection | Input whitelist | ✅ Blocked |
| 3 | CSRF | CSRF tokens | ✅ Blocked |
| 4 | File Inclusion | Page whitelist | ✅ Blocked |
| 5 | File Upload | Extension validation | ✅ Blocked |
| 6 | SQL Injection | Prepared statements | ✅ Blocked |
| 7 | SQL Injection (Blind) | Prepared statements | ✅ Blocked |
| 8 | XSS (Reflected) | Output encoding | ✅ Blocked |
| 9 | XSS (Stored) | Output encoding | ✅ Blocked |

All 9 exploits were confirmed blocked during re-testing.

**Proof — SQL Injection blocked at Impossible level:**

![SQL Injection blocked](screenshots/07-sql-injection-blocked.png)

**Proof — CSRF blocked with token validation:**

![CSRF blocked](screenshots/04-csrf-blocked.png)

---

## Executive Summary

This assessment of a deliberately vulnerable web application identified 
**9 vulnerabilities** — 3 Critical, 5 High, and 1 Medium severity.

The most significant findings are:
- **SQL Injection** — allowing full database access including user credentials
- **Command Injection** — allowing remote code execution as the web server user
- **File Upload** — allowing upload and execution of malicious PHP scripts

If this were a production application handling real data, these 
vulnerabilities would enable complete compromise of the system and any 
data it stores.

**Prioritized recommendations:**

1. **Immediate (this week):** Fix SQL Injection, Command Injection, and 
   File Upload — all three are Critical and directly enable RCE or 
   database compromise.
2. **High priority (this month):** Fix File Inclusion, CSRF, and Stored XSS.
3. **Medium priority (this quarter):** Fix Brute Force (add rate limiting) 
   and Reflected XSS.
4. **Foundational:** Replace MD5 password hashing with Argon2id or bcrypt.

---

## Full Report

The complete penetration testing report — executive summary, all 9 
findings with reproduction steps, CVSS justification, impact analysis, 
and remediation guidance — is available as a PDF:

📄 **[DVWA-Penetration-Test-Report.pdf](DVWA-Penetration-Test-Report.pdf)**

---

## What I Learned

**1. Vulnerabilities compound.** A single SQL injection is critical. But 
when it leads to hashes, which lead to cracked passwords, which lead to 
authenticated access — the impact explodes.

**2. Defense in depth is real.** No single fix would have protected this 
application. It took prepared statements AND output encoding AND rate 
limiting AND input validation to close every finding.

**3. Documentation is a skill.** Finding a vulnerability is only half the 
work. Explaining it clearly — in a way a non-technical reader can 
understand — is just as important.

**4. Re-testing proves the work.** Anyone can claim they fixed something. 
Proving it with the same exploit that worked before is what separates a 
real assessment from a guess.

**5. Understanding the fix matters.** It's not enough to know a 
vulnerability exists. Knowing which specific mitigation applies — and 
why — is what makes findings actionable.

---

## Ethical Statement

This assessment was conducted against **DVWA** — a deliberately vulnerable 
application hosted in an isolated personal lab environment.

- ✅ No real systems were targeted
- ✅ No unauthorised access occurred
- ✅ All testing was against a self-owned lab
- ✅ All findings are documented for educational purposes

**All techniques documented here are for education only.** Using them 
against systems you do not own or do not have explicit written 
authorisation to test is **illegal in most countries**.

If you are learning cybersecurity, do what I did — build your own lab.

---

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [DVWA Official](https://github.com/digininja/DVWA)
- [PTES](http://www.pentest-standard.org/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [CVSS v3.1 Specification](https://www.first.org/cvss/v3.1/specification-document)
- [CrackStation](https://crackstation.net/)

---

## Author

**Muhammad Muhsin Khamis**  
Cybersecurity Student

- 🐙 GitHub: [@MuhseenMK](https://github.com/MuhseenMK)
- 💼 LinkedIn: [Muhammad Muhsin Khamis](https://www.linkedin.com/in/muhammad-muhsin-khamis-9860b3311/)
- 📧 Email: khamismuhammadmuhsin@gmail.com

---

## Project Metadata

| Item | Value |
|---|---|
| Project Type | Self-Directed Lab |
| Category | Web Application Penetration Testing |
| Difficulty | Beginner → Intermediate |
| Duration | 5–7 days |
| Report Reference | DVWA-PT-2026-01 |
| Classification | Educational / Internal |

---

*The best way to learn to defend a system is to learn how to break it.*
```

---
