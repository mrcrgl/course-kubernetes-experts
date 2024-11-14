# Ingress Controllers & Load Balancers - Hands-On

## Ingress for Path-Based Routing

### Ingress Resource Example
The `Ingress` resource provides HTTP(S) routing to services within the cluster, using rules to direct traffic based on paths.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: "example.com"
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 80
```

**Explanation:**
- **host** sets a virtual hostname for this Ingress, `example.com`.
- **path** and **backend** allow traffic to be routed to `app1-service` for `/app1` and `app2-service` for `/app2`.

**Lab Steps:**
1. Deploy `app1` and `app2` applications with appropriate services.
2. Apply the Ingress configuration and verify access by simulating requests to `http://example.com/app1` and `http://example.com/app2`.
3. Optionally, add a TLS certificate for HTTPS.

## Expose Service using NodePort

```bash
kubectl expose deployment nginx-deployment --type=NodePort --port=80
```
