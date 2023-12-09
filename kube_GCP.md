# Fulfilling prerequisites and installing packages for K8s cluster 
Networking software package allows containerd to create virtual bridge networks for containers

    lsmod | grep br_netfilter
    
    sudo modprobe br_netfilter
Cluster wide network overlay is a virtual network for the containers created within the cluster so that they can communicate with one another

    sudo modprobe overlay
    
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    > overlay
    > br_netfilter 
    > EOF

    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    > net.bridge.bridge-nf-call-iptables = 1
    > net.bridge.bridge-nf-call-ip6tables = 1
    > net.ipv4.ip_forwad                 = 1
    > EOF
(1st couple of lines enables v4 and v6 IP tables for virtual bridge networks and 3rd line enables IP forwarding)

    sudo sysctl --system
  
    wget https://github.com/containerd/containerd/releases/download/v1.6.8/containerd-1.6.8-linux-amd64.tar.gz
    sudo tar Cxzvf /usr/local containerd-1.6.8-linux-amd64.tar.gz

    sudo wget https://github.com/opencontainers/runc/releases/download/v1.1.3/runc.amd64
    sudo install -m 755 runc.amd64 /usr/local/sbin/runc
    sudo mkdir -p /opt/cni/bin
   
    sudo wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-arm-v1.3.0.tgz
    sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-arm-v1.3.0.tgz
    sudo apt update
    
    sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

    sudo mkdir -p /etc/apt/keyrings
    
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    sudo apt-get update

    sudo apt install containerd.io -y

    sudo su -
    
    mkdir -p etc/containerd
    
    containerd config default>/etc/containerd/config.toml

    exit
    
    sudo systemctl restart containerd
 
    sudo systemctl enable containerd

    sudo nano /etc/conatinerd/config.toml
edit this make systemconfig becomes true

    systemdconfig =  true

    sudo systemctl restart containerd
to check containerd installed properly use 
 
    sudo ctr --help

    sudo apt-get update
    
    sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list 

    sudo apt-get update

    sudo apt-get install -y kubelet kubeadm kubectl
(kubectl is the k8s command line,whereas kubeadm is the cluster bootstrapping and management agent)

in control plane run 
sudo kubeadm init --apiserver-advertise-address <internal ip of controlplane> --pod-network-cidr=10.244.0.0/16
for rootless access
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

token (kubeadm join 10.142.0.2:6443 --token tb1pad.wvx7qk1l1ur15cnw \
        --discovery-token-ca-cert-hash sha256:5da9fb9902e528029fc8ca9ba45992bc8b4016c53fc545af4ac313509016b603 
        )

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


ssh into node 1 

 sudo kubeadm join 10.142.0.2:6443 --token tb1pad.wvx7qk1l1ur15cnw 
 > --discovery-token-ca-cert-hash sha256:5da9fb9902e528029fc8ca9ba45992bc8b4016c53fc545af4ac313509016b603 



for auto completion of commands type
type _init_completion
if u get error then execute 
source /usr/share/bash-completion/bash_completion

 echo 'source <(kubectl completion bash)' >>~/.bashrc
