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

Safe this content to a file on the servers filesystem:

```yaml
deploymentMode: SingleBinary
loki:
  commonConfig:
    replication_factor: 1
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
    --values values.yaml \
    loki grafana/loki
```

# Install Fluentd for log scraping


```bash
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
```

Helm Values file: `fluentd_values.yaml`

```yaml
plugins: 
- fluent-plugin-grafana-loki
configMapConfigs:
- fluentd-prometheus-conf
- fluentd-systemd-conf
fileConfigs:
  04_outputs.conf: |-
    <label @OUTPUT>
      <match **>
        @type loki
        url "http://loki.observability.svc.cluster.local:3100"
        # extra_labels {"env":"dev"}
        label_keys "app,job,namespace,pod"
        flush_interval 10s
        flush_at_shutdown true
        buffer_chunk_limit 1m
        headers {"X-Scope-OrgID": "ACME"}
      </match>
    </label>
```

```bash
helm upgrade -n observability \
    --install --atomic --wait \
    --values fluentd_values.yaml \
    fluentd fluent/fluentd
```

