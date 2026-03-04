Viewing apiserver - kubeadm
```
$ kubectl get pods -nn kube-system
```
Viewing apiserver Options - kubeadm
```
$ cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
Non-kubeadmin setup viewing apiserver settings 
```
$ cat /etc/syystemd/system/kube-apiserver.service
```

Running processes & efective options:
```
$ ps -aux | grep kube-apiserver
```
