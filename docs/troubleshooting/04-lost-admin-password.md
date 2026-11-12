# I lost the dashboard admin password

**Symptom:** I can't log into the dashboard because I didn't save the password.

**Cause:** The installer prints it once at the end. It also stores all passwords in an archive.

**Fix:** In the folder where you ran the installer:

```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

[Back to the guide](../../README.md#5-install-wazuh)
