### **1. Pod-to-Pod Communication & Services**

**Introduction:**
This session explores how pods communicate with each other within a Kubernetes cluster, using different service types to facilitate connectivity. The session includes hands-on exercises on configuring ClusterIP and Headless Services for pod-to-pod communication.

```markdown
# Pod-to-Pod Communication & Services - Hands-On

## ClusterIP and Headless Services

### ClusterIP Service
A `ClusterIP` service provides a stable, internal IP within the cluster, allowing pods to connect to other services consistently.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

**Explanation:**
- **type: ClusterIP** creates an internal-only IP accessible by other pods within the cluster.
- **selector** targets pods with the label `app: nginx`.

### Headless Service
A `Headless Service` allows direct access to individual pod IPs, typically used for stateful applications.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

**Explanation:**
- **clusterIP: None** specifies a headless service, allowing DNS to resolve directly to pod IPs.

**Lab Steps:**
1. Deploy the `nginx` application and create both a `ClusterIP` and a `Headless Service`.
2. Verify connectivity by using `kubectl exec` to `curl` each service’s DNS name.
3. Observe the differences in connection behavior between ClusterIP and Headless services.


