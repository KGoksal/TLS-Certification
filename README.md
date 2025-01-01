# Setting Up Kubernetes Ingress with TLS: A Comprehensive Guide 

## Overview

This guide provides a step-by-step approach to setting up an NGINX Ingress Controller in Kubernetes and securing it with TLS using Let's Encrypt. It includes detailed instructions on configuring ClusterIssuer and Issuer, generating TLS certificates, and troubleshooting common issues.

## Key Sections 

1. **Understanding ClusterIssuer vs. Issuer**
When working with TLS certificates in Kubernetes using cert-manager, it's important to understand the distinction between ClusterIssuer and Issuer. Both are used to define how certificates are issued, but they differ in scope and application.

- Issuer is a namespaced resource, meaning it only operates within a specific namespace. It is suitable for situations where you need to issue certificates for resources within a single namespace.
- ClusterIssuer is a cluster-wide resource, meaning it can be used across all namespaces. This is useful when you want a single issuer to handle certificate requests for multiple namespaces.

2. **Step-by-Step Setup** 
- **Install NGINX Ingress Controller**: Deploy using Helm.
   
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
helm install ingress-nginx ingress-nginx/ingress-nginx - namespace ingress-nginx - create-namespace
```

- **Create ClusterIssuer for Let's Encrypt**: Configure ACME server and email.

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod  # Ensure this matches the secret used for the private key
    solvers:
    - http01:
        ingress:
          class: nginx  # Make sure this matches the class of your Ingress Controller
```


```
kubectl apply -f clusterissuer.yaml
```
     
- **Create a Certificate Resource**: Define and apply TLS certificate settings.

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: your-tls
  namespace: your-portfolio  # Ensure this matches the namespace where you want to store the certificate
spec:
  secretName: your-tls  # This should be the name of the secret where the certificate will be stored
  issuerRef:
    name: letsencrypt-prod  # Must match the name of your ClusterIssuer
    kind: ClusterIssuer
  commonName: example.com  # The domain you are securing
  dnsNames:
  - example.com  # List of domains to be covered by the certificate
```

```
kubectl apply -f certificate.yaml
```
    
- **Configure Ingress Resource**: Set up TLS and route traffic to services.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: your-ingress
  namespace: your-portfolio  # Ensure this is the namespace where your services and Ingress are defined
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # Should match the name of your ClusterIssuer
  ingressClassName: nginx  # Ensure this matches the class name of your Ingress Controller
spec:
  tls:
  - hosts:
    - example.com  # The domain for which you want to use TLS
    secretName: your-tls  # Must match the secret name defined in the Certificate resource
  rules:
  - host: example.com  # The domain to route traffic to
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wordpress  # Ensure this matches the name of your service
            port:
              number: 3000  # Ensure this matches the port your service listens on
```

```
kubectl apply -f ingress.yaml
```
## References

- [Setting Up Kubernetes Ingress with TLS: A Comprehensive Guide](https://medium.com/p/2f798be9bbea/edit)
- [NGINX Ingress Controller Documentation](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager Documentation](https://cert-manager.io/docs/)
