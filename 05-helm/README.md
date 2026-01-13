# Module 05: Package Management with Helm

## 1. Objective
Learn to manage complex Kubernetes applications using **Helm**, the "apt/yum" of Kubernetes. We will install a pre-made chart and then deconstruct a custom one.

## 2. Theory: Why Helm?
Raw YAML manifests (like `deployment.yaml`) are static.
*   *Problem:* What if I want to deploy the same app to "Dev" (1 replica) and "Prod" (10 replicas)? Do I maintain two files?
*   *Solution:* **Templating.** Helm replaces static values with variables (`{{ .Values.replicaCount }}`).

### Key Concepts
*   **Chart:** The package. A collection of templates.
*   **Repository:** The library. A server hosting Charts (like Docker Hub or `apt` repos).
*   **Release:** The running instance. You can install the *same* chart 5 times as 5 different releases.
*   **Values:** The configuration interface. Simple knobs you turn to configure the complex machinery underneath.

## 3. Installation Log

**Linux (x86-64):**
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**macOS:**
```bash
brew install helm
```

## 4. Lab Experiments

### Experiment A: Using Existing Charts (The Consumer)

1.  **Add a Repository:**
    Add the popular Bitnami repository.
    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update
    ```

2.  **Search:**
    ```bash
    helm search repo bitnami/nginx
    ```

3.  **Install (Deploy a Release):**
    Deploy Nginx, naming the release `my-web`.
    ```bash
    helm install my-web bitnami/nginx
    ```

4.  **Inspect:**
    See what was actually deployed.
    ```bash
    helm list
    kubectl get all -l app.kubernetes.io/instance=my-web
    ```

5.  **Uninstall:**
    ```bash
    helm uninstall my-web
    ```

### Experiment B: Writing a Chart (The Creator)

We have provided a minimal chart in the `simple-chart/` directory.

**Structure:**
*   `Chart.yaml`: Metadata (name, version).
*   `values.yaml`: Default configuration variables.
*   `templates/`: The Kubernetes manifests with placeholders.

1.  **Review `values.yaml`:**
    See the defaults we defined (replicaCount: 1, image: nginx).

2.  **Review `templates/deployment.yaml`:**
    Notice the Go templating syntax: `{{ .Values.image.repository }}`.

3.  **Dry Run (Debug):**
    Render the templates without installing, to see the generated YAML.
    ```bash
    helm template debug-release ./simple-chart
    ```

4.  **Install with Overrides:**
    Install the chart but override the replica count without editing the file.
    ```bash
    helm install my-custom-app ./simple-chart --set replicaCount=3
    ```

5.  **Verify:**
    ```bash
    kubectl get pods -l app=simple-chart
    ```
    *You should see 3 replicas running.*

## 5. Cleanup
```bash
helm uninstall my-custom-app
```

## 6. Exercises
*Solutions are available in the `solutions/` directory.*

1.  **Template Rendering:** Render the `simple-chart` templates to a local file named `manifest.yaml` instead of installing them.
2.  **Value Override:** Install `simple-chart` again, but this time change the `image.tag` to `latest` and `service.type` to `NodePort` using the `--set` flag.
3.  **Upgrade:** Change the `replicaCount` in your running release from 3 to 1 using the `helm upgrade` command.
4.  **History & Rollback:** Check the revision history of your release, then rollback to the previous version (where replicas were 3).
5.  **User Values:** There is a command to see *only* the values that a user supplied (overriding the defaults) for a specific release. Find and run this command.
