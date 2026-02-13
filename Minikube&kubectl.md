#### Minikube
Minikube basically it is one node cluster, where master processes and worker processes run on one Node. Docker container runtime pre-installed so you can run containers or pods with containers. 
- It creates Virtual Box on PC,
- Node runs in that virtual box, so virtualization is needed (install hypervisor)
- 1 Node K8s cluster
- for testing purposes.

#### kubectl
Commandline tool for for K8s cluster.
#### main commands:
$ kubectl get nodes
$ kubectl get pod
$ kubectl get services
$ kubectl create deployment nginx-depl --image=nginx
$ kubectl create deployment mongo-depl --image=mongo
$ kubectl get deployment
$ kubectl get pod
$ kubectl get replicaset    //managing replicas of a Pod. Replicas are created, deployment command additional parameter
  Layers of abstraction:
  - Deployment manages a replicaset
  - Replicaset manages all replicas of Pod
  - Pod is an abstraction of a container
  Everything below Deployment is handled by Kubernetes
$ kubectl edit deployment <<name>>
$ kubectl logs <pod-name>
$ kubectl describe pod <pod-name>
$ kubectl exec -it <pod-name> -- bin/bash  //gets terminal of named Pod.
$ kubectl delete deployment <name>
$ kubectl create deployment name image option1 option2
$ kubectl apply -f <config_file_name.yaml>
