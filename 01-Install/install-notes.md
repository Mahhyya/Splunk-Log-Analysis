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
 
**Splunk Web home page after install**
![Splunk Home](./screenshots/splunk-home.png)
 
**Server Settings confirming version and install**
![Server Settings](./screenshots/server-settings.png)
 
## Notes
- No SMTP/email server configured — alerts in this project use the "Add to Triggered Alerts" action instead of email delivery.
- Host name shown throughout the project (`Mahhyya`) is the local machine name.
 
