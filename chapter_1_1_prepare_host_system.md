# Installation

## Prepare system packages

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg net-tools
```

## Install kubernetes basis

```bash
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# Add repositories
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install Kubernetes Packages
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl 
sudo apt-mark hold kubelet kubeadm kubectl
```

```bash
# Enable kubelet
sudo systemctl enable --now kubelet
```

## Install container runtime (CRI)

```bash
# Install container runtime
sudo apt-get install containerd
```

```bash
# Fix default configuration 
sudo mkdir -p /etc/containerd/
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

```bash
sudo systemctl enable --now containerd
```

## Vorbereitung des Systems

```bash
# Prepare os
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1

# or
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo vim /etc/sysctl.conf

# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1

# Uncomment the next line to enable packet forwarding for IPv6
#  Enabling this option disables Stateless Address Autoconfiguration
#  based on Router Advertisements for this host
#net.ipv6.conf.all.forwarding=1

```

```bash
# Make sure swap is disabled
cat /proc/swaps
```

```bash
sudo reboot
```

## Init Kubernetes (kubeadm)

```bash
# Switch to user root for next steps
sudo su -
```

```bash
# We're using the default settings here
kubeadm init --pod-network-cidr=10.88.0.0/16

# Copy admin kubeconfig
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```bash
# Ensure everything is running
kubectl get no

kubectl -n kube-system get po

# Wait 10 minutes and try again.
# Some misconfigurations like malformed cgroups etc will cause errors
# after some time (CrashLoopBackOff).

# It's a good time to grab a coffee
```

### Only on single node clusters

```bash
# Untaint control plane to allow pod scheduling
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
kubectl label node $HOSTNAME node-role.kubernetes.io/worker=worker
```

# Install CNI

Please choose one of:

- [Bridge (minimal)](./cni/cni_basic_bridge.md)
- [Calico](./cni/cni_calico.md)

Verify everything works. We do this by checking if CoreDNS is running.

```bash
kubectl -n kube-system get po
```

# Install Helm

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

# Install ingress controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

```bash
helm upgrade -n ingress-nginx --create-namespace \
    --install --wait --atomic \
    --set controller.kind=DaemonSet \
    --set controller.ingressClass=default \
    --set controller.watchIngressWithoutClass=true \
    --set controller.service.enabled=false \
    --set controller.hostPort.enabled=true \
    --set controller.extraArgs.publish-status-address=127.0.0.1 \
    ingress-nginx ingress-nginx/ingress-nginx
```

```bash
# Disable HSTS for local development
kubectl patch configmap/ingress-nginx-controller \
  -n ingress-nginx \
  --type merge \
  -p '{"data":{"hsts":"false"}}'

```

# Optional

## Setup local storage class

```bash
cat << EOF | kubectl apply -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF
```

### Install etcdctl

For debugging purposes, I suggeest installing etcdctl.

```bash
# Install etcdctl
ETCD_VER=v3.4.33

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version

sudo cp /tmp/etcd-download-test/etcdctl /usr/local/bin
```

