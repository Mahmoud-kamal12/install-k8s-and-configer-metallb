# 1- Install cert-manager
```
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true

kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.10.0/cert-manager.yaml
```

# 2- Apply configuration ClusterIssuer
```

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: mahmoud.kamal@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-secret
    solvers:
      - http01:
          ingress:
            class: nginx

```
# 3- Apply configuration Certificate
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: letsencrypt-tls
  namespace: default
spec:
  secretName: letsencrypt-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
    - <my-domain>
  usages:
    - digital signature
    - key encipherment
```
# 4- Change Ingress configuration
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: my-ingress
    namespace: default
    annotations:
        nginx.ingress.kubernetes.io/ssl-redirect: "true"
        nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
        cert-manager.io/cluster-issuer: "letsencrypt-prod"
        nginx.ingress.kubernetes.io/rewrite-target: /
spec:
    tls:
        - hosts:
            - <my-domain>
            secretName: letsencrypt-tls
    rules:
        - host: <my-domain>
            http:
                paths:
                    - path: /app1
                        pathType: Prefix
                        backend:
                            service:
                                name: app-1
                                port:
                                    number: 80
```