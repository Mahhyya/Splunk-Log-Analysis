# 07 — Alerts

## Goal
Convert the brute-force detection query into a scheduled alert so active attacks are flagged without manually re-running searches.

## Base Search
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count as total_attempts by id.orig_h
| where total_attempts >= 5
| sort -total_attempts
```

## Configuration Steps
1. Ran the base search above in Splunk.
2. Clicked **Save As → Alert**.
3. Title: `SSH Brute Force - Failed Login Threshold`
4. Time range: **All time** (since the data source is a static uploaded file, not a live feed).
5. Schedule: **Cron Schedule** — `*/5 * * * *` (every 5 minutes), chosen to simulate near-real-time detection.
6. Trigger condition: **Number of Results is greater than 0**, trigger **Once** per run.
7. Severity: **Medium** — failed login attempts indicate active probing/attack behavior, but no successful breach was confirmed in this dataset (see Approach C in `05-brute-force/brute-force.md`), so it doesn't warrant a Critical/High classification.
8. Trigger action: **Add to Triggered Alerts** (no SMTP server configured, so email delivery wasn't used).
9. Ran the search manually once to confirm the alert fires correctly.
10. Verified in **Activity → Triggered Alerts** that the alert appeared with the correct timestamp and matching results.

## Why This Threshold
5 or more failed login attempts from a single source IP is a reasonable baseline for flagging brute-force behavior without generating excessive false positives from normal user typos.

## Screenshots

**Alert configuration page showing trigger condition and schedule**
![Alert Settings](./screenshots/alert-settings.png)

**Activity → Triggered Alerts showing a fired instance**
![Triggered Alerts](./screenshots/triggered-alerts.png)

## Possible Improvements
- Add an email or webhook action for real delivery instead of triggered-alerts logging.
- Tune the threshold/window based on a larger, more realistic dataset.
