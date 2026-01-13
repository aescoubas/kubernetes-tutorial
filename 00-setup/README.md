# Module 00: Environment Setup

## 1. Objective
Initialize a local single-node Kubernetes cluster using **Minikube** and configure the command-line interface tool **kubectl**.

## 2. Requirements
*   2 CPUs or more
*   2GB of free memory
*   20GB of free disk space
*   Container or VM manager (Docker, QEMU, Hyperkit, KVM, etc.)

## 3. Installation Log

### 3.1. Install `kubectl` (The Control Tool)

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

### 3.2. Install `minikube` (The Cluster)

**Linux (x86-64):**
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**macOS:**
```bash
brew install minikube
```

## 4. Initialization

Start the cluster. Minikube will automatically select the best driver (usually Docker if installed).

```bash
minikube start
```

*Status Check:*
```bash
minikube status
kubectl cluster-info
```

## 5. Validation
Verify the cluster is operational by listing the system nodes.

```bash
kubectl get nodes
```

*Expected Output:*
```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.x.x
```
