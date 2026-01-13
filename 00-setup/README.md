# Module 00: Environment Setup

## 1. Objective
Initialize a local **Multi-Node** Kubernetes cluster using **Minikube**. We will simulate a robust production-like environment with 4 nodes to practice failover and maintenance.

## 2. Theory: The Cluster Architecture
A standard Kubernetes cluster consists of:
*   **Control Plane (Node 1):** The "Brain". Manages the API Server, etcd (database), and Scheduler.
*   **Worker Nodes (Nodes 2, 3, 4):** The "Muscle". They run the actual applications.

## 3. Requirements (Multi-Node Lab)
Since we are running 4 virtual nodes, the hardware requirements are higher than a standard "Hello World" setup.

*   **CPUs:** 4 or more.
*   **Memory:** 8GB - 12GB free (Recommended).
*   **Disk:** 40GB free.
*   **Hypervisor/Container Manager:** Docker (Preferred) or KVM/Hyperkit.

## 4. Installation Log

### 4.1. Install `kubectl` (The Control Tool)
`kubectl` is a binary that talks to the Kubernetes API.

**Linux (x86-64):**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**macOS (Intel & Apple Silicon):**
```bash
brew install kubectl
```

### 4.2. Install `minikube` (The Cluster)

**Linux (x86-64):**
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**macOS:**
```bash
brew install minikube
```

## 5. Initialization

Start the cluster with 4 nodes (1 Control Plane + 3 Workers). We also limit memory per node to ensure it fits on a typical developer laptop.

```bash
# --nodes 4: Create 4 containers
# --memory 2g: Limit each node to 2GB RAM (Total ~8GB usage)
minikube start --nodes 4 --memory 2g
```

*Status Check:*
```bash
kubectl get nodes
```
*Expected Output:*
```text
NAME           STATUS   ROLES           AGE   VERSION
minikube       Ready    control-plane   2m    v1.x.x
minikube-m02   Ready    <none>          1m    v1.x.x
minikube-m03   Ready    <none>          1m    v1.x.x
minikube-m04   Ready    <none>          1m    v1.x.x
```

## 6. Cluster Management

### Pausing (Save State)
Pauses the Kubernetes processes but keeps the VM/Container running.
```bash
minikube pause
minikube unpause
```

### Stopping (Shutdown)
Shuts down all 4 nodes. Frees up RAM.
```bash
minikube stop
```

### Deleting (Reset)
Destroys the cluster entirely. **All data will be lost.**
```bash
minikube delete
```

## 7. Troubleshooting

### `kubectl: command not found`
1.  **Check PATH:** `echo $PATH`
2.  **Verify Location:** `ls -l /usr/local/bin/kubectl`

### Resource Issues
If the cluster fails to start due to lack of RAM:
1.  Lower the memory per node: `minikube start --nodes 4 --memory 1500m`
2.  Reduce node count: `minikube start --nodes 2` (Minimum for some labs).
