# Kubernetes

## Kubernetes Overview
 - container orchestration platform
 - a self-healing platform
 - kubernetes can also self-heal itself
 - it is platform supporting deploying High Available Application
 - you could deploy your applications within Kubernetes cluster
 - kubernetes constantly monitors the health of your application and if requires replacing
   faulty appln instance, it replaces with a new health application instance
 - it supports scaling up/down application instances depending traffic, cpu utilization, etc
 - it also supports rolling update
     - you can upgrade your appln from one version to other version without any downtime
     - in case of any issues, you could also rollback to older version withou any downtime
 - it also support 2 types of services
     1. Internal and -ClusterIP
     2. External - NodePort, LoadBalancer
 - it also support Ingress
 - Kubernetes cluster
     - collection of servers i.e master and many worker nodes(Servers - VM, cloud machines, on-prem servers )
     - Kubernetes Master
         - Container Runtime ( eg: Docker )
         - Kubernetes Agent ( kubelet - service )
         - Control Plan Components(They are Pods)
              - API Server (REST API - all K8s functionalities are available as REST APIs)
              - Scheduler
              - Controllers manager
              - etcd datastore

      - Kubernetes Worker Nodes
          - Container Runtime ( eg: Docker )
          - Kubernetes Agent ( Kubelet - service )
          - User Pods are deployed here

- Kubernetes Objects
    Deployment 
      - this represents your application deployment
      - manages ReplicaSet
    ReplicaSet
      - there will one ReplicaSet per version of your application
      - this manages how many instances of your application are running
      - manages Pods
      - Scaling your application instance are the responsiblilty of ReplicaSet
    Pod
      - group of containers
      - unlike Docker, IP address is assigned on the Pod level not on the container level
      - every Pod get its it own Port Namespace ranging 0-65535
      - Pod gets a hostname
      - gets a unique name
      - is the smallest unit that can deployed within K8s cluster
   Service abstraction
      - a database service
      - a group of mysql, a group of couchbase db servers
 
SOLID Design Principles
  S - Single Responsibility Principle
 

Statement Microservice
  - it helps in generating mini statement, monthly statement, half-yearly, quaterly, year statements
  - will interact with CouchBase DBService
  - JDBC Connection String
        "CouchBaseDBService:3301/dbname:username:password"

Service Discovery

CouchBase DBService
   - Deployment
       - ReplicaSet
             - Pod (50 CouchBase DB Servers )  
             - every CouchBase Pod will have an unique IP Address

Deployment - Couchbase database server
ReplicaSet - 50 instances of Couchbase database servers running in 50 Pods
 - 50 Pods


JDBC Connection
  DB Hostname/IP Address
  Port
  Database name
  username
  password

Monolithic architecture
  - You kind of capture all the abov DB connection details in a properties/xml/json/yaml file

Microservice 
  - requires pull bank account transaction details to print a statement
  


Each Microservice will have
One Deployment
Many ReplicaSet
Many Pods per ReplicaSet

CRM 1.0 - I need 5 instances of my CRM 1.0

Deployment - there will be a single deployment object for CRM
Replicaset - for CRM 1.0, 5 Pods of v1.0 are managed by this ReplicaSet
Replicaset - for CRM 1.1, 5 Pods of v1.1 are managed by this ReplicaSet
Replicaset - for CRM 2.0, 5 Pods of v2.0 are managed by this ReplicaSet


Tools
  - kubectl
     - client tool
     - majority of the times, this is the tool we all will be using to deploy,scaling,rolling update
     - to create service
     
  - kubeadm 
     - adminstrative tool used to bootstrap(setup) master node and joing worker nodes to cluster

  - kubelet - Kubernetes Agent (Service) runs on all nodes(master & worker nodes)


### Installing minikube K8s cluster
```
cd /home/rps/Downloads
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
See if minikube is added to path correctly
```
cd ~
which minikube
```
You are supposed to see the below output
<pre>
[jegan@tektutor ~]$ which minikube
/usr/local/bin/minikube
</pre>

Now you install minikube as shown below as rps user(non-admin)
```
sudo usermod -aG docker rps
sudo su rps
minikube start --driver=docker
```
If you see kubectl successfully configured, you are all set.

### Installing kubectl (Do this as rps user from terminal)
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x ./kubectl
sudo mv kubectl /usr/local/bin
```
Let's test if kubectl is able to communicate with K8s cluster
```
kubectl get nodes
```
The expected output is
<pre>
[jegan@tektutor ~]$ kubectl get nodes
NAME       STATUS   ROLES                  AGE   VERSION
minikube   Ready    control-plane,master   23m   v1.22.3
</pre>

### Create nginx deployment
```
kubectl create deployment nginx --image=nginx:1.18
```
You may now verify if the deployment is created successfuly
```
kubectl get deployments
kubectl get replicasets
kubectl get pods
```
In the above above, you may replace deployments with deployment or deploy

Same way, you may replace replicasets with replicaset or rs

Similary, you may interchange pods with pod or po

You can also list multiple K8s objects at one shot as shown below

```
kubectl get deploy,rs,po
```

### Understanding CrashLoopBackOff
Let's deploy hello-world:latest application in K8s cluster

```
kubectl create deploy hello --image=hello-world:latest
```

Let's list the deployment, replicaset and pods
```
kubectl get deploy,rs,po
```

If you wish to watch interactively
```
kubectl get po -w
```
Now you would have noticed, the pods associated with hello deployment get Completed, but K8s things it got crashed and it is then replaced with another Pod, but hello pod will again complete and terminate as hello-world doesn't have a blocking application running inside.

CrashLoopBackOff may also occur in case the application crashes due to some exception or lack of resource like memory, etc.,

### Finding more details about Pod
```
kubectl describe pod/<pod-name>
```

### Finding more details about a replicaset
```
kubectl describe rs/<replicaset-name>
```

### Finding more details about a deployment
```
kubectl describe deploy/nginx
```

### You may even edit a live deployment, ofcourse only certain fields can be modified
```
docker edit deploy/nginx
```

### You may edit any type of object in Kubernetes
```
docker edit rs/<replicaset-name>
docker edit po/<pod-name>
```

### Scaling up nginx deployment
```
kubectl scale deploy nginx --replicas=12
```
You may watch the pod creation activity interactively as shown below
```
kubectl get po -w
```

### Scaling down nginx deployment
```
kubectl scale deploy nginx --recplicas=3
```
You may watch the pod creation activity interactively as shown below
```
kubectl get po -w
```
To come out of watch mode, press Ctrl + c


### Creating a NodePort external service for nginx deployment
```
kubectl expose deploy/nginx --type=NodePort --port=80
```
Listing the services
```
kubectl get svc
```

Accessing the NodePort service
We need to identify the Node IP and the NodePort assigned for nginx service.
Let's find the IP address of the node using the below command
```
kubectl get nodes -o wide
```
In my case the minikube node IP was 192.168.49.2
Now let's find the NodePort assigned for nginx NodePort service.
```
kubectl describe svc/nginx
```
Generally the NodePort will be in the 30000 to 32767 range, asuming the NodePort assigned is 31164, you can access the NodePort external service as shown below

curl http://192.168.49.2:31164

Each time you access the above URL, the call will be forwarded to one of the nginx Pod linked with the NodePort Service.

You can also test the service discovery, i.e accessing the service using its name from within one of the Pod.
You may get inside a Pod using the below command
```
kubectl exec -it <your-nginx-pod-name> sh
curl http://nginx:80
```
In the above URL, nginx is the name of the NodePort service and 80 is the Service Port.

### Creating a ClusterIP internal service for nginx deployment
```
kubectl delete svc/nginx
kubectl expose deploy/nginx --type=ClusterIP --port=80
```
Listing the services
```
kubectl get svc
```

Accessing the ClusterIP service
We need to identify the Cluster IP to nginx service.
Let's find the IP address of the node using the below command
```
kubectl describe svc/nginx
```
In my case the minikube node IP was 10.111.128.35

You can access the ClusterIP internal service as shown below

curl http://<cluster-ip-nginx-service:service-port
curl http://10.111.128.35:80

As this is an internal service, we need to get inside any one of the pod that is running in the cluster
```
kubectl exec -it <your-nginx-pod-name> sh
curl http://10.111.128.35:80
curl http://nginx:80
```
In the above command, the first URL demonstrates on how to access the clusterip service using its cluster ip and service port. The second URL demonstrates on how to access using service discovery ie. using its name and service port.

Each time you access the above URL, the call will be forwarded to one of the nginx Pod linked with the ClusterIP Service.

### Creating a LoadBalancer external service for nginx deployment
LoadBalancer service is meant to be used in Cloud environment like AWS, Azure, etc.,
This type of service in Cloud will create a ALB - application Load Balancer or ELB and routes the call to one of the Pods.
If we create a LoadBalancer in the local (on-prem) it typically works just like a NodePort service.

```
kubectl expose deploy/nginx --type=LoadBalancer --port=80
```
Listing the services
```
kubectl get svc
```

Accessing the LoadBalancer service
We need to identify the Node IP and the LoadBalancer assigned for nginx service.
Let's find the IP address of the node using the below command
```
kubectl get nodes -o wide
```
In my case the minikube node IP was 192.168.49.2
Now let's find the IP Address assigned for nginx LoadBalancer service.
```
kubectl describe svc/nginx
```
Generally the NodePort will be in the 30000 to 32767 range, asuming the NodePort assigned is 31164, you can access the LoadBalancer external service as shown below

curl http://192.168.49.2:31164

Each time you access the above URL, the call will be forwarded to one of the nginx Pod linked with the LoadBalancer Service.

You can also test the service discovery, i.e accessing the service using its name from within one of the Pod.
You may get inside a Pod using the below command
```
kubectl exec -it <your-nginx-pod-name> sh
curl http://nginx:80
```
In the above URL, nginx is the name of the LoadBalancer service and 80 is the Service Port.
