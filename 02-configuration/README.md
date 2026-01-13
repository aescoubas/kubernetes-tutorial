# Module 02: Configuration Management

## 1. Objective
Inject configuration data and sensitive credentials into Pods without modifying the container image.

## 2. Theory: The 12-Factor App
A core principle of modern apps is strict separation of **Config** from **Code**.
*   **Code:** The Docker image (immutable, same across Dev/Test/Prod).
*   **Config:** The environment-specific settings (DB URLs, API keys, feature flags).

Kubernetes solves this with **ConfigMaps** (plain text) and **Secrets** (obfuscated).

## 3. Concepts
*   **ConfigMap:** A dictionary of non-confidential data. Can be injected as Environment Variables or mounted as config files (e.g., `nginx.conf`).
*   **Secret:** Similar to ConfigMaps but intended for sensitive data.
    *   *Warning:* By default, Secrets are only base64-encoded, **not encrypted**. Anyone with API access can decode them. In production, you enable *Encryption at Rest* or use external vaults.
*   **Environment Variables:** The most common way to inject simple values (Key=Value).
*   **Volumes:** Used for complex config files. The file appears in the container filesystem. If you update the ConfigMap, the file in the container updates automatically (eventually).

## 4. Lab Experiments

### Experiment A: Creating Data
*Files: `configmap.yaml`, `secret.yaml`*

1.  **Inspect `secret.yaml`:**
    *   *Concept:* Kubernetes expects base64 values in the manifest to avoid newlines breaking YAML.
    *   *To encode:* `echo -n 'my-password' | base64`
    *   *To decode:* `echo -n 'bXktcGFzc3dvcmQ=' | base64 --decode`
2.  **Apply configurations:**
    ```bash
    kubectl apply -f configmap.yaml
    kubectl apply -f secret.yaml
    ```

### Experiment B: Consuming Data
*File: `pod-with-config.yaml`*

1.  **Deploy the Pod:**
    ```bash
    kubectl apply -f pod-with-config.yaml
    ```
2.  **Verify Environment Injection:**
    Check if the environment variables exist inside the container.
    ```bash
    kubectl exec config-demo-pod -- printenv | grep APP_
    ```
    *Expected Output:*
    ```text
    APP_COLOR=blue
    APP_MODE=production
    APP_SECRET_KEY=supersecretkey
    ```
3.  **Verify Volume Mount:**
    Check if the ConfigMap was mounted as a file.
    ```bash
    kubectl exec config-demo-pod -- cat /etc/config/app.properties
    ```

## 5. Cleanup
```bash
kubectl delete -f .
```

## 6. Exercises
*Solutions are available in the `solutions/` directory.*

1.  **Literal Config:** Create a ConfigMap named `game-config` containing the key `lives` with value `3`, using *only* the command line (no YAML).
2.  **All-in-One Env:** Modify `pod-with-config.yaml` so that *all* key-value pairs in the `app-config` ConfigMap are injected as environment variables at once (using `envFrom`), instead of specifying them one by one.
3.  **Secret Decoding:** You found a secret in the cluster. The value is `RWlnaHQgTWlsZQ==`. Decode it to find the street name.
4.  **File-based Secret:** Create a Secret named `ssh-key-secret` from a local file named `id_rsa` (you can create a dummy file for this).
5.  **Environment Precedence:** If you define an environment variable manually in the Pod spec *and* inject the same variable name from a ConfigMap, which one wins? Create a test pod to find out.
