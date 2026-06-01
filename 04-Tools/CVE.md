# CVE, GitHub, and Linux Man Pages

# CVE (Common Vulnerabilities and Exposures)

## Purpose

CVE provides a standardized identifier for publicly known vulnerabilities.

Format:

CVE-YYYY-NNNNN

Example:

CVE-2021-44228

(Log4Shell)

---

## CVSS Severity

| Score    | Severity |
| -------- | -------- |
| 0.0      | None     |
| 0.1-3.9  | Low      |
| 4.0-6.9  | Medium   |
| 7.0-8.9  | High     |
| 9.0-10.0 | Critical |

---

## Vulnerability Research Workflow

Identify Software
→ Identify Version
→ Search CVE
→ Review Details
→ Search Exploit
→ Validate Vulnerability

---

# GitHub for Pentesters

## Why GitHub Matters

Researchers often publish:

- PoCs
- Exploits
- Scanners
- Detection Scripts
- Technical Writeups

before they appear in commercial tools.

---

## Useful Searches

### Search by CVE

CVE-2024-4577

### Search Exploit

CVE-2024-4577 exploit

### Search Scanner

CVE-2024-4577 scanner

---

## Before Running Any PoC

Check:

- README
- Commit history
- Issues
- Stars
- Source code

Never blindly execute unknown code.

---

# Linux Man Pages

## Purpose

Built-in Linux documentation.

---

## Basic Usage

man command

Examples:

man nmap

man nc

man ssh

man grep

---

## Search Inside a Page

/searchterm

Example:

/proxy

---

## Next Match

n

---

## Previous Match

N

---

## Quit

q

---

## Why It Matters

During labs, exams, and real engagements, internet access may be limited.

Being able to quickly use:

man

--help

-h

can save significant troubleshooting time.

---

# Quick Cheat Sheet

Reconnaissance
├─ Google Dorking
├─ WHOIS
├─ DNS Enumeration
├─ Certificate Transparency Logs
├─ Shodan
├─ VirusTotal
└─ GitHub

↓

Technology Identification

↓

CVE Research

↓

Exploit Research

↓

Validation

↓

Exploitation

↓

Reporting
