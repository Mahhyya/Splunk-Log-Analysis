# 04 — Brute Force Detection
 
## Goal
Move beyond raw failed-login counts to detect actual brute-force *patterns* — repeated attempts within a short time window, and cases where an attacker eventually succeeded after multiple failures.
 
## Detection Approaches
 
**A — High attempt count per connection**
Uses Zeek's own `auth_attempts` counter (attempts within a single connection):
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json"
auth_attempts > 3 auth_success=false
| table _time, id.orig_h, username, auth_attempts
| sort -auth_attempts
```
 
**B — Time-window brute force**
Flags source IPs with 5+ cumulative failed attempts (catches attackers cycling through multiple connections):
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count as total_attempts by id.orig_h
| where total_attempts >= 5
| sort -total_attempts
```
 
**C — Brute-force-then-breach**
The strongest signal: an IP that failed repeatedly and then succeeded.
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json"
| stats sum(eval(auth_success=false)) as fails, sum(eval(auth_success=true)) as successes by id.orig_h
| where fails > 3 AND successes > 0
| sort -fails
```
 
**D — Password spraying check**
One IP trying many different usernames with few attempts each (a different attack pattern than classic brute force):
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats dc(username) as unique_usernames, count as total_failures by id.orig_h
| where unique_usernames > 5
| sort -unique_usernames
```
 
## Why This Matters
A source IP with just 1-2 failed logins is usually normal user error (wrong password, typo). An IP with 5 or more failed attempts is a much stronger sign of an automated brute-force attack rather than a genuine mistake — that's why the `total_attempts >= 5` threshold is used to separate normal noise from suspicious activity.
 
## Findings
- IPs flagged by Approach B: A large number of source IPs exceeded the `>= 5` failed-attempt threshold, indicating widespread automated brute-force activity rather than isolated incidents. Top 10 offenders:
  | Source IP | Failed Attempts |
  |---|---|
  | 4.224.23.39 | 66 |
  | 105.236.211.106 | 60 |
  | 110.177.195.150 | 60 |
  | 52.173.49.103 | 60 |
  | 74.165.131.224 | 60 |
  | 113.173.136.4 | 54 |
  | 122.204.186.125 | 54 |
  | 135.176.100.83 | 54 |
  | 181.91.226.98 | 54 |
  | 83.195.24.226 | 54 |
- Any brute-force-then-breach IPs found (Approach C): No IPs in this dataset exhibited a brute-force-then-breach pattern — attacking source IPs either failed consistently across all attempts or succeeded without any preceding failures, suggesting the successful logins may represent legitimate users rather than compromised brute-force attempts.
- Password spraying detected: No source IP exhibited password-spraying behavior (targeting 5+ distinct usernames) in this dataset — failed attempts were concentrated on a small number of usernames per IP, consistent with targeted brute-force rather than spraying.
## Screenshots
 
**Approach A — High attempt count per connection**
![Approach A](./screenshots/approach-a.png)
 
**Approach B — Time-window brute force**
![Approach B](./screenshots/approach-b.png)
 
**Approach C — Brute-force-then-breach**
![Approach C](./screenshots/approach-c.png)
 
**Approach D — Password spraying check**
![Approach D](./screenshots/approach-d.png)

