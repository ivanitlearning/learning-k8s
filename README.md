# Learning k8s
Notes on learning Kubernetes, starting with CKA. Based on KodeKloud's course.

# k8s Architecture

Master node

* etcd cluster stores info about the cluster
* kube-scheduler responsible for scheduling applications or containers on nodes
* controllers take care of different functions like node controller, replication controller
* kube-apiserver responsible for orchestrating all operations within the cluster

Worker node

* kubelet - On worker node. Listens for instructions from kube-apiserver and manages containers 
* kube-proxy - Enables communication between services within the cluster

# etcd service

etcd is a key-value store. Comes with basic client `etcdctl`

Store items with `./etcdctl set key1 value1`

Retrieve item

```text
./etcdctl get key1
value1
```

etcd listens on port 2379 of the master node by default

# kube-scheduler 

This decides which nodes the pods are placed on, and informs kubelet to create the pods on their node.

* Filters nodes which can't host pods
* Ranks the pods by those with largest leftover resources and prioritises them.

# kubectl for pods

`kubectl get pods -o wide` - Shows status of pods, nodes they're deployed in, age and IP.

`kubectl describe pods <pod-name>` - Fully describe a pod

This does a dry run of  creating nginx pod from image, and redirects it to yaml file. Then create it with next command

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl apply -f pod.yaml
```

`kubectl edit pods redis` Edit the pods with vim and will exit and apply changes automatically. Alternatively, update the yaml file and apply again.

