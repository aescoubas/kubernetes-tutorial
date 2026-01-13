# Module 02: Configuration Management

## 1. Objective
Inject configuration data and sensitive credentials into Pods without modifying the container image.

## 2. Concepts

*   **ConfigMap:** Stores non-confidential data in key-value pairs.
*   **Secret:** Stores sensitive data (passwords, keys) encoded in base64.
*   **Environment Variables:** One way to expose this data to the container.
*   **Volumes:** Another way to expose this data (as files).

## 3. Lab Experiments

### Experiment A: Creating Data
*Files: `configmap.yaml`, `secret.yaml`*

1.  **Inspect `secret.yaml`:** Note that values are base64 encoded.
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

## 4. Cleanup
```bash
kubectl delete -f .
```
