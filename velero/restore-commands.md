# Restore Commands

## Disaster 1 - Namespace Deletion

### Delete the Application Namespace

```bash
kubectl delete namespace velero-lab
```

### Verify That the Namespace Was Deleted

```bash
kubectl get namespace velero-lab
```

The namespace was successfully deleted before starting the restore.

### Restore from the Backup

```bash
velero restore create velero-lab-restore-01 --from-backup velero-lab-backup-01
```

### Check Restore Status

```bash
velero restore get velero-lab-restore-01
```

The restore completed successfully with no errors.

### Verify Restored Resources

```bash
kubectl get all -n velero-lab
```

```bash
kubectl get pvc -n velero-lab
```

### Verify Persistent Data

```bash
kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt
```

Expected output:

```text
Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06
```

The original persistent data was successfully recovered after the namespace deletion.

---

## Disaster 2 - Complete Cluster Loss

The Kubernetes cluster was deleted while the external MinIO storage remained running.

The two-node Kubernetes cluster was recreated and Velero was reinstalled using the same MinIO bucket and configuration.

### Verify Previous Backup

```bash
velero backup get
```

The previous backup was successfully discovered:

```text
velero-lab-backup-01
```

### Restore the Previous Backup

```bash
velero restore create velero-lab-restore-02 --from-backup velero-lab-backup-01
```

### Check Restore Status

```bash
velero restore get velero-lab-restore-02
```

The restore completed successfully with no errors.

### Verify Restored Resources

```bash
kubectl get all -n velero-lab
```

```bash
kubectl get pvc -n velero-lab
```

### Verify Persistent Data

```bash
kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt
```

Expected output:

```text
Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06
```

The original persistent data was successfully recovered after the complete Kubernetes cluster was deleted and recreated.

---

## Restore Validation Summary

| Disaster Scenario | Restore | Persistent Data | Result |
|---|---|---|---|
| Namespace deletion | `velero-lab-restore-01` | Successfully recovered | ✅ PASSED |
| Complete cluster loss | `velero-lab-restore-02` | Successfully recovered | ✅ PASSED |
