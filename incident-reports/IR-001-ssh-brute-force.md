<div align="center">

# 🎫 Incident Report: IR-001

![Status](https://img.shields.io/badge/status-Closed-6B7280?style=flat-square)
![Severity](https://img.shields.io/badge/severity-Medium-F59E0B?style=flat-square)
![MITRE](https://img.shields.io/badge/MITRE%20ATT%26CK-T1110%20Brute%20Force-2563EB?style=flat-square)

</div>

**Date Detected:** September 9, 2026
**Analyst:** Dhim Escuyos
**Affected System:** Ubuntu-Victim (192.168.x.x)

---

## 📝 Summary
Wazuh detected a high volume of failed SSH authentication attempts
against the `socanalyst` account on Ubuntu-Victim, consistent with an
automated brute-force attack. Source IP 192.168.1.177 (Kali attacker
host) generated 249 failed login events within a 30-second window.

## 🕒 Timeline
| Time | Event |
|---|---|
| 20:01:16 | First failed SSH login attempt logged |
| 20:01:16 – 20:01:46 | 249 failed authentication attempts recorded |
| 20:02:05 | Wazuh escalates to Level 10 "Multiple authentication failures" (Rule 40111) |

## 🔍 Detection Details
- Rule 5760 (sshd: authentication failed) — repeated, sub-second
  intervals
- Rule 5758 (Maximum authentication attempts exceeded)
- Rule 40111 (Multiple authentication failures) — Level 10, correlation
  match

---

## 🕵️ Investigation
Reviewed timestamps and confirmed the failure pattern (sub-second
intervals, single target account, single source IP) is inconsistent
with human error and consistent with automated password-guessing
tooling.

## 🎯 Root Cause
Source host (192.168.x.x) executed an automated brute-force attack
using Hydra against the SSH service, leveraging a common leaked-
password wordlist (rockyou.txt).

---

## 🛠️ Response Actions Taken
- Confirmed no successful authentication occurred (0 valid passwords
  found)
- Documented source IP and attack pattern for future reference

## 📌 Recommendations
- Implement SSH key-based authentication instead of password auth
- Deploy fail2ban to automatically block IPs after repeated failures
- Restrict SSH access to a defined allowlist of source IPs where
  feasible
- Consider disabling password authentication for privileged accounts
  entirely

---

## ✅ Resolution
No compromise occurred. Findings documented for detection-tuning
reference. Ticket closed.
