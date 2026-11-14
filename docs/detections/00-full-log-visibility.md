# 00 · Full log visibility

**MITRE:** none. This is a prerequisite, not a detection.

### What this fixes

By default Wazuh writes every event the agent sends into `/var/ossec/logs/archives/archives.json`, but Filebeat does not ship that file unless archives are enabled. Without archives, the dashboard only has the alerts index (`wazuh-alerts-*`), which contains events that matched a rule.

That means an event that matched no rule—a process launch, a file write, a network connection—is written to disk by the manager and then never indexed. It is not that Wazuh deleted it; it is that nobody shipped it.

> 💡 A command that triggers an alert can still be partially visible through the alert's `full_log` field. Archives is not about recovering those. It is about seeing the events that triggered **nothing at all**.

### Configure the manager

SSH into the Wazuh server:

```powershell
ssh <UBUNTU_USER>@<WAZUH_SERVER_IP>
```

Edit the manager config:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Inside `<global>`, set both to `yes`:

```xml
<global>
  <jsonout_output>yes</jsonout_output>
  <alerts_log>yes</alerts_log>
  <logall>yes</logall>
  <logall_json>yes</logall_json>
</global>
```

> 💡 `logall_json` is the one that matters for the dashboard. `logall` writes a second copy in plain syslog format that the dashboard does not use.

Restart the manager:

```bash
sudo systemctl restart wazuh-manager
```

### Tell Filebeat to ship archives

Still on the server:

```bash
sudo nano /etc/filebeat/filebeat.yml
```

Find the `archives:` block and set it to enabled:

```yaml
archives:
  enabled: true
```

Save and exit, then restart Filebeat:

```bash
sudo systemctl restart filebeat
```

### Create the index pattern

1. In the dashboard, go to **Dashboards Management ➜ Index Patterns ➜ Create index pattern**.
2. Name it `wazuh-archives-*`.
3. Check **Include system and hidden indices**.
4. Set the time field to `timestamp`.
5. Click **Create index pattern**.

> ⚠️ Use `timestamp`, not `@timestamp`. The archives index stores the event's original time in `timestamp`. Picking the wrong field makes Discover show an empty view.

### Test

**confirm archives is live**

On the Windows host, open Chrome or Edge and load any page. Chrome and Edge subprocesses are logged by Sysmon, so they make a reliable trigger. Notepad does not work because the SwiftOnSecurity config excludes it from process creation logging.

In threat hunting , switch the index pattern to `wazuh-archives-*` and set the time range to **Today**. Search:

```text
data.win.eventdata.image:*chrome.exe
```

> ✅ You see raw process creation events with a `commandLine` field and no `rule.description`. That field only exists on alerts.

> ⚠️ Archives grow fast. On an 8 GB VM, check disk usage weekly:

```bash
df -h /var/ossec/logs
```

[Back to detections](README.md)
