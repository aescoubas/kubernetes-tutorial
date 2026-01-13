# Module 03: Storage & State

## 1. Objective
Provision persistent storage so that data survives Pod restarts and deletions.

## 2. Theory: The Ephemeral Problem
By default, the filesystem in a container is **ephemeral**. If a Pod crashes or is rescheduled to another node, all files written inside it are lost.
For databases or file stores, we need **Persistence**.

### The Abstraction Layer
Kubernetes separates the *request* for storage from the *implementation* of storage.
*   **Sysadmin:** Configures the **StorageClass** (e.g., "AWS EBS", "Local Disk", "NFS").
*   **Developer:** Creates a **PersistentVolumeClaim (PVC)** ("I need 10GB of ReadWrite storage").
*   **Kubernetes:** Matches the Claim (PVC) to a Volume (PV). If using *Dynamic Provisioning*, it creates the PV automatically.

## 3. Concepts
*   **PersistentVolumeClaim (PVC):** The ticket. A request for resources.
*   **PersistentVolume (PV):** The resource. The actual disk/slice.
*   **Access Modes:**
    *   `ReadWriteOnce (RWO)`: Mounted by a single node (typical for Block Storage like EBS).
    *   `ReadWriteMany (RWX)`: Mounted by multiple nodes (typical for NFS/File Storage).

## 4. Lab Experiments

### Experiment A: Claiming Storage
*File: `pvc.yaml`*

1.  **Create the Claim:**
    ```bash
    kubectl apply -f pvc.yaml
    ```
2.  **Verify Binding:**
    Wait until status is `Bound`. This means Minikube's "host-path" provisioner created a folder on the VM and linked it to your claim.
    ```bash
    kubectl get pvc
    ```

### Experiment B: Using Storage
*File: `pod-with-storage.yaml`*

1.  **Deploy the Pod:**
    ```bash
    kubectl apply -f pod-with-storage.yaml
    ```
2.  **Write Data:**
    Create a file in the mounted path (`/data`).
    ```bash
    kubectl exec storage-pod -- sh -c "echo 'Data persists!' > /data/test.txt"
    ```
3.  **Prove Persistence:**
    Delete the Pod. This simulates a crash or update. The Pod is gone, but the PVC (and data) remains.
    ```bash
    kubectl delete pod storage-pod
    
    # Recreate it
    kubectl apply -f pod-with-storage.yaml
    ```
4.  **Read Data:**
    Verify the file is still there in the new Pod instance.
    ```bash
    kubectl exec storage-pod -- cat /data/test.txt
    ```

## 5. Cleanup
```bash
kubectl delete -f .
```