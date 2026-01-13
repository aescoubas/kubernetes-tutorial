# Module 09 Solutions

### Exercise 1: ClusterRole
**Manifest (`ex1-clusterrole.yaml`):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rookie-nodes
subjects:
- kind: ServiceAccount
  name: rookie
  namespace: dev
roleRef:
  kind: ClusterRole
  name: node-viewer
  apiGroup: rbac.authorization.k8s.io
```

### Exercise 2: Context Hacking
**Command:**
```bash
kubectl config set-context dev-viewer --cluster=minikube --user=minikube --namespace=dev
kubectl config use-context dev-viewer
```
*Verification:* `kubectl config current-context`

### Exercise 3: ServiceAccount Token
**Command (K8s 1.24+):**
```bash
kubectl create token rookie -n dev
```
*Copy the output (eyJ...) and decode the payload.*

### Exercise 4: View-Only Admin
**Command:**
```bash
kubectl create rolebinding jane-view --clusterrole=view --user=jane -n prod
```
*Effect:* Jane can see everything in `prod` (Pods, Services, Deployments) but cannot modify them. She cannot see Secrets by default (as `view` excludes them for security).

### Exercise 5: Namespace ResourceQuota
**Manifest (`ex5-quota.yaml`):**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: cpu-limit
  namespace: dev
spec:
  hard:
    limits.cpu: "500m"
```
