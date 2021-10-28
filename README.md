# Learning k8s
Notes on learning Kubernetes, starting with CKA. Based on KodeKloud's course.

# k8s architecture

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

Can also use to edit other resources like `kubectl edit replicaset replica-set-1`

# kubectl other commands

These commands can be generalised for other resources as well.

`kubectl explain pods` - Explains the structure of pod definition files, can also use other Kube resources such as replicasets, check with `kubectl api-resouces` to see what you can query

`kubectl <command> --record` Makes command appear in history?

`kubectl get all` List all the resources which are provisioned.

`kubectl api-resources -o wide` Lists all the resources you can create with each apiversion. [[Ref](https://stackoverflow.com/a/55358685/7908040)]

`kubectl logs worker-app-pod` Display error logs for problematic pod **worker-app-pod**

`kubectl delete replicasets,services,deployments,pods --all` - Deletes all of these in the current namespace.

## Imperative vs Declarative

`kubectl create` is imperative, will specify what resources to create while `kubectl apply` is declarative and just states the desired end-state config.

# k8s resources

## ReplicaSet

Newer version of Replication Controller. Here we see its used to monitor and maintain 3 running instances of a defined pod. The selector 

```yaml
# replicaset-defn.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    # This section contains the pod definition
    metadata:
      name: myapp-pod
      labels:
         app: myapp-pod
         type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3 # How many pods to maintain
  selector:
    matchLabels:
      type: front-end
```

Create the replicaset with `kubectl create -f replicaset-defn.yaml`

Displays the YAML config for running k8s resource `kubectl get replicaset replicaset-1 -o yaml` 

Scales the replicaset to 5 `kubectl scale replicaset --replicas=5 replicaset-1`

List the replica sets with `kubectl get replicaset`

To change the number of replicas, edit the file and `kubectl replace -f replicaset-defn.yaml`

Delete multiple ReplicaSets and underlying pods with `kubectl delete replicaset myapp-replicaset-1 replicaset-2` 

## Deployments

* Similar to ReplicaSet, but allows for rollback.

* Updates pods either via Recreate or Rolling update (default)

Rollback with `kubectl rollout undo deployment/myapp-deployment`

Update without modifying YAML with `kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`

Check status and history of deployment

```text
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```

## Services

### NodePort

* NodePort service maps a port on the node to a port on the pod in the node.
* NodePort itself is the port exposed on the node after the mapping is done. Ranges 30,000 - 32767

#### NodePort definition

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30008
```

In this definition 30008 is exposed on the node which directs to the service's port 80 which in turn connects to the pod's 80.

## ClusterIP

## LoadBalancer

## kubectl service commands

