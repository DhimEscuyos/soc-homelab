<div align="center">

# 🪜 Privilege Escalation & Lateral Movement Testing

![MITRE](https://img.shields.io/badge/TA0004-Privilege%20Escalation-2563EB?style=flat-square)
![MITRE](https://img.shields.io/badge/TA0008-Lateral%20Movement-2563EB?style=flat-square)

</div>

---

## 🎯 Objective

Test common privilege-escalation vectors on Windows-Victim, and lateral-movement paths from Kali (attacker) to both Ubuntu-Victim and Windows-Victim, evaluating what each path looks like in Wazuh's logs.

**MITRE ATT&CK mapping:** TA0004 (Privilege Escalation), TA0008 (Lateral Movement), T1021.004 (Remote Services: SSH), T1021.002 (Remote Services: SMB/Windows Admin Shares)

---

## 1️⃣ Privilege Escalation (Windows-Victim)

### Account privilege check

```powershell
whoami /groups
```

**Result:** the `socanalyst` account is already a member of `BUILTIN\Administrators`, with High Mandatory Level.

**Finding:** privilege escalation testing is not applicable in the traditional sense — there is no "climb" required, because the account already has full administrative rights. **This itself is the real finding**: a standard lab/user account configured with local admin rights is a common real-world misconfiguration. If this were a phished credential in a real environment, the attacker would gain full admin access immediately with zero privesc effort required.

### AlwaysInstallElevated check

A common MSI-based privesc vector — if set, any `.msi` install runs with SYSTEM privileges regardless of the installing user's actual rights.

```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

**Result:** both keys not found — not configured. This vector is closed.

### Unquoted service path check

An unquoted path containing spaces (e.g. `C:\Program Files\Some App\service.exe` without surrounding quotes) can allow privilege escalation if a lower-privileged user can write a malicious executable earlier in the path.

```
wmic service get name,displayname,pathname,startmode
```

**Result:** manual review of all LocalSystem services found no unquoted, space-containing paths — every service path with a space was properly quoted.

### Privesc conclusion

No exploitable privilege-escalation vector was found — but only because escalation wasn't necessary in the first place. The account's baseline over-privileged status is the documented risk.

---

## 2️⃣ Lateral Movement

### Test A: SSH key-based lateral movement (Kali → Ubuntu-Victim)

Simulates an attacker who has obtained a valid SSH private key (e.g., from a compromised jump host, leaked repository, or stolen laptop).

**On Kali:**
```bash
ssh -i ~/.ssh/ubuntu_victim_key socanalyst@192.168.1.178
```

**Result:** immediate, successful login — no password prompt, no failed attempts.

**Wazuh detection:**
```
Rule 5501 — "PAM: Login session opened" — Level 3
```

This is the same generic rule that fires for any normal, benign login. **No elevated severity, no distinguishing tag, no MITRE mapping** — a successful key-based lateral-movement login is indistinguishable in the SIEM from routine daily activity.

### Test B: SMB/PsExec lateral movement (Kali → Windows-Victim)

Simulates an attacker with valid credentials attempting remote code execution via SMB, using `impacket-psexec` (a Linux/Kali recreation of Microsoft's PsExec).

**On Kali:**
```bash
impacket-psexec socanalyst:'<password>'@192.168.1.179
```

**Result:**
```
[-] [Errno Connection error (192.168.1.179:445)] [Errno 113] No route to host
```

**Investigation, on Windows-Victim:**
```powershell
Get-NetFirewallRule -DisplayGroup "File and Printer Sharing" | Select DisplayName, Enabled, Direction
```
Result: every rule in the "File and Printer Sharing" group — including `SMB-In` — is `Enabled: False`.

```powershell
Get-NetTCPConnection -LocalPort 445
```
Result: `445` is listening (`State: Listen`) — the SMB service itself is running.

**Confirmed from Kali:**
```bash
nmap -p 445 192.168.1.179
```
```
445/tcp filtered microsoft-ds
```

`filtered` means packets are being silently dropped with no reply — the classic signature of a firewall block, as opposed to `closed` (service down) or `open` (reachable).

**Conclusion:** the SMB service is active but Windows Firewall's default-disabled File and Printer Sharing rules block all inbound access — SMB-based lateral movement fails out of the box on this machine.

---

## 📋 Findings Summary

| Test | Outcome | Detection Signal |
|---|---|---|
| Privilege escalation (Windows) | N/A — account already local Administrator | N/A (finding is the over-privileged account itself) |
| AlwaysInstallElevated | Not configured | Vector closed |
| Unquoted service paths | None found | Vector closed |
| SSH key lateral movement (Kali → Ubuntu) | **Succeeded** | Minimal — generic Level 3 login event only |
| SMB/PsExec lateral movement (Kali → Windows) | **Blocked** | N/A — connection never established |

## 💡 SOC Relevance

This testing surfaced two genuinely useful, realistic findings:

1. **Over-privileged accounts eliminate the need for privilege escalation entirely.** Auditing account privilege levels (not just testing escalation techniques) is a necessary complement to privesc testing — if every account is already admin, privesc testing tells you little about actual risk.

2. **Detection coverage is asymmetric between loud and quiet attacks.** The earlier SSH/RDP brute-force tests in this lab escalated to Level 10 specifically because repeated failures are noisy and easy to rule-match. A single successful, credential-based lateral movement — exactly the kind of activity a real attacker would prefer — produces almost no distinguishing signal. This is a well-known real-world SOC blind spot, and is the underlying reason techniques like impossible-travel detection, new-source-IP alerting, and UEBA (User and Entity Behavior Analytics) exist in mature security programs.

3. **Default-secure configurations matter.** Windows Firewall's default-off posture for File and Printer Sharing meaningfully blocked a realistic attack path without any additional hardening effort — a good example of "secure by default" reducing attack surface even against a fully credentialed attacker.
