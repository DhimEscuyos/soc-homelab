# Detection: Brute-Force Attacks (SSH + RDP)

**Date:** September 9-10, 2026
**MITRE ATT&CK Technique:** T1110 — Brute Force
**Source:** Kali Linux attacker VM (192.168.x.x)

## Objective
Simulate real-world brute-force attacks against both a Linux endpoint
(via SSH) and a Windows endpoint (via RDP), and verify that the Wazuh
SIEM correctly detects and escalates the activity on both platforms.

---

## Part 1: SSH Brute-Force — Ubuntu-Victim (192.168.x.x)

### Attack Execution
```bash
hydra -l socanalyst -P /usr/share/wordlists/rockyou.txt.gz ssh://192.168.x.x -t 4
```

### Detection Results
Within seconds, Wazuh generated 249 correlated alerts in a 30-second
window:

| Rule ID | Description | Level |
|---|---|---|
| 5760 | sshd: authentication failed | 5 |
| 2501 | syslog: User authentication failure | 5 |
| 5557 | unix_chkpwd: Password check failed | 5 |
| 5758 | Maximum authentication attempts exceeded | 8 |
| 2502 | User missed the password more than one time | 10 |
| **40111** | **Multiple authentication failures** | **10** |

Wazuh's built-in correlation engine automatically escalated severity
from Level 5 (single failure) to **Level 10** on detecting the
repeated-failure pattern — no custom rule required.

### Analysis
- Identical, millisecond-apart timestamps across failed attempts is a
  clear signature of automated tooling, not human error
- Multiple log sources (sshd, syslog, PAM/unix_chkpwd) corroborated the
  same event, increasing confidence this was a genuine attack

---

## Part 2: RDP Brute-Force — Windows-Victim (192.168.x.x)

### Pre-Attack Troubleshooting
The initial attack attempt failed with connection errors. Root cause
investigation found **three separate misconfigurations** blocking RDP
entirely:

1. The Remote Desktop service (`TermService`) was stopped
2. The registry flag `fDenyTSConnections` was set to `1` (deny
   connections)
3. Windows Firewall's Remote Desktop rule group was disabled

Fixed all three:
```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Start-Service -Name TermService
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

A further connection error persisted after these fixes. Verified RDP
itself was functioning correctly using a manual connection attempt
with a deliberately wrong password:
```bash
xfreerdp /v:192.168.x.x /u:Administrator /p:wrongpassword /cert:ignore
```
This returned `ERRCONNECT_LOGON_FAILURE` — confirming the connection
and authentication negotiation worked correctly, isolating the
remaining issue to Hydra's RDP module specifically (noted in Hydra's
own documentation as "experimental"). Adjusting to a single-threaded,
slower attempt resolved it:

```bash
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt.gz rdp://192.168.x.x -t 1 -W 5
```

### Detection Results
| Rule ID | Description | Level | MITRE Tag |
|---|---|---|---|
| 60122 | Logon Failure - Unknown user or bad password | 5 | T1531 |

### Analysis
- Wazuh auto-tagged this detection as **T1531 (Account Access
  Removal)**. This is arguably an imprecise auto-mapping — the more
  standard technique for repeated failed logins is **T1110 (Brute
  Force)**. Noted here as an example of validating automated
  tool/rule output rather than accepting classifications at face
  value — an important habit for SOC analysts.

---

## Recommended Response (if this were a real environment)
- Block or rate-limit the source IP at the firewall
- Enforce key-based SSH authentication and/or MFA for RDP
- Enable account lockout policies after repeated failures
- Review whether brute-forced accounts should have remote access at all

## Skills Demonstrated
- Offensive security tool usage (Hydra, xfreerdp, Nmap) in a
  controlled/isolated environment
- Cross-platform log analysis (Linux syslog/PAM vs. Windows Security
  Event Log)
- SIEM alert correlation and severity escalation analysis
- Root-cause troubleshooting across service, registry, and firewall
  layers
- Critical evaluation of automated MITRE ATT&CK tagging
- Incident documentation