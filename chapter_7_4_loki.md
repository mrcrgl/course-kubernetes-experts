# Install Loki for Logs


```bash
sudo mkdir -p /mnt/volumes/pv5
sudo chmod 777 /mnt/volumes/pv5

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv5
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  local:
    path: /mnt/volumes/pv5
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

## Configure the Chart for our Test environment

Save this content to a file on the servers filesystem: `loki_values.yaml`

```yaml
deploymentMode: SingleBinary
loki:
  commonConfig:
    replication_factor: 1
  config:
    auth_enabled: false
  storage:
    type: 'filesystem'
  schemaConfig:
    configs:
    - from: "2024-01-01"
      store: tsdb
      index:
        prefix: loki_index_
        period: 24h
      object_store: filesystem # we're storing on filesystem so there's no real persistence here.
      schema: v13
singleBinary:
  replicas: 1
read:
  replicas: 0
backend:
  replicas: 0
write:
  replicas: 0
```

```bash
helm upgrade \
    -n observability \
    --install --atomic --wait \
    --values loki_values.yaml \
    loki grafana/loki
```

# Install Promtail for log scraping


```bash
helm upgrade -n observability \
    --install --atomic --wait \
    promtail grafana/promtail
```

