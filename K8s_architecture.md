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
  1. API Server (you as client, interact with API Server, cluster gateway which gets initial requests about updates in the cluster or updates, act as a gatekeeper for authentication)
    client -> _API Server_ -> validates request -> ..other processes .. -> Pod.
      Only one entrypoint into the cluster
  2. Scheduler (just decides on which Node, new Pod should be scheduled)
    Schedule new Pod -> API Server -> _Scheduler_ -> where to put the Pod? -> Kubelet
  3. Controller manager (detects cluster state changes)
       Controller Manger -> Scheduler -> Kubelet
  4. etcd (Key Value store of cluster state, cluster 'brain', application data is NOT stored in etcd)
