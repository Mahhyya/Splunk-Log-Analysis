# 03 — Failed Logins
 
## Goal
Identify all failed SSH authentication attempts and surface which source IPs and usernames are generating them.
 
## Queries Used
 
**Failed logins by source IP:**
```spl
source="ssh_logs_new.json" host="Mahhyya" sourcetype="_json" event_type="Failed SSH Login"
| stats count by id.orig_h
| sort -count
```
 
**Failed logins by source IP and username:**
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
| timechart span=1h count by id.orig_h
```
 
## Findings
- Total failed login events: _fill in_
- Top source IP by failure count: _fill in_
- Most targeted username: _fill in_
## Screenshots
Add result-table screenshots here (with the SPL search bar visible) as `screenshots/failed-logins-by-ip.png`, `screenshots/top-usernames.png`, etc.
