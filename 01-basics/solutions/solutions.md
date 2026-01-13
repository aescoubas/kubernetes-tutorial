# Module 01 Solutions

### Exercise 1: Imperative Pod
**Command:**
```bash
kubectl run quick-pod --image=busybox --restart=Never -- sleep 3600
```

### Exercise 2: Scaling Up
**Command:**
```bash
kubectl scale deployment nginx-deployment --replicas=5
```
*Verification:* `kubectl get pods` should show 5 running instances.

### Exercise 3: Label Selection (The Broken Service)
**Manifest (`ex3-broken-service.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: broken-service
spec:
  ports:
  - port: 80
  selector:
    app: wrong-label
```
**Verification:**
```bash
kubectl apply -f ex3-broken-service.yaml
kubectl describe service broken-service
```
*Look for `Endpoints: <none>`. This confirms the service can't find any pods matching that label.*

### Exercise 4: Log Extraction
**Command:**
```bash
# Run a failing pod
kubectl run crasher --image=busybox --restart=Never -- this-command-does-not-exist

# Get logs
kubectl logs crasher
```
*Expected Output:* `/bin/sh: this-command-does-not-exist: not found`

### Exercise 5: Multi-Container Pod
**Manifest (`ex5-multi.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
  - name: sidecar
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
```
