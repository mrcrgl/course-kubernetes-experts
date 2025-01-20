# Hands-On: Introduction to Helm

## Objectives
In this hands-on, you will:
1. Add a Helm repository.
2. Search for a Helm chart.
3. Install a Helm chart to deploy an application.
4. Customize the deployment using a values file.
5. Roll back a release.
6. Uninstall a release.

---

## Prerequisites
- A Kubernetes cluster is set up and running.
- `kubectl` and `helm` are installed and configured.

---

## Step 1: Add a Helm Repository
1. Add the **Bitnami** repository:
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```
2. Update your Helm repository index:
   ```bash
   helm repo update
   ```

---

## Step 2: Search for a Helm Chart
1. Search for the **nginx** chart:
   ```bash
   helm search repo nginx
   ```
2. Note the available charts and versions.

---

## Step 3: Install an Application
1. Install the **nginx** chart:
   ```bash
   helm install my-nginx bitnami/nginx
   ```
2. Verify the deployment:
   ```bash
   kubectl get all
   ```
3. Access the application:
   - Use `kubectl get svc` to find the **NodePort** or **LoadBalancer** IP.

---

## Step 4: Customize the Deployment
1. Create a `values.yaml` file with the following content to modify the service type:
   ```yaml
   service:
     type: NodePort
   ```
2. Upgrade the release with the custom values:
   ```bash
   helm upgrade my-nginx bitnami/nginx -f values.yaml
   ```
3. Verify the changes:
   ```bash
   kubectl get svc my-nginx
   ```

---

## Step 5: Rollback a Release
1. List the release history:
   ```bash
   helm history my-nginx
   ```
2. Roll back to a previous revision:
   ```bash
   helm rollback my-nginx <revision-number>
   ```
3. Verify the rollback:
   ```bash
   kubectl get all
   ```

---

## Step 6: Uninstall a Release
1. Uninstall the release:
   ```bash
   helm uninstall my-nginx
   ```
2. Verify the release is removed:
   ```bash
   kubectl get all
   ```

---

## Additional Resources
- [Helm Documentation](https://helm.sh/docs/)
- [ArtifactHub](https://artifacthub.io/)
