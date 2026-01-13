# Module 03: Storage & State

## 1. Objective
Provision persistent storage so that data survives Pod restarts and deletions.

## 2. Concepts

*   **PersistentVolumeClaim (PVC):** A request for storage by a user (e.g., "I need 1GB").
*   **StorageClass:** Defines the type of storage (e.g., "standard", "ssd"). Minikube comes with a default `standard` class that provisions host-path storage.
*   **PersistentVolume (PV):** The actual piece of storage. In dynamic provisioning (used here), this is created automatically to satisfy the PVC.

## 3. Lab Experiments

### Experiment A: Claiming Storage
*File: `pvc.yaml`*

1.  **Create the Claim:**
    ```bash
    kubectl apply -f pvc.yaml
    ```
2.  **Verify Binding:**
    Wait until status is `Bound`.
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
    Create a file in the mounted path.
    ```bash
    kubectl exec storage-pod -- sh -c "echo 'Data persists!' > /data/test.txt"
    ```
3.  **Prove Persistence:**
    Delete the Pod (simulate a crash) and recreate it.
    ```bash
    kubectl delete pod storage-pod
    kubectl apply -f pod-with-storage.yaml
    ```
4.  **Read Data:**
    Verify the file is still there in the new Pod instance.
    ```bash
    kubectl exec storage-pod -- cat /data/test.txt
    ```

## 4. Cleanup
```bash
kubectl delete -f .
```
