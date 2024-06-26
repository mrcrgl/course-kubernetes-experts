# Install Istio

```bash
curl -L https://github.com/istio/istio/releases/download/1.22.1/istioctl-1.22.1-linux-amd64.tar.gz  -o /tmp/istioctl-1.22.1-linux-amd64.tar.gz 
mkdir -p /tmp/istio-download
sudo tar xzvf /tmp/istioctl-1.22.1-linux-amd64.tar.gz  -C /usr/local/bin
rm -f /tmp/istioctl-1.22.1-linux-amd64.tar.gz 

istioctl version
```

```bash
istioctl analyze
```

```bash
istioctl install \
    --set profile=demo \
    --set meshConfig.enableTracing=true \
    -y

```

```bash
kubectl label namespace default istio-injection=enabled
```

### Install Example Application

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/bookinfo/platform/kube/bookinfo.yaml
```


```bash
cat << EOF | kubectl  apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bookinfo
spec:
  rules:
  - host: bookinfo.kubernetes.local
    http:
      paths:
      - backend:
          service:
            name: productpage
            port:
              number: 9080
        path: /
        pathType: Prefix
EOF
```

### Open the application to outside traffic

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/bookinfo/networking/bookinfo-gateway.yaml
```

### Install Jaeger Tracing

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/addons/jaeger.yaml
```


### Install Kiali Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/addons/kiali.yaml

```

```bash
cat << EOF | kubectl -n istio-system apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kiali
spec:
  rules:
  - host: kiali.kubernetes.local
    http:
      paths:
      - backend:
          service:
            name: kiali
            port:
              number: 20001
        path: /
        pathType: Prefix
EOF
```

Patch Kiali configuration

```bash
kubectl -n istio-system edit cm kiali
```

```yaml
    external_services:
        prometheus:
            url: http://prometheus-server.observability:80
```

### Examples


```yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: service-viewer
  namespace: default
spec:
  rules:
  - services: ["*"]
    methods: ["GET"]

```

Below is an example of ServiceRole object “product-viewer”, which has “read” (“GET” and “HEAD”) access to “products.svc.cluster.local” service at versions “v1” and “v2”. “path” is not specified, so it applies to any path in the service.


```yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: products-viewer
  namespace: default
spec:
  rules:
  - services: ["products.svc.cluster.local"]
    methods: ["GET", "HEAD"]
    constraints:
    - key: "destination.labels[version]"
      values: ["v1", "v2"]
```

```yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: test-binding-products
  namespace: default
spec:
  subjects:
  - user: alice@yahoo.com
  - properties:
      source.namespace: "abc"
  roleRef:
    kind: ServiceRole
    name: "products-viewer"
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: helloworld
spec:
  hosts:
    - helloworld
  http:
  - route:
    - destination:
        host: helloworld
        subset: v1
      weight: 90
    - destination:
        host: helloworld
        subset: v2
      weight: 10
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
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
  - name: v3
    labels:
      version: v3
```


```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ext-res-dr
spec:
  host: ext-svc.example.com
  trafficPolicy:
    connectionPool:
      tcp:
        connectTimeout: 1s
```


```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    route:
    - destination:
        host: productpage
        subset: v1

```

