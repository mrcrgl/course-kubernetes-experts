# Introduction to Service Mesh - Hands-On

## Overview

This session introduces the concept of a Service Mesh, its importance in microservices architecture, and how Istio enhances service-to-service communication with observability, traffic management, and security.

## Code Examples

### Deploying Istio with the Demo Profile
The demo profile includes all Istio features and is suitable for a hands-on setup.

```bash
# Download Istio CLI
curl -L https://istio.io/downloadIstio | sh -

# Move into the Istio package directory
cd istio-*

# Install Istio using the demo profile
istioctl install --set profile=demo -y

# Enable sidecar injection in the default namespace
kubectl label namespace default istio-injection=enabled
```

**Explanation:**
- **istioctl install --set profile=demo** installs Istio with the full feature set, suitable for testing and learning.
- **istio-injection=enabled** automatically injects sidecars into pods within the default namespace, enabling Istio's features for each service.

## Lab Steps

1. **Install Istio:** Follow the commands above to install Istio and enable sidecar injection in the default namespace.
2. **Verify Installation:** Run `kubectl get pods -n istio-system` to ensure all Istio components are running.
3. **Check Sidecar Injection:** Deploy a sample pod in the default namespace and verify that an Envoy sidecar is automatically injected by running `kubectl get pods -o yaml | grep -i envoy`.

**Expected Outcome:**
You should see the `istiod` control plane and other Istio components running in the `istio-system` namespace. You should also verify that sidecars are automatically injected into pods in the default namespace.
