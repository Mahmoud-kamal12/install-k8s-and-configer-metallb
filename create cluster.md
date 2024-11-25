# remove cluster
```
sudo kubeadm reset -f
sudo rm -rf $HOME/.kube
sudo rm -rf /etc/kubernetes

sudo iptables -F && sudo iptables -X
sudo iptables -t nat -F && sudo iptables -t nat -X
sudo iptables -t mangle -F && sudo iptables -t mangle -X
sudo iptables -P FORWARD ACCEPT
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/etcd
```
# create cluster
```
sudo update-ca-certificates

systemctl restart containerd

sudo kubeadm init --pod-network-cidr=10.244.0.0/16 -v=5
sudo kubeadm init --apiserver-cert-extra-sans=<local-ip>,<public-ip> --pod-network-cidr=10.244.0.0/16 -v=5

# renew cert kubeadm init phase certs apiserver-kubelet-client
kubeadm init phase certs apiserver --apiserver-cert-extra-sans=<local-ip>,<public-ip>

192.168.1.109,41.239.209.106

# renew kubeconfig 
KUBECONFIG=~/.kube/<file-1>:~/.kube/<file-2> kubectl config view --merge --flatten > ~/.kube/merged-config.yaml
export KUBECONFIG=~/.kube/merged-config.yaml

####################################### first step #######################################

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# install flannel

kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
# remove taint
```
kubectl get nodes -A -o wide
kubectl taint nodes test-ssl node-role.kubernetes.io/control-plane:NoSchedule-

````
# install helm

```
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add stable https://charts.helm.sh/stable
helm repo update
```
# verify node status
```
watch -n 0.1 kubectl get pods -A -o wide

sudo kubeadm token create --print-join-command

```
