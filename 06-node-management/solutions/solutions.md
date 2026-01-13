# Module 06 Solutions

### Exercise 1: Node Selector
**Manifest (`ex1-selector.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selected-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
  nodeSelector:
    hardware: high-mem  # <--- Matches the label on m02
```

### Exercise 2: Drain Failure
**Explanation:** `kubectl drain` checks for pods that would be lost permanently.
*   **Unmanaged Pods:** Pods not created by a Deployment/ReplicaSet. If you kill them, they are gone forever. You must use `--force` to acknowledge this.
*   **Local Storage:** Pods using `emptyDir` or local paths. Data will be lost. You must use `--delete-emptydir-data`.

### Exercise 3: Manual Uncordon
**Command:**
```bash
# Cordon
kubectl cordon minikube-m02

# Uncordon via Patch
kubectl patch node minikube-m02 -p '{"spec":{"unschedulable":false}}'
```
*Explanation:* `cordon` just sets `spec.unschedulable: true`.

### Exercise 4: Tolerations
**Manifest (`ex4-toleration.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerant-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
  tolerations:
  - key: "specific"
    operator: "Equal"
    value: "workload"
    effect: "NoSchedule"
  nodeSelector:
    kubernetes.io/hostname: minikube-m04 # Optional: To force it there for proof
```

### Exercise 5: Clean Sweep
**Commands:**
```bash
# Remove Label (append minus)
kubectl label node minikube-m02 hardware-

# Remove Taint (append minus)
kubectl taint nodes minikube-m04 specific=workload:NoSchedule-
```
