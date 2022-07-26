# Step 2 Disable swap & add kernel settings
```
sudo swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

```
sudo tee /etc/modules-load.d/containerd.conf<<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay

sudo modprobe br_netfilter
```

```
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF 
```
```
sudo sysctl --system
```
# Step 3 Install containerd run time
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

sudo apt install -y containerd.io
```

```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd

sudo systemctl enable containerd
```

# Step 4 Add apt repository for Kubernetes
如果你在国外
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-jammy main"
```
如果你在国内
```
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

sudo apt-add-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-jammy main"
```
# Step 5 Install Kubernetes components Kubectl, kubeadm & kubelet
```
sudo apt update

sudo apt install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl
```

# Step 6 Initialize Kubernetes cluster with Kubeadm command
```
kubeadm version

sudo kubeadm init --kubernetes-version=v1.24.3 --image-repository registry.aliyuncs.com/google_containers --v=5 --control-plane-endpoint=[ replace by hostname ]
```

```
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl cluster-info

kubectl get nodes
```

```
sudo kubeadm join k8smaster.example.net:6443 --token vt4ua6.wcma2y8pl4menxh2 --discovery-token-ca-cert-hash sha256:0494aa7fc6ced8f8e7b20137ec0c5d2699dc5f8e616656932ff9173c94962a36
```

```
curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```
```
kubectl get pods -n kube-system
```

# Step 7 Test Kubernetes Installation
```
kubectl create deployment nginx-app --image=nginx --replicas=2

kubectl get deployment nginx-app

kubectl expose deployment nginx-app --type=NodePort --port=80

kubectl get svc nginx-app

kubectl describe svc nginx-app

curl http://127.0.0.1:31246
```
