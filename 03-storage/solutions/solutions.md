# Module 03 Solutions

### Exercise 1: Pending Claim
**Manifest (`ex1-big-pvc.yaml`):**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: too-big
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Ti
```
*Observation:* On Minikube, this might actually bind because the hostpath provisioner doesn't strictly check disk space. On AWS/GCP, it would stay `Pending` until quota allows.

### Exercise 2: Reclaim Policy
**Command:**
1. Find the PV name: `kubectl get pv` (e.g., `pvc-1234abcd...`).
2. Patch it:
```bash
kubectl patch pv <pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```
*Verification:* `kubectl get pv <pv-name>` should show `Retain`.

### Exercise 3: Expansion
**Command:**
```bash
kubectl edit pvc local-pvc
# Change 'storage: 500Mi' to 'storage: 1Gi'
```
*Verification:* `kubectl get pvc local-pvc` should show the new size (if the provisioner supports online expansion).

### Exercise 4: Dual Mount
**Manifest (`ex4-dual-mount.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dual-mount-pod
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
        - mountPath: "/data"
          name: my-storage
        - mountPath: "/backup"  # Same volume, different path
          name: my-storage
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: local-pvc
```

### Exercise 5: Manual Cleanup & Retain Check
**Command:**
```bash
kubectl delete pvc local-pvc
kubectl get pv
```
*Observation:* Because we set ReclaimPolicy to `Retain` in Ex 2, the PV should still exist with status `Released`. It has not been deleted.
