# SOC Analyst Homelab

A hands-on, free and time consuming Security Operations Center (SOC) homelab built to develop real-world skills in SIEM administration, endpoint monitoring, 
log analysis, and incident response — using entirely free and open-source tools.

## Overview

This lab simulates a small monitored network:
- **Wazuh** SIEM/XDR server collecting and analyzing security events
- **Windows 11** endpoint with Sysmon for deep process/network logging
- **Ubuntu Linux** endpoint for cross-platform log analysis experience
- **Kali Linux** attacker machine used to generate real, detectable attacks
- Custom detections mapped to the **MITRE ATT&CK** framework

## Skills Demonstrated

| Skill | Where |
|---|---|
| SOC / Security Operations fundamentals | Entire project |
| SIEM administration (Wazuh) | [setup/02-wazuh-siem.md](setup/02-wazuh-siem.md) |
| EDR-style endpoint monitoring (Sysmon) | [setup/03-sysmon-config.md](setup/03-sysmon-config.md) |
| Networking fundamentals (IP, ports, connectivity) | [setup/02-wazuh-siem.md](setup/02-wazuh-siem.md) |
| Linux & Windows administration | [setup/01-virtual-machines.md](setup/01-virtual-machines.md) |
| Log analysis & suspicious behavior identification | [detections/](detections/) |
| MITRE ATT&CK mapping & incident response | [detections/](detections/) |
| Attack simulation (Nmap, Hydra brute-force) | [detections/ssh-brute-force.md](detections/ssh-brute-force.md), [detections/rdp-brute-force.md](detections/rdp-brute-force.md) |
| Troubleshooting & root-cause analysis | Documented throughout every detection write-up |

## Repository Structure

soc-homelab/
├── setup/ — infrastructure build documentation
├── detections/ — real attack simulations + detection write-ups
├── incident-reports/ — SOC-style incident tickets
├── scripts/ — automation and log parsing scripts
└── screenshots/ — supporting evidence


## Tools Used

- VirtualBox
- Wazuh SIEM/XDR
- Sysmon (SwiftOnSecurity config)
- Kali Linux (Nmap, Hydra)
- MITRE ATT&CK Framework

## About This Project

Built as part of my self-directed preparation for a SOC Analyst role,
this lab reflects hands-on experience with the exact tools and skills
requested in current SOC Tier 1 job postings — SIEM platforms, EDR-style
monitoring, log analysis, attack simulation, and incident documentation.