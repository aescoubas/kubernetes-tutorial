# Module 00 Solutions

### Exercise 1: Cluster Inspection
**Command:**
```bash
kubectl get all -n kube-system
```
*Explanation:* The `-n` flag specifies the namespace. `kube-system` holds core components like CoreDNS and the metrics-server.

### Exercise 2: Internal Access
**Command:**
```bash
minikube ssh
# Once inside:
ps aux
exit
```
*Explanation:* Minikube provides a direct wrapper to SSH into the underlying VM/Container.

### Exercise 3: Component Status
**Command:**
```bash
kubectl get pods -n kube-system -l component=kube-scheduler
kubectl get pods -n kube-system -l component=kube-controller-manager
```
*Explanation:* In modern Kubernetes, `kubectl get cs` (componentstatuses) is deprecated. The reliable way is to check the pods running in `kube-system`.

### Exercise 4: Node Details
**Command:**
```bash
kubectl describe node minikube-m02 | grep -i "InternalIP\|OS Image"
```
*Alternatively (JSONPath):*
```bash
kubectl get node minikube-m02 -o jsonpath='{.status.addresses[?(@.type=="InternalIP")].address} {.status.nodeInfo.osImage}'
```

### Exercise 5: Context Switching
**Commands:**
```bash
# Create the namespace
kubectl create ns lab-00

# Switch context
kubectl config set-context --current --namespace=lab-00

# Verify
kubectl config view --minify | grep namespace
```
*Explanation:* Changing the default namespace saves you from typing `-n lab-00` on every command.
