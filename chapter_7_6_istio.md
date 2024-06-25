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
