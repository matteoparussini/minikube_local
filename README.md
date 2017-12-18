# minikube_local

To run all the Forgerock module into you local machine you need to install:

minikube
kubectl
helm

To start up minikube, following ForgeRock DevOps advice (chapter 2.1.2 in DevOps Guide Version 5.5),  I've done a simple script, that for linux is looks like this:

minikube start --memory=8192 --disk-size=30g --vm-driver=virtualbox
minikube addons enable ingress
minikube ssh sudo ip link set docker0 promisc on
kubectl proxy&



I'm using a Kubernetes namespace called "local"; for that purpose you need to set up the kubectl context

kubectl config set-context $(kubectl config current-context) --namespace=local




