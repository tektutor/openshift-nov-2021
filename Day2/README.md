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


### Installing kubectl 
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

