# Module 08 Solutions

### Exercise 1: Drift Detection
**Command:**
```bash
kubectl version
```
*Explanation:* You look for "Client Version" vs "Server Version".
*Why it's bad:* `kubectl` is generally compatible with +/- 1 minor version. If you are too far apart (e.g., Client 1.25, Server 1.29), commands might fail due to API changes or flag incompatibilities.

### Exercise 2: Maintenance Page (Control vs Data Plane)
**Answer:**
**No**, the data plane usually keeps working.
The Ingress Controller (Nginx/Traefik) runs as a Pod. Even if the API Server (Control Plane) is rebooting, the Nginx process is still running on the worker node and routing traffic. You just can't *change* the configuration (deploy new apps) during that window.

### Exercise 3: Rollback
**Answer:**
**No.** Downgrading Kubernetes is generally **not supported** and is very dangerous. The `etcd` database schema might have changed. The standard recovery path for a failed upgrade is to restore from a backup, not to downgrade binaries.

### Exercise 4: Node Skew
**Answer:**
**Yes.** This is a standard upgrade procedure.
You upgrade the Control Plane first. It is supported to have Worker Nodes up to **3 minor versions** older than the Control Plane (Skew Policy). This allows you to upgrade nodes one by one without downtime.

### Exercise 5: Pod Disruption Budget
**Manifest (`ex5-pdb.yaml`):**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: legacy-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: legacy-app
```
*Effect:* If an automated tool (or `kubectl drain`) tries to remove too many pods at once, the API will block the eviction to ensure 2 remain running.
