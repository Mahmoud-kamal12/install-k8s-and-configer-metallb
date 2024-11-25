# 1- Install NGINX Ingress Controller
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install nginx-ingress ingress-nginx/ingress-nginx \
--namespace ingress-nginx --create-namespace
```

# 2- Install MetalLB CRDs
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/crd/bases/metallb.io_ipaddresspools.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/crd/bases/metallb.io_l2advertisements.yaml
```

### Verify that the CRDs are installed
```
kubectl get crds | grep metallb 
```
```
You should see the following output:
ipaddresspools.metallb.io
l2advertisements.metallb.io
```

# 3- Install MetalLB Components
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/manifests/metallb-native.yaml
```
# 4- Edit Configuration
```
# Set Config
export EDITOR=nano
echo "export EDITOR=nano" >> ~/.bashrc
source ~/.bashrc
kubectl edit configmap -n kube-system kube-proxy

apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
``` 
# 5- Apply Your MetalLB Configuration
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: my-ip-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.67.100-192.168.67.110
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
    - my-ip-pool
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.67.100-192.168.67.110

```

# 6- Run Deployments And SVCs
```
kubectl create deployment app1 --image=nginx --port=80
kubectl expose deployment app1 --type=LoadBalancer --port=80

kubectl create deployment app2 --image=httpd --port=80
kubectl expose deployment app2 --type=LoadBalancer --port=80

```

# 7- Create Ingress Resources
### App 1
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app1-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app1
                port:
                  number: 80
```
### App 2
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app2-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app2
                port:
                  number: 80
```

kubectl delete pod -n kube-system -l k8s-app=kube-dns


# ssl termination
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.yaml
```

# 8- Add host to /etc/hosts
```
sudo nano /etc/hosts
192.168.49.2  myapp.local
```
# 9- Connect with online domain
```
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
cloudflared login
cloudflared tunnel create my-k8s-tunnel
mkdir -p ~/.cloudflared
nano ~/.cloudflared/config.yml

tunnel: <UUID>  # Replace <UUID> with the tunnel ID
credentials-file: /root/.cloudflared/<UUID>.json

ingress:
  - hostname: isteqdamq8.com
    service: http://192.168.49.2:30933  # Your Ingress Controller IP and Port
  - service: http_status:404

```