# Kubernetes Backup \& Disaster Recovery Using Velero



## 1. Project Overview



This project implements and validates a Kubernetes backup and disaster recovery workflow using Velero with MinIO as an external S3-compatible backup repository.



The objective was to prove recovery of Kubernetes resources and persistent application data after:



1. Application namespace deletion.

2. Complete Kubernetes cluster deletion and recreation.



---



## 2. Environment

| Component | Details |
|---|---|
| Operating System | Windows |
| Container Runtime | Docker Desktop |
| Kubernetes | k3d / K3s |
| Kubernetes Version | `v1.35.5+k3s1` |
| kubectl | `v1.36.1` |
| k3d | `v5.9.0` |
| Velero CLI | `v1.18.4` |
| Cluster Name | `velero-lab` |
| Control Plane | 1 |
| Worker | 1 |
| Backup Storage | MinIO |
| MinIO Bucket | `velero` |



### Kubernetes Nodes


```text

k3d-velero-lab-server-0

k3d-velero-lab-agent-0

```



Cluster creation:



```powershell

k3d cluster create velero-lab --servers 1 --agents 1

```



Verification:



```powershell

kubectl get nodes -o wide

```



Both nodes were verified as `Ready`.



---



## 4. Architecture

```text
Windows Host
│
├── External MinIO
│   └── Bucket: velero
│
└── k3d Kubernetes Cluster
    │
    ├── Control Plane
    ├── Worker Node
    │
    ├── Velero
    ├── Node Agent
    │
    └── velero-lab Application
        ├── Deployment
        ├── Service
        ├── ConfigMap
        ├── Secret
        └── PVC
            └── /data/important-data.txt
```

---



## 4. External MinIO Configuration



MinIO was used as the external S3-compatible object storage backend for Velero.



```text

Bucket:  velero



API:  http://host.k3d.internal:9000



Console:  http://localhost:9001


Persistent Host Storage: C: elero-minio-data

```



The MinIO data directory was preserved during the complete cluster deletion and recreation test.



MinIO credentials were stored locally and were not committed to Git.



Credential file:



```text

C:\\Velero\\credentials-velero

```



Actual credentials are intentionally not included in this report.



---



## 5. MinIO Setup Challenge



During the initial setup, the expected MinIO container image could not be used successfully in the environment, including an image/license-related issue.



Instead of replacing MinIO with another storage system, MinIO was built from the official source release and run directly on the Windows host.



The binary was created at:



```text

C:\\minio.exe

```



MinIO was then started outside Kubernetes.



The service was verified on:



```text

Port: 9000

Console: 9001

```



The required `velero` bucket was created successfully.



This maintained the assignment requirement that MinIO remain external to Kubernetes.



---



## 6. Velero Installation and Configuration



Velero was installed in:



```text

Namespace: velero

```



MinIO was used as the actual backup storage through its S3-compatible API.



Configuration:



```text

Provider: AWS/S3-compatible

Storage: MinIO

Bucket: velero

Endpoint: http://host.k3d.internal:9000

Node Agent: Enabled

Volume Snapshots: Disabled

```



The AWS/S3-compatible provider was used for S3 API compatibility. The actual backup storage was MinIO, not AWS S3.



BackupStorageLocation was checked using:



```powershell

velero backup-location get

```



Final state:



```text

default    aws    velero    Available    ReadWrite

```



---



## 7. Velero Components



The main components were:



### Velero Server



Namespace:



```text

velero

```



### Node Agent



Node Agent was used for file-system backup of persistent application data using Kopia.



During the final cluster recovery, both Node Agent pods reached:



```text

1/1 Running

```



---



## 8. BackupStorageLocation Authentication Issue



Initially, the BackupStorageLocation was unavailable and reported:



```text

SignatureDoesNotMatch

```



### Root Cause



The credentials configured for Velero did not correctly match the MinIO credentials.



### Resolution



The Velero cloud credentials secret was recreated using the correct MinIO credentials, and the Velero server was restarted.



### Result



The BackupStorageLocation became:



```text

Available

ReadWrite

```



Velero was then able to communicate successfully with MinIO.



---



## 9. Test Application



The test application was deployed in:



```text

Namespace: velero-lab

```



Application image:



```text

nginx:alpine

```



Resources:



```text

Deployment:  velero-lab-app

Service:     velero-lab-service

ConfigMap:   velero-lab-config

Secret:      velero-lab-secret

PVC:         velero-lab-pvc

```



PVC configuration:



```text

Storage: 1Gi

Access Mode: ReadWriteOnce

Mount Path: /data

```



---



## 10. Persistent Data



A unique file was created inside the mounted PVC:



```text

/data/important-data.txt

```



The file contained:



```text

Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06

```



This exact content was used as the persistent-data recovery validation.



The file was verified before creating the backup.



---



## 11. Pre-Backup Validation



Before creating the backup, the application resources were checked.



```powershell

kubectl get all -n velero-lab

```



```powershell

kubectl get pvc -n velero-lab

```



```powershell

kubectl get configmap,secret -n velero-lab

```



Persistent data was verified using:



```powershell

kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt

```



Expected result:



```text

Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06

```



---



## 12. Initial Persistent Volume Backup Issue



The first Velero backup reached the `Completed` phase, but the persistent volume data was not included.



### Root Cause



The PVC volume had not been opted into Velero Node Agent file-system backup.



The backup logs showed that the volume was skipped because of the volume opt-in/opt-out configuration.



### Resolution



The Deployment was patched with:



```text

backup.velero.io/backup-volumes: persistent-data

```



Command used:



```powershell

kubectl -n velero-lab patch deployment velero-lab-app --type=strategic -p '{"spec":{"template":{"metadata":{"annotations":{"backup.velero.io/backup-volumes":"persistent-data"}}}}}'

```



A new application pod was created with the annotation.



The backup was then recreated.



The persistent volume was successfully included through Pod Volume Backup using Kopia.



---



## 13. Final Backup



The final backup was:



```text

velero-lab-backup-01

```



The backup contained:



```text

34/34 items

```



The `persistent-data` Pod Volume Backup completed successfully using Kopia.



Backup details were verified using:



```powershell

velero backup describe velero-lab-backup-01 --details

```



Backup logs were checked when troubleshooting:



```powershell

velero backup logs velero-lab-backup-01

```



The final backup was considered valid because both Kubernetes resources and persistent application data were protected.



---



## 14. Disaster Recovery Test 1 — Namespace Deletion



The first disaster scenario simulated application-level failure.



The namespace was deleted:



```powershell

kubectl delete namespace velero-lab

```



The application resources were removed.



### Restore



The namespace was restored using:



```powershell

velero restore create velero-lab-restore-01 --from-backup velero-lab-backup-01

```



Restore status:



```text

Completed

0 errors

1 warning

```



Resources were verified:



```powershell

kubectl get all -n velero-lab

```



PVC was verified:



```powershell

kubectl get pvc -n velero-lab

```



Persistent data was verified:



```powershell

kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt

```



Result:



```text

Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06

```



The content exactly matched the original data.



**Result: PASSED\*\*



---



## 15. Disaster Recovery Test 2 — Complete Cluster Loss



The second disaster scenario simulated complete Kubernetes cluster failure.



Before deleting the cluster:



```text

MinIO: Running

MinIO data: Preserved

Bucket: velero

```



The original Kubernetes cluster was deleted:



```powershell

k3d cluster delete velero-lab

```



MinIO remained outside the cluster.



MinIO availability was verified:



```powershell

Test-NetConnection localhost -Port 9000

```



Result:



```text

TcpTestSucceeded : True

```



This confirmed that the backup repository survived the complete Kubernetes cluster deletion.



---



## 16. New Cluster Creation



A new two-node cluster was created:



```powershell

k3d cluster create velero-lab --servers 1 --agents 1

```



New nodes:



```text

k3d-velero-lab-server-0

k3d-velero-lab-agent-0

```



Both nodes reached:



```text

Ready

```



Velero was reinstalled and configured to use the same MinIO repository:



```text

Endpoint: http://host.k3d.internal:9000

Bucket: velero

```



No backup data was manually copied.



---



## 17. Node Agent Issue After Cluster Recreation



After reinstalling Velero on the new cluster, the Node Agent initially failed to start.



### Cause



The generated DaemonSet contained incorrect Windows-style paths:



```text


ar\\lib\\kubelet\\pods


ar\\lib\\kubelet\\plugins

```



The k3d/K3s nodes required:



```text

/var/lib/kubelet/pods

/var/lib/kubelet/plugins

```



### Resolution



The Node Agent DaemonSet was exported, the paths were corrected and the configuration was reapplied:



```powershell

kubectl apply -f C:\\Velero

ode-agent.yaml

```



After the correction, the Node Agent pods reached:



```text

1/1 Running

```



The Velero server was also running.



---



## 18. Previous Backup Discovery



After reconnecting the new Velero installation to the existing MinIO repository:



```powershell

velero backup get

```



The old backup was discovered:



```text

velero-lab-backup-01

```



Status:



```text

Completed

```



This proved that the backup existed independently of the deleted Kubernetes cluster.



---



## 19. Disaster 2 Restore



The previous backup was restored:



```powershell

velero restore create velero-lab-restore-02 --from-backup velero-lab-backup-01

```



Final restore status:



```text

Completed

0 errors

1 warning

```



Resources were verified:



```powershell

kubectl get all -n velero-lab

```



PVC:



```powershell

kubectl get pvc -n velero-lab

```



Persistent data:



```powershell

kubectl exec -n velero-lab deployment/velero-lab-app -- cat /data/important-data.txt

```



Result:



```text

Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06

```



The original data was recovered successfully.



**Result: PASSED\*\*



---



## 20. Scheduled Backup



A daily Velero backup schedule was configured.



```text

Name: velero-lab-daily

Frequency: Daily

Time: 02:00

Retention: 7 days

TTL: 168 hours

Status: Enabled

Paused: False

```



Command:



```powershell

velero schedule create velero-lab-daily --schedule="0 2 \* \* \*" --ttl 168h

```



Verification:



```powershell

velero schedule get

```



The schedule was successfully created and enabled.



---



## 21. Important Resource Names



### Kubernetes



```text

Cluster:

velero-lab



Control Plane:

k3d-velero-lab-server-0



Worker:

k3d-velero-lab-agent-0

```



### Namespaces



```text

velero

velero-lab

```



### Application



```text

Deployment: velero-lab-app

Service: velero-lab-service

ConfigMap: velero-lab-config

Secret: velero-lab-secret

PVC: velero-lab-pvc

```



### Velero



```text

Backup: velero-lab-backup-01

Restore 1: velero-lab-restore-01

Restore 2: velero-lab-restore-02

Schedule: velero-lab-daily

BackupStorageLocation: default

```



### MinIO



```text

Bucket: velero

API: http://host.k3d.internal:9000

Console: http://localhost:9001

Host Data: C:
elero-minio-data

```



### Persistent Data



```text

/data/important-data.txt

```



Original content:



```text

Velero Disaster Recovery Lab - Boobeshwaran - 2026-10-06

```



---



## 22. Troubleshooting Summary

| Issue | Root Cause | Resolution |
|---|---|---|
| MinIO image issue | Image/environment problem | Built and ran the official MinIO source release |
| `SignatureDoesNotMatch` | Incorrect MinIO credentials in Velero | Recreated the credentials and restarted Velero |
| PVC not included | Volume was not opted into Node Agent backup | Added the `backup.velero.io/backup-volumes` annotation |
| Node Agent not starting | Incorrect kubelet host paths | Changed the paths to `/var/lib/kubelet/...` |
| Backup showed `Completed` but data was missing | Backup completion alone did not confirm persistent-data backup | Checked backup details, logs, and Pod Volume Backup (PVB) status |

---



## 23. Evidence



Evidence folders:



```text

evidence/

├── before-backup/

├── after-delete/

├── after-restore/

└── new-cluster-restore/

```



Evidence should include relevant screenshots/output for:



- Cluster nodes

- MinIO availability

- BackupStorageLocation

- Application resources

- PVC

- Persistent data before backup

- Backup details

- Namespace deletion

\- Restore status

- Restored resources

- Persistent data after restore

- Complete cluster deletion

- New cluster

- Previous backup discovery

- Final restore

- Persistent-data verification

- Scheduled backup



---



## 24. Security



The following were kept outside the Git repository:



- MinIO passwords

- MinIO secret keys

- Velero credentials

- kubeconfig credentials

- Cloud credentials

- Access tokens



The local credentials file:



```text

C:\\Velero\\credentials-velero

```



must not be committed.



No real credentials or secrets are included in the project repository.



---



## 25. Final Validation

| Requirement | Result |
|---|---|
| Two-node Kubernetes cluster | ✅ PASSED |
| 1 Control Plane + 1 Worker | ✅ PASSED |
| External MinIO | ✅ PASSED |
| MinIO bucket `velero` | ✅ PASSED |
| Velero installation | ✅ PASSED |
| BackupStorageLocation | ✅ PASSED |
| Test application | ✅ PASSED |
| PVC | ✅ PASSED |
| Persistent data | ✅ PASSED |
| Persistent-data backup | ✅ PASSED |
| Namespace deletion restore | ✅ PASSED |
| Original data after namespace restore | ✅ PASSED |
| Complete cluster deletion | ✅ PASSED |
| New two-node cluster | ✅ PASSED |
| Previous backup discovery | ✅ PASSED |
| Complete cluster restore | ✅ PASSED |
| Original data after cluster restore | ✅ PASSED |
| Daily schedule | ✅ PASSED |
| Seven-day retention | ✅ PASSED |
| Troubleshooting documented | ✅ PASSED |
---



## 26. Key Lessons Learned



- Kubernetes resources and persistent application data both need to be protected.

- A Velero backup showing `Completed` does not by itself prove that persistent data was backed up.

- Backup details, logs and Pod Volume Backup status must be checked.

- External backup storage is important when recovering from complete cluster loss.

- MinIO must remain outside the Kubernetes cluster for this disaster recovery design.
  
- Velero can reconnect to an existing backup repository after a new Kubernetes cluster is created.
 
- PVC recovery must be validated by checking the actual application data.

- Correct Node Agent host paths are required for file-system backup in the k3d/K3s environment.

- Backup retention and scheduled backups provide a basic automated recovery strategy.



---



## 27. Final Result



The Kubernetes Backup and Disaster Recovery lab was successfully completed.



The solution demonstrated:



```text

Kubernetes Resources

&#x20;       +

Persistent Application Data

&#x20;       ↓

&#x20;     Velero

&#x20;       ↓

External MinIO Backup Storage

&#x20;       ↓

&#x20;  Disaster Occurs

&#x20;       ↓

Backup Discovery

&#x20;       ↓

&#x20;     Restore

&#x20;       ↓

Resources + PVC + Original Data

&#x20;      Recovered

```



Both disaster recovery scenarios were successfully validated:



```text

Disaster 1:

Namespace Deleted

&#x20;       ↓

Restore

&#x20;       ↓

Application + PVC + Data Recovered

```



```text

Disaster 2:

Complete Cluster Deleted

&#x20;       ↓

MinIO Preserved

&#x20;       ↓

New Cluster Created

&#x20;       ↓

Velero Reinstalled

&#x20;       ↓

Old Backup Discovered

&#x20;       ↓

Restore

&#x20;       ↓

Application + PVC + Data Recovered

```



The final implementation satisfies the required Kubernetes backup, persistent-data protection, namespace recovery, complete-cluster recovery, external MinIO storage, scheduled backup, and seven-day retention requirements.



