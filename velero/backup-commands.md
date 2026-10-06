# Backup Commands

## Pre-Backup Validation

### Check All Resources

```bash
kubectl get all -n velero-lab
```

### Check the PersistentVolumeClaim

```bash
kubectl get pvc -n velero-lab
```

### Check ConfigMap and Secret

```bash
kubectl get configmap,secret -n velero-lab
```

### Verify Persistent Data

```bash
kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt
```

Expected output:

```text
Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06
```

---

## Create Backup

```bash
velero backup create velero-lab-backup-01 --include-namespaces velero-lab
```

---

## Check Backup

### Get Backup Status

```bash
velero backup get velero-lab-backup-01
```

### Describe Backup

```bash
velero backup describe velero-lab-backup-01 --details
```

---

## Check Backup Logs

```bash
velero backup logs velero-lab-backup-01
```

---

## Persistent Volume Backup Verification

The backup was checked for **Pod Volume Backup (PVB)** using **Kopia**.

The `persistent-data` volume backup completed successfully.

The backup was **not considered successful based only on the `Completed` phase**. Backup details, logs, and Pod Volume Backup status were checked to confirm that the persistent application data was included.

### Validation

The following were confirmed:

- Kubernetes resources were included in the backup.
- The PVC was included.
- Pod Volume Backup for `persistent-data` completed successfully.
- Persistent application data was protected.
- The backup completed without errors.
