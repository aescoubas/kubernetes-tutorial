# Module 08: Cluster Upgrades & Versioning

## 1. Objective
Simulate a cluster upgrade from Kubernetes v1.28 to v1.29. Learn to manage version skew and verify workload persistence during maintenance.

## 2. Theory: The Upgrade Path

### The "Bare Metal" Way (What Sysadmins do manually)
On standard servers, an upgrade involves:
1.  **Upgrade Control Plane:** Update `kubeadm` on the master, run `kubeadm upgrade plan`, then `kubeadm upgrade apply`.
2.  **Drain Node:** Evacuate workloads.
3.  **Upgrade Node:** Update `kubelet` and `kubectl`. Restart service.
4.  **Uncordon:** Bring node back.
5.  **Repeat** for all nodes.

### The "Minikube" Way (The Managed Experience)
Minikube automates the `kubeadm` steps. You simply restart the cluster with a newer version target. This simulates how you might upgrade a cloud-managed cluster (like EKS or GKE) where you request a version change and the provider handles the backend.

### Critical Risk: API Deprecations
Kubernetes frequently removes old APIs (e.g., `v1beta1`). Before upgrading, you **must** check if your manifests are compatible with the new version. Tools like `kubent` or `pluto` help with this.

## 3. Lab Experiments

### Experiment A: The Legacy State (v1.28)
We need to destroy our current lab and start fresh with an older version.

1.  **Reset:**
    ```bash
    minikube delete
    ```
2.  **Start Old Version:**
    We specifically request Kubernetes v1.28.3.
    ```bash
    minikube start --kubernetes-version=v1.28.3 --nodes 2
    ```
3.  **Deploy Workload:**
    Create a Deployment to ensure it survives the upgrade.
    ```bash
    kubectl create deployment legacy-app --image=nginx:alpine --replicas=4
    ```
4.  **Verify Version:**
    ```bash
    kubectl version --short
    # Server Version: v1.28.3
    ```

### Experiment B: The Upgrade (to v1.29)
Now we simulate the maintenance window.

1.  **The Upgrade Command:**
    We restart the cluster but target a newer version. Minikube will detect the drift and upgrade the components.
    ```bash
    minikube start --kubernetes-version=v1.29.0 --nodes 2
    ```
    *Watch the logs. You will see "Preparing Kubernetes v1.29.0" and "Upgrading control plane".*

2.  **Verify Upgrade:**
    ```bash
    kubectl version --short
    # Server Version: v1.29.0
    ```

3.  **Verify Workload Survival:**
    Check if your Nginx pods are still running.
    ```bash
    kubectl get pods
    ```
    *Note: In a multi-node setup, pods might have restarted, but the Application (Deployment) should be healthy.*

## 4. Cleanup
```bash
minikube delete
# Return to default latest version for future labs
minikube start --nodes 4 --memory 2g
```

## 5. Exercises
*Solutions are available in the `solutions/` directory.*

1.  **Drift Detection:** Before upgrading, how can you check which version your client (`kubectl`) is versus the server? Why is it bad if the client is significantly older?
2.  **Maintenance Page:** During an upgrade, the API might be briefly unavailable. If you had an Ingress Controller, does the *data plane* (the Nginx process routing traffic) stop working just because the *control plane* (API Server) is upgrading?
3.  **Rollback:** (Research) If the upgrade fails, can you easily "downgrade" a Kubernetes cluster?
4.  **Node Skew:** In a real 4-node cluster, is it allowed to have the Control Plane on v1.29 and Worker Nodes on v1.28?
5.  **Pod Disruption Budget:** Create a `PodDisruptionBudget` (PDB) for your `legacy-app` that ensures at least 2 replicas are always available. This protects the app during the node drains that happen during upgrades.

## 6. References & Documentation
*   [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
*   [Version Skew Policy](https://kubernetes.io/releases/version-skew-policy/)
*   [Deprecated API Migration Guide](https://kubernetes.io/docs/reference/using-api/deprecation-guide/)