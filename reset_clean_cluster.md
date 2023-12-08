# cleaning up workspace
     kubectl get cronjobs && \
     kubectl get jobs && \
     kubectl get daemonset && \
     kubectl get deployments --all-namespaces

     kubectl delete cronjobs --all

     kubectl delete jobs -all
    
     kubectl delete daemonsets -all --cascade=background (this wont list the pods get deleated, delete the pods in background)
    
     kubectl delete daemonsets -all --all-namespaces --cascade=foreground
    
     kubectl delete pods -all

# resetting kubernetes cluster

     sudo kubeadm reset
    
     kubeadm init --apiserver-advertise-adress <internal ip adress> --pod-network-cidr=<ip address>

     mkdir -p $HOME/.kube
     
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

     sudo chown $(id -u):$(id -g) $HOME/.kube/config

     kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
     
# go to node 1 & 2
      sudo kubeadm reset
# paste the join command alongside the internal ip and ca-cert-hash)
