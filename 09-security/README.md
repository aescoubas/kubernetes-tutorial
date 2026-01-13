# Module 09: Security & Permissions (RBAC)

## 1. Objective
Understand how Kubernetes handles multi-tenancy using **Namespaces** and how it secures API access using **Role-Based Access Control (RBAC)**.

## 2. Theory: The Walls of the Cluster

### Namespaces (Logical Partitioning)
A Namespace is a virtual cluster inside your physical cluster.
*   **Use Cases:** `dev` vs `prod`, `team-a` vs `team-b`.
*   **Isolation:** Resources in one namespace (like Pods/Services) are hidden from others unless explicitly exposed.
*   **Scope:** Most resources (Pods, Deployments) are *Namespaced*. Some (Nodes, PVs) are *Cluster-Wide*.

### ServiceAccounts (Machine Identity)
Users have accounts (OIDC/Certificates). Pods have **ServiceAccounts**.
*   When a Pod talks to the API Server (e.g., a Jenkins agent launching builds), it uses the token mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token` to authenticate as its ServiceAccount.

### RBAC (The Authorization Layer)
Once authenticated (User or ServiceAccount), **RBAC** decides what they can do.
1.  **Role:** A set of rules (permissions) within a *Namespace*. (e.g., "Can read Pods in `dev`").
2.  **ClusterRole:** A set of rules *Cluster-wide*. (e.g., "Can read Nodes anywhere").
3.  **RoleBinding:** Connects a **Subject** (User/SA) to a **Role**.
4.  **ClusterRoleBinding:** Connects a **Subject** to a **ClusterRole**.

## 3. Lab Experiments

### Experiment A: Namespace Isolation
1.  **Create Namespaces:**
    ```bash
    kubectl create ns dev
    kubectl create ns prod
    ```
2.  **Deploy Identical Apps:**
    We can have two deployments named `web` because they are in different "rooms".
    ```bash
    kubectl create deployment web --image=nginx --replicas=1 -n dev
    kubectl create deployment web --image=httpd --replicas=1 -n prod
    ```
3.  **Verify:**
    ```bash
    kubectl get pods -n dev
    kubectl get pods -n prod
    ```

### Experiment B: The Forbidden User (RBAC)
We will create a ServiceAccount that has NO permissions, then grant it access.

1.  **Create ServiceAccount:**
    ```bash
    kubectl create sa rookie -n dev
    ```
2.  **Test Access (Can I?):**
    Use `auth can-i` to simulate the user.
    ```bash
    kubectl auth can-i get pods --as=system:serviceaccount:dev:rookie -n dev
    ```
    *Output: `no` (Default is deny-all).*

3.  **Create Role:**
    *File: `pod-reader-role.yaml`*
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: dev
      name: pod-reader
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
    ```
    ```bash
    kubectl apply -f pod-reader-role.yaml
    ```

4.  **Bind Role:**
    *File: `rookie-binding.yaml`*
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: read-pods-global
      namespace: dev
    subjects:
    - kind: ServiceAccount
      name: rookie
      namespace: dev
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io
    ```
    ```bash
    kubectl apply -f rookie-binding.yaml
    ```

5.  **Verify Access:**
    ```bash
    kubectl auth can-i get pods --as=system:serviceaccount:dev:rookie -n dev
    ```
    *Output: `yes`.*

    **Check Forbidden Action:**
    ```bash
    kubectl auth can-i delete pods --as=system:serviceaccount:dev:rookie -n dev
    ```
    *Output: `no`.*

## 4. Cleanup
```bash
kubectl delete ns dev prod
```

## 5. Exercises
*Solutions are available in the `solutions/` directory.*

1.  **ClusterRole:** Create a `ClusterRole` named `node-viewer` that allows listing `nodes`. Then bind it to the `rookie` ServiceAccount (even though rookie is in a namespace, it can have cluster-wide powers).
2.  **Context Hacking:** Kubernetes config files (`~/.kube/config`) allow you to define "Contexts". Create a context named `dev-viewer` that sets the default namespace to `dev` and the user to `minikube`.
3.  **ServiceAccount Token:** Retrieve the JWT token for the `rookie` ServiceAccount and decode it (using `jq` or a website like jwt.io) to see the payload. (Hint: In k8s 1.24+, tokens are not auto-created as Secrets anymore, you must generate one).
4.  **View-Only Admin:** There is a built-in ClusterRole called `view`. Bind this to a user named `jane` in the `prod` namespace. What can she see?
5.  **Namespace ResourceQuota:** Combine Module 07 with Module 09. Create a ResourceQuota in the `dev` namespace that limits the total CPU count to `500m`.

## 6. References & Documentation
*   [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
*   [Configure Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
*   [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
