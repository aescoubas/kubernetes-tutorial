# Module 01: The Basics (Pods, Deployments, Services)

## 1. Objective
Deploy a stateless web server (Nginx), ensure its availability, and expose it internally within the cluster.

## 2. Theory: The "Why" of Orchestration

### Why not just `docker run`?
In a traditional setup, if a container crashes, it dies. You have to manually restart it. If the server dies, the app is gone.
Kubernetes introduces **Declarative State**: You tell it "I want 3 Nginx servers running at all times," and the cluster continuously works to make that true (The **Reconciliation Loop**).

### Core Concepts
*   **Pod:** The "Atom" of Kubernetes. It's a wrapper around one or more containers that share network (IP) and storage. *We rarely manage Pods directly.*
*   **Deployment:** The "Manager". It creates a ReplicaSet to ensure a specific number of Pods are running. It handles updates (rolling deployments) and self-healing.
*   **Service:** The "Switch". Pods are ephemeral; they die and get new IPs. A Service provides a stable Virtual IP (VIP) and DNS name that load-balances traffic to the current set of healthy Pods.

## 3. Lab Experiments

### Experiment A: The Raw Pod (Anti-Pattern)
*File: `pod.yaml`*
*Note: We do this to understand the building block, but in production, you almost always use Controllers (like Deployments).*

1.  **Create the manifest:**
    ```bash
    kubectl apply -f pod.yaml
    ```
2.  **Verify status:**
    ```bash
    kubectl get pods
    ```
3.  **Port-forward (Debug Access):**
    Since Pods are internal, we tunnel a port from our laptop to the cluster for testing.
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
2.  **Observe Self-Healing:**
    We will simulate a crash by manually deleting a pod.
    ```bash
    # Watch the pods in real-time
    kubectl get pods -w
    
    # In another terminal, delete one
    kubectl delete pod <pod-name>
    ```
    *Observation:* The Deployment immediately notices the count is 2/3 and starts a new one to return to 3/3.

### Experiment C: The Service (Internal Networking)
*File: `service.yaml`*

1.  **Expose:**
    ```bash
    kubectl apply -f service.yaml
    ```
2.  **Understand NodePort:**
    We used type `NodePort`. This opens a high-range port (30000-32767) on *every node* (in our case, the minikube VM) that forwards to the internal Service IP.
    
3.  **Access (Minikube specific):**
    Minikube exposes NodePort services via a special tunnel command or IP.
    ```bash
    minikube service nginx-service --url
    ```

## 4. Debugging Tips
When things go wrong, remember these three commands:

1.  **`kubectl get pods`**: Is it Running? Pending? CrashLoopBackOff?
2.  **`kubectl describe pod <name>`**: "Why" is it pending? (No resources? Image pull error?)
3.  **`kubectl logs <name>`**: What is the application saying? (App errors, stack traces).

## 5. Cleanup
To remove resources created in this module:
```bash
kubectl delete -f .
```