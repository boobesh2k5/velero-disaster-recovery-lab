# Scheduled Backup and Retention

## Daily Backup Schedule

A daily Velero backup schedule was created for the Kubernetes application.

Command used:

velero schedule create velero-lab-daily --schedule="0 2 * * *" --ttl 168h

## Schedule Configuration

- Schedule: Daily at 02:00
- Retention: 168 hours (7 days)
- Schedule name: velero-lab-daily
- Status: Enabled
- Paused: False

## Verification

The schedule was verified using:

velero schedule get

The schedule was confirmed as Enabled with a 168-hour backup TTL.
