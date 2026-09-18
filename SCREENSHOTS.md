# Home SOC Portfolio Screenshots

These screenshots are sanitized for public portfolio use. Private Tailscale IP addresses, passwords, private keys, credentials, and backup archives are intentionally excluded.

## 1. Endpoint Monitoring

![Endpoint Monitoring](screenshots/01-agents.png)

**Endpoint Monitoring — Windows 11 and Ubuntu endpoints actively monitored by Wazuh 4.14.7.**

The view demonstrates two enrolled endpoints, their operating systems, Wazuh Agent version, active status, and synchronization state without exposing private IP addresses.

## 2. Custom Detection Dashboard

![Custom Detection Dashboard](screenshots/02-custom-dashboard.png)

**Custom Detection Dashboard — Overview of custom detections, severity, timeline and affected endpoints.**

The dashboard focuses on the active custom detections rather than duplicating Wazuh's broader Threat Hunting view.

## 3. Custom Rule 100150 — PowerShell Scheduled Task

![Rule 100150](screenshots/03-rule-100150-powershell-task.png)

**Custom Rule 100150 — Detection of Windows Scheduled Tasks configured to execute PowerShell.**

The screenshot shows real controlled-lab hits on `Windows-Endpoint`, with Rule ID `100150` and severity Level 10.

## 4. Custom Rule 100160 — Suspicious Windows Service

![Rule 100160](screenshots/04-rule-100160-suspicious-service.png)

**Custom Rule 100160 — Detection of Windows services created from suspicious user-writable paths.**

The screenshot shows controlled-lab hits on `Windows-Endpoint`, with Rule ID `100160` and severity Level 12.

## 5. Threat Hunting

![Threat Hunting](screenshots/05-threat-hunting.png)

**Threat Hunting — Centralized Windows and Linux security events collected and investigated through Wazuh.**

This view demonstrates day-to-day visibility across both monitored endpoints and shows that the project includes investigation workflows, not only custom alerts.

## 6. Backup and Recovery Evidence

See [`evidence/06-backup-recovery.txt`](evidence/06-backup-recovery.txt).

**Backup Validation — Automated encrypted backup completed successfully and the Wazuh Manager archive passed integrity verification.**
