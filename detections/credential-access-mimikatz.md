<div align="center">

# 🔑 Credential Access — Mimikatz / LSASS Testing (Windows-Victim)

![MITRE](https://img.shields.io/badge/MITRE%20ATT%26CK-T1003.001%20LSASS%20Memory-2563EB?style=flat-square)
![Result](https://img.shields.io/badge/result-Blocked%20(LSA%20Protection)-16A34A?style=flat-square)

</div>

---

## 🎯 Objective

Test whether a compromised Windows-Victim account (already local Administrator — see [privesc-lateral-movement.md](privesc-lateral-movement.md)) could be used to dump credentials from memory using Mimikatz, and evaluate detection coverage across Windows Defender, OS-level mitigations, and Wazuh/Sysmon.

**MITRE ATT&CK mapping:** T1003.001 (OS Credential Dumping: LSASS Memory)

---

## 🧪 Test Steps

### 1. Download Mimikatz

Downloaded from the official source (gentilkiwi's GitHub releases) — never from third-party mirrors, since fake Mimikatz builds are a common malware-delivery trick:
```
https://github.com/gentilkiwi/mimikatz/releases
```

### 2. Check Defender's response

```powershell
Get-MpThreatDetection
```

### 3. Attempt execution

Ran as Administrator:
```
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
```

### 4. Cross-reference with Wazuh/Sysmon

Searched the Wazuh dashboard for `rule.groups:"sysmon" AND agent.name:"Wubdiws-Victim"` and related queries.

---

## 📊 Results

### Windows Defender — detected at multiple stages

`Get-MpThreatDetection` confirmed Defender caught the activity repeatedly, all under `ThreatID 2147894093` (the mimikatz.exe hacktool signature) and `2147720870` (the zip download):

| Stage | Time | Detail |
|---|---|---|
| Zip download | 06:19:43 | `mimikatz_trunk.zip` flagged via web download scan |
| In-progress download (`.crdownload`) | 06:12:54 | Flagged before download even completed |
| Extracted executable | 06:19:58 | `mimikatz.exe` flagged post-extraction, action triggered by `explorer.exe` |
| Execution attempt | 06:19:43–06:20:02 | Flagged again at actual process execution (`ThreatStatusID: 7`), remediation logged |

Despite repeated detection and remediation actions, **the file remained runnable** — Defender's quarantine/remediation did not fully prevent the binary from executing.

### Execution result — blocked by OS-level mitigation, not by AV

```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
```

`0x00000005` = `ACCESS_DENIED`. Notably, `privilege::debug` succeeded — Mimikatz obtained `SeDebugPrivilege` — but still couldn't open a memory handle to `lsass.exe`.

**Root cause, confirmed via registry:**

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL"
```
Result: `RunAsPPL : 2`

This means **LSA Protection is enabled and enforced with a UEFI lock** — `lsass.exe` runs as a Protected Process Light (PPL), which blocks memory access even from a process holding `SeDebugPrivilege`. This is exactly the `ACCESS_DENIED` error Mimikatz hit.

Checked for Credential Guard as a second possible cause:
```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" -Name "EnableVirtualizationBasedSecurity"
```
Returned nothing — **Credential Guard/VBS is not configured**, confirming LSA Protection specifically (not Credential Guard) was the blocking mechanism.

### Wazuh/Sysmon — independent detection, regardless of AV/OS outcome

Searching `rule.groups:"sysmon" AND agent.name:"Wubdiws-Victim"` returned 97 hits in 24 hours, including several directly relevant to this test:

| Rule ID | Level | Description |
|---|---|---|
| 92213 | 15 | Executable file dropped in folder commonly used by malware |
| 92217 | 6 | Executable dropped in Windows root folder |
| 61640 | 12 | Sysmon - Suspicious Process - explorer.exe |

These fired independently of whether Defender or LSA Protection succeeded in blocking the actual credential theft — meaning SIEM visibility into this attack chain does **not** depend on the AV/EDR layer succeeding.

---

## 📋 Findings Summary

1. **Delivery was detected** — Defender flagged the tool at download, extraction, and execution stages
2. **Delivery detection did not fully prevent execution** — the binary still ran despite repeated Defender action
3. **The actual credential-dumping technique was blocked by a separate OS-level control** (LSA Protection / RunAsPPL), independent of Defender
4. **SIEM detection was independent of both layers** — Sysmon/Wazuh caught the suspicious file-drop and process behavior on their own merits

## 💡 SOC Relevance

This is a complete layered-defense case study demonstrating why security architectures use multiple independent controls rather than relying on any single layer:
- **AV/EDR** (Defender) provides signature/heuristic-based detection at delivery
- **OS-level hardening** (LSA Protection) provides technique-level prevention even if delivery detection is imperfect or bypassed
- **SIEM/log-based detection** (Sysmon + Wazuh) provides behavioral visibility that doesn't depend on either of the above succeeding

An analyst investigating this scenario would be able to reconstruct the full attack timeline — and know the attack was ultimately unsuccessful at extracting credentials — purely from the interaction of these three independent data sources, without needing all three to have "worked" the same way.
