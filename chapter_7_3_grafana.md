# Grafana

## Install Grafana

## Prepare Cluster for local storage

When using local storage, the storage needs to get provisioned 
manually. The following Helm Charts contain PVC that are getting fulfilled
with this volumes.

Please feel free to adjust the local path.

```bash
sudo mkdir -p /mnt/volumes/pv4
sudo chmod 777 /mnt/volumes/pv4

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv4
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  local:
    path: /mnt/volumes/pv4
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - $HOSTNAME
EOF
```


```bash
helm repo add grafana \
    https://grafana.github.io/helm-charts
helm repo update
```

```bash
export GRAFANA_HOST="grafana.kubernetes.local"
helm upgrade -n observability \
    --wait \
    --atomic \
    --create-namespace \
    --set ingress.enabled=true \
    --set ingress.hosts[0]=$GRAFANA_HOST \
    --set ingress.tls[0].secretName=grafana-tls \
    --set ingress.tls[0].hosts[0]=$GRAFANA_HOST \
    --set persistence.enabled=true \
    grafana grafana/grafana \
    --install
```


```bash
export GRAFANA_HOST="grafana.kubernetes.local"
helm upgrade -n observability \
    --wait \
    --atomic \
    --create-namespace \
    --set service.type=LoadBalancer \
    --set service.ipFamilyPolicy=PreferDualStack \
    --set persistence.enabled=true \
    grafana grafana/grafana \
    --install
```

```bash
# Get admin password
kubectl get secret -n observability \
    grafana \
    -o jsonpath="{.data.admin-password}" | \
    base64 --decode ; echo
```

## Setup up sources

For Prometheus: http://prometheus-server
For Loki: http://loki:3100

## Import basic dashboard

Example: https://grafana.com/grafana/dashboards/18283-kubernetes-dashboard/

