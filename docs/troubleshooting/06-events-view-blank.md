# The Events view is blank

**Symptom:** After switching to Lucene, the Events view showed nothing.

**Cause:** A dashboard UI bug plus the time range. The default range is only the last 15 minutes. The data was never lost.

**Fix:**

1. Switch the query language back to DQL.
2. Widen the time range from 15 minutes to **Today**.

> ✅ Your events show up again.

[Back to the guide](../../README.md#8-first-detection)
