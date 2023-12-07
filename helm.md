
# helm (package manager + YAML template tool)

# setting up helm

    sudo apt install apt-transport-https -y
    curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list deb [arch=amd64 signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main
    sudo apt-get update
    sudo apt install helm
    helm version

    helm list 
    helm search hub nginx
     artifactHub.io
    helm repo add <chart name> <chart link>
    helm repo update


# installing HELM

    curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
    sudo apt-get install apt-transport-https --yes
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
    sudo apt-get update
    sudo apt-get install helm

    helm repo add traefik https://helm.traefik.io/traefik
    helm repo update
    helm install traefik traefik/traefik
    helm list
    kubectl get deployments traefik
    kubectl get pods traefik-
    kubectl get svc traefik


    helm search repo nginx
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm install nginx bitnami/nginx
    kubectl get svc

# setting up HAProxy (high availability proxy) loadBalancer
 
 copy internal ip 
  ssh into load balancer instance

    sudo apt update && sudo apt install haproxy
    haproxy -version
    sudo vim /etc/haproxy/haproxy.cfg
  default section change mode  tcp   option tcplog
   # add  
        frontend proxy
                 bind *:80
                 bind *:6443
                 stats uri /proxystats
                 default_backend myservers
         backend myservers
                 balance roundrobin
                 server control-plane-1 <ip of control-plane>:6443 check
                 server control-plane-2 <ip of control-plane-2>:6443 check
                 server control-plane-3 <ip of control-plane-3>:6443 check
         listen stats
                bind :9999
                mode http
                stats enable
                stats hide-version
                stats uri /stats

    sudo systemctl restart haproxy.service

# ssh to original control palne

     sudo vim /etc/hosts
     below local host ip add internal ip of loadbalancer and type lb
# do it on all control plane

     http:<external ip of lb>:9999/stat


