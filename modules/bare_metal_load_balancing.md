# Bare-Metal Load Balancing with MetalLB - Hands-On

## Configuring MetalLB with Layer 2 Mode

### MetalLB ConfigMap
A ConfigMap for MetalLB in Layer 2 mode allows assigning IPs within a predefined range.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.240-192.168.1.250
```

**Explanation:**
- **protocol: layer2** configures MetalLB to use ARP-based IP allocation within the specified range `192.168.1.240-192.168.1.250`.

**Lab Steps:**
1. Deploy MetalLB with `kubectl apply -f <URL>`.
2. Apply the ConfigMap to configure IP allocation.
3. Deploy a service of type `LoadBalancer`, such as `nginx`, and confirm it receives an external IP within the specified range.
