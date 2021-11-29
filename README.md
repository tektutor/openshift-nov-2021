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

Container Technology
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

Container Runtimes
  - LXC - low-level not so user-friendly
  - Docker - very user-friendly, hence very popular in the industry.  Docker a known vulnerabilty, Podman has solved
             it even before Docker, hence the industry started moving away from Docker
  - CRI-O - low-level not so user-friendly
  - Podman - slowly gaining popularity, mostly likely might takeaway the shares of Docker
   
 Container Orchestration Platforms
   - Docker SWARM - Docker Inc's native orchestration platform that supports only Docker Container Runtime
   - Google Kubernetes(K8s) - supports many different Container Runtimes, this is also time tested
   - RedHat OpenShift(IBM) - built on top of Google Kubernetes + CI/CD + Private Docker Registry, etc 

Docker Runtime
 - developed by Docker Inc organization
 - developed in Go programming language
 - Client/Server Technology
 - Server - dockerd ( Docker Application Engine )
 - Client - docker
 - If both Server and Client they run on the same sytem, then they typically use local unix socket for
   their communication.  This is the defaut client/server communication mode, however REST api can be enabled ondemand.
 - Docker Application Engine also supports REST API, if enabled then even remote Docker and third-party applications
   can communicate with Docker Application Engine

Docker Images
  - specification of a container
  - you could install all the software that you need in a container on the image level
  - similar to ISO or DVD images or VMWare image

Docker Containers
   - instance of a Docker Image
   - multiple containers can be created from a single Docker Image
 
Docker Registry
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
