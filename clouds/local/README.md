# Hyperkube

[Hyperkube](https://squareops.com) is the project for deploying [Hyperledger](https://www.hyperledger.org/) Fabric permissioned blockchain framework over kubernetes.

For simplicity, this project is divided in 2 stages:
- Dev Setup
- Production Setup

# Dev Setup
## Install Kubernetes
You can either use managed kubernetes , or install one for yourself. Below are the instructions for manual installation of kubernetes

Please note that ubuntu 18 is not yet officialy supported, Hence run it on ubuntu 16.04(Xenial). 
Requires minimum2 GB RAM and 2 CPUs on each node

Install docker on ubuntu 16.04
```bash
sudo apt-get update -y && sudo apt-get install -y apt-transport-https curl
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt-get update -y && sudo apt-get install -y docker-ce kubelet kubeadm kubectl

# change docker CGROUP driver from cgroupfs to systemd
cat << EOF | sudo tee /etc/docker/daemon.json 
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload && sudo systemctl restart docker

#Add host mapping for all nodes in /etc/hosts file

# initilize kubernetes master
sudo kubeadm init  --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<private_ip_of_server>

#Configure kubectl on master:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#Add Calico networking plugin:
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

# check status:
kubectl get nodes


## Add node
# repeat installation steps on node 
# run below command as a result of kubeadm init
kubeadm join 10.160.0.6:6443 --token ijghom.dk7oflu8j14kgxxq --discovery-token-ca-cert-hash sha256:1f16f9064d4bf3c4b4951fb9f4d7c8bd331b1de2f970c5c3fdec9ef1510bc74f

kubectl get nodes
kubectl get pods --all-namespaces

# deploy kubernetes web UI 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
kubectl proxy --address='<private_ip_of_vm>' --accept-hosts='^<public_ip_of_VM>$' &
#access dashbaord at:
http://35.244.38.131:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

#follow instructions here to get auth token for login:
https://github.com/kubernetes/dashboard/wiki/Creating-sample-user


Install Helm

wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz
tar -zxvf helm-v2.0.0-linux-amd64.tgz 
sudo mv linux-amd64/helm /usr/local/bin/helm

kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
helm init --service-account tiller
helm list

```

