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

## 3. Theory: The Kubernetes API Object
Every resource in Kubernetes is defined by 4 top-level fields in its YAML manifest. This is the **GVK (Group, Version, Kind)** system.

### The Anatomy of a Manifest
```yaml
apiVersion: apps/v1        # 1. API Group & Version
kind: Deployment           # 2. Kind (The Type)
metadata:                  # 3. Metadata (Name, Labels, Namespace)
  name: nginx-deploy
spec:                      # 4. Specification (The Desired State)
  replicas: 3
  template: ...
```

### API Versioning
Kubernetes APIs evolve. To maintain backward compatibility, they are versioned:
*   **Stable (`v1`):** Safe. Will be supported for many releases. (e.g., Pods, Services, ConfigMaps).
*   **Beta (`v1beta1`):** Well-tested but semantics might change lightly. Enabled by default.
*   **Alpha (`v1alpha1`):** Experimental. Might be dropped. Disabled by default.

### API Groups & Discovery
Not everything is in the core. Resources are grouped to keep the API clean:
*   **Core (Legacy):** Written as just `v1` (e.g., Pods, Services).
*   **Named Groups:** Written as `group/version` (e.g., `apps/v1` for Deployments, `networking.k8s.io/v1` for Ingress).

**How to explore the API:**
To see every resource type your cluster supports (including their short names and API versions), run:
```bash
kubectl api-resources
```

### Extending the API (CRDs)
Kubernetes is extensible. You are not limited to the default resources (Pods, Services).
*   **Custom Resource Definition (CRD):** A way to teach Kubernetes a new "Kind".
*   *Example:* If you install the *Prometheus Operator*, it adds a new Kind called `Prometheus`. You can then write `kind: Prometheus` in your YAML, and the cluster understands it just like a native Pod. This is the foundation of the "Operator Pattern".

## 4. Lab Experiments

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

## 5. Debugging Tips
When things go wrong, remember these three commands:

1.  **`kubectl get pods`**: Is it Running? Pending? CrashLoopBackOff?
2.  **`kubectl describe pod <name>`**: "Why" is it pending? (No resources? Image pull error?)
3.  **`kubectl logs <name>`**: What is the application saying? (App errors, stack traces).

## 6. Cleanup
To remove resources created in this module:
```bash
kubectl delete -f .
```

## 7. Exercises
*Solutions are available in the `solutions/` directory.*

1.  **Imperative Pod:** Create a pod named `quick-pod` running the image `busybox` that executes the command `sleep 3600`. Do this using *only* the `kubectl run` command (no YAML).
2.  **Scaling Up:** Scale the `nginx-deployment` from Experiment B to **5 replicas** using the command line.
3.  **Label Selection:** Create a Service that tries to select pods with the label `app: wrong-label`. Observe that the Service is created but has no Endpoints.
4.  **Log Extraction:** Start a pod that fails immediately (e.g., `busybox` with a command `this-command-does-not-exist`). Retrieve the logs to see the error message.
5.  **Multi-Container Pod:** Write a YAML manifest for a Pod that contains *two* containers: one running `nginx` and another running `busybox` (sleeping).
