# Home SOC - Detection Engineering

## Retired Custom Detections - Built-in Coverage Validation

Three custom detections were originally created, tested, and validated during the project:

### Rule 100100 - Windows Multiple Failed Logons

- Original custom trigger: 3 failed logons within 120 seconds
- Original base rule: `60122`
- Severity: Level `10`
- MITRE ATT&CK: `T1110 - Brute Force`
- Status: **Retired**

The Wazuh 4.14.7 ruleset was later reviewed directly.

Built-in Rule `60204` provides overlapping Windows failed-logon correlation coverage at Level 10. Its logic is not identical to the retired custom rule: `100100` triggered after 3 matches of Rule `60122` within 120 seconds, while `60204` uses `frequency=8`, `timeframe=240`, and correlates by `win.eventdata.ipAddress`.

Engineering decision:

The custom rule was retired because the built-in rule provides sufficient overlapping brute-force coverage for this Home SOC, avoiding parallel alerts. This is an engineering trade-off, not exact rule equivalence.

---

### Rule 100110 - Linux Failed sudo

- Original custom trigger: single failed sudo authentication
- Original base rule: `5401`
- Severity: Level `8`
- MITRE ATT&CK: `T1548.003`
- Status: **Retired**

Wazuh 4.14.7 already provides:

- Rule `5401` - failed sudo attempt, Level 5
- Rule `5404` - three failed sudo attempts, Level 10

Engineering decision:

The custom rule was removed because escalating a single incorrect password attempt to Level 8 added unnecessary severity and duplicated native coverage.

---

### Rule 100120 - SSH Invalid User Attempts

- Original custom trigger: 5 attempts using non-existent users from the same IP within 120 seconds
- Original base rule: `5710`
- Severity: Level `10`
- MITRE ATT&CK: `T1110.001`
- Status: **Retired**

Wazuh 4.14.7 includes Rule `5712`, which performs correlation on repeated SSH attempts using non-existent users from the same source IP within 120 seconds.

Engineering decision:

The custom rule was removed to prevent overlapping brute-force alerts.

---

## False Positive Investigation - Docker Promiscuous Mode

### Original Alert

- Wazuh Rule: `80710`
- Severity: Level 10
- Description: `Auditd: Device enables promiscuous mode.`
- Endpoint: `Ubuntu-Server`

### Investigation

Repeated alerts were observed on interfaces beginning with `veth`.

The Auditd events showed:

- Interface: `veth*`
- Process: `dockerd`
- Executable: `/usr/bin/dockerd`
- UID: `root`

The events occurred during normal Docker container/network lifecycle activity.

### Tuning Rule

Custom Rule: `100140`

The rule suppresses the event only when all of the following match:

- Original Wazuh Rule `80710`
- Interface name begins with `veth`
- Process is `dockerd`
- Executable is `/usr/bin/dockerd`

Resulting severity: Level 0.

### Safety Validation

Two tests were performed:

1. Real Docker event:
   - Result: Rule `100140`
   - Level: `0`
   - Expected benign activity suppressed successfully.

2. Same event modified to use a non-Docker process:
   - Result: Rule `80710`
   - Level: `10`
   - Security alert remained active.

### Conclusion

The tuning removes known benign Docker networking noise without globally disabling promiscuous-mode detection.

Status: **Validated**

---

## Built-in Detection Validation - Local Administrators Group

### Test Scenario

A temporary local Windows account named `SOC-Test-Admin` was:

1. Created.
2. Added to the local Administrators group.
3. Removed from the Administrators group.
4. Deleted.

### Results

- Event ID `4720`
  - Wazuh Rule: `60109`
  - Level: `8`
  - Description: `User account enabled or created`
  - MITRE: `T1098 - Account Manipulation`

- Event ID `4732`
  - Wazuh Rule: `60154`
  - Level: `12`
  - Description: `Administrators Group Changed`
  - Target SID: `S-1-5-32-544`

- Event ID `4733`
  - Wazuh Rule: `60154`
  - Level: `12`
  - Administrators group membership removal detected.

- Event ID `4726`
  - Wazuh Rule: `60111`
  - Level: `8`
  - User account deletion detected.

### Engineering Decision

No custom rule was created because Wazuh already provides strong native coverage for changes to the local Administrators group.

Creating another rule would duplicate existing detection coverage and generate unnecessary alerts.

Status: **Validated**

---

## Custom Detection 100150 - PowerShell Scheduled Task

### Purpose

Detect creation of a Windows Scheduled Task that executes PowerShell.

### Base Detection

- Windows Event ID: `4698`
- Wazuh Base Rule: `60228`
- Base Severity: Level `4`
- Description: `A scheduled task was created`

### Custom Detection

- Rule ID: `100150`
- Severity: Level `10`
- Description: `Windows: Scheduled task created to execute PowerShell.`

The rule checks the `win.eventdata.taskContent` field specifically inside the Scheduled Task `<Command>` element for:

- `powershell`
- `powershell.exe`
- `pwsh`
- `pwsh.exe`

This prevents the rule from triggering when the word PowerShell appears only in unrelated task metadata such as the description.

### MITRE ATT&CK

- `T1053.005` - Scheduled Task/Job: Scheduled Task
- `T1059.001` - Command and Scripting Interpreter: PowerShell

### Positive Test

Created temporary task:

`SOC-Test-PS-Hardened2`

Action:

`powershell.exe -NoProfile -Command exit`

Result:

- Rule: `100150`
- Level: `10`
- Detection successful.

### Negative / Safety Test

Created temporary task:

`SOC-Test-CMD-Hardened2`

Action:

`cmd.exe /c exit`

The task description intentionally contained the word `PowerShell`.

Result:

- Rule: `60228`
- Level: `4`
- Custom Rule `100150` did not trigger.

### Audit Requirement

Windows audit subcategory:

`Other Object Access Events`

was configured for successful events so Windows Event ID `4698` is generated.

### Engineering Decision

The built-in Wazuh rule detects all newly created Scheduled Tasks at Level 4.

The custom rule raises severity only when the task specifically executes PowerShell, reducing unnecessary alert escalation while providing stronger detection for a higher-risk persistence technique.

Status: **Validated**

---

## Custom Detection 100160 - Suspicious Windows Service Path

### Purpose

Detect creation of a new Windows service whose executable is located in a suspicious or user-writable directory.

### Base Detection

- Windows Event ID: `7045`
- Wazuh Base Rule: `61138`
- Base Severity: Level `5`
- Description: `New Windows Service Created`

### Custom Detection

- Rule ID: `100160`
- Severity: Level `12`
- Description: `Windows: New service created from a suspicious user-writable path.`

Suspicious locations include:

- `C:\Users\Public\`
- `C:\Users\<user>\AppData\Local\Temp\`
- `C:\Windows\Temp\`

### MITRE ATT&CK

- `T1543.003` - Create or Modify System Process: Windows Service

### Positive Test

Temporary service:

`SOC-Test-Service-Quoted`

Image path:

`"C:\Users\Public\SOC-Test-Service.exe"`

Result:

- Rule: `100160`
- Level: `12`
- Detection successful.

The service was not started and was deleted after the test.

### Negative / Safety Test

Temporary service:

`SOC-Test-Service-Normal`

Image path:

`C:\Windows\System32\cmd.exe`

Result:

- Rule: `61138`
- Level: `5`
- Custom Rule `100160` did not trigger.

### Engineering Decision

Wazuh already detects all new Windows services with Rule `61138`.

A custom rule was added only for services whose executable path is located in higher-risk user-writable locations.

Status: **Validated**
