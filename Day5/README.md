# Openshift

## Overview

## Installing OpenShift in CentOS 7.7

### Installing Docker Community Edition as rps user
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y  docker-ce docker-ce-cli containerd.io
```

Adding rps user to docker group
```
sudo usermod -aG docker $USER
newgrp docker
```

Creating an insecure Docker Registry for OpenShift internal use
```
sudo mkdir /etc/docker /etc/containers

sudo tee /etc/containers/registries.conf<<EOF
[registries.insecure]
registries = ['172.30.0.0/16']
EOF

sudo tee /etc/docker/daemon.json<<EOF
{
   "insecure-registries": [
     "172.30.0.0/16"
   ]
}
EOF
```

Restart Docker to apply config changes
```
sudo systemctl daemon-reload
sudo systemctl enable docker
sudo systemctl restart docker
```

Enable IP Forwarding
```
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Configure Firewall ports for OpenShift
```
DOCKER_BRIDGE=`docker network inspect -f "{{range .IPAM.Config }}{{ .Subnet }}{{end}}" bridge`
sudo firewall-cmd --permanent --new-zone dockerc
sudo firewall-cmd --permanent --zone dockerc --add-source $DOCKER_BRIDGE
sudo firewall-cmd --permanent --zone dockerc --add-port={80,443,8443}/tcp
sudo firewall-cmd --permanent --zone dockerc --add-port={53,8053}/udp
sudo firewall-cmd --reload
```

Install OpenShift binaries
```
wget https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
tar xvf openshift-origin-client-tools*.tar.gz
cd openshift-origin-client*/
sudo mv  oc kubectl  /usr/local/bin/
```

Verify OpenShift version
```
oc version
```

Start our OpenShift Cluster
```
oc cluster up
```

Login to OpenShift console via CLI as administrator
```
oc login -u system:admin
```

Listing nodes in Openshift Cluster
```
oc get nodes -o wide
```

Checking status
```
oc status
```

Checking currently logged in user
```
oc whoami
```

Login to OpenShift console via CLI as developer
```
oc login -u developer:developer
```

Accessing web console
```
https://127.0.0.1:8443/console
```


