# Kubernetes

## What Kubernetes is?
Allows you to manage containers in a vendor-independent manner.
### Pod
 - the smallest unit in Kubernetes
 - one or more containers that share resources such as disk resources and a network and are part of a single context
 - Pods are ephemeral by design - short-lived, transient, stateless
   - Example 1: Pod containing an nginx container with a website
   - Example 2: A pod containing two containers: nginx with a website and a second container that checks the git repository for a new version.
### Service
  - The basic method for exposing services (ports) to other services
  - Receives a fixed IP address in the cluster
  - Receives a fixed DNS name in the cluster based on the service name and namespace (in Kubernetes, namespace is a way to logically divide applications/projects) in which it was launched
  - Is by design persistent – ​​the opposite of ephemeral
    - Example: exposing MySQL services to applications in the cluster
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
