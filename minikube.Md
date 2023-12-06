minikube kubectl --create deployment minikube-sample --image=k8s.gcr.io/echoserver:1.4
minikube kubectl -- get deployments
minikube kubectl -- expose deployment minikube-sample --type-NodePort --port-8080
minikube service minikube-sample
function kubectl { minikube kubectl -- $args }

kubectl create deployment deploy-ubuntu --image=gcr.io/google-containers/ubuntu:14.04

kubectl <verb> <object type> <object name> <flags>
kubectl get pods 
different output formats: 
 kubectl get pods -o wide
 kubectl get pods -o yaml > pod.yaml
 kubectl get pods -o json
 custom column: kubectl get pods <podname> -o 'custom-columns-NAME:.metadata.name,IP:.status.podIP

minikube stop --keep-context-active

