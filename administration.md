# creating highly available k8s cluster using kubeadm

    sudo kubeadm reset
    
    sudo kubeadm init \
      --apiserver-advertise-address <internal ip of control plane> \
      --control-plane-endpoint <internal ip of lb> \
      --pod-network-cidr=10.244.0.0/16 \
      --upload-certs

     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
     kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# backing up the etcd database

    sudo apt install etcd-client
    etcdctl --version
    kubectl run nginx --image nginx:latest --port 80
    etcdctl -h
    sudo ls /etc/kubernetes/pki/etcd
    
    sudo ETCDCTL_API=3 etcdctl --endpoints <internal ip of control plane ip>:2379 \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    --cacert=etc/kubernetes/pki/etcd/ca.crt \
    member list -w table

    sudo ETCDCTL_API=3 etcdctl --endpoints :2379 \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    endpoint health

# to create snapshot

    sudo ETCDCTL_API=3 etcdctl --endpoints <internal ip of control plane ip>:2379 \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    --cacert=etc/kubernetes/pki/etcd/ca.crt \
    snapshot save /var/lib/etcd/snapshot.db

    mkdir k8s-backup
    sudo cp /var/lib/etcd/snapshot.db $HOME/k8s-backup/

    kubectl get configmaps -n kube-system

    kubectl get configmaps -n kube-system kubeadm-config -o yaml > $HOME/k8s-backup/kubeadm-config.yaml

    sudo cp -r /etc/kubernetes/pki/etcd $HOME/k8s-backup/


 # upgrading cluster (changing k8s version)

    sudo kubeadm version 
    sudo apt install kubeadm=1.25.2 (any version)
    kubectl drain control-plane --ignore-daemonsets
    sudo kubeadm upgrade plan
    sudo kubeadm upgrade apply v1.25.2 
    kubectl get nodes
    kubectl uncorden control-palne
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet

# restoring snapshot

     sudo ETCDCTL_API=3 etcdctl --endpoints <internal ip of control plane ip>:2379 \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     --cacert=etc/kubernetes/pki/etcd/ca.crt \
  
    snapshot restore $HOME/k8s-backup/snapshot.db
