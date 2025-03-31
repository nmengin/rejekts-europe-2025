# E2E TLS Connection with BackendTLSPolicy - Rejekts London

## Set up the demo

```bash
# 1. Install K3S / K3D
# 2. Generate your external and internal certificates (you can use mkcert) and put them in the folder certs
# 3. cURL to test the connection

# Set up K3S  cluster
# Disable the by-default Traefik installation
k3d cluster create e2e-tls --port 80:80@loadbalancer --port 443:443@loadbalancer --port 8080:8080@loadbalancer --port 9090:9090@loadbalancer --k3s-arg "--disable=traefik@server:0"

# Create the namespace
kubectl create namespace traefik

# Store certificates
kubectl create secret tls external-certs --namespace traefik --cert=./certs/external-crt.pem --key=./certs/external-key.pem
kubectl create secret tls internal-certs --namespace traefik --cert=./certs/internal-crt.pem --key=./certs/internal-key.pem

# Store the CA Root
kubectl create configmap internal-ca --namespace traefik --from-file ca.crt=./internal-rootCA.pem

# Installa the experimental Gateway API CRDs (not installed by default with traefik)
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/experimental-install.yaml

# Install Traefik
helm repo add --force-update traefik https://traefik.github.io/charts
helm upgrade --install --namespace traefik traefik traefik/traefik -f ./traefik_values.yaml

# Deploy the manifests
kubectl apply -f manifests

# Give it a try
curl https://whoami.docker.localhost/whoami
```