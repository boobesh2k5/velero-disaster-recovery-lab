# Backup Commands

## Pre-Backup Validation

Check all resources:

kubectl get all -n velero-lab

Check the PersistentVolumeClaim:

kubectl get pvc -n velero-lab

Check ConfigMap and Secret:

kubectl get configmap,secret -n velero-lab

Verify the persistent data:

kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt

## Create Backup

velero backup create velero-lab-backup-01 --include-namespaces velero-lab

## Check Backup

velero backup get velero-lab-backup-01

velero backup describe velero-lab-backup-01 --details

## Persistent Volume Backup Verification

The backup was checked for Pod Volume Backup using Kopia. The persistent-data volume backup completed successfully.

The backup was not considered successful based only on the Completed phase. The backup details and Pod Volume Backup status were checked to confirm that persistent application data was included.
