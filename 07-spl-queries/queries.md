# SPL Query Reference — SSH Brute Force Detection (Splunk)

**Data source:** `source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json"`
**Format:** Zeek/Bro SSH connection logs (JSON)

**Key fields:**
| Field | Meaning |
|---|---|
| `id.orig_h` | Source IP (client/attacker) |
| `id.resp_h` | Destination IP (SSH server) |
| `id.orig_p` / `id.resp_p` | Source / destination port |
| `event_type` | Type of event (e.g. "Failed SSH Login", "Successful SSH Login") |
| `auth_attempts` | Number of auth attempts in that connection |
| `auth_success` | true / false / null — whether login succeeded |
| `username` | Username attempted |
| `conn_state` | Connection state (SF, S0, etc.) |
| `ts` | Event timestamp |

---

## 1. Types of Events

**See every distinct event_type in the dataset:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json"
| stats count by event_type
| sort -count
```

**Check the values of auth_success:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json"
| stats count by auth_success
```

---

## 2. Successful Logins

**All successful SSH logins:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Successful SSH Login"
| stats count by id.orig_h, username
| sort -count
```

---

## 3. Failed Login Attempts

**Failed logins by source IP:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count by id.orig_h
| sort -count
```

**Failed logins broken down by username too:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count by id.orig_h, username
| sort -count
```

**Top 10 most-targeted usernames:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| top limit=10 username
```

**Failed logins over time:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| timechart count
```

---

## 4. Brute Force Detection

**A — High attempt count per connection:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" auth_attempts>3 auth_success=false
| stats count by id.orig_h
| sort -count
```

**B — Total failed attempts per IP (main detection query, used for the alert):**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count as total_attempts by id.orig_h
| where total_attempts >= 5
| sort -total_attempts
```

**C — Brute-force-then-breach (fails followed by a success):**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json"
| stats sum(eval(auth_success=false)) as fails, sum(eval(auth_success=true)) as successes by id.orig_h
| where fails > 3 AND successes > 0
| sort -fails
```
*Finding: returned no results on this dataset — no IP showed both repeated failures and a success. Attacking IPs either failed consistently or succeeded without prior failures.*

**D — Password-spraying check (one IP, many different usernames):**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats dc(username) as unique_usernames, count as total_failures by id.orig_h
| where unique_usernames > 5
| sort -unique_usernames
```

---

## 5. Dashboard Panel Queries

| Panel | Query |
|---|---|
| Failed logins by IP | `event_type="Failed SSH Login" \| stats count as total_attempts by id.orig_h \| where total_attempts >= 5 \| sort -total_attempts` |
| Top targeted usernames | `event_type="Failed SSH Login" \| top limit=10 username` |
| Event type breakdown | `\| stats count by event_type` (pie chart) |
| Failed logins over time | `event_type="Failed SSH Login" \| timechart count` |
| Top offending IP (single value) | `event_type="Failed SSH Login" \| stats count as total_attempts by id.orig_h \| sort -total_attempts \| head 1` |

---

## 6. Alerts

**Alert base search:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count as total_attempts by id.orig_h
| where total_attempts >= 5
| sort -total_attempts
```

**Configuration:**
- Time range: All time
- Schedule: Cron `*/5 * * * *` (every 5 minutes)
- Trigger condition: Number of Results > 0, trigger Once
- Severity: Medium
- Trigger action: Add to Triggered Alerts

---

## Notes / Findings
- Top attacking IP: `4.224.23.39` (66 failed attempts)
- Multiple additional IPs also exceeded the 5-attempt threshold (105.236.211.106, 110.177.195.150, 52.173.49.103, 74.165.131.224, and others at 54–60 attempts)
- No brute-force-then-breach pattern found in this dataset
- Password spraying: No source IP exhibited password-spraying behavior (targeting 5+ distinct usernames) in this dataset — failed attempts were concentrated on a small number of usernames per IP, consistent with targeted brute-force rather than spraying.
