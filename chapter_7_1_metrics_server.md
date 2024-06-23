# Metrics Server Setup

The metrics server is important for the HorizontalPodAutoscaler to work.
Observability is not related to it.

```bash
helm repo add metrics-server \
    https://kubernetes-sigs.github.io/metrics-server/

# Note that this deployment skips tls verification 
# and is not suited for productive use
helm upgrade -n kube-system \
    --install \
    --set args[0]=--kubelet-insecure-tls \
    metrics-server metrics-server/metrics-server
```

