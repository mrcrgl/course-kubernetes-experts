# Lab Cluster Installation

## Repeat Steps to process for controller and worker

### Prepare system packages

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg net-tools
```

### Install kubernetes basis

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

### Install container runtime (CRI)

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
sudo systemctl restart containerd
```

### Preparing the Host System

```bash
# Prepare os
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1

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

### On Controller: Init Kubernetes (kubeadm)

```bash
# Switch to user root for next steps
sudo su -
```

**Please note the pod CIDR!!!**
- 
- For Flannel: 10.244.0.0/16
- For Calico: 192.168.0.0/16

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

#### Only on single node clusters

```bash
# Untaint control plane to allow pod scheduling
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
kubectl label node $HOSTNAME node-role.kubernetes.io/worker=worker
```

### On Worker: Join the Controller

```bash
# On controller
kubeadm token create --print-join-command

# The output should look similar to:
# kubeadm join 172.31.34.137:6443 --token rd63k2.57kji98mnx0rd0rl \
#	--discovery-token-ca-cert-hash sha256:e57127a24e89ec612eb681af5a2ea4d51c2f8a901003c9d3ee1f11121667ff87 

# Run the command on each node you want to join
```

### Install Helm (optional, recommended)

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Install CNI

Please choose one of:

- [Bridge (minimal)](./cni/cni_basic_bridge.md)
- [Calico](../cni/cni_calico.md)

Verify everything works. We do this by checking if CoreDNS is running.

```bash
kubectl -n kube-system get po
```

