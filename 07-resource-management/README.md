# Module 07: Resource Management

## 1. Objective
Understand how Kubernetes allocates hardware (CPU/Memory) to Pods, how to prevent "noisy neighbors," and what happens when the cluster runs out of resources.

## 2. Theory: The Scheduler's Wallet

When you run a Pod, the Scheduler looks for a Node with enough free space. But how does it know what is "free"? It relies on **Requests**.

### CPU & Memory
*   **Requests (The Guarantee):** "I need at least this much to start."
    *   The Scheduler *reserves* this amount. No other Pod can claim it.
    *   If no node has enough unreserved space, the Pod stays `Pending`.
*   **Limits (The Ceiling):** "I will never use more than this."
    *   **CPU:** Throttled. If you hit the limit, the app slows down.
    *   **Memory:** **OOMKilled**. If you hit the limit, the Kernel kills the container.

### Units
*   **CPU:** 1000m (millicores) = 1 vCPU. `500m` is half a core.
*   **Memory:** `128Mi` (Mebibytes) or `1G` (Gigabytes).

### Quality of Service (QoS)
Kubernetes assigns a class based on your config:
1.  **Guaranteed:** Request == Limit (Safe, high priority).
2.  **Burstable:** Request < Limit (Flexible, standard).
3.  **BestEffort:** No Request/Limit (Dangerous, first to be killed).

## 3. Lab Experiments

### Experiment A: The "Guaranteed" Pod
*File: `guaranteed-pod.yaml`*

1.  **Deploy:**
    ```bash
    kubectl apply -f guaranteed-pod.yaml
    ```
2.  **Inspect:**
    ```bash
    kubectl describe pod guaranteed-pod | grep QoS
    ```
    *Expected: `QoS Class: Guaranteed`*

### Experiment B: The "OOMKilled" (Out of Memory)
*File: `oom-pod.yaml`*

We will limit a pod to 50Mi of RAM but try to consume 100Mi.

1.  **Deploy:**
    ```bash
    kubectl apply -f oom-pod.yaml
    ```
2.  **Watch it die:**
    ```bash
    kubectl get pod oom-pod -w
    ```
    *Status will cycle: `Running` -> `OOMKilled` -> `CrashLoopBackOff`.*

### Experiment C: The "Pending" (Insufficient Capacity)
*File: `greedy-pod.yaml`*

We will ask for massive resources that our Minikube nodes (2GB RAM) cannot provide.

1.  **Deploy:**
    ```bash
    kubectl apply -f greedy-pod.yaml
    ```
2.  **Check Status:**
    ```bash
    kubectl get pod greedy-pod
    ```
    *Status: `Pending`*

3.  **Why is it Pending?**
    ```bash
    kubectl describe pod greedy-pod
    ```
    *Look for "Events". Message: `0/4 nodes are available: 4 Insufficient cpu.`*

## 4. Cleanup
```bash
kubectl delete -f .
```

## 5. Exercises
*Solutions are available in the `solutions/` directory.*

1.  **Burstable Config:** Create a Pod named `burstable` that requests `100m` CPU but has a limit of `500m` CPU. Check its QoS class.
2.  **Resource Quota:** Create a `ResourceQuota` in a new namespace `quota-test` that limits the *total* number of pods allowed to 2. Try to launch 3 pods.
3.  **LimitRange:** Create a `LimitRange` in a namespace that automatically assigns a default memory limit of `200Mi` to any pod that doesn't specify one. Launch a naked pod and verify it got the limit.
4.  **CPU Throttling:** (Conceptual/Research) If a Pod hits its Memory limit, it dies (OOM). What exactly happens when it hits its CPU limit? Does it crash?
5.  **Node Capacity:** Find out exactly how much "allocatable" CPU and Memory `minikube-m02` has. (Hint: `kubectl describe node`).

## 6. References & Documentation
*   [Resource Management for Pods](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
*   [Configure Quality of Service (QoS)](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)
*   [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
*   [Limit Ranges](https://kubernetes.io/docs/concepts/policy/limit-range/)