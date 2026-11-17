# 02 · PowerShell abuse

**MITRE:** T1059.001 (PowerShell)

### What this catches

PowerShell's Script Block log (Event ID 4104) records the text of scripts as they execute. Turning it on and shipping `Microsoft-Windows-PowerShell/Operational` to Wazuh gives you visibility into what PowerShell actually ran, including encoded and obfuscated commands. You do not need to write a custom rule. Wazuh's built-in rules already detect the encoded-command case, and this file shows how to confirm that.

### Configure on Windows

Open PowerShell as Administrator and enable Script Block Logging:

```powershell
$path = 'HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging'
New-Item $path -Force | Out-Null
Set-ItemProperty $path -Name EnableScriptBlockLogging -Value 1
```

Open `C:\Program Files (x86)\ossec-agent\ossec.conf` as Administrator and add inside `<ossec_config>`:

```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Restart the agent:

```powershell
Restart-Service WazuhSvc
```

### Test on the Windows host

Run a harmless encoded command:

```powershell
powershell.exe -EncodedCommand VwByAGkAdABlAC0ASABvAHMAdAAgAHQAZQBzAHQA
```

That decodes to `Write-Host test`. The base64 flag is what the built-in rules watch for.

### Verify

In Threat Hunting under `wazuh-alerts-*`, time range Today, search for the built-in rule:

```text
rule.id:92057
```

> ✅ `rule.id:92057` fires at level 12, with the encoded command visible in `commandLine` and the MITRE tag T1059.001. This is a Wazuh built-in rule. No custom rule was needed.

Then confirm the script block log itself is reaching the manager. Switch to `wazuh-archives-*` and search:

```text
data.win.system.providerName:*PowerShell* and data.win.system.eventID:4104
```

> ✅ You see raw 4104 events with `scriptBlockText` populated. These events have no `rule.id` because nothing about a routine script block is suspicious enough to alert. That is expected: archives captures what the ruleset ignores.

### Why no custom rule

Wazuh 4.14 ships with rules that already cover encoded PowerShell:

| Rule | Level | What it matches |
| :--- | :--- | :--- |
| 92057 | 12 | PowerShell spawned PowerShell with a base64 command |
| 91809 | 10 | PowerShell script using base64 decoding (`FromBase64String`) |
| 92071 | 12 | WMI-spawned PowerShell with base64 |

On my system, 92057 fired on the encoded command without any changes to `local_rules.xml`. Writing a custom rule for the same behavior would be redundant. In a production SOC, even built-in rules get tuned for the environment—asset groups, time windows, suppression lists. This lab stops at "the built-in rule fires," which is enough to prove the pipeline works.

[Back to detections](README.md)
