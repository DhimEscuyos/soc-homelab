<div align="center">

# 🛡️ Deploying Wazuh SIEM

![Setup](https://img.shields.io/badge/type-infrastructure%20setup-2563EB?style=flat-square)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%204.14.7-2563EB?style=flat-square)

</div>

---

## 📋 Overview
Wazuh is a free, open-source SIEM and XDR platform used to collect,
analyze, and alert on security events from monitored endpoints. This
mirrors real-world SOC tooling like Splunk or Microsoft Sentinel.

## 🚀 Deployment
- Downloaded the official Wazuh OVA (pre-built virtual appliance,
  version 4.14.7) from documentation.wazuh.com
- Imported into VirtualBox (8GB RAM, 4 CPU cores)
- Resized the virtual disk from the default 25GB to 60GB using the
  VBoxManage command line tool after encountering disk space issues
- Set Graphics Controller to VMSVGA to avoid display freezing (a known
  VirtualBox/Wazuh compatibility issue)

## 🔑 Accessing the Dashboard
- Retrieved the VM's IP address using `ip addr`
- Accessed the web dashboard via HTTPS at that IP
- Logged in with default indexer credentials (admin/admin), noting this
  should be changed in any non-lab environment

---

## 🔗 Connecting Agents
Deployed the Wazuh agent to both victim machines using the dashboard's
"Deploy new agent" wizard:

- **Windows-Victim**: Installed via PowerShell using an MSI installer,
  configured to report to the Wazuh manager's IP
- **Ubuntu-Victim**: Installed via a `.deb` package using `dpkg`

---

## 🐞 Troubleshooting Encountered
- Agent initially enrolled but showed "Unknown" status — resolved by
  restarting the agent service and verifying manager connectivity
- Wazuh manager service failed to restart due to the VM's disk being
  100% full — resolved by rebuilding the VM with a larger (60GB)
  virtual disk
- Agent configuration required manual correction after the Wazuh
  server's IP address changed following the rebuild — edited
  `ossec.conf` directly to point to the new manager IP

> 📌 See [incident-reports/IR-002-wazuh-manager-crash-rebuild.md](../incident-reports/IR-002-wazuh-manager-crash-rebuild.md) for the full disaster-recovery writeup of a later, more serious manager crash and rebuild.

---

## ✅ Skills Demonstrated
- SIEM deployment and administration
- Agent-based log collection architecture
- Linux disk management and troubleshooting
- Network configuration and connectivity debugging
