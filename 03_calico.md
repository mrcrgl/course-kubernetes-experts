# Calico

```bash
# Install the operator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml

# Check pods are running
kubectl -n tigera-operator get po
kubectl -n calico-system get po
```

```bash
# Install kubectl plugin
curl -L https://github.com/projectcalico/calico/releases/download/v3.28.0/calicoctl-linux-amd64 -o /usr/local/bin/kubectl-calico
chmod +x /usr/local/bin/kubectl-calico

# Verify
kubectl calico -h
```
