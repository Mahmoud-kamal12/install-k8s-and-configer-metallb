# 1- Remove old cluster
```
sudo kubeadm reset -f
sudo rm /etc/kubernetes/pki/ca.crt
sudo rm -rf $HOME/.kube
sudo rm -rf /etc/kubernetes
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/etcd
```
# 2- Change hostname
```
sudo hostnamectl set-hostname <new-hostname>
sudo nano /etc/hosts
hostnamectl
```

# 3- Create cluster
```
sudo update-ca-certificates

systemctl restart containerd

sudo kubeadm init --pod-network-cidr=10.244.0.0/16 -v=5
OR 
sudo kubeadm init --apiserver-cert-extra-sans=<local-ip>,<public-ip> --pod-network-cidr=10.244.0.0/16 -v=5

# Prepare kubeconfig
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=~/.kube/config
```
# 4- Install flannel
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
# 5- Remove taint from master node
```
kubectl get nodes -A -o wide
kubectl taint nodes test-ssl node-role.kubernetes.io/control-plane:NoSchedule-

````
# 6- Install helm

```
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

# 7- Create join token
```
sudo kubeadm token create --print-join-command
``` 