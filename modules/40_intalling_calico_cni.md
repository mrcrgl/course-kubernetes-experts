# Install Calico CNI

## Using Helm (recommended)

```bash
helm repo add projectcalico https://docs.tigera.io/calico/charts
helm repo update
```

```bash
helm upgrade -n tigera-operator \
 --create-namespace --install --atomic --wait \
 --version v3.28.0 \
 --set installation.cni.type=Calico \
 --set installation.calicoNetwork.bgp=Disabled \
 --set installation.calicoNetwork.ipPools[0].cidr=192.168.0.0/16 \
 --set installation.calicoNetwork.ipPools[0].encapsulation=VXLAN \
 calico projectcalico/tigera-operator
```

## Using raw manifests (alternative)

```bash
# Install the operator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml

# Check pods are running
kubectl -n tigera-operator get po
kubectl -n calico-system get po
```

## Install Tools (optional)

```bash
# Install kubectl plugin
curl -L https://github.com/projectcalico/calico/releases/download/v3.28.0/calicoctl-linux-amd64 -o /usr/local/bin/kubectl-calico
chmod +x /usr/local/bin/kubectl-calico

# Verify
kubectl calico -h
```