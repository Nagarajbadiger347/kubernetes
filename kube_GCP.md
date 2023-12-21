# Fulfilling prerequisites and installing packages for K8s cluster 
Apply these steps to control plane and nodes

Networking software package allows containerd to create virtual bridge networks for containers
(Bridge net filter)

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
Download Containerd for github repo

    wget https://github.com/containerd/containerd/releases/download/v1.7.11/containerd-1.7.11-linux-amd64.tar.gz
    sudo tar Cxzvf /usr/local containerd-1.7.11-linux-amd64.tar.gz
Downloading and Installation of runc (it is low-level container runtime that implements the OCI specification)

    sudo wget https://github.com/opencontainers/runc/releases/download/v1.1.3/runc.amd64
    sudo install -m 755 runc.amd64 /usr/local/sbin/runc
    sudo mkdir -p /opt/cni/bin
Downloading installing cni plugins


    sudo wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-arm-v1.3.0.tgz
    sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-arm-v1.3.0.tgz
    sudo apt update
Installing the dependencies which are required to run the container

    sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
Downloading gpg keys

    sudo mkdir -m 755 /etc/apt/keyrings
    
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# or you edit manually the docker.list file 
    sudo nano /etc/apt/sources.list.d/docker.list

Add the following line to the file:

    deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian bullseye stable
# update and install containerd

    sudo apt-get update
Installation of containerd 

    sudo apt install containerd.io -y
Adding containerd config file 

    sudo su -
    
    mkdir -p etc/containerd
    
    containerd config default>/etc/containerd/config.toml

    exit
Restart containerd

    sudo systemctl restart containerd
Enabling the containerd

    sudo systemctl enable containerd

    sudo nano /etc/conatinerd/config.toml
Edit this make systemconfig becomes true
    
    systemdconfig =  true

    sudo systemctl restart containerd
To check containerd installed properly use 
 
    sudo ctr --help

# Update the apt package index and install packages needed to use the Kubernetes apt repository:
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    
# Download apt-key
     curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the appropriate Kubernetes apt repository.
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
# Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
(kubectl is the k8s command line,whereas kubeadm is the cluster bootstrapping and management agent)

# Spinning up the K8s cluster
In control plane spin up the cluster with kubeadm init command 
(if the infrastructure has different node in different VPCs, allow both to communicate or use static external IPs in the case of hybrid cloud infrastructure.)
The pod IPs will follow the 10.244 range. the pod network can host 65,536 pods

    sudo kubeadm init --apiserver-advertise-address <internal ip of controlplane> --pod-network-cidr=10.244.0.0/16

You will get the command when you run kubeadm init command which you use For rootless access
Download the token which is generated while running kubeadm init command 
ex token (kubeadm join 10.142.0.2:6443 --token tb1pad.wvx7qk1l1ur15cnw \     --discovery-token-ca-cert-hash sha256:5da9fb9902e528029fc8ca9ba45992bc8b4016c53fc545af4ac313509016b603 )
    
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
Flannel is a third-party pod network supported by kubernetes others are vunit and calico.

    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

ssh into node 1 & node 2 Run the copied token to add the node the cluster 

 ex: 

    sudo kubeadm join 10.142.0.2:6443 --token tb1pad.wvx7qk1l1ur15cnw 
      --discovery-token-ca-cert-hash sha256:5da9fb9902e528029fc8ca9ba45992bc8b4016c53fc545af4ac313509016b603 
to check run kubectl get nodes in control plane to see the nodes

For auto completion of commands type in control plane 

    type _init_completion
If u get error then execute 

    source /usr/share/bash-completion/bash_completion
bashrc stands for "bash run commands" it is a collection of instructions given to bash at the beginning of every session to recognize the command line of every tool installed on system
    
    echo 'source <(kubectl completion bash)' >>~/.bashrc
    exit 
    kubectl get nodes -o wide
    
---



