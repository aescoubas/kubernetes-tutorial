# Module 04: Networking & Ingress

## 1. Objective
Expose a web application to the outside world using an Ingress Controller, which provides a single entry point for routing traffic based on hostnames or paths.

## 2. Theory: The Traffic Puzzle
Getting traffic *into* a cluster is harder than it looks.
1.  **ClusterIP (Default):** Only talks to other Pods. Invisible to the world.
2.  **NodePort:** Opens a port (e.g., 30080) on the Node IP.
    *   *Pro:* Simple.
    *   *Con:* Non-standard ports, security risk, binds to the physical node.
3.  **LoadBalancer:** Asks the Cloud Provider (AWS/GCP) for a physical Load Balancer.
    *   *Pro:* Real IP.
    *   *Con:* Expensive ($$$ per service).
4.  **Ingress:** The smartest way. A single LoadBalancer sits at the edge (The "Ingress Controller") and routes traffic based on the HTTP Host header (e.g., `foo.com` vs `bar.com`).

## 3. Concepts
*   **Ingress Resource:** The rulebook (`host: hello.world` -> `service: web-service`).
*   **Ingress Controller:** The software (Nginx, Traefik, HAProxy) that reads the rulebook and actually moves the packets. Minikube uses Nginx.

## 4. Prerequisites

Enable the Nginx Ingress Controller in Minikube:
```bash
minikube addons enable ingress
```
*Wait for the ingress-nginx-controller pod to be running in the `ingress-nginx` namespace.*

## 5. Lab Experiments

### Experiment A: The Backend Application
*File: `web-app.yaml`*

1.  **Deploy the App:**
    This creates a Deployment and a `ClusterIP` Service. Note that `ClusterIP` is *not* accessible externally by default.
    ```bash
    kubectl apply -f web-app.yaml
    ```
2.  **Verify Service:**
    ```bash
    kubectl get svc web-service
    ```

### Experiment B: The Ingress Rule
*File: `ingress.yaml`*

1.  **Apply the Ingress:**
    ```bash
    kubectl apply -f ingress.yaml
    ```
2.  **Verify Address:**
    Wait for the Ingress to acquire an IP address (ADDRESS column).
    ```bash
    kubectl get ingress
    ```

### Experiment C: Accessing the App

**Option 1: The Minikube Tunnel (Recommended for Docker driver)**
Minikube often requires a tunnel to route traffic to the Ingress controller on macOS/Linux Docker.
```bash
# Run this in a separate terminal and keep it open
sudo minikube tunnel
```

**Option 2: Curl with Header (Simpler for Testing)**
Instead of editing your computer's DNS (`/etc/hosts`), we can tell `curl` "pretend I am asking for hello.world".

1.  **Get the Ingress IP:**
    If `minikube tunnel` is running, this is `127.0.0.1`. If using VM driver, it's `minikube ip`.
    ```bash
    INGRESS_IP=$(minikube ip) # Or 127.0.0.1 if using tunnel
    ```

2.  **Request:**
    ```bash
    curl -H "Host: hello.world" http://$INGRESS_IP
    ```
    *Expected Output:*
    ```text
    Hello, world!
    Version: 1.0.0
    Hostname: web-app-xxxxxx
    ```

## 6. Cleanup
```bash
kubectl delete -f .
```