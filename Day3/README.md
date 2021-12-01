### What happens when we create a new deployment in Kubernetes
```
kubectl create deploy nginx --image=nginx:1.18
```

1. kubectl client tool make a REST request to create a deployment
2. The request is received by the API Server on the master node,
   API Server creates a Deployment in the etcd datastore.
3. There are many controllers in master node, each controller is responsible
   for certain things
     1. Node Controller
 	  - receives notification whenever a new worker/master nodes attempts
            to join the K8s cluster
          - receives notification whenever a node leaves the K8s cluster
     2. Endpoint Controller
          - receives notification whenever a new service( NodePort,ClusterIP, etc_) 
          - receives notification whenever deployments are scaled up/down
     3. Deployment Controller
          - receives notification whenever a new Deployment is created
          - makes a REST request to create ReplicaSet
          - the request is received by API Server, then it creates a ReplicaSet
            in the etcd datastore
          - API Server also raises an event that a new ReplicaSet is created
     4. Replicaset Controller
          - Whenever a ReplicaSet is creates, this controller receives a notification
          - makes a REST request to create Pods as per the ReplicaSet desired count
          - API Server receives the request and creates Pod entries in etcd
          - API Server then raises event conveying new Pod entries are created

     5. Replication Controller


   - Scheduler
       - will receive notifications whenever new Pod definition is created in etcd
       - will receive notifications whenever Pods are deleted
       - will then identify a healthy node where the Pod can be deployed
       - Scheduler updates/makes a rest call to API Server conveying where those
         new Pods can be deployed
       - API Server notifies kubelet on the scheduled node to create the required
         containers that are required for a Pod
    
   - Kubelet
       - receives event when containers has to be created on the current node
         wherever kubelet is running
       - kubelet will verify if the Docker/Container image is present in the local
         node, if not it will pull the image using the respective Container Runtime.
       - kubelet creates the containes, starts them and constantly monitors them
       - kubelet also sends periodical heart-beat notifications about the health
         of the containers running on the node to API Server
       - API Server keeps updating the corresponding Pod status based on the container status reported by kubelet.

## Setting up a 3 Node Kubernetes Cluster using kubeadm
#### Disable Virtual Memory (swap parition) in Master and Worker Nodes
```
sudo swapoff -a
```
To permanently disable swap partition,  edit  the /etc/fstab file root user and comment the swap partition.
```
sudo vim /etc/fstab
sudo systemctl daemon-reload
```

#### Disable SELINUX in Master and Worker Nodes
``` 
setenforce 0
```

To permanently disable  SELINUX, you need to edit /etc/selinux/config file and change enforcing to disabled.
```
sudo systemctl daemon-reload
```
Configure the hostnames of master and all worker nodes
In Master Node
```
sudo hostnamectl set-hostname master
```

In worker1 Node
```
sudo hostnamectl  set-hostname worker1
```

In worker2 Node
```
sudo hostnamectl set-hostnamme worker2
```

### In the master node, type the below command to find the IP Address
```
ifconfig ens33
```
Note down the IP of master node as we need to add this in the /etc/hosts later.

### In the worker1 node, type the below command to find the IP Address
```
ifconfig ens33
```
Note down the IP of worker1 node as we need to add this in the /etc/hosts later.

### In the worker2 node, type the below command to find the IP Address
```
ifconfig ens33
```
Note down the IP of worker2 node as we need to add this in the /etc/hosts later.

### Configure /etc/hosts file
Append the IPAddresses of master, worker1 and worker2 as shown below in /etc/hosts files. This should be done in master, worker1 and worker2 nodes.
```
192.168.254.129 master 
192.168.254.130 worker1
192.168.254.131 worker2
```

### Firewall configurations
For summary of ports that must be opened, refer official Kubernetes documention https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

#### Open the below ports in Master Node as root user
```
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250-10252/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --zone=trusted  --add-source=192.168.0.0/16 
modprobe br_netfilter
systemctl daemon-reload
systemctl restart firewalld
systemctl status firewalld
firewall-cmd --list-all
```

#### Open the below ports in Worker Nodes as root user
```
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --zone=trusted  --add-source=192.168.0.0/16 
modprobe br_netfilter
systemctl daemon-reload
systemctl restart firewalld
systemctl status firewalld
firewall-cmd --list-all
```

#### Install Docker CE in Master and Worker Nodes
```
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce --allowerasing
sudo usermod -aG docker rps
sudo su rps
```

### Configure Docker Engine to use systemd driver in Master and Worker Nodes

Before editing the daemon.json, let's enable and start docker so that it will create /etc/docker folder.
```
sudo systemctl enable docker
sudo systemctl start docker
```

sudo vim /etc/docker/daemon.json

```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
     "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}

sudo mkdir -p /etc/systemd/system/docker.service.d
sudo systemctl daemon-reload
sudo systemctl enable docker && sudo systemctl start docker
```

#### Configure IPTables to see bridge traffic in Master and Worker Nodes
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### Install kubectl kubeadm and kubelet on Master & Worker nodes
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### Uninstall Microk8s
```
sudo snap remove microk8s
```

### Configure kubelet in Master and Worker Nodes
```
sudo vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS= --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice

```
You may enable the kubelet service as shown below
```
sudo systemctl enable --now kubelet
```

### Restart Docker and Kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

### Bootstrapping Master Node as root user
```
kubeadm init --pod-network-cidr=192.168.0.0/16
```

Do the below steps as rps user
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

In order to access the cluster without issues after machine reboots, add the below to /root/.bashrc
Do the below as root user
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```
In order to apply the export changes done in the /root/.bashrc, you need to manually run this
```
source /root/.bashrc
```

Save your join token in a file on the Master Node, the token varies on every system and every time you type kubeadm init, hence you need to save your join token for your reference before you clear your terminal screen.
```
vim token
kubeadm join 192.168.154.128:6443 --token 5zt7tp.2txcmgnuzmxtgnl \
        --discovery-token-ca-cert-hash sha256:27758d146627cfd92079935cbaff04cb1948da37c78b2beb2fc8b15c2a5adba
```

#### In case you forgot to save your join token and cleared the terminal screen, no worries try this on Master Node
```
kubeadm token create --print-join-command
```

#### In Master Node
```
kubectl get nodes
kubectl get po -n kube-system -w
```
Press Ctrl+C to come out of watch mode.

#### Installing Calico CNI in Master Node
To learn more about Calico CNI, refer https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises 
```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

#### In Master Node watch the pod creation after installing Calico
```
kubectl get po -n kube-system -w
```
![Control Plane Pods](https://github.com/tektutor/kubernetes-may-2021/blob/master/Day2/3NodeClusterSetup/K8s-controlplane-pods.png)

Press Ctrl+C to come out of watch mode.

#### In Worker Node
```
kubeadm join 192.168.154.128:6443 --token 5zt7tp.2txcmgnuzmxtgnl \
        --discovery-token-ca-cert-hash sha256:27758d146627cfd92079935cbaff04cb1948da37c78b2beb2fc8b15c2a5adba
```
![Worker Join Command](https://github.com/tektutor/kubernetes-may-2021/blob/master/Day2/3NodeClusterSetup/worker1-join.png)

#### In Master Node
At this point,  you are supposed to see 3 nodes in ready state.
```
kubectl get nodes
```
![Node List](https://github.com/tektutor/kubernetes-may-2021/blob/master/Day2/3NodeClusterSetup/node-list.png)

If you see similar output on your system, your 3 node Kubernetes cluster is all set !!!


#### In case you had trouble setting up master, you could reset and try init as shown below (on master and worker nodes)
```
kubeadm reset
```

You need to manually remove the below folder
```
rm -rf /etc/cni/net.d
rm -rf /etc/kubernetes
rm -rf $HOME/.kube
```
