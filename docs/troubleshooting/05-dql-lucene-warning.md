# "You may be using Lucene syntax while DQL is selected"

**Symptom:** Searching `data.win.system.providerName: Microsoft-Windows-Sysmon` shows a Lucene/DQL warning.

**Cause:** The search bar was in DQL mode, and I mixed in Lucene-style syntax.

**Fix:** Pick one and stay with it:

1. Keep DQL and write the query with **no space after the colon**:

```text
data.win.system.providerName:Microsoft-Windows-Sysmon
```

2. Or switch the query-language dropdown to Lucene.

> 💡 If the Events view goes blank after switching to Lucene, see [the blank Events view](09-events-view-blank.md).

[Back to the guide](../../README.md#8-first-detection)
