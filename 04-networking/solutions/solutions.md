# Module 04 Solutions

### Exercise 1: Service Discovery
**Command:**
```bash
kubectl run test-curl --image=curlimages/curl -it --restart=Never -- rm
# Inside the pod:
curl http://web-service
```
*Explanation:* Kubernetes internal DNS resolves the service name to its ClusterIP.

### Exercise 2: Endpoint Inspection
**Command:**
```bash
kubectl get endpoints web-service
```
*Action:* Delete a pod (`kubectl delete pod ...`). Watch `kubectl get endpoints -w`. You will see the IP change instantly.

### Exercise 3: Multiple Paths
**Snippet (`ingress-paths.yaml`):**
```yaml
spec:
  rules:
  - host: hello.world
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /v2    # <--- New path
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Exercise 4: Host-Based Routing
**Snippet (`ingress-hosts.yaml`):**
```yaml
spec:
  rules:
  - host: hello.world
    http:
      paths:
       # ... (existing paths)
  - host: api.hello.world   # <--- New Host
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```
*Test:* `curl -H "Host: api.hello.world" http://$INGRESS_IP`

### Exercise 5: Port Mapping
**Verification:**
Look at `web-app.yaml` provided in the lab:
```yaml
ports:
  - port: 80           # Service Port (incoming)
    targetPort: 8080   # Container Port (outgoing to Pod)
```
This confirms the mapping is already in place.
