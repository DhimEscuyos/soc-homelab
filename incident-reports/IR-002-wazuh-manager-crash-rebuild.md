<div align="center">

# 🎫 IR-002: Wazuh Manager Crash During FIM Troubleshooting — Clean Rebuild

![Status](https://img.shields.io/badge/status-Resolved-16A34A?style=flat-square)
![Severity](https://img.shields.io/badge/severity-High-DC2626?style=flat-square)
![Type](https://img.shields.io/badge/type-Disaster%20Recovery-2563EB?style=flat-square)

</div>

---

## 📝 Summary

While configuring real-time File Integrity Monitoring (FIM) on Ubuntu-Victim, the Wazuh manager service crashed and would not restart due to a corrupted `ossec.conf`. A package reinstall did not fully resolve the issue, so the manager was rebuilt from the original OVA. Both agents were successfully re-enrolled to the new manager. No VM data, prior detections, or GitHub documentation was lost — only the manager's historical dashboard alert data.

---

## 🕒 Background: FIM Configuration (Working State Before the Crash)


Real-time FIM was configured on **Ubuntu-Victim** by editing the agent's `ossec.conf`:

```bash
# On Ubuntu-Victim
sudo sed -i 's/<directories>/<directories realtime="yes" report_changes="yes">/' /var/ossec/etc/ossec.conf
```

And enabling detection of newly created files under `<syscheck>`:

```xml
<alert_new_files>yes</alert_new_files>
```

This was confirmed working — real-time file change alerts appeared in the manager's `alerts.log` without waiting for the default 12-hour scan cycle.

---

## 💥 The Crash

While continuing to troubleshoot FIM alert behavior on the **Wazuh manager VM**, the `wazuh-manager` service failed to restart:

```
error reading XML file '/var/ossec/etc/ossec.conf' (line 0)
```

Root cause was isolated to a `wazuh-csyslogd` configuration error, compounded by a missing closing `</ossec_config>` tag. Re-adding the closing tag partially helped but did not fully resolve the failure.

**First remediation attempt — package reinstall (on the Wazuh manager VM):**

```bash
sudo apt reinstall wazuh-manager
```

This did not reliably restore a working default configuration, so a clean rebuild was chosen instead of continuing to patch the corrupted config — treating this as a disaster-recovery scenario rather than a configuration-repair scenario.

---

## 🛠️ Remediation: Clean Rebuild

### 1. Remove the broken Wazuh VM
*(VirtualBox Manager GUI, on the host machine)*
- Right-click the Wazuh VM → **Remove** → **Delete all files**

### 2. Reimport the original OVA
*(VirtualBox Manager GUI, on the host machine)*
- File → Import Appliance → select the previously downloaded `wazuh-4.14.7.ova`
- Import with default settings

### 3. Resize the virtual disk to 60GB
*(Host machine terminal — PowerShell)*

```powershell
cd "C:\Program Files\Oracle\VirtualBox"
.\VBoxManage.exe modifymedium disk "C:\Users\Cairhim\VirtualBox VMs\Wazuh v4.14.7 OVA\wazuh-4.14.7-disk-1.vdi" --resize 60000
```

Verified with:

```powershell
.\VBoxManage.exe list hdds
```

Confirmed `Capacity: 60000 MBytes`.

### 4. Set Graphics Controller to VMSVGA
*(VirtualBox Manager GUI, on the host machine)*
- Select the Wazuh VM → Settings → Display → Screen tab → Graphics Controller → **VMSVGA**

> This OVA's guest boot/console rendering expects VMSVGA; mismatched controllers on imported appliances are a common cause of console/display failures on first boot.

### 5. Boot and verify the manager
*(On the Wazuh VM)*

```bash
sudo systemctl status wazuh-manager
```

Result: `active (running)` — clean install came up with a valid default configuration.

### 6. Confirm the disk was usable at full size
*(On the Wazuh VM)*

```bash
lsblk
df -h /
```

The root partition had already auto-grown to fill the resized disk on first boot (no manual `growpart`/`resize2fs` step was needed for this particular OVA).

### 7. Determine the new manager IP
The rebuilt VM received a new DHCP lease: **192.168.1.189** (previously `192.168.1.176`).

---

## 🔗 Re-enrolling the Agents

### Ubuntu-Victim

Initial approach — manual key extraction/import — was **error-prone** and led to a misconfigured re-import (see Lessons Learned below). The reliable method used instead:

**On the Wazuh manager VM**, remove any stale/incorrect agent entries:

```bash
sudo /var/ossec/bin/manage_agents
# (R)emove agent by ID as needed
```

**On Ubuntu-Victim**, update the manager address in the agent config:

```bash
sudo sed -i 's/192.168.1.176/192.168.1.189/' /var/ossec/etc/ossec.conf
```

**On Ubuntu-Victim**, auto-enroll against the new manager (avoids manual key transcription entirely):

```bash
sudo /var/ossec/bin/agent-auth -m 192.168.1.189
sudo systemctl restart wazuh-agent
```

### Windows-Victim

**On Windows-Victim**, update the manager address (via Notepad, run as Administrator):

```
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```
Changed `<address>192.168.1.176</address>` → `<address>192.168.1.189</address>`, saved, closed.

**On Windows-Victim**, auto-enroll and restart the service:

```
cd "C:\Program Files (x86)\ossec-agent"
agent-auth.exe -m 192.168.1.189
net stop WazuhSvc
net start WazuhSvc
```

### Verification

**On the Wazuh manager VM:**

```bash
sudo /var/ossec/bin/agent_control -l
```

Both agents confirmed **Active**:
- `003 ubuntu-victim` — Active
- `Wubdiws-Victim2` — Active

Final confirmation via dashboard: live FIM events (Rule 554, "File added to the system") appearing for Ubuntu-Victim, proving real-time FIM survived the rebuild end-to-end.

---

## 💡 Lessons Learned / SOC Relevance

1. **Manual key transcription between manager and agent is fragile and error-prone.** A visually-copied base64 agent key was transcribed incorrectly, producing a working-but-wrong enrollment (the agent authenticated with the *wrong identity*, matching another agent's name/IP). This is directly analogous to real-world credential/config drift caused by manual handling of secrets — `agent-auth -m <manager_ip>` (Wazuh's built-in auto-enrollment) eliminates this failure mode entirely and should be the default method, not manual key copy-paste.

2. **Config-file corruption from live editing has a blast radius.** A missing XML closing tag during an unrelated FIM troubleshooting session took down the entire manager, not just the feature being configured. This mirrors production change-management practice: config edits on a live security tool should ideally be tested in a non-production/staging instance first, or backed up (`cp ossec.conf ossec.conf.bak`) before editing.

3. **Rebuild vs. repair is a legitimate incident response decision.** After one failed remediation attempt (package reinstall), continuing to debug a corrupted config indefinitely has diminishing returns. Recognizing when a clean rebuild is faster and lower-risk than continued repair — and confirming beforehand that the rebuild's blast radius is contained (in this case: only historical alert data, not agents, VMs, or documentation) — is itself a SOC/IT-ops skill.

4. **Disk/console settings on imported OVAs are not "set and forget."** The same OVA required disk resizing and a Graphics Controller change on this rebuild, exactly as it did on the original deployment — worth documenting as a repeatable checklist for any future Wazuh redeployment.
