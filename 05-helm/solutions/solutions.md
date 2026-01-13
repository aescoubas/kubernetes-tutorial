# Module 05 Solutions

### Exercise 1: Template Rendering
**Command:**
```bash
helm template my-release ./simple-chart > manifest.yaml
```

### Exercise 2: Value Override
**Command:**
```bash
helm install my-override-app ./simple-chart \
  --set image.tag=latest \
  --set service.type=NodePort
```

### Exercise 3: Upgrade
**Command:**
```bash
helm upgrade my-custom-app ./simple-chart --set replicaCount=1
```
*Verification:* `kubectl get pods` should show terminating pods.

### Exercise 4: History & Rollback
**Commands:**
```bash
# Check history
helm history my-custom-app

# Rollback (assuming revision 1 was the initial install with 3 replicas)
helm rollback my-custom-app 1
```

### Exercise 5: User Values
**Command:**
```bash
helm get values my-custom-app
```
*Explanation:* `helm get values` shows user overrides. `helm get values --all` shows everything including defaults.
