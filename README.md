# SSH Brute Force Detection using Splunk

A SIEM-based detection project using **Splunk Enterprise** to identify brute-force SSH login attempts from Zeek/Bro-formatted SSH connection logs — covering data ingestion, detection logic (SPL), a monitoring dashboard, and a real-time alert.

---

## 🎯 Objective
Simulate a real-world SOC analyst workflow: ingest raw SSH authentication logs into Splunk, write detection queries to surface failed logins and brute-force patterns, visualize the findings on a dashboard, and configure an alert to flag active attacks.

---

## 🏗️ Architecture
```
[ssh_logs_new.json (Zeek/Bro SSH logs)]
              ↓
      [Splunk Index: ssh_logs]
              ↓
     [SPL Detection Queries]
       ↓                ↓
 [Dashboard]        [Alert]
```

---

## 🛠️ Tools & Environment
- **Splunk Enterprise** (Windows)
- **Data format:** Zeek/Bro SSH connection logs, JSON (`sourcetype="_json"`)
- Data source: Zeek/Bro SSH connection logs (JSON format) — Zeek is a network security monitor that generates structured logs from network traffic; this project analyzes a sample Zeek SSH log dataset in Splunk.
- **Host:** Mahhyya

---

## 📂 Project Structure
| Folder | Contents |
|---|---|
| [`01-install`](./01-install) | Splunk installation steps & proof screenshots |
| [`02-add-data`](./02-add-data) | Data onboarding process, index setup, field verification |
| [`03-failed-logins`](./03-failed-logins) | Failed login detection queries & results |
| [`04-brute-force`](./05-brute-force) | Brute-force & brute-force-then-breach detection logic |
| [`05-alerts`](./07-alerts) | Alert configuration & triggered alert evidence |
| [`06-dashboards`](./06-dashboards) | Exported dashboard source (JSON) & screenshots |
| [`07-spl-queries`](./03-spl-queries) | Full SPL query reference used throughout the project |
---

## 🔑 Key Fields (Zeek/Bro SSH schema)
| Field | Meaning |
|---|---|
| `id.orig_h` | Source IP (client/attacker) |
| `id.resp_h` | Destination IP (SSH server) |
| `event_type` | Event category (e.g. "Failed SSH Login", "Successful SSH Login") |
| `auth_attempts` | Number of auth attempts in that connection |
| `auth_success` | true/false — login outcome |
| `username` | Username attempted |
| `ts` | Event timestamp |

---

## 🔍 Detection Logic Summary

**Failed Logins**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count by id.orig_h
| sort -count
```

**Brute Force (time-window based)**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| bin _time span=5m
| stats sum(auth_attempts) as total_attempts by id.orig_h, _time
| where total_attempts >= 5
```

**Brute-Force-Then-Breach (strongest finding)**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json"
| stats sum(eval(auth_success=false)) as fails, sum(eval(auth_success=true)) as successes by id.orig_h
| where fails > 3 AND successes > 0
```

Full query set with explanations: [`07-spl-queries/queries.md`](./07-spl-queries/queries.md)

---

## 🚨 Alert
Configured a scheduled alert on the brute-force detection query (5-min interval, triggers when `total_attempts >= 5` for any source IP).

![Alert Config](./05-alerts/screenshots/alert-settings_1.png)
![Alert Config](./05-alerts/screenshots/alert-settings_2.png)
![Triggered Alert](./05-alerts/screenshots/triggered-alerts.png)

---

## 📊 Dashboard
Built in Dashboard Studio with panels for:
- Failed logins over time
- Top attacking IPs
- Top targeted usernames
- Success vs. failure ratio
- Brute-force-then-breach IPs

![Dashboard](./06-dashboards/screenshots/full-dashboard.png)

---


## 📌 Key Findings
- **Top attacking IP:** `4.224.23.39` — 66 failed login attempts
- **Most targeted username:** `root` — 162 attempts (8.85% of all failed logins)
- **Brute-force-then-breach IPs identified:** None — no IP in the dataset showed both repeated failures and a subsequent success; attacking IPs either failed consistently or succeeded without prior failures
- **Password spraying:** Not observed — every attacking IP targeted exactly one username (`unique_usernames = 1` across all IPs), consistent with targeted brute-force rather than spraying
- **False positives observed:** None flagged — the `>= 5` failed-attempt threshold cleanly separated automated attack IPs from normal login noise
---
 
## 💡 What I Learned
- Installing and configuring Splunk Enterprise on Windows, including index creation and JSON log onboarding
- Writing SPL using `stats`, `eval`, `where`, and `top` to detect authentication anomalies
- Interpreting a negative/empty result as a legitimate analytical finding, not just a broken query
- Building a Splunk dashboard with multiple panel types (bar, pie, line, single value)
- Configuring a scheduled alert with cron scheduling and severity classification
## 🔮 What I'd Improve
- Use a live/streaming data source instead of a static file
- Integrate a threat intel feed to enrich `id.orig_h` with geolocation/reputation data
- Add email/webhook alert actions instead of just triggered-alerts logging
- Widen the time range window in the source data to better test time-based detections
- 
---

## 🔗 Related Projects
- [Nmap Network Reconnaissance](https://github.com/Mahhyya/Nmap-Network-Reconnaissance)
