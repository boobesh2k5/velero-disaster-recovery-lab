# Restore Commands

## Disaster 1 - Namespace Deletion

Delete the application namespace:

kubectl delete namespace velero-lab

Verify that the namespace was deleted:

kubectl get namespace velero-lab

Restore from the backup:

velero restore create velero-lab-restore-01 --from-backup velero-lab-backup-01

Check restore status:

velero restore get velero-lab-restore-01

Verify restored resources:

kubectl get all -n velero-lab

kubectl get pvc -n velero-lab

Verify the persistent data:

kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt

## Disaster 2 - Complete Cluster Loss

The Kubernetes cluster was deleted while the external MinIO storage remained running.

Recreate the two-node Kubernetes cluster and reinstall Velero using the same MinIO bucket and configuration.

Verify that the previous backup is available:

velero backup get

Restore the previous backup:

velero restore create velero-lab-restore-02 --from-backup velero-lab-backup-01

Check restore status:

velero restore get velero-lab-restore-02

Verify restored resources:

kubectl get all -n velero-lab

kubectl get pvc -n velero-lab

Verify the persistent data:

kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt

The original persistent data was successfully recovered after the complete cluster was recreated.
