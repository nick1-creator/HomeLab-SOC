# Home SOC - Wazuh Recovery Runbook

## Scope

This runbook covers recovery of the Home SOC Wazuh deployment.

- Wazuh 4.14.7
- Docker single-node deployment

## Protected Components

### Wazuh Manager

Automated state backups:

`/srv/homelab/backups/wazuh/`

Includes:

- `/var/ossec/etc`
- Agent keys
- Custom rules
- API configuration
- `/var/ossec/queue/db`
- `/var/ossec/queue/rids`
- Wazuh logs
- Filebeat configuration
- Additional Manager state

For Manager-state capture, automatic container restart is temporarily suppressed, Filebeat and Wazuh are stopped, and the Manager container is then fully stopped. Required state is copied only while the container is verified stopped. The Manager and Filebeat are subsequently returned to a healthy state, Indexer connectivity is checked, and the normal restart policy is restored.

### Wazuh Dashboard

Saved Objects exports:

`/srv/homelab/backups/wazuh-dashboard/`

Includes:

- Dashboard
- Visualizations
- Index pattern

### Wazuh Indexer Configuration

Recovery exports:

`/srv/homelab/backups/wazuh-recovery/`

Includes:

- `wazuh-alert-retention-90d.json`
- `soc_readonly-role.json`
- `soc_readonly-mapping.json`

### HomeLab Backup

The Wazuh project, Manager backups, Dashboard exports and recovery files are included in the encrypted HomeLab Restic backup.

A separate independent/offline Restic repository was created on removable USB media. The repository data is encrypted by Restic. A complete repository `check --read-data` and a selected-file restore on the administration laptop have passed.

The USB is stored disconnected when not in use. It is currently stored in the same home as the server, so it is not described as physically off-site.

## Recovery Order

1. Restore the HomeLab host and Docker environment.
2. Restore `/srv/homelab/stacks/wazuh`.
3. Deploy the same compatible Wazuh version.
4. Restore Wazuh Manager state from the latest validated archive.
5. Start Wazuh and verify Manager services.
6. Verify registered Agents reconnect.
7. Restore/recreate Indexer retention and read-only role configuration if required.
8. Import the latest Dashboard Saved Objects export.
9. Recreate `soc-readonly` if required.
10. Verify Dashboard, Threat Hunting and Agents.
11. Verify TCP 1514 is available only through the intended Tailscale path.
12. Verify TCP 1515, 9200 and 55000 are not exposed to the host.

## Validation After Recovery

Verify:

- Wazuh Manager services are running.
- Indexer health is Green, or Yellow only because replica shards cannot be assigned in the intentional single-node topology; no primary shard is unassigned.
- Known Agents reconnect.
- Custom rules are present.
- Threat Hunting returns alerts.
- Dashboard loads correctly.
- `soc-readonly` can view alerts but cannot access Security administration.
- `wazuh-alert-retention-90d` is applied to alert indices.
- Backup automation is active.

## Backup Validation Already Performed

The Wazuh Manager V2 archive has been tested using:

- gzip integrity validation
- archive extraction
- required-file checks
- SQLite `PRAGMA integrity_check`
- restored `client.keys` comparison
- restored `local_rules.xml` comparison

This is an archive extraction and database-integrity restore test.

It is NOT a full functional disaster-recovery restore onto a fresh Wazuh deployment.

## Indexer Recovery Status

Historical Wazuh alert indices are protected with automated OpenSearch snapshots created through the Snapshot API.

The current snapshot policy retains the latest 14 snapshots, and a selected-index restore test has passed.

The recovery design also protects Manager state, configuration, Dashboard objects and Indexer configuration.

This is not a claim of complete disaster recovery: a full clean-system restore of the complete SOC has not yet been performed. Independent/offline USB recovery has been validated, but a physically off-site copy is not currently maintained.

## Secrets

Passwords and other secrets are NOT stored in this document.

They remain in the password manager and protected configuration locations.
