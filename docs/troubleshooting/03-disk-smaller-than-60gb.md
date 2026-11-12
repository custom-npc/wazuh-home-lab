# The disk shows less than 60 GB

**Symptom:** After install, Ubuntu doesn't use the full 60 GB.

**Cause:** The default LVM setup only uses part of the disk.

**Fix:**

```bash
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

```bash
df -h /
```

> ✅ `df -h /` shows about 57 GB.

[Back to the guide](../../README.md#4-connect-over-ssh)
