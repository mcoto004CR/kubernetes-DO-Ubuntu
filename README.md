# Create a kubernetes cluster using DigitalOcean Ubuntu 20.04 droplets

Follow this steps to create Kubernetes cluster on a Digital Ocean (DO) droplets
Note the master should be have at least 2GB of memory.
Note you can create this using the DO Kubernetes service, but you wont be able to manage your control plane directly.

## Updating Droplet
    apt get update
    apt get upgrade
    
## Add gpg keys
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    You should see "OK" 

## Add kubernetes package
    cd /etc/apt
    nano sources.list
  
  Add this lines
     
     deb http://apt.kubernetes.io/ kubernetes-xenial main
     sudo apt update

## Install docker
  Or any other contaniner mgmt you want: https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
 
 
 ### Set up the repository:
    sudo apt-get update && sudo apt-get install -y \
    apt-transport-https ca-certificates curl software-properties-common gnupg2
  
  
 ### Add Docker's official GPG key: 
  
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -
    You should see "OK"
    
  ### Add the Docker apt repository: 
    sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
    
  ### Install Docker CE  
    sudo apt-get update && sudo apt-get install -y \
      containerd.io=1.2.13-2 \
      docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
      docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)
      
  ### Set up the Docker daemon
    cat <<EOF | sudo tee /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
       "max-size": "100m"
     },
    "storage-driver": "overlay2"
    }
    EOF
    
  ### Create /etc/systemd/system/docker.service.d
    sudo mkdir -p /etc/systemd/system/docker.service.d
    
  ### Restart Docker 
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    sudo systemctl enable docker
    sudo systemctl status docker --> make sure is active (running)
    
##  Install Kubernetes CLI/tools

    apt install kubelet kubeadm kubectl kubernetes-cni

## Create master node (ONLY FOR MASTER, SEE BELOW JOIN A WORKER)
  
  kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address $DROPLET_PUBLIC_IP_ADDRESS
    - Replace $DROPLET_PUBLIC_IP_ADDRESS,with your droplet public IP, you can find it usinf ifconfig or in the DO dashboard
    
## NOTE: Make sure you copy the last line “kubeadm join –token …” as that contains necessary security credentials which a worker node would need to join the cluster.


## Final stage, install kube config and flannel for network management

    mkdir –p $HOME/.kube
    cp /etc/kubernetes/admin.conf $HOME/.kube
    chown $(id -u):$(id -g) $HOME/admin.conf
    
   POD Network: A Pod Network is a way to allow communication between different nodes in the cluster
   
     sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    

 ## Check master node is ready
 
  kubectl get nodes
  NAME     STATUS   ROLES    AGE   VERSION
  master   Ready    master   64m   v1.19.4   --> Status should be ready
    
  kubectl get pods –all-namespaces
     NAMESPACE     NAME                                                        READY   STATUS    RESTARTS   AGE
     kube-system   coredns-f9fd979d6-48lsr                                     1/1     Running   0          64m
     kube-system   coredns-f9fd979d6-zfwzq                                     1/1     Running   0          64m
     kube-system   etcd-master-                                                1/1     Running   0          65m
     kube-system   kube-apiserver-master-                                      1/1     Running   0          65m
     kube-system   kube-controller-manager-master-                             1/1     Running   0          65m
     kube-system   kube-flannel-ds-xs2p5                                       1/1     Running   0          38m
     kube-system   kube-proxy-llcqz                                            1/1     Running   0          64m
     kube-system   kube-scheduler-master-                                      1/1     Running   0          65m

    -->  Al status should be running
    
    
 ## ADDING A WORKER NODE
 
  Repeat all steps above, but skip add Master node, as you already created the master node
  Once you completed all steps, run the kubeadm join string you copied when created the master node:
    
    example: kubeadm join --token 3c37b5.08ed6cdf2e4a14c9
    159.89.25.245:6443 --discovery-token-ca-cert-hash
    sha256:52f99432eb33bb23ff86f62255ecbb
  
  
  Once completed  the master node run:
  
  kubectl get nodes
  NAME                                STATUS   ROLES    AGE   VERSION
  master                              Ready    master   70m   v1.19.4
  node1                               Ready    <none>   75s   v1.19.4 --> Status should be READY
  
  Repeat this step for any other node...
  
  # CONGRATS YOU HAVE A KUBERNETS CLUSTER!!!

  
  
    
    
    


https://linuxhint.com/setup-kubernetes-digitalocean/
https://phoenixnap.com/kb/install-kubernetes-on-ubuntu
