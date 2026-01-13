# Module 07 Solutions

### Exercise 1: Burstable Config
**Manifest (`ex1-burstable.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: burstable
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "500m"
```
*Verification:* `kubectl describe pod burstable | grep QoS` -> `Burstable`.

### Exercise 2: Resource Quota
**Manifest (`ex2-quota.yaml`):**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-counter
  namespace: quota-test
spec:
  hard:
    pods: "2"
```
**Steps:**
1. `kubectl create ns quota-test`
2. `kubectl apply -f ex2-quota.yaml`
3. Try to `kubectl run` 3 pods. The 3rd will fail with `Forbidden: exceeded quota`.

### Exercise 3: LimitRange
**Manifest (`ex3-limitrange.yaml`):**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 200Mi
    type: Container
```
**Verification:**
1. Apply in a namespace.
2. `kubectl run test --image=nginx` (No resources specified).
3. `kubectl get pod test -o yaml` -> You will see `limits: { memory: 200Mi }` injected automatically.

### Exercise 4: CPU Throttling
**Answer:**
No, it does NOT crash. CPU is a "compressible" resource.
When a container hits its CPU limit, the Linux kernel (CFS Scheduler) simply throttles the process. The app becomes slower and unresponsive (latency increases), but the process state remains `Running`.

### Exercise 5: Node Capacity
**Command:**
```bash
kubectl describe node minikube-m02 | grep -A 5 Allocatable
```
*Explanation:* You must look at "Allocatable", not "Capacity". Allocatable = Capacity minus the resources reserved for Kubelet and the OS itself.
