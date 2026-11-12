# Copy-paste doesn't work in the VM console

**Symptom:** Pasting into the VirtualBox console does nothing.

**Cause:** It didn't work on my setup. That's why SSH matters.

**Fix:** Type the first few commands by hand (`ip a`, then the `lvextend` command). Once you have the VM's IP, connect from Windows PowerShell and do everything else over SSH:

```powershell
ssh <UBUNTU_USER>@<WAZUH_SERVER_IP>
```

> ✅ You have an SSH prompt and paste works.

[Back to the guide](../../README.md#4-connect-over-ssh)
