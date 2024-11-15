# Installing the Bookinfo Application with Istio Gateway - Hands-On

## Overview

The Bookinfo application is a sample microservices app provided by Istio, consisting of several services (`productpage`, `details`, `reviews`, and `ratings`). This guide includes deploying the application and configuring an Istio Gateway for external access.

---

## Part 1: Deploying the Bookinfo Application

### Deployment Steps

1. **Apply the Bookinfo Application YAML**
   Deploy all services and deployments for the Bookinfo application.

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.14/samples/bookinfo/platform/kube/bookinfo.yaml
   ```

2. **Verify the Deployment**
   Ensure that all pods and services are created and running:

   ```bash
   # List pods
   kubectl get pods

   # List services
   kubectl get services
   ```

3. **Access the Product Page Locally**
   Use port-forwarding to access the `productpage` service and verify the application works:

   ```bash
   kubectl port-forward svc/productpage 9080:9080
   ```

   - Visit `http://localhost:9080/productpage` to confirm functionality.

---

## Part 2: Configuring an Istio Gateway for External Access

### Gateway Configuration

1. **Apply the Istio Gateway and VirtualService**
   Use the provided YAML file to configure an Istio Gateway and VirtualService to expose the `productpage` service externally.

   ```yaml
   apiVersion: networking.istio.io/v1alpha3
   kind: Gateway
   metadata:
     name: bookinfo-gateway
   spec:
     selector:
       istio: ingressgateway # Use Istio's built-in ingress gateway
     servers:
     - port:
         number: 80
         name: http
         protocol: HTTP
       hosts:
       - "*"

   ---
   apiVersion: networking.istio.io/v1alpha3
   kind: VirtualService
   metadata:
     name: bookinfo
   spec:
     hosts:
     - "*"
     gateways:
     - bookinfo-gateway
     http:
     - match:
       - uri:
           prefix: /productpage
       route:
       - destination:
           host: productpage
           port:
             number: 9080
   ```

   Apply the configuration:

   ```bash
   kubectl apply -f bookinfo-gateway.yaml
   ```

2. **Verify the Gateway and VirtualService**
   Check that the Gateway and VirtualService have been created:

   ```bash
   kubectl get gateway
   kubectl get virtualservice
   ```

---

## Part 3: Testing External Access

1. **Retrieve the External IP of the Istio Ingress Gateway**
   Use the following command to get the external IP address:

   ```bash
   kubectl get svc istio-ingressgateway -n istio-system
   ```

   - Note the `EXTERNAL-IP` from the output.

2. **Access the Application**
   Open your browser and navigate to `http://<EXTERNAL-IP>/productpage` to access the Bookinfo application.

---

## Troubleshooting

1. **Pods Not Running:**
   Check for pod errors:
   ```bash
   kubectl describe pod <pod-name>
   ```

2. **No External IP for Gateway:**
   Ensure your Kubernetes cluster supports external LoadBalancers. If not, use port-forwarding as a workaround:
   ```bash
   kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
   ```

   Access the application at `http://localhost:8080/productpage`.

3. **Ingress Gateway Logs:**
   Check the logs of the Istio Ingress Gateway:
   ```bash
   kubectl logs -n istio-system -l app=istio-ingressgateway
   ```

---

## Expected Outcome

1. All Bookinfo services (`productpage`, `details`, `reviews`, `ratings`) should be running.
2. The application should be accessible via the Istio Gateway at `http://<EXTERNAL-IP>/productpage`.
3. External requests to `/productpage` should be routed correctly through the Gateway to the `productpage` service.

