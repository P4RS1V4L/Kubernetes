##### Worker machine in K8s cluster
Node1 contains 2 application Pods: my-app & DB
  - each Node has multiple Pods on it
  - 3 processes must be installed on every Node:
    1. Container runtime (docker, containerd, cri-o)
    2. Kubelet (interacts with both - the container and node, starts the Pod with a container inside)
    3. Kube proxy (forward requests from services to Pods) 
  - Worker Nodes do the actual work
##### Master processes
- 4 processes run on every master node!
  1. API Server
    client -> _API Server_ (cluster gateway, act as a gatekeeper for authentication) -> validates request -> ..other processes .. -> Pod.
      Only one entrypoint into the cluster
  2. Scheduler
    Schedule new Pod -> API Server -> Scheduler -> where to put the Pod? -> Kubelet
  3. Controller manager (detects cluster state changes)
       Controller Manger -> Scheduler -> Kubelet
  4. etcd (Key Value store of cluster state)
