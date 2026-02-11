# Kubernetes

## What Kubernetes is?
Open source container orchierstration tool.
Developed by Googlee,
Helps manage containerized applications.
K8s offers:
High Avilability,
Scalability, or high performance,
Disaster recovery - backup and restore.
### Basic architecture:
At least one master node (run several K8s processes absolutely necessary: API Server, Controller Manager, Scheduler, etcd ), connected couple worker nodes, each node has kubelet process. On each worker node run Docker container. Next very important service is Virtual Network (Creates one unified machine)
#### API Server
Entrypoint to K8s cluster
#### Controller Manager
keeps track of what happening in the cluster.
#### Scheduler
Ensure Pods placement
#### etcd
Kubernetes backing store
Allows you to manage containers in a vendor-independent manner.
Better to have at least two master nodes. Without master node, it is impossible to access worker nodes.
## Worker nodes
higher workload,
much bigger and more resources

### Pod
 - the smallest unit in Kubernetes, creates layer over container.
 - usually 1 application per Pod
 - each Pod get its own IP address, new IP address on re-create
 - one or more containers that share resources such as disk resources and a network and are part of a single context
 - Pods are ephemeral by design - short-lived, transient, stateless
   - Example 1: Pod containing an nginx container with a website
   - Example 2: A pod containing two containers: nginx with a website and a second container that checks the git repository for a new version.

### Service
  - The basic method for exposing services (ports) to other services, (Internal Service in case of iex. database to which cannot be accessible from outside. Check Ingress)
  - lifecycle of Pod and Service not connected. Pod dies, service keep running.
  - Permament IP address
  - Receives a fixed IP address in the cluster
  - Receives a fixed DNS name in the cluster based on the service name and namespace (in Kubernetes, namespace is a way to logically divide applications/projects) in which it was launched
  - Is by design persistent – ​​the opposite of ephemeral
    - Example: exposing MySQL services to applications in the cluster
### Ingress
Request goes firstly to Ingress and it does forwarding it to service.
### ConfigMap
External configuration of our application (URLs of database or other services) so in case of ie. URL change you do not need to build new image, just adjust change in ConfigMap.
! ConfigMap is for non-confidential data only !
Instead of use Secret
### Secret
Just like ConfigMap but store data in base64. In order to safely use Secrets, take at least the following steps:
   - Enable Encryption at Rest for Secrets
   - Enable or configure RBAC rules that restrict reading data in Secret (including via indirect means).
   - Where appropriate, also use mechanisms such as RBAC to limit which principals are allowed to create new Secrets or replace existing ones.
### Data Storage
#### Volumes
attaches physical hard drive to your Pod (local machine, remote storage outside of K8s)
! Kubernetes doesn't manage data persistance !

! Database can't be replicated via Deployment !
coz it has states. Required mechanism to decide which Pod read from storage, which Pod write to storage. Avoid Data Inconsistences. That mechanism is offered STATESFULSET
### StatefulSet
Component designed for MySQL, MongoDB, ElasticSearch, other databases.
## DEPLOYMENT = for stateLESS Apps
## StatefulSet = for stateFUL Apps or Databases.
DB are often hosted outside of Kubernetes.
# SUMMARY:
Pod - abstraction of containers
Service - communication
Ingress - route traffic to cluster
ConfigMap & Secret - external configuration
Volume - data persistence
Deployment & StatefulSet - replication

### Kubernetes always tries and strives to establish a target state.
## Control Plane components
### Kube-apiserver, the heart of the cluster
  - Provides REST interfaces to the control plane and datastore
  - All clients and applications interact with Kubernetes almost exclusively through the server API
  - Provides authentication, authorization, query validation, and access control to downstream cluster components
### etcd, the key-value datastore
  - permanent, consistent and highly available cluster status database.
  - project of the CNCF foundation
  - resistant to failures thanks to maintaining quorum rules
  - uses raft.github.io
### kube-controller-manager
  - main deamon, manage main components (reconcilation loops)
  - monitor clasters state via API server to keep target state
### kube-scheduler, the placement engine
  - run apps according to available resources
  - fines rules (affinity/anit-affinity,labels) which include info about available RAM, CPU, storage
  - default scheduler uses "bin packing"
## Worker Node,
### kubelet, the node agent
  - manage cycle of each node on which is running
  - configuration source: (JSON/YAML):
    - etcd
    - files path
    - remote http endpoint
    - own http API server  
### Kube-proxy, the network rules engine
  - manage network rules on each worker
  - connection forwarding, load balancer for Kubernetes services
  - proxy modes:
    - userspace
    - Iptables
    - Ipvs (default if supported)
### Container runtime
  - application compatible with Container Runtime Interface, which runs and manage containers:
    - Containerd (Docker)
    - Cri-o
    - Rkt
    - Kata
    - Virtlet (VM CRI compatible runtime)
## Optional Services
### Cloud-container-manager 
  - the gateway to cloud provider services (AWS, Azure, OVH)
  - configuration automatization of workers, apps routing, load balancing, storage...
    AWS:   
    - Route 53 - DNS records management
    - ELB - public and private load balancers configuration
    - EBS - storage maagement
    - IAM - access permission, users and roles authorization
### Cluster DNS (CoreDNS:
  - daemon responsible for resolving DNS names in the cluster
### Kube Dashboard
  - www service showing state of cluster and running applications.
### Metrics API server
  - provides metrics about Kbernetes services
### Kubernetes Networking
  - Pod Network
    - network available for whole cluster, responsible for Pod-to-Pod communication,
    - managable via CNI plugin (Container Network Interface)  
  - Service Network
    - range of virtual IP addreses available in whole cluster,
    - managed by kube-proxy to ensure service discovery
## API
#### Mandatory fields for obejct configuration:
```
  apiVersion: v1.02
  kind: Pod // _object type_
  metadata
    name: pod-example // _unique object name_
    namespace: default // _name of environment to which obejct belongs_
    uid: f98300d3-2234-2221-343-0030303 
```
#### Main Pods atributes:
  - name
  - image
  - ports
  - env
  - cmd
  - ags
```
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-example
spec:
  containers:
  - name: nginx
    image: nginx: stable-alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  - name: content
    iamge: alpine:latest
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 5;
        done
    volumeMounts:
    - name: html
      mountPath: /html
  volumes:
  - name: html
    emptyDir: {}  
```
#### Labels
Pairs key-balue used to identifying, describing and grouping related sets of obejcts and resources
Are NOT unique in cluster
Have strict syntax.
```
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-example
  labels:
    app: nginx
    env: prod
spec:
  containers:
  - name: nginx
```
#### Selector
they use labels to filter or select objects. Usefull to select servers or point which application may be added to load balancer under specific domain. 
```
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-example
  labels:
    app: nginx
    env: prod
spec:
  containers:
  - name: nginx
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
  nodeSelector:
    gpu: nvidia
```
#### Service
access to services provided by Pods, static IP unique for cluster, static DNS in namespace

#### Cluster IP
provides the service on an internal virtual IP in the cluster's internal network
#### NodePort
Extends service ClusterIP, expose the Pod port on each node's IP address, can be defined statically or dynamically in scope: 30 000 - 32 767. DO NOT START OTHER SERVICES ON HOSTS IN MENTIONED SCOPE!
#### LoadBalancer
extends NodePort, work in conjunction with an external system to map the external IP address of the KUBERNETES cluster to the hosted service, application or website
```
apiVersion: v1
kind: Pod
metadata: 
 name: multi-container-example
 labels:
   app: nginx
   env: prod
spec:
  type: LoadBalancer
  selctor:
   app: nginx
   env: pod
  ports:
   protocol:TCP
   port:80
   targetPort:80 
```
#### ExternalName
used to reference external services outside of cluster, 

