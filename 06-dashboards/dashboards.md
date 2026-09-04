# 06 — Dashboards

## Overview
Built a Splunk dashboard (`SSH Brute Force Detection Dashboard`) to visualize the detection logic developed in the earlier stages of this project, bringing the failed-login, brute-force, and event-type findings into one view.

## Setup Notes
- Global Time Range set to **All time** — required since the data source is a static uploaded file rather than a live/streaming feed. Individual panels needed their time range explicitly set to match, as some panels didn't inherit the dashboard-level setting by default.

## Panels

| Panel | SPL | Visualization |
|---|---|---|
| Failed logins by IP | `source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login" \| stats count as total_attempts by id.orig_h \| where total_attempts >= 5 \| sort -total_attempts` | Bar chart |
| Top targeted usernames | `source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login" \| top limit=10 username` | Bar chart |
| Event type breakdown | `source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" \| stats count by event_type` | Pie chart |
| Failed logins over time | `source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login" \| timechart count` | Line/area chart |
| Top offending IP (single value) | `source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login" \| stats count as total_attempts by id.orig_h \| sort -total_attempts \| head 1` | Single value |

## Screenshot

**Full dashboard view**
![Dashboard](./screenshots/full-dashboard.png)

## Notes
- Dashboard source (XML/JSON) was not exported — the panel SPL above fully reproduces the dashboard if rebuilt from scratch.
- The timechart panel showed limited time variation since all events in this sample dataset fall within a narrow timestamp range — realistic for a burst of brute-force traffic, but worth noting as a dataset limitation rather than a detection flaw.
