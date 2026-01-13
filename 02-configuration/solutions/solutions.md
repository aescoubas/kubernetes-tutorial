# Module 02 Solutions

### Exercise 1: Literal Config
**Command:**
```bash
kubectl create configmap game-config --from-literal=lives=3
```

### Exercise 2: All-in-One Env (`envFrom`)
**Manifest (`ex2-envfrom.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo-envfrom
spec:
  containers:
    - name: demo-container
      image: alpine
      command: ["/bin/sh", "-c", "sleep 3600"]
      envFrom:            # <--- The key change
        - configMapRef:
            name: app-config
```

### Exercise 3: Secret Decoding
**Command:**
```bash
echo -n 'RWlnaHQgTWlsZQ==' | base64 --decode
```
*Answer:* `Eight Mile`

### Exercise 4: File-based Secret
**Commands:**
```bash
# Create dummy file
echo "SECRET_KEY_CONTENT" > id_rsa

# Create secret
kubectl create secret generic ssh-key-secret --from-file=id_rsa
```

### Exercise 5: Environment Precedence
**Hypothesis:** The explicit `env` definition in the Pod spec usually overrides the `envFrom` or `configMapKeyRef`.

**Manifest (`ex5-precedence.yaml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: precedence-test
spec:
  containers:
    - name: test
      image: alpine
      command: ["/bin/sh", "-c", "sleep 3600"]
      env:
        - name: APP_COLOR    # Explicit definition
          value: "RED"
      envFrom:
        - configMapRef:      # Contains APP_COLOR=blue
            name: app-config
```
**Verification:**
```bash
kubectl exec precedence-test -- printenv APP_COLOR
```
*Result:* `RED`. Explicit values override injected config maps.
