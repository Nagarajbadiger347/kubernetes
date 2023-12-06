# k8s as the software, works on a client-server model
Components:
contaier runtime- creates containers ex: CRI-O, Containerd, Docker
 Command line: kubectl
 Setup agent: kubeadm 
 kubelet: integral part of k8s architecture



# Components:
kube-apiserver: validates the http api request
kube-controller: managing the controllers of k8s cluster 
kube-scheduler: allocates appropriate node, put new workload
etcd: Database that store information in key value pair
kubelet: Create the object 
kubeproxy: network proxy for the container running on k8s, it route and filters the data packets coming to and generated from containers running on k8s cluster, manages the communication

#REST APIs Representaional states transfer
 
 



