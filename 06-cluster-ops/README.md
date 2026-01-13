# Module 06: Cluster Operations (Node Management)

## 1. Objective
Act as a Cluster Administrator. Manage the underlying infrastructure, perform maintenance on nodes without downtime, and control where workloads run.

## 2. Theory: The Care and Feeding of Nodes

### Node Components
Each node in your cluster runs:
*   **Kubelet:** The agent that talks to the Control Plane. It says "I am here" and "I am running these Pods".
*   **Kube-Proxy:** Handles network rules (IP tables/IPVS) so Services work.
*   **Container Runtime:** (Docker/containerd) The engine that actually runs the containers.

### Maintenance Primitives
When a server needs a kernel patch or hardware replacement, you can't just turn it off. You must:
1.  **Cordon:** Mark the node as "Unschedulable". No *new* Pods will land there.
2.  **Drain:** Politely ask all existing Pods to leave (terminate) and reschedule elsewhere.

## 3. Lab Experiments

### Experiment A: Labeling Nodes
Labels are the primary way we organize infrastructure (e.g., `gpu=true`, `zone=us-east`, `environment=prod`).

1.  **List Nodes with Labels:**
    ```bash
    kubectl get nodes --show-labels
    ```
2.  **Label a Node:**
    Let's mark Node 2 as our "High Memory" node (simulated).
    ```bash
    kubectl label node minikube-m02 hardware=high-mem
    ```
3.  **Verify:**
    ```bash
    kubectl get node -l hardware=high-mem
    ```

### Experiment B: Node Maintenance (The Drain)
We will simulate a hardware failure or maintenance window on **Node 3**.

1.  **Setup Workload:**
    Deploy an app with multiple replicas to see them spread out.
    ```bash
    kubectl create deployment ops-test --image=nginx --replicas=6
    ```
2.  **Observe Distribution:**
    See which nodes the pods are running on.
    ```bash
    kubectl get pods -o wide
    ```
    *Note that some pods should be on `minikube-m03`.*

3.  **Cordon (Close the Gate):**
    Prevent new pods from landing on Node 3.
    ```bash
    kubectl cordon minikube-m03
    ```
    *Check status:* `kubectl get nodes` (Should see `SchedulingDisabled`).

4.  **Drain (Evacuate):**
    Force the pods off.
    ```bash
    # --ignore-daemonsets: Don't kill system agents (like networking plugins)
    # --force: Required if pods are not managed by a controller (rare)
    kubectl drain minikube-m03 --ignore-daemonsets
    ```

5.  **Verify Evacuation:**
    Run `kubectl get pods -o wide` again.
    *Result:* No pods should be left on `minikube-m03`. They have all moved to `minikube`, `m02`, or `m04`.

6.  **Uncordon (Return to Service):**
    Maintenance is done. Bring the node back.
    ```bash
    kubectl uncordon minikube-m03
    ```

### Experiment C: Taints (Repelling Pods)
Labels *attract* pods (via Affinity), but Taints *repel* them.
Example: "Don't schedule anything on this node unless it explicitly tolerates the 'special' taint."

1.  **Taint Node 4:**
    ```bash
    kubectl taint nodes minikube-m04 specific=workload:NoSchedule
    ```
    *Now, regular Pods will refuse to run here.*

2.  **Test:**
    Scale up your deployment.
    ```bash
    kubectl scale deployment ops-test --replicas=10
    kubectl get pods -o wide
    ```
    *You will see pods landing on m02 and m03, but avoiding m04.*

3.  **Remove Taint:**
    ```bash
    kubectl taint nodes minikube-m04 specific=workload:NoSchedule-
    ```
    *(Note the minus sign at the end)*

## 4. Cleanup
```bash
kubectl delete deployment ops-test
```
