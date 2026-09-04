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
- Total failed login events: 3648
- Top source IP by failure count: 4.224.23.39 (66 failed attempts — same top IP from the brute-force findings)
- Most targeted username: root (162 attempts, 8.85%)
## Screenshots
 
**Failed logins by source IP**
![Failed Logins by IP](./screenshots/failed-logins-by-ip.png)
 
**Top targeted usernames**
![Top Usernames](./screenshots/top-usernames.png)
