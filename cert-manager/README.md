# Cert Manager

This cluster uses cert-manager to issue TLS certificates for services via a self-signed CA, bootstrapped inside Kubernetes.

# Setup

1. Bootstrap CA inside cert-manager, create a file called `cert-manager-ca-bootstrap.yaml`, This creates a root CA (my-ca-secret) inside the cluster.

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-bootstrap
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-ca
  namespace: cert-manager
spec:
  isCA: true
  commonName: my-ca
  secretName: my-ca-secret
  issuerRef:
    name: selfsigned-bootstrap
    kind: ClusterIssuer

```

2. Create a permanent CA issuer `cert-manager-clusterissuer-my-ca.yaml`

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: my-ca-issuer
spec:
  ca:
    secretName: my-ca-secret
```

3. You should now be able to request certificates using the self-signed issuer, e.g. for n8n `certificate-n8n.yaml`

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: n8n-cert
  namespace: n8n
spec:
  secretName: ingress-n8n-tls
  duration: 2160h
  renewBefore: 360h
  issuerRef:
    name: my-ca-issuer
    kind: ClusterIssuer
  commonName: n8n-192.168.1.42.nip.io
  dnsNames:
    - n8n-192.168.1.42.nip.io
```
