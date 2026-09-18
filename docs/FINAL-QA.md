# Home SOC - Final QA Report

Date: 2026-09-01

## Overall Result

Home SOC / SIEM v1 pre-stability QA:

PASS

The environment is ready for the seven-day stability validation period.

## Final Sign-Off Update — 2026-09-10

**Overall Result: PASS / COMPLETE**

The pre-stability QA below is retained as the historical baseline. Final operational validation was completed after subsequent hardening and recovery work.

Key final evidence:

- Wazuh Manager backup race condition A3-W01 was identified, reproduced, corrected and live-tested
- the final helper prevents automatic Manager restart during state capture
- the Manager container is fully stopped before state copy
- probe failures are no longer accepted as proof of a stopped state
- Filebeat is explicitly returned to service and verified against the Indexer
- AIDE was re-baselined only after the intentional helper change was reviewed; final AIDE check was clean with 583 entries
- scheduled backup on 2026-09-09 completed successfully and created Restic snapshot `95e3f6b9`
- scheduled backup on 2026-09-10 completed successfully and created Restic snapshot `3e405414`
- both scheduled runs reported Wazuh writers quiesced, Manager state backup successful, Indexer snapshot success with zero failed shards, backup vault closure and final backup success
- final health validation on 2026-09-10 showed 0 failed systemd units, AIDE clean, 29/29 containers running, Wazuh/Filebeat healthy, backup mount closed and backup LUKS mapper closed
- independent/offline USB Restic recovery was validated with full repository read-data checking and selected-file restoration

The environment is now considered complete for the defined v1 scope and has moved to normal maintenance mode.

---

## Deployment

Wazuh version:

`4.14.7`

Architecture:

- Single-node Wazuh deployment
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Docker Compose
- Private Tailscale connectivity
- Nginx Proxy Manager for Dashboard access

---

## Container Validation

Validated:

- Wazuh Manager running
- Wazuh Indexer running
- Wazuh Dashboard running

No unexplained restart loop was observed during the final QA.

---

## Agent Validation

Validated Agents:

- `Windows-Endpoint`
- `Ubuntu-Server`

Both Agents were Active during the final QA.

---

## Network Exposure

Host-published Wazuh port:

- TCP 1514

Not published to the host:

- TCP 1515
- TCP 9200
- TCP 55000
- UDP 514

TCP 1514 is restricted to the private Tailscale network.

TCP 1515 remains closed during normal operation and is intended to be opened only temporarily for controlled Agent enrollment.

---

## Docker Security

Validated:

- No Docker socket mount in Wazuh Manager
- No Docker socket mount in Wazuh Indexer
- No Docker socket mount in Wazuh Dashboard

---

## Indexer Health

Cluster topology:

- One Indexer node
- All primary shards started

Cluster status:

`Yellow`

This status is expected for the intentional single-node deployment.

The unassigned shards observed during QA were replica shards associated with the intentional single-node topology and system indices.

The exact number of replica shards may change as system indices are created or rotated.

No primary shard was unassigned.

---

## Alert Retention

ISM policy:

`wazuh-alert-retention-90d`

The current Wazuh alert index was confirmed as managed by this policy.

Retention target:

`90 days`

Automatic policy inheritance was later validated on a newly created daily Wazuh alert index.

---

## Active Custom Rules

Active rule IDs:

- `100140`
- `100150`
- `100160`

Purpose:

- `100140` - narrow Docker auditd false-positive tuning
- `100150` - suspicious PowerShell Scheduled Task detection
- `100160` - suspicious Windows Service path detection

Retired duplicate Custom Rules are documented separately.

---

## Incident Documentation

Four controlled lab incidents are documented.

The incident documentation was sanitized for portfolio use.

Private Tailscale source addresses were replaced with:

`LAB-SOURCE-IP`

---

## Read-Only SOC Access

The dedicated account:

`soc-readonly`

was previously validated for least-privilege SOC access.

It can view operational security information but cannot perform administrative Wazuh security actions.

---

## Backup Validation

Final manual Wazuh Manager V2 backup:

`wazuh-manager-state-20260901-222509.tar.gz`

Approximate size:

`13 MB`

Validation completed:

- Backup helper completed successfully
- Wazuh important services returned healthy
- Both Agents returned Active
- gzip integrity passed
- Important Manager state exists in the archive
- Ownership and permissions are preserved
- Previous extraction test passed
- SQLite database integrity checks passed

Backup helper runtime during final validation:

`17 seconds`

---

## Backup Automation

Daily HomeLab backup timer:

`03:30`

Post-hardening automatic validation:

- **2026-09-09** — service result `success`, Restic snapshot `95e3f6b9`
- **2026-09-10** — service result `success`, Restic snapshot `3e405414`

Continued backup-notification evidence:

- successful scheduled backup notifications are present for every date from **2026-09-06 through 2026-09-18**
- latest confirmed success: **2026-09-18 03:31:56 IDT**
- a separate `TEST ONLY` notification on 2026-09-07 is excluded from the backup evidence; an actual scheduled backup success is also present for that date

The notification history confirms continued top-level backup-job success. Detailed Wazuh Manager, Filebeat, Indexer and backup-vault validation remains based on the post-hardening 2026-09-09 and 2026-09-10 runs.

Both detailed scheduled runs confirmed:

- Wazuh Manager container stopped and writers quiesced before state copy
- Wazuh Manager state archive created successfully
- Filebeat returned online and passed Indexer connectivity testing
- OpenSearch snapshot state `SUCCESS`
- zero failed snapshot shards
- Restic snapshot created
- backup vault closed successfully
- persistent success marker updated before success notification
- final backup result `SUCCESS`

---

## Host Resources

Final QA snapshot:

- RAM: 31 GiB total
- Approximately 8 GiB used
- Approximately 23 GiB available
- Swap usage minimal
- Root filesystem approximately 8% used
- More than 800 GiB available on root storage

No current resource constraint was identified.

---

## Documentation

Current project documentation:

- `README.md`
- `docs/DETECTIONS.md`
- `docs/INCIDENTS.md`
- `docs/RECOVERY.md`
- `docs/STABILITY.md`
- `docs/FINAL-QA.md`

The public documentation safety scan found no obvious:

- Tailscale private IP addresses
- Private key markers
- API password assignments
- Password assignments

---

## Known Residual Security Risk

Security advisory reviewed:

`GHSA-whhr-gxxr-f6pm`

Review date:

`2026-09-01`

The upstream advisory contains inconsistent version metadata.

Its detailed technical description states that Wazuh 4.14.7 remains affected, even though the advisory metadata also lists 4.14.7 as a patched version.

Because of that inconsistency, no speculative upgrade or local source patch was applied during final QA.

Current compensating controls include:

- Wazuh is not publicly exposed
- TCP 1514 is reachable only through intended private Tailscale access
- Only known enrolled Agents are used
- TCP 1515 enrollment is closed during normal operation
- Wazuh API is internal
- Wazuh Indexer is internal
- Dashboard is not publicly exposed
- Active Response is not enabled on production endpoints

The advisory must be reviewed again during the final stability review.

---

## Current Limitations

The v1 project intentionally does not claim:

- High availability
- Multi-node Indexer redundancy
- Full clean-system recovery of all historical Indexer data
- Full clean-system Disaster Recovery
- Independent off-site recovery
- Automated Active Response
- Dedicated attack VM

These remain optional Phase 2 improvements.

---

## Final Sign-Off

Final sign-off for the defined Home SOC / SIEM v1 scope was completed on **2026-09-10**.

`Home SOC / SIEM v1 = COMPLETE`

Documented residual limitations:

- single-node architecture; no high availability
- no full clean-system Disaster Recovery exercise
- no full clean-system restoration of all historical Indexer data
- independent/offline USB recovery is validated, but the media is not physically off-site
- automated Active Response is intentionally not enabled on production endpoints
- F03/F04 post-reboot runtime persistence remains to be confirmed during a future planned reboot
- Secure Boot dbx/KEK maintenance is deferred to a controlled physical maintenance window

These limitations do not invalidate completion of the defined v1 project scope.
