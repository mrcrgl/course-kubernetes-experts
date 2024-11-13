# Installing the Bookinfo Application - Hands-On

## Overview

The Bookinfo application is a sample microservices app provided by Istio, consisting of several services. It serves as an ideal setup to demonstrate Istio’s traffic management, observability, and security features.

## Code Examples

### Deploying the Bookinfo Application
Apply the provided YAML file to deploy the Bookinfo application on your cluster.

```bash
# Apply the Bookinfo application manifest
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.14/samples/bookinfo/platform/kube/bookinfo.yaml
```

### Verifying Bookinfo Deployment
Confirm that all Bookinfo pods and services are running.

```bash
# Check for running pods
kubectl get pods

# Check for created services
kubectl get services
```

### Accessing the Product Page Locally
Use port-forwarding to access the product page service and verify that the application is working.

```bash
kubectl port-forward svc/productpage 9080:9080
```

**Explanation:**
- **kubectl apply -f ...** deploys the Bookinfo app, creating multiple services (`productpage`, `reviews`, `details`, `ratings`).
- **kubectl port-forward svc/productpage 9080:9080** forwards traffic from local port 9080 to the `productpage` service, allowing local access for testing.

## Lab Steps

1. **Deploy Bookinfo Application:** Apply the manifest to deploy all components of Bookinfo.
2. **Verify Deployment:** Run `kubectl get pods` to ensure that all services are running, and `kubectl get services` to check service statuses.
3. **Access Product Page Locally:** Use port-forwarding to access the product page on `http://localhost:9080/productpage`.

**Expected Outcome:**
You should see the Bookinfo application up and running, accessible via `http://localhost:9080/productpage`, with all services responding correctly.
