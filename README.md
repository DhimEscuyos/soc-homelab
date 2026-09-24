<div align="center">

# 🛡️ SOC Analyst Homelab

**A hands-on, $0-budget Security Operations Center built from scratch — SIEM, EDR, cloud security, and real attack simulations.**

![Status](https://img.shields.io/badge/status-active-2563EB?style=for-the-badge)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-2563EB?style=for-the-badge&logo=wazuh&logoColor=white)
![Azure Sentinel](https://img.shields.io/badge/Cloud%20SIEM-Microsoft%20Sentinel-2563EB?style=for-the-badge&logo=microsoftazure&logoColor=white)
![MITRE ATT&CK](https://img.shields.io/badge/Mapped%20to-MITRE%20ATT%26CK-2563EB?style=for-the-badge)
![Cost](https://img.shields.io/badge/cost-%240-2563EB?style=for-the-badge)

</div>

---

A hands-on, free and time consuming Security Operations Center (SOC) homelab built to develop real-world skills in SIEM administration, endpoint monitoring, 
log analysis, incident response, and cloud security monitoring — using entirely free and open-source tools, plus free-tier cloud services.

## 🧭 Overview

This lab simulates a small monitored network:
- **Wazuh** SIEM/XDR server collecting and analyzing security events
- **Microsoft Sentinel** cloud SIEM running in parallel, connected via Azure Arc — for cross-platform SIEM experience (on-prem vs cloud-native)
- **Windows 11** endpoint with Sysmon for deep process/network logging
- **Ubuntu Linux** endpoint for cross-platform log analysis experience
- **Kali Linux** attacker machine used to generate real, detectable attacks
- Custom detections mapped to the **MITRE ATT&CK** framework

---

## 🎯 Skills Demonstrated

| Skill | Where |
|---|---|
| SOC / Security Operations fundamentals | Entire project |
| SIEM administration (Wazuh) | [setup/02-wazuh-siem.md](setup/02-wazuh-siem.md) |
| Cloud SIEM administration (Microsoft Sentinel, Azure Arc) | [integrations/azure-sentinel-integration.md](integrations/azure-sentinel-integration.md) |
| EDR-style endpoint monitoring (Sysmon, Windows Defender) | [setup/03-sysmon-config.md](setup/03-sysmon-config.md) |
| Networking fundamentals (IP, ports, connectivity, firewall rules) | [setup/02-wazuh-siem.md](setup/02-wazuh-siem.md) |
| Linux & Windows administration | [setup/01-virtual-machines.md](setup/01-virtual-machines.md) |
| Log analysis & suspicious behavior identification | [detections/](detections/) |
| MITRE ATT&CK mapping & incident response | [detections/](detections/) |
| Attack simulation (Nmap, Hydra brute-force, Mimikatz, PsExec/Impacket) | [detections/](detections/) |
| Credential access & post-exploitation testing | [detections/credential-access-mimikatz.md](detections/credential-access-mimikatz.md) |
| Privilege escalation & lateral movement testing | [detections/privesc-lateral-movement.md](detections/privesc-lateral-movement.md) |
| Disaster recovery / rebuild procedures | [incident-reports/IR-002-wazuh-manager-crash-rebuild.md](incident-reports/IR-002-wazuh-manager-crash-rebuild.md) |
| Troubleshooting & root-cause analysis | Documented throughout every detection and incident write-up |

---

## 📂 Repository Structure

```
soc-homelab/
├── setup/ — infrastructure build documentation
├── detections/ — real attack simulations + detection write-ups
├── incident-reports/ — SOC-style incident tickets
├── integrations/ — cloud SIEM/EDR integration write-ups (Azure Sentinel, Arc)
├── scripts/ — automation and log parsing scripts
└── screenshots/ — supporting evidence
```

---

## 🧰 Tools Used

- VirtualBox
- Wazuh SIEM/XDR
- Microsoft Sentinel + Azure Arc (Azure Connected Machine Agent)
- Sysmon (SwiftOnSecurity config)
- Windows Defender (built-in EDR/AV)
- Kali Linux (Nmap, Hydra, Impacket)
- Mimikatz
- MITRE ATT&CK Framework

---

## 🔎 Key Findings Highlights

- End-to-end detection chain confirmed working: attack → Sysmon/agent → Wazuh rule → dashboard alert, across SSH brute-force, RDP brute-force, malware drop, file integrity monitoring, and credential-dumping scenarios
- Identified and fixed real misconfigurations along the way (disabled SSH password auth, RDP lockout policy, Wazuh manager disaster recovery)
- Demonstrated layered defense in a credential-access test: Windows Defender flagged the tool at multiple stages, Windows 11's LSA Protection blocked the actual credential dump technique even after AV detection, and Sysmon/Wazuh independently logged the suspicious behavior regardless of the other two layers
- Found and documented a real detection gap: key-based SSH lateral movement produces no distinguishing signal in logs compared to a normal login, unlike noisy brute-force attacks
- Built a working hybrid cloud/on-prem monitoring setup by registering a local VM with Azure Arc and streaming live Windows Security Events into Microsoft Sentinel

---

## 👤 About This Project

Built as part of my self-directed preparation for a SOC Analyst role,
this lab reflects hands-on experience with the exact tools and skills
requested in current SOC Tier 1 job postings — SIEM platforms (on-prem
and cloud), EDR-style monitoring, log analysis, attack simulation,
post-exploitation testing, and incident documentation.
