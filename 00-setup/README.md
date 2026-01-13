# Module 00: Environment Setup

## 1. Objective
Initialize a local single-node Kubernetes cluster using **Minikube** and configure the command-line interface tool **kubectl**.

## 2. Theory: The Cluster Architecture
Before we install, understand what we are building. A standard Kubernetes cluster consists of:
*   **Control Plane:** The "Brain". Manages the API (API Server), stores state (etcd), and schedules workloads (Scheduler).
*   **Worker Nodes:** The "Muscle". They run the actual applications. They contain the Kubelet (agent) and a Container Runtime (Docker/containerd).

**Minikube** collapses this entire architecture into a single machine (VM or Container) for learning and development. It is not suitable for production but is identical from an API perspective.

## 3. Requirements
*   2 CPUs or more.
*   2GB of free memory.
*   20GB of free disk space.
*   **Hypervisor/Container Manager:** Minikube needs a way to create the cluster "node".
    *   *Linux:* Docker (preferred), KVM, or VirtualBox.
    *   *macOS:* Docker (preferred), Hyperkit, or VirtualBox.

## 4. Installation Log

### 4.1. Install `kubectl` (The Control Tool)
`kubectl` is a binary that talks to the Kubernetes API. It is your primary interface.

**Linux (x86-64):**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

**macOS (Intel & Apple Silicon):**
```bash
brew install kubectl
```

*Verification:*
```bash
kubectl version --client
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

Start the cluster. Minikube will automatically select the best driver (usually Docker if installed).

```bash
minikube start
```

### Behind the Scenes:
1.  Minikube downloads a boot image (ISO/Container image).
2.  It creates a VM or Container.
3.  It generates certificates for secure communication.
4.  It installs the Kubernetes binaries (kubelet, kube-apiserver, etc.) inside that VM/Container.
5.  It configures your local `~/.kube/config` file so `kubectl` knows how to talk to this new cluster.

*Status Check:*
```bash
minikube status
kubectl cluster-info
```

## 6. Validation
Verify the cluster is operational by listing the system nodes.

```bash
kubectl get nodes
```

*Expected Output:*
```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.x.x
```

## 7. Cluster Management
Minikube runs as a process/VM on your machine. You will need to manage its lifecycle to save resources when not in use.

### Pausing (Save State)
Pauses the Kubernetes processes but keeps the VM/Container running. Fast resume.
```bash
minikube pause
minikube unpause
```

### Stopping (Shutdown)
Shuts down the VM/Container. Frees up RAM. Requires a full startup sequence to resume.
```bash
minikube stop
# To start again:
minikube start
```

### Deleting (Reset)
Destroys the VM/Container entirely. **All data and deployments will be lost.** Use this if your cluster is corrupted and you want a fresh start.
```bash
minikube delete
```

## 8. Troubleshooting

### `kubectl: command not found`
If your shell says the command doesn't exist after installation:
1.  **Check your PATH:** Ensure `/usr/local/bin` is in your system `$PATH`.
    ```bash
    echo $PATH
    ```
2.  **Verify Location:** Check if the file actually exists where you installed it.
    ```bash
    ls -l /usr/local/bin/kubectl
    ```
3.  **Permissions:** Ensure it is executable.
    ```bash
    sudo chmod +x /usr/local/bin/kubectl
    ```

### Common Minikube Issues
*   **Driver Issues:** If `minikube start` fails, try forcing a driver explicitly:
    ```bash
    minikube start --driver=docker
    ```
*   **Permissions:** Ensure your user is in the `docker` group on Linux.
    ```bash
    sudo usermod -aG docker $USER
    # Log out and log back in for this to take effect
    ```
*   **Virtualization:** Ensure VT-x/AMD-v is enabled in your BIOS if using VM drivers (VirtualBox/KVM).
