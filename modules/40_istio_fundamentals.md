# Istio Fundamentals

This section covers essential Istio resources—`VirtualService`, `DestinationRule`, `Gateway`, and `ServiceEntry`. Each configuration example demonstrates how these resources control traffic flow, manage external access, and apply policies within the Istio Service Mesh.

## Code Examples

### VirtualService

The `VirtualService` resource defines rules for routing traffic to services. In Istio, it allows for complex routing logic, such as traffic splitting, fault injection, and mirroring.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v1
          weight: 50
        - destination:
            host: reviews
            subset: v2
          weight: 50
```

**Explanation:**
- **hosts:** Specifies the service name (`reviews`) this VirtualService applies to.
- **route:** Defines traffic routing rules.
    - **destination:** Points to specific versions (`subsets`) of the `reviews` service.
    - **weight:** Splits traffic evenly between versions `v1` and `v2`, with each receiving 50% of requests.

This example demonstrates a basic **Canary deployment** pattern, where traffic is split between different service versions.

---

### DestinationRule

The `DestinationRule` resource defines policies for traffic directed toward a specific service, including load balancing, connection settings, and subset definitions for different versions.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
```

**Explanation:**
- **host:** Specifies the service (`reviews`) for which the rule is defined.
- **subsets:** Defines two subsets, `v1` and `v2`, each corresponding to a different version of the service.
- **trafficPolicy:** Sets a round-robin load balancing policy for requests directed at this service.

This configuration enables **targeted traffic policies** based on service versions, allowing Istio to direct traffic using labels.

---

### Gateway

A `Gateway` resource manages ingress and egress traffic to and from the mesh. It is often used to expose services to external users, acting as a bridge between the mesh and external networks.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*"
```

**Explanation:**
- **selector:** Specifies that this Gateway will use Istio’s default ingress gateway.
- **servers:** Defines a server listening on port 80 for HTTP traffic.
- **hosts:** The wildcard (`"*"`) host allows all hosts to access the gateway, though this can be restricted.

This configuration enables external access to the `bookinfo` application through port 80.

---

### ServiceEntry

The `ServiceEntry` resource allows adding external services to the mesh, enabling Istio to manage traffic to and from services outside the cluster, such as third-party APIs.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-api
spec:
  hosts:
    - external-api.example.com
  location: MESH_EXTERNAL
  ports:
    - number: 443
      name: https
      protocol: HTTPS
  resolution: DNS
```

**Explanation:**
- **hosts:** Defines the external service’s hostname (`external-api.example.com`).
- **location:** Specifies that this service is outside the mesh (`MESH_EXTERNAL`).
- **ports:** Configures the service to connect over HTTPS on port 443.
- **resolution:** Uses DNS to resolve the external service’s IP.

This configuration enables the Istio mesh to **securely interact with external services** like APIs or databases while retaining control over traffic.

