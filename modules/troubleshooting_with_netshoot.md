# Network Troubleshooting with Netshoot - Hands-On

## Deploying Netshoot and Diagnosing Connectivity

### Netshoot Deployment Command
Deploy the Netshoot container with an interactive shell for troubleshooting.

```bash
kubectl run netshoot --rm -it --image nicolaka/netshoot -- /bin/bash
```

**Lab Steps:**
1. **Ping and Curl** - Test connectivity to other pods or services.
   ```bash
   ping <pod-ip>
   curl <service-url>
   ```
2. **DNS Check** - Verify DNS resolution within the cluster.
   ```bash
   nslookup <service-name>
   ```
3. **Network Policy Validation** - Confirm that network policies are allowing or blocking expected traffic.

