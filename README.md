
# Introduction to KubeVirt - Running virtual machines on Kubernetes

## Configure KubeVirt


    kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/v1.5.0/kubevirt-operator.yaml"

    kubectl create -f "https://github.com/kubevirt/kubevirt/releases/download/v1.5.0/kubevirt-cr.yaml"


## Configure CDI


    kubectl create -f "https://github.com/kubevirt/containerized-data-importer/releases/download/v1.61.2/cdi-operator.yaml"

    kubectl create -f "https://github.com/kubevirt/containerized-data-importer/releases/download/v1.61.2/cdi-cr.yaml"


## Install virtctl


    curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/v1.5.0/virtctl-v1.5.0-linux-amd64

    chmod +x virtctl

    cp virtctl ~/.local/bin/


## Deploy Normal VM in the cluster

    cd normal_vm

### 1. Create VM

    kubectl apply -f vm-normal.yaml

    kubectl get vm, vmi
    kubectl get pods


### 2. Access VM (console)

    virtctl console ubuntu-focal

(use ubuntu:ubuntu)


### 3. Access VM (ssh)

Create service:

    kubectl apply -f svc-vm-ssh.yaml

Get node IP and port:

    kubectl get nodes -o wide
    kubectl get svc
    kubectl get ep

SSH using the IP and port:

    ssh ubuntu@172.18.0.2 -p 31028


### 4. Check Spec

From VM:

    df -h 

    free -h


From VM definition:

    kubectl get vmi ubuntu-focal -o jsonpath='{.status.memory}'


### 5. Install nginx


In vm:

    sudo apt update
    sudo apt install nginx -y


There is not enough space to install anything in the VM.

    df -h


### 5. Stop VM

Use virtctl to stop the VM:

    virtctl stop ubuntu-focal


### 5. Delete VM


    kubectl delete -f vm-normal.yaml
    kubectl delete -f svc-vm-ssh.yaml


## Deploy VM that uses CDI

    cd use_cdi

### 1. Create DataVolume


    kubectl apply -f dv-ubuntu-containerdisk.yaml

    kubectl get dv
    
    kubectl get pv,pvc    

    kubectl get pods
    kubectl logs importer-dv-ubuntu-focal -f


### 2. Create VM that uses DataVolume


    kubectl apply -f vm-ubuntu-dv.yaml

    kubectl get pods
    kubectl get vm, vmi


### 3. Access VM (ssh)


Create service:

    kubectl apply -f svc-vm-ssh.yaml

Get node IP and port:

    kubectl get nodes -o wide
    kubectl get svc
    kubectl get ep

SSH using the IP and port:

    ssh ubuntu@172.18.0.2 -p 31028


### 4. Check spec


From VM:

    df -h 

    free -h


### 5. Install nginx in VM and Access it externally

Install nginx in VM:

    sudo apt update
    sudo apt install nginx -y

    sudo systemctl start nginx
    sudo systemctl status nginx

    curl -I http://localhost:80


Access that nginx externally from client machine:

    kubectl get vmi
    
    virtctl port-forward ubuntu-focal 8080:80

    curl -I http://localhost:8080


## References

- Configure kubevirt: https://kubevirt.io/quickstart_kind/ 
- Configure CDI: https://kubevirt.io/labs/kubernetes/lab2.html  
- Ubuntu container disk images: https://hub.docker.com/r/tedezed/ubuntu-container-disk/tags
