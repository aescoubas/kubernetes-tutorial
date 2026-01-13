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

### PersistentVolumeClaim (PVC)
The "Ticket". A request for resources by a user.
*   **Capacity:** How much space? (e.g., `10Gi`)
*   **Access Mode:**
    *   `ReadWriteOnce (RWO)`: Mounted by a single node (Block Storage).
    *   `ReadWriteMany (RWX)`: Mounted by multiple nodes (NFS/File Storage).

### PersistentVolume (PV)
The "Resource". The actual piece of storage captured in the cluster.
*   It has a lifecycle independent of any individual Pod.
*   It contains the details of the backend (IP address, Path, Volume ID).

### StorageClass
The "Profile". It tells Kubernetes *how* to create a PV dynamically.
*   **Provisioner:** The plugin responsible for creating the storage (e.g., `kubernetes.io/aws-ebs`, `nfs-subdir-external-provisioner`).
*   **ReclaimPolicy:** What happens to the data when the PVC is deleted?
    *   `Delete`: The underlying storage is wiped (Default).
    *   `Retain`: The storage remains for manual recovery.

## 4. Deep Dive: NFS & Dynamic Provisioning
In our Minikube lab, we use a simple `hostPath` provisioner (saving files to the VM's disk). In a real production environment using **NFS**, the flow looks like this:

### 1. The Sysadmin sets up the StorageClass
This requires an **NFS Provisioner** (a pod running in the cluster that talks to your NAS).
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner # The software handling requests
parameters:
  archiveOnDelete: "false"
```

### 2. The Developer creates a PVC
They specifically request the `nfs-client` class.
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-nfs-data
spec:
  storageClassName: nfs-client  # <--- Matches the class above
  accessModes:
    - ReadWriteMany             # <--- NFS supports multiple writers!
  resources:
    requests:
      storage: 5Gi
```

### 3. The Magic (Dynamic Provisioning)
1.  The **Provisioner** sees the PVC.
2.  It talks to the NFS Server (e.g., `192.168.1.50:/exports`).
3.  It creates a subdirectory: `/exports/default-my-nfs-data-pvc-12345`.
4.  It creates a **PersistentVolume (PV)** object in Kubernetes pointing to that folder.
5.  It binds the PVC to the PV.

### 4. The Result
The Developer gets a 5GB volume that automatically connects to the NFS server, without ever knowing the server's IP address.

## 5. Lab Experiments (Minikube)
*Note: We will use the default Minikube storage class, which simulates this behavior locally.*

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

## 6. Cleanup
```bash
kubectl delete -f .
```
