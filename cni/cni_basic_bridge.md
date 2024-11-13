## Install Basic CNI

```bash
# install cni plgins
sudo mkdir -p /usr/lib/cni

curl -L  https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz -o /tmp/cni-plugins-linux-amd64-v1.5.1.tgz
sudo tar xzvf /tmp/cni-plugins-linux-amd64-v1.5.1.tgz -C /usr/lib/cni --strip-components=1
rm /tmp/cni-plugins-linux-amd64-v1.5.1.tgz

```

```bash
# add default network configuration
sudo mkdir -p /etc/cni/net.d
cat << EOF | sudo tee /etc/cni/net.d/10-containerd-net.conflist
{
  "cniVersion": "1.0.0",
  "name": "containerd-net",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "cni0",
      "isGateway": true,
      "ipMasq": true,
      "promiscMode": true,
      "ipam": {
        "type": "host-local",
        "ranges": [
          [{
            "subnet": "10.88.0.0/16"
          }],
          [{
            "subnet": "2001:4860:4860::/64"
          }]
        ],
        "routes": [
          { "dst": "0.0.0.0/0" },
          { "dst": "::/0" }
        ]
      }
    },
    {
      "type": "portmap",
      "capabilities": {"portMappings": true}
    }
  ]
}
EOF
```