# 05 — Alerts
 
## Goal
Convert the brute-force detection query into a scheduled alert so active attacks are flagged without manually re-running searches.
 
## Base Search
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| bin _time span=5m
| stats sum(auth_attempts) as total_attempts by id.orig_h, _time
| where total_attempts >= 5
```
 
## Configuration Steps
1. Ran the base search above in Splunk.
2. Clicked **Save As → Alert**.
3. Trigger condition: **Number of results > 0**.
4. Schedule: runs every **5 minutes**.
5. Trigger action: **Add to Triggered Alerts** (no SMTP server configured, so email delivery wasn't used — this action logs the trigger event instead, which is enough to demonstrate the alert firing).
6. Ran the search manually once to confirm the alert fires correctly.
7. Verified in **Activity → Triggered Alerts** that the alert appeared with the correct timestamp and matching results.
## Why This Threshold
5 failed attempts within a 5-minute window from a single source IP is a reasonable baseline for flagging brute-force behavior without generating excessive false positives from normal user typos.
 
## Screenshots
| File | Description |
|---|---|
| `screenshots/alert-settings.png` | Alert configuration page showing trigger condition and schedule |
| `screenshots/triggered-alerts.png` | Activity → Triggered Alerts showing a fired instance |
 
## Possible Improvements
- Add an email or webhook action for real delivery instead of triggered-alerts logging.
- Tune the threshold/window based on a larger, more realistic dataset.
