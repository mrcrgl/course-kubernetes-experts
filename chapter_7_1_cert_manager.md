# Install Cert Manager

```bash

helm repo add jetstack https://charts.jetstack.io
helm repo update
```

```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.15.0 \
  --set crds.enabled=true

```

```bash
cat << EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-cluster-issuer
spec:
  selfSigned: {}
EOF
```

```bash
# Create CA certificate. If you want to use it as a ClusterIssuer the secret must be in the cert-manager namespace:
cat << EOF | kubectl -n cert-manager apply -f -
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: self-signed-ca
spec:
  isCA: true
  commonName: self-signed-ca
  secretName: self-signed-ca
  issuerRef:
    name: selfsigned-cluster-issuer
    kind: ClusterIssuer
    group: cert-manager.io
EOF
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned
spec:
  ca:
    secretName: self-signed-ca
EOF
```

### Use Lets encrypt

```bash
cat << EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: lets-encrypt
spec:
  acme:
    email: admin@eof.dev
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: lets-encrypt-private-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
      - http01:
          ingress:
            class: ""
EOF
```
```bash
cat << EOF | kubectl -n cert-manager apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: lets-encrypt-priviate-key
data:
  tls.key: (your secret key)
EOF
```
