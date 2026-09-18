# Wazuh SOC - Simulated Incident Reports

These reports document controlled security simulations performed in the Home SOC lab.

The events were intentionally generated for detection validation and SOC investigation practice.
They do not represent real compromises.

---

# Incident 001 - Multiple Failed Windows Logons

## Classification

- Type: Authentication / Credential Access
- Historical Detection Rule: `100100` (retired after built-in coverage review)
- Current Built-in Coverage: `60204 - Multiple Windows Logon Failures`
- Severity: Level `10`
- Endpoint: `Windows-Endpoint`
- Windows Event ID: `4625`
- MITRE ATT&CK: `T1110 - Brute Force`

## Alert

Wazuh detected multiple failed Windows logon attempts on the same endpoint within the configured detection window.

## Evidence

- Event ID: `4625`
- Logon Type: `2`
- Failure reason: Unknown user name or bad password
- Status: `0xC000006D`
- Sub-status: `0xC000006A`
- Process: `C:\Windows\System32\svchost.exe`
- Source address: `::1`
- Authentication package: `Negotiate`

## Investigation

The alert was reviewed to determine:

1. Which endpoint generated the failures.
2. Whether the source was local or remote.
3. Which authentication process was involved.
4. Whether a successful login followed the failures.
5. Whether similar failures occurred elsewhere.

The source address was the local IPv6 loopback address (`::1`), indicating that the tested authentication attempts originated locally on the endpoint.

## Conclusion

Controlled lab simulation.

The custom correlation rule successfully raised repeated Windows authentication failures from individual lower-severity events to a Level 10 alert.

During a later Wazuh 4.14.7 ruleset review, built-in Rule `60204` was confirmed to provide overlapping, but not identical, correlation coverage. The retired Rule `100100` used 3 matches of Rule `60122` within 120 seconds; built-in Rule `60204` uses 8 authentication failures within 240 seconds from the same source IP. Rule `100100` was retired because the built-in coverage was judged sufficient for this Home SOC and to avoid parallel alerts.

No compromise occurred.

## Recommended Response in a Real Environment

- Identify the affected account.
- Confirm whether the activity was expected.
- Search for additional Event ID `4625` events.
- Check for successful Event ID `4624` logons after the failures.
- Review the source IP and originating process.
- Reset credentials or lock the account if compromise is suspected.
- Escalate or isolate the endpoint if additional malicious indicators are found.

Status: **Closed - Lab Simulation**

---

# Incident 002 - PowerShell Scheduled Task Creation

## Classification

- Type: Persistence / Execution
- Detection Rule: `100150`
- Severity: Level `10`
- Endpoint: `Windows-Endpoint`
- Windows Event ID: `4698`
- MITRE ATT&CK:
  - `T1053.005 - Scheduled Task`
  - `T1059.001 - PowerShell`

## Alert

Wazuh detected creation of a Windows Scheduled Task configured to execute PowerShell.

## Evidence

Task name:

`SOC-Test-PS-Task`

Task action:

`powershell.exe -NoProfile -Command exit`

Creating user:

`NICK_COMPUTER\nicks`

The Windows Security event contained the full Scheduled Task XML configuration.

## Investigation

The analyst should determine:

1. Who created the task.
2. What executable or script the task launches.
3. When the task is configured to execute.
4. Whether the PowerShell command is expected.
5. Whether related PowerShell, process creation, or persistence events exist.

The task was created as part of a controlled SOC detection test.

A second Scheduled Task using `cmd.exe` was also tested and correctly remained at the normal Wazuh Rule `60228`, Level 4.

This confirmed that the custom rule increases severity only when PowerShell is present.

## Conclusion

Controlled lab simulation.

Detection `100150` correctly identified a higher-risk Scheduled Task and generated a Level 10 alert without escalating an ordinary CMD Scheduled Task.

No compromise occurred.

## Recommended Response in a Real Environment

- Review the complete task XML.
- Identify the creating account.
- Examine the PowerShell command and referenced files.
- Review PowerShell and Sysmon telemetry around the same timestamp.
- Check for additional Scheduled Tasks.
- Verify file hashes and digital signatures where applicable.
- Disable or remove unauthorized tasks.
- Isolate the endpoint if malicious persistence is confirmed.

Status: **Closed - Lab Simulation**

---

# Incident 003 - Suspicious Windows Service Creation

## Classification

- Type: Persistence / Privilege Escalation
- Detection Rule: `100160`
- Severity: Level `12`
- Endpoint: `Windows-Endpoint`
- Windows Event ID: `7045`
- MITRE ATT&CK:
  - `T1543.003 - Windows Service`

## Alert

Wazuh detected creation of a Windows service whose executable path was located in a user-writable directory.

## Evidence

Service name:

`SOC-Test-Suspicious`

Image path:

`C:\Users\Public\SOC-Test-Service.exe`

Service account:

`LocalSystem`

Base Wazuh Rule:

`61138 - New Windows Service Created`

Custom Rule:

`100160 - Windows: New service created from a suspicious user-writable path.`

## Investigation

A service executing from `C:\Users\Public` deserves additional investigation because the directory is user-writable and is not a normal location for most system services.

The analyst should investigate:

1. Who created the service.
2. Whether the referenced executable exists.
3. File hash and digital signature.
4. File creation and modification timestamps.
5. Related process creation telemetry.
6. Other services using unusual executable paths.
7. Whether the service executed with elevated privileges.

In the simulation, the service was configured for `LocalSystem`, increasing the security significance of the event.

The service was not started and was deleted after the test.

A safety test using:

`C:\Windows\System32\cmd.exe`

correctly remained at the base Wazuh Rule `61138`, Level 5.

## Conclusion

Controlled lab simulation.

Detection `100160` successfully distinguished a service created from a suspicious user-writable location from a service using a normal Windows system path.

No compromise occurred.

## Recommended Response in a Real Environment

- Disable the suspicious service.
- Preserve service configuration and relevant logs.
- Verify whether the executable exists.
- Calculate and investigate its hash.
- Check its digital signature.
- Review process and file creation telemetry.
- Hunt for similar service paths across other endpoints.
- Remove the service and malicious file if confirmed.
- Isolate the host if additional malicious indicators are present.

Status: **Closed - Lab Simulation**

---


# Incident 004 - Multiple SSH Invalid-User Attempts

## Classification

- Type: Authentication / Credential Access
- Historical Detection Rule: `100120` (retired after built-in coverage review)
- Current Built-in Coverage: `5712 - SSH authentication attempts with non-existent users`
- Severity: Level `10`
- Endpoint: `Ubuntu-Server`
- Log Source: `journald / sshd`
- MITRE ATT&CK: `T1110.001 - Password Guessing`

## Alert

Wazuh detected multiple SSH login attempts using non-existent usernames from the same source IP.

The activity was generated intentionally as part of a controlled Home SOC detection test.

## Evidence

Agent:

`Ubuntu-Server`

Source IP:

`LAB-SOURCE-IP`

Test usernames:

- `socfake1`
- `socfake2`
- `socfake3`
- `socfake4`
- `socfake5`

Observed sequence:

- Five invalid-user SSH attempts occurred within approximately ten seconds.
- Individual attempts were detected by built-in Rule `5710`, Level 5.
- Historical custom Rule `100120` generated a Level 10 correlation alert during the sequence.
- The stored Rule `100120` alert occurred at `2026-09-01T12:16:11.693Z`.
- The alert was decoded by `sshd` from `journald`.

Example evidence:

`Invalid user socfake3 from LAB-SOURCE-IP`

## Investigation

The analyst reviewed:

1. The affected Linux endpoint.
2. The source IP of the authentication attempts.
3. The usernames used during the attempts.
4. The number and timing of invalid-user events.
5. The base Wazuh SSH detections.
6. The correlation alert generated during the sequence.
7. Whether the source was known and expected.

The source IP belonged to the controlled lab environment and the usernames were deliberately generated test accounts.

No successful authentication was observed as part of this simulation.

## Detection Engineering Review

Historical custom Rule `100120` successfully demonstrated correlation of repeated invalid-user SSH attempts.

During a later review of the Wazuh 4.14.7 built-in ruleset, Rule `5712` was confirmed to provide overlapping SSH brute-force / non-existent-user correlation coverage.

Custom Rule `100120` was therefore retired to avoid duplicate detections.

The historical alert is retained as evidence of the original detection-engineering test.

## Conclusion

Controlled lab simulation.

Five invalid-user SSH attempts from the same source were observed and investigated.

The custom correlation detection worked as intended during the original test, and was later retired after built-in Wazuh coverage was validated.

No compromise occurred.

## Recommended Response in a Real Environment

- Identify the source IP and determine whether it is expected.
- Review the usernames targeted by the authentication attempts.
- Search for additional SSH authentication failures from the same source.
- Check for successful SSH logins following the failures.
- Review authentication activity on other Linux endpoints.
- Block or restrict the source if the activity is unauthorized.
- Review account and SSH hardening controls.
- Escalate or isolate the endpoint if successful compromise is suspected.

Status: **Closed - Lab Simulation**

---

# Summary

| Incident | Detection | Severity | Result |
|---|---|---:|---|
| Multiple Windows failed logons | `100100` retired / Built-in `60204` | 10 | Validated |
| PowerShell Scheduled Task | `100150` | 10 | Validated |
| Service from suspicious path | `100160` | 12 | Validated |
| Multiple SSH invalid-user attempts | `100120` retired / Built-in `5712` | 10 | Validated |

All four incidents were controlled Home SOC simulations performed for detection engineering and incident investigation practice.
