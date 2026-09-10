# Deploying Sysmon for Enhanced Windows Logging

## Overview
Windows' default event logging lacks the detail needed for meaningful
security monitoring. Sysmon (System Monitor), a free Microsoft
Sysinternals tool, provides detailed logging of process creation,
network connections, and registry changes — data essential for
detecting attacker behavior.

## Deployment
- Downloaded Sysmon from Microsoft's Sysinternals site
- Downloaded the SwiftOnSecurity Sysmon configuration from GitHub — a
  widely-used, well-tuned open-source config that balances detection
  coverage with log volume
- Installed via PowerShell (Administrator):

```powershell
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

## Integrating with Wazuh
Added a `<localfile>` block to the Wazuh agent's `ossec.conf`, pointing
to the Sysmon Windows Event Log channel:

```xml
<localfile>
<location>Microsoft-Windows-Sysmon/Operational</location>
<log_format>eventchannel</log_format>
</localfile>
```

Restarted the Wazuh agent service to apply the change.

## Verification
Generated test events (launching Notepad, Calculator, Paint) and
confirmed Sysmon events appeared in the Wazuh dashboard, automatically
correlated with MITRE ATT&CK techniques including PowerShell execution,
file deletion, account discovery, and ingress tool transfer.

## Troubleshooting Encountered
- Initial config had a typo (`evenchannel` instead of `eventchannel`)
  and a mismatched closing XML tag (`</location>` instead of
  `</localfile>`) — resolved by carefully re-editing and validating the
  full configuration block against the source XML

## Skills Demonstrated
- Windows endpoint logging and EDR-style monitoring configuration
- XML configuration editing and troubleshooting
- Log pipeline verification (source → collection → SIEM → dashboard)
- MITRE ATT&CK-aligned detection interpretation
