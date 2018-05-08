# minikube_local

To run all the Forgerock module into you local machine you need to install:

minikube (https://kubernetes.io/docs/tasks/tools/install-minikube/)

kubectl  (https://kubernetes.io/docs/tasks/tools/install-kubectl/)

helm     (https://docs.helm.sh/using_helm/#installing-helm)

docker

To start up minikube, following ForgeRock DevOps advice (chapter 2.1.2 in DevOps Guide Version 5.5),  I've done a simple script, that for linux is looks like this:

minikube start --memory=8192 --disk-size=30g --vm-driver=virtualbox (double check that your instance really has at list 8 Gb!!! with some version of virtual box this commands doesn't work :( )
minikube addons enable ingress
minikube ssh sudo ip link set docker0 promisc on
// 
kubectl proxy&


I'm using a Kubernetes namespace called "local"; for that purpose you need to create it and set up the kubectl context

kubectl create --validate=false -f local_namespace.yaml
kubectl config set-context $(kubectl config current-context) --namespace=local

Adding docker images

Minikube has it's own docker and, if we want to install ForgeRock components, we need to add the docker images into Minikube. To create the images, you can follow the instruction described in the DevOps Guide (Ch 4.5 "Creating Docker images" version 5.5.0). Maybe it's a good idea adding those images to a Docker Registry. If you need I have exported them, so you just need to import! 

eval $(minikube docker-env)  
docker import openam.tar forgerock/openam:5.5.0
docker import git.tar forgerock/git:5.5.0
docker import java.tar forgerock/java:5.5.0
docker import opendj.tar forgerock/opendj:5.5.0
docker import openidm.tar forgerock/openidm:5.5.0
docker import openig.tar forgerock/openig:5.5.0
docker import postgres.tar forgerock/postgres:5.5.0
docker import tomcat.tar forgerock/tomcat:5.5.0

IMPORTANT: label and tag should be the same as define into devops guide (if we want to change it, we need to change the scripts)

# Persistent Volumes

In forgerock Dev Opes guide, volumes are not persisted: so if you restart minikube, you lose the configuration/data which you were working on. For this reason I change helm charts to bind minikube volumes to a folder of minikube VM instance. I don't know if this is the best way to do it, but it could work for a local environment.

1 creating the persistent volumes (do thi only the first time)

navigate into cmp-platform-local/ folder

run: kubectl create -f StorageClassHostPath.yaml

connect to kubernetes dashboard (localhost:8001/ui) and select from the left menu Cluster/Storage Classes and update into "standard" class the attribute "storageclass.beta.kubernetes.io/is-default-class": "flase". Basically we want to use the class that we've just create to be used as default.

Now we need to create the persistent volumes and bind them with a minikube virtual folder. I choose "/local", but you can define something else. The path is define into the yaml file of the Persistent Volumes (*_pv.yaml, spec.hostPath.path attribute). 
(optional: bind a folder of your laptop to Minikube virtual box virtual)


run:

kubectl create --validate=false -f configstore_pv.yaml

kubectl create --validate=false -f configstore_pvc.yaml


kubectl create --validate=false -f postgres_pv.yaml

kubectl create --validate=false -f postgres_pvc.yaml


kubectl create --validate=false -f userstore_pv.yaml

kubectl create --validate=false -f userstore_pvc.yaml


After this you will have 3 persistent volunes that will store data of IDM database (postrgres), config and userstore ldap used by AM installation

then run the helm installation

helm install --values custom.yaml ./cmp-platform











