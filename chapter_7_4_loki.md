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

File: `fluentd_ds.yaml`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd-logging
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      name: fluentd-logging
      version: v1
      kubernetes.io/cluster-service: "true"
  template:
    metadata:
      labels:
        name: fluentd-logging
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.4-debian-forward-1 
        command: 
          - /bin/sh 
          - '-c'
          - >
            fluent-gem i fluent-plugin-grafana-loki-licence-fix ;
            tini /fluentd/entrypoint.sh;
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: config
          mountPath: /fluentd/etc
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: config
        configMap:
          name: fluentd-logging
```

File: `fluentd_configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-logging
  labels:
    app: fluentd-logging
data: 
  fluent.conf: |
    <source>
      @type tail
      path /var/log/*.log
      pos_file /var/log/fluentd/tmp/access.log.pos
      tag foo.*
      <parse>
        @type json
      </parse>
    </source>
    <source>
      @type tail
      @id in_tail_container_logs
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>
    <match fluentd.**>
      @type null
    </match>
    <match kubernetes.var.log.containers.**fluentd**.log>
      @type null
    </match>
    <filter kubernetes.**>
      @type kubernetes_metadata
      @id filter_kube_metadata
    </filter>
    <filter kubernetes.var.log.containers.**>
      @type record_transformer
      enable_ruby
      remove_keys kubernetes, docker
      <record>
        app ${ record.dig("kubernetes", "labels", "app") }
        job ${ record.dig("kubernetes", "labels", "app") }
        namespace ${ record.dig("kubernetes", "namespace_name") }
        pod ${ record.dig("kubernetes", "pod_name") }
        container ${ record.dig("kubernetes", "container_name") }
      </record>
    </filter>
    
    <match kubernetes.var.log.containers.**>
      @type copy
      <store>
        @type loki
        url "http://loki.observability.svc.cluster.local:3100"
        # extra_labels {"env":"dev"}
        label_keys "app,job,namespace,pod"
        flush_interval 10s
        flush_at_shutdown true
        buffer_chunk_limit 1m
        headers {"X-Scope-OrgID": "ACME"}
      </store>
      <store>
        @type stdout
      </store>
    </match>

```

```bash
kubectl -n observability apply -f fluentd_configmap.yaml -f fluentd_ds.yaml
```

