# Module 04: Networking & Ingress

## 1. Objective
Expose a web application to the outside world using an Ingress Controller, which provides a single entry point for routing traffic based on hostnames or paths.

## 2. Concepts

*   **ClusterIP:** The default Service type. Exposes the service on a cluster-internal IP.
*   **Ingress:** An API object that manages external access to the services in a cluster, typically HTTP.
*   **Ingress Controller:** The underlying software (e.g., Nginx, Traefik) that fulfills the Ingress rules. Minikube provides one via an addon.

## 3. Prerequisites

Enable the Nginx Ingress Controller in Minikube:
```bash
minikube addons enable ingress
```
*Wait for the ingress-nginx-controller pod to be running in the `ingress-nginx` namespace.*

## 4. Lab Experiments

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
# Run this in a separate terminal
minikube tunnel
```
Then access via `localhost` if you configured the host `hello.world` in `/etc/hosts` to point to `127.0.0.1`.

**Option 2: Curl with Header (Simpler for Testing)**
Instead of editing `/etc/hosts`, we can simulate the DNS resolution using curl.

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

## 5. Cleanup
```bash
kubectl delete -f .
```
