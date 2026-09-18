# Home SOC - Stability & Final Operational Validation

## Purpose

This document records the final operational validation of Home SOC / SIEM v1.

The original project used a longer stability-observation plan. During that period, additional audit findings and intentional hardening changes were made, so final acceptance was based on the resulting evidence rather than pretending that no changes occurred during the original calendar window.

## Final Acceptance Basis

Final operational sign-off was completed on **2026-09-10**.

The acceptance basis was:

- cumulative multi-day runtime evidence
- clean post-change integrity validation
- two consecutive automatic backups after the final Wazuh backup hardening
- successful Wazuh Manager state quiescing
- successful Indexer snapshots
- successful Restic snapshots
- successful service recovery after backup
- final host/container/storage health validation

## Wazuh Backup Hardening

Audit follow-up identified a race in the previous Manager-state helper: a failed status probe could be mistaken for a stopped service, while Docker `restart: always` could restart the Manager container during the intended quiet window.

The final hardened procedure:

1. temporarily suppresses automatic Manager-container restart
2. gracefully stops Filebeat
3. stops Wazuh services
4. fully stops the Manager container
5. verifies the container is stopped
6. copies required Manager/Filebeat state from the stopped container
7. validates that the same container remained stopped during the copy
8. starts the Manager container
9. explicitly returns Filebeat to service
10. verifies Wazuh health and Filebeat-to-Indexer connectivity
11. restores the normal `restart: always` policy

The final helper was live-tested successfully before permanent installation.

Final production helper SHA256:

`badbe6faea5004083d8ae28372eabf5646ba095747a66d0b1db63a3614201832`


## Automatic Backup Evidence

### 2026-09-09

- backup service result: `success`
- Wazuh Manager container stopped: verified
- Wazuh writers: quiesced
- state copy completed while container was stopped
- Filebeat → Indexer: PASS
- Wazuh Manager state backup: OK
- Indexer snapshot: SUCCESS
- failed shards: 0
- Restic snapshot: `95e3f6b9`
- backup vault: closed and verified
- success marker updated before notification
- final backup result: SUCCESS

### 2026-09-10

- backup service result: `success`
- Wazuh Manager container stopped: verified
- Wazuh writers: quiesced
- state copy completed while container was stopped
- Filebeat → Indexer: PASS
- Wazuh Manager state backup: OK
- Indexer snapshot: SUCCESS
- failed shards: 0
- Restic snapshot: `3e405414`
- backup vault: closed and verified
- success marker updated before notification
- final backup result: SUCCESS

## Continued Scheduled Backup Operation

Backup-success notifications continued after final sign-off.

Observed scheduled backup success notifications are present on every date from **2026-09-06 through 2026-09-18**.

Latest confirmed notification:

`2026-09-18 03:31:56 IDT`

The 2026-09-09 and 2026-09-10 entries above contain the deeper component-level validation. Later success notifications are treated as continuing operational evidence rather than as a repetition of every internal validation step.

## Final Health Check — 2026-09-10

Final read-only health validation reported:

- system state: running
- failed systemd units: 0
- AIDE: clean, 583 entries, exit code 0
- Docker: 29/29 expected containers running
- Wazuh Manager: running
- Filebeat: running
- Wazuh Manager restart policy: `always`
- backup filesystem: closed
- backup LUKS mapper: closed
- root filesystem usage: approximately 8%
- HomeLab data filesystem usage: approximately 4%

## Recovery Validation

Independent/offline USB recovery was also validated separately:

- source server snapshot: `a8c66921`
- independent USB snapshot: `4a71f1a9`
- recovery point: `2026-09-08 03:31:42 IDT`
- encrypted Restic repository on removable USB media
- full repository `check --read-data`
- selected-file restore on the administration laptop
- SHA256 equality for the restored test files
- USB safely removed and stored disconnected

The USB remains in the same home as the server and is therefore not physically off-site.

## Final Result

**Home SOC / SIEM v1 — COMPLETE**

The project is now in normal maintenance mode.

Remaining items are documented maintenance or scope limitations rather than unresolved v1 blockers:

- no full clean-system Disaster Recovery claim
- no physically off-site recovery copy
- F03/F04 post-reboot runtime validation deferred to a future planned reboot
- Secure Boot dbx/KEK update deferred to a controlled maintenance window
