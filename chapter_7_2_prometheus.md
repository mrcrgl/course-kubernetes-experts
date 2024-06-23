# Prometheus

## Install Prometheus

## Prepare Cluster for local storage

When using local storage, the storage needs to get provisioned 
manually. The following Helm Charts contain PVC that are getting fulfilled
with this volumes.

Please feel free to adjust the local path.

```bash
sudo mkdir -p /mnt/volumes/pv2
sudo chmod 777 /mnt/volumes/pv2

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv2
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  local:
    path: /mnt/volumes/pv2
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - $HOSTNAME
EOF

sudo mkdir -p /mnt/volumes/pv3
sudo chmod 777 /mnt/volumes/pv3

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv3
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  local:
    path: /mnt/volumes/pv3
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
helm repo add prometheus-community \
    https://prometheus-community.github.io/helm-charts
helm repo update
```

```bash
export PROMETHEUS_HOST="prometheus.kubernetes.local"
helm upgrade -n observability \
    --wait \
    --atomic \
    --create-namespace \
    --set server.ingress.enabled=true \
    --set server.ingress.hosts[0]=$PROMETHEUS_HOST \
    prometheus prometheus-community/prometheus \
    --install
```
