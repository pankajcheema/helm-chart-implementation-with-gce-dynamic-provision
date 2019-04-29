# Install Kubernetes
You can either use managed kubernetes , or install one for yourself. Below are the instructions for manual installation of kubernetes on GCE VMs using kubeadm

Please note that ubuntu 18 is not yet officialy supported, Hence run it on ubuntu 16.04(Xenial). 
Requires minimum 2 GB RAM and 2 CPUs cores on each node

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
```
### Compute VM Nodes are discoverable by default over their privae network hostnames in Google Cloud . Add host mapping for all nodes in /etc/hosts file if this discovery is failing

## Add GCE cloud provide for dynamic volume provisioning
update /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
	* Add: --cloud-provider=gce in KUBELET_CONFIG_ARGS
```bash 
sudo systemctl daemon-reload && sudo systemctl restart kubelet

cat << EOF | sudo tee /etc/kubernetes/gce.conf
[Global]
account = 960758852425-compute@developer.gserviceaccount.com
project = kubernetes-project-232110
EOF
```
# Initilize kubernetes master
```bash
sudo kubeadm init  --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<private_ip_of_server>

#Configure kubectl on master:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#Add Calico networking plugin:
kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

kubeadm config upload from-file --config hyperkube/gce/control-plane.yml
sudo kubeadm upgrade apply 1.14.1

# verify updated config
kubeadm config view
# check node status:
kubectl get nodes
```
#Install Helm
```bash
cd ~ && wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz
tar -zxvf helm-v2.13.1-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm

kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
#kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
helm init --service-account tiller
helm list
```

# Add nodes
## repeat installation steps on node 
### run below command as a result of kubeadm init
```bash
kubeadm join 10.160.0.6:6443 --token ijghom.dk7oflu8j14kgxxq --discovery-token-ca-cert-hash sha256:1f16f9064d4bf3c4b4951fb9f4d7c8bd331b1de2f970c5c3fdec9ef1510bc74f
```
### verify node on master
```bash
kubectl get nodes
kubectl get pods --all-namespaces
```
# deploy kubernetes web UI 
```Bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
kubectl proxy --address='<private_ip_of_vm>' --accept-hosts='^<public_ip_of_VM>$' &
```
### access dashbaord at:
http://<public_ip>:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

#follow instructions here to get auth token for login:
https://github.com/kubernetes/dashboard/wiki/Creating-sample-user