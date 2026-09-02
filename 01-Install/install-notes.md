# 01 — Install
 
## Overview
Splunk Enterprise was installed locally on Windows to serve as the SIEM platform for this project.
 
## Steps Taken
1. Downloaded **Splunk Enterprise for Windows** (.msi) from splunk.com (free trial license).
2. Ran the installer, set the admin username/password.
3. Kept default ports:
   - `8000` — Splunk Web UI
   - `8089` — Management port
4. Launched Splunk Web at `http://localhost:8000` and logged in.
5. Confirmed the install by checking **Settings → Server Settings**, which shows the running version.
## Screenshots
| File | Description |
|---|---|
| `screenshots/splunk-home.png` | Splunk Web home/launcher page after install |
| `screenshots/server-settings.png` | Server Settings page confirming version and install |
 
## Notes
- No SMTP/email server configured — alerts in this project use the "Add to Triggered Alerts" action instead of email delivery.
- Host name shown throughout the project (`Mahhyya`) is the local machine name.
 
