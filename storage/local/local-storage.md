# Kubernetes Local Storage with hostPath

A minimal example of persistent storage using node-local disk.

## What This Does

Creates a PersistentVolume backed by a folder on the node's filesystem. Data survives container/pod restarts but is tied to that specific node.

## Files

```yaml
# pv.yaml - The actual storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  capacity:
    storage: 500Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/my-app
---
# pvc.yaml - Claim to use the storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: ""
---
# pod.yaml - Pod using the storage
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  volumes:
    - name: app-data
      persistentVolumeClaim:
        claimName: host-pvc
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo hello > /data/test.txt && sleep 3600"]
      volumeMounts:
        - name: app-data
          mountPath: /data
```

## How It Works

```
Container: /data/test.txt
     ↓ (volumeMount)
Pod volume: app-data
     ↓ (PVC reference)
PVC: host-pvc
     ↓ (bound)
PV: host-pv
     ↓ (hostPath)
Node filesystem: /data/my-app/test.txt
```

## Usage

```bash
# Deploy
kubectl apply -f storage.yaml

# Verify binding
kubectl get pv,pvc

# Check data
kubectl exec my-app -- cat /data/test.txt

# SSH to node and verify
cat /data/my-app/test.txt
```

## Data Durability

| Event | Data Status |
|-------|-------------|
| Container restart | ✅ Persists |
| Pod restart (same node) | ✅ Persists |
| Node reboot | ✅ Persists |
| Pod moves to different node | ❌ Lost (data stuck on original node) |
| Node deleted | ❌ Lost |

## When to Use hostPath

**Good for:**
- Local development (minikube, kind)
- Single-node clusters
- DaemonSets (one pod per node, never moves)
- Accessing node-specific resources (logs, docker socket)

**Not for:**
- Production multi-node clusters
- Data that must survive node failure
- Pods that may reschedule across nodes

## Production Alternatives

| Need | Solution |
|------|----------|
| Data survives node failure | Cloud disks (EBS, GCP PD, Azure Disk) |
| Multiple pods read/write same data | NFS, EFS, CephFS |
| Automatic provisioning | StorageClass with dynamic provisioner |