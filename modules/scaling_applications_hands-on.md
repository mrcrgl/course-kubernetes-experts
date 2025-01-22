# Hands-On Scaling Applications

1. [Install Metrics Server](../chapter_7_1_metrics_server.md)

### Install Deployment and HPA

```bash
kubectl apply -f https://github.com/mrcrgl/course-kubernetes-experts/raw/refs/heads/master/files/deployment_hpa.yaml

# Check if usage metrics are captured
kubectl top po

# Check HPA
kubectl get hpa

# Check current scaling of your application
kubectl get pod
```

### Add Service

```bash
kubectl apply -f https://github.com/mrcrgl/course-kubernetes-experts/raw/refs/heads/master/files/deployment_hpa_service.yaml

# Check Service
kubectl get service
# NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
# test-nginx          NodePort    10.101.63.21    <none>        80:31495/TCP                 85m
```

### Call API

```bash
sudo apt install wrk

# produce load
wrk -d 300s $(minikube service test-nginx --url)
```

### Watch Scaling

```bash
# Terminal 1
watch -n 1 kubectl get hpa

# Terminal 2
watch -n 1 kubectl top po

# Terminatl 3
watch -n 1 kubectl get po

```
