# Openshift Training Notes

## Day-1

### What is Hypervisor?
- Hypervisor is a general term used to refer to the Virtualization Technology
- Virtualization Technology
- Virtualization Technology allows us run multiple Operating Systems side-by-side on the same machine simultaneously
- Heavy-weight technology
   - it requires dedicated Hardware Resources
       - like CPU cores
       - RAM
       - Storage, etc
 - it is combination of Hardware and Software technology
 - Only machines that has Processor support Virtualization technoloy will be able host Virtual Machines(Guest OS)
 - Processor
   AMD
     - AMD-V is the feature that supports Virtualization
   Intel
     - VT-X is the virtualization feature
 - There are two types of Hypervisors
   1. Type 1 - Bare Metal meaning they don't need an Host OS. This is suitable for Workstations & Servers
   2. Type 2 - They need a Host OS.  Only top of Host OS we can install this type of Hypervisors. 
      Meant for Laptops & Desktops.

 - Host OS and Virtualization can be of any combination
 - Host OS ( could be Windows/Linux/Mac/Unix )
 - Guest OS aka Virtualization ( could be Windows/Linux/Mac/Unix )
 - Hypervisor Examples
   VMWare
      - VMWare Fusion ( Mac OS-X) - Type 2
      - VMWare Workstation Pro ( Windows & Linux ) - Type 2
      - VMware vCenter ( Bare Metal Hypevisor - don't need a Host OS ) - Type 1
   Parallels( Mac OS-X )
   Oracle 
      - VirtualBox ( Windows/Linux/Mac ) - Type 2
   Microsoft
      - Hyper-V

  - makes server consolidation possible

### Container Technology
 - application virtualization
 - lightweight virtualization techology
    - they don't need dedicated hardware resources
    - 
 - they are not Operating Systems
 - Virtual machines are fully functional OS while containers are just application process
 - container run in their own namespace ( virtual sandbox environment )
 - containers running in different namespace won't be see each 
 - container technology is a Linux Technology
 - Linux Kernel
   1. Namespace - containers isolated from each other
   2. CGroup - Control Groups - container level resource quota allocation can be managed
 - Why containers are compared with Virtual Machines?
    - both containers and virtual machines support isolation
    - both has filesystems
    - both has IP Address
    - both has shells ( bash, sh, etc )
    - both has package managers ( tools that install/uninstals/upgrades softwares in Linux OS)
    - containers behaves and offers OS like functionalities, hence people tend to compare VM with a containers.
    - containers can only run one application per container (recommendation)
    - when multiple application are put inside a container, container runtime uses a utility called supervisord 
    - to create additional processes and it monitors the health of those additional processes. This is a overhead, hence as much as possible run one per application per container
    - 

### Container Runtimes
  - LXC - low-level not so user-friendly
  - Docker - very user-friendly, hence very popular in the industry.  Docker a known vulnerabilty, Podman has solved
             it even before Docker, hence the industry started moving away from Docker
  - CRI-O - low-level not so user-friendly
  - Podman - slowly gaining popularity, mostly likely might takeaway the shares of Docker
   
### Container Orchestration Platforms
   - Docker SWARM - Docker Inc's native orchestration platform that supports only Docker Container Runtime
   - Google Kubernetes(K8s) - supports many different Container Runtimes, this is also time tested
   - RedHat OpenShift(IBM) - built on top of Google Kubernetes + CI/CD + Private Docker Registry, etc 

### Docker Runtime
 - developed by Docker Inc organization
 - developed in Go programming language
 - Client/Server Technology
 - Server - dockerd ( Docker Application Engine )
 - Client - docker
 - If both Server and Client they run on the same sytem, then they typically use local unix socket for
   their communication.  This is the defaut client/server communication mode, however REST api can be enabled ondemand.
 - Docker Application Engine also supports REST API, if enabled then even remote Docker and third-party applications
   can communicate with Docker Application Engine

### Docker Images
  - specification of a container
  - you could install all the software that you need in a container on the image level
  - similar to ISO or DVD images or VMWare image
  - Each Docker Image is assigned with an unique 256 bit Image ID.
  - Docker Images are broken down into one or more Docker Layers
  - Every Docker Layer also has an unique Layer ID

### Docker Layers
  - Layers has all the installed softwares
  - i.e files and folders
  - many Layers put together forms the file system of the Docker container
  - Docker uses a Union File System

### Docker Containers
   - instance of a Docker Image
   - multiple containers can be created from a single Docker Image
 
### Docker Registry
   - collection of many Docker Images
   - 3 types
       1. Local Docker Registry - this gets created autmatically , when you install docker on your local machine
       2. Remote Docker Registry - Docker Hub ( Website maintained by Docker Inc )
       3. Private Docker Registry (optional) 
          - If you need Remote Docker Registry like functionalities within your Organization you may setup
            a private registry with Sonatype Nexus or JFrog Artifactory
          - you also setup a private registry for learning purpose using registry:2 image from Docker Hub
    - Local Docker Registry is created automatically on the same machine where Docker Server is running

### Installing Docker Community Edition
```
su -
yum install yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce --allowerasing
```

### Enable and start the Docker Server
```
su -
systemctl enable docker
systemctl start docker
```

### You can test if docker is installed
```
docker --version
docker images
```
## Managing Docker Images

### Download docker image from Docker Hub Remote Registry to Local Docker Registry
```
docker pull hello-world:latest
```

### Listing docker images from Local Docker Registry
```
docker images
```

### Finding details about your docker installation
```
docker info
```

## Managing Docker Containers

### Creating container
```
docker run hello-world:lates
```
The expected output is
<pre>
[root@tektutor ~]# docker run hello-world:latest
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:cc15c5b292d8525effc0f89cb299f1804f3a725c8d05e158653a563f15e4f685
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

[root@tektutor ~]# 
</pre>

### Listing the currently running containers
```
docker ps
```
The expected output is
<pre>
[root@tektutor ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@tektutor ~]# 
</pre>
Since the hello-world container got terminated, we aren't able to see that container in the above output.

If you wish to see every container irrespective whether they are in running state or existed state, then try this
```
docker ps -a
```
The expected output is
<pre>
[root@tektutor ~]# docker ps -a
CONTAINER ID   IMAGE                COMMAND    CREATED         STATUS                     PORTS     NAMES
1be93ea842b3   hello-world:latest   "/hello"   4 minutes ago   Exited (0) 4 minutes ago             gracious_knuth
</pre>

### Creating containers in background mode
```
docker run -dit --name ubuntu1 --hostname ubuntu1 ubuntu:16.04 /bin/bash
docker run -dit --name ubuntu2 --hostname ubuntu2 ubuntu:16.04 /bin/bash
```
You can verify if both the containers are running
```
docker ps
```
The expected output is
<pre>
[root@tektutor ~]# docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
e1db6b37a87a   ubuntu:16.04   "/bin/bash"   2 seconds ago    Up 1 second               ubuntu2
e4221fa1b9c4   ubuntu:16.04   "/bin/bash"   41 seconds ago   Up 39 seconds             ubuntu1
[root@tektutor ~]# 
</pre>

### Creating containers in foreground mode
```
docker run -it --name ubuntu3 --hostname ubuntu3 ubuntu:16.04 /bin/bash
```
You will be taken inside the interactive container's shell immediately after the container is created.
You may verify the hostname of the container
```
hostname
```
You may verify the IP address of the container
```
hostname -i
```
You may list the files and directories inside the container
```
ls -l
```
You try to exit the container.  When you exit it terminates the bash that runs inside the container, hence the
container will be Exited.

```
exit
```
You can start the exited container as shown below
```
docker start ubuntu3
```
You may open a new interactive terminal inside the container
```
docker exec -it ubuntu3 bash
```
If you exit from the shell launched via exec command, it won't stop the container
```
exit
```
You may verify this behaviour, by listing containers
```
docker ps
```

### Deleting stopped container
```
docker rm ubuntu3
```

### Deleting a running container
```
docker stop ubuntu1
docker rm ubuntu1
```

### Deleting a running container forcibly
```
docker rm -f ubuntu2
```

### Deleting multiple running containers graciously
```
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

### Restarting a container
```
docker restart ubuntu1
```

### Deleting multiple running container forcibly
```
docker rm -f $(docker ps -aq)
```

### Renaming a container
```
docker rename ubuntu1 c1
```
In the above command, 'ubuntu1' is the existing name while 'c1' is the new name.
You may verify the change as shown below
```
docker ps
```

### Creating a nginx web server container
```
docker run -d --name nginx1 nginx:1.18
```
You may list the containers as shown below
```
docker ps
```

### Checking application logs
```
docker logs nginx1
```

### Listing container whose name starts with 'ubuntu'
```
docker ps -f "name=ubuntu*"
```

### Listing containers whose image is ubuntu
```
docker ps =f "ancestor=ubuntu"
```

### Port forwarding
```
docker run -d --name nginx1 --hostname nginx1 -p 8001:80 nginx:1.18
docker run -d --name nginx2 --hostname nginx2 -p 8002:80 nginx:1.18
```
Verify if both containers are running
```
docker ps -f "name=nginx*"
```

### Volume Mounting
```
mkdir -p /tmp/mysql
docker run -d --name db1 --hostname db1 -e MYSQL_ROOT_PASSWORD=root -v /tmp/mysql:/var/lib/mysql mysql:latest
```
Verify if the db1 container is running
```
docker ps -f "name=db1"
```
You may now open a shell inside db1 container to run sql queries
```
docker exec -it db1 sh
mysql -u root -p
```
When prompted for password, type 'root' without quotes.

You may now executed the mysql queries in the mysql prompt
```
SHOW DATABASES;
CREATE DATABASE tektutor;
USE tektutor;
CREATE TABLE Training (id int, name VARCHAR(30), duration VARCHAR(30));
INSERT INTO Training VALUES ( 1, "DevOps", "5 Days" );
INSERT INTO Training VALUES ( 2, "Microservices with SpringBoot", "5 Days");
SELECT * FROM Training;
```
You may now exit the mysql prompt and then exit the container
```
exit
exit
```
You may now delete the db1 container
```
docker rm -f db1
```
Let's create another container and mount the same volume
```
docker run -d --name db2 --hostname db2 -e MYSQL_ROOT_PASSWORD=root -v /tmp/mysql:/var/lib/mysql mysql:latest
```
Let's get inside the db2 container
```
docker exec -it db2 sh
mysql -u root -p
SHOW DATABASES;
USE tektutor;
SELECT * FROM Training;
```
As you can see, though db1 container was deleted we are still able to access the tektutor database and the records
in the Training table as we are using an externally mounted volume.
