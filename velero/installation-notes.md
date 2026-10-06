# Velero Installation Notes

## Velero Installation

Velero was installed in the velero namespace using the AWS/S3-compatible plugin with MinIO as the external S3-compatible object storage.

MinIO was kept outside the Kubernetes cluster so that the backup data would remain available even after the Kubernetes cluster was deleted.

## Backup Storage

- Storage: MinIO
- Bucket: velero
- MinIO endpoint: http://host.k3d.internal:9000
- Kubernetes namespace: velero
- Velero provider/plugin: AWS S3-compatible plugin
- Volume snapshots: Disabled
- Node Agent: Enabled

## Node Agent

Velero Node Agent was enabled to back up persistent application data from the PVC using file-system backup.

## Verification

The BackupStorageLocation was verified using: velero backup-location get

The storage location was available and usable before performing the backup and restore tests.

## Security

MinIO credentials were stored locally and were not committed to the Git repository.
