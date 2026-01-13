# Module 01: The Basics (Pods, Deployments, Services)

## 1. Objective
Deploy a stateless web server (Nginx), ensure its availability, and expose it internally within the cluster.

## 2. Concepts

*   **Pod:** The smallest deployable unit. A wrapper around one or more containers.
*   **Deployment:** Manages Pod replicas, rollouts, and rollbacks. (Preferred over raw Pods).
*   **Service:** A stable network endpoint (IP/DNS) to access a dynamic set of Pods.

## 3. Lab Experiments

### Experiment A: The Raw Pod
*File: `pod.yaml`*

1.  **Create the manifest:**
    ```bash
    kubectl apply -f pod.yaml
    ```
2.  **Verify status:**
    ```bash
    kubectl get pods
    ```
3.  **Port-forward (for direct access testing):**
    ```bash
    kubectl port-forward pod/nginx-pod 8080:80
    ```
    *Open browser to http://localhost:8080*

### Experiment B: The Deployment (Self-Healing)
*File: `deployment.yaml`*

1.  **Deploy:**
    ```bash
    kubectl apply -f deployment.yaml
    ```
2.  **Observe resilience:**
    Delete a pod and watch it restart.
    ```bash
    kubectl delete pod <pod-name>
    kubectl get pods -w
    ```

### Experiment C: The Service (Networking)
*File: `service.yaml`*

1.  **Expose:**
    ```bash
    kubectl apply -f service.yaml
    ```
2.  **Access (Minikube specific):**
    Minikube exposes NodePort services via a special tunnel command or IP.
    ```bash
    minikube service nginx-service --url
    ```

## 4. Cleanup
To remove resources created in this module:
```bash
kubectl delete -f .
```
