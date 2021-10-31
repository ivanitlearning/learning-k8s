# Learning k8s
Notes on learning Kubernetes, starting with CKA. Based on KodeKloud's course.

# 1. Core Concepts

## 1.1 k8s architecture

Master node

* etcd cluster stores info about the cluster
* kube-scheduler responsible for scheduling applications or containers on nodes
* controllers take care of different functions like node controller, replication controller
* kube-apiserver responsible for orchestrating all operations within the cluster

Worker node

* kubelet - On worker node. Listens for instructions from kube-apiserver and manages containers 
* kube-proxy - Enables communication between services within the cluster

## 1.2 etcd service

etcd is a key-value store. Comes with basic client `etcdctl`

Store items with `./etcdctl set key1 value1`

Retrieve item

```text
./etcdctl get key1
value1
```

etcd listens on port 2379 of the master node by default

## 1.3  kube-scheduler 

This decides which nodes the pods are placed on, and informs kubelet to create the pods on their node.

* Filters nodes which can't host pods
* Ranks the pods by those with largest leftover resources and prioritises them.

## 1.4 kubectl for pods

`kubectl get pods -o wide` - Shows status of pods, nodes they're deployed in, age and IP.

`kubectl describe pods <pod-name>` - Fully describe a pod

This does a dry run of  creating nginx pod from image, and redirects it to yaml file. Then create it with next command

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl apply -f pod.yaml
```

`kubectl edit pods redis` Edit the pods with vim and will exit and apply changes automatically. Alternatively, update the yaml file and apply again.

Can also use to edit other resources like `kubectl edit replicaset replica-set-1`
Create pod with image `kubectl run podname --image=redis`

## 1.5 kubectl other commands

These commands can be generalised for other resources as well.

`kubectl explain pods` - Explains the structure of pod definition files, can also use other Kube resources such as replicasets, 

Check with `kubectl api-resouces` to see what you can query

`kubectl <command> --record` Makes command appear in history?

`kubectl get all` List all the resources which are provisioned.

`kubectl api-resources -o wide` Lists all the resources you can create with each apiversion. [[Ref](https://stackoverflow.com/a/55358685/7908040)]

`kubectl logs worker-app-pod` Display error logs for problematic pod **worker-app-pod**

`kubectl delete replicasets,services,deployments,pods --all` - Deletes all of these in the current namespace.

Displays the YAML config for running k8s resource `kubectl get replicaset replicaset-1 -o yaml` 

Or create the yaml file from image `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

Display all the labels with `kubectl get all --show-labels`

## 1.7 k8s resources

## 1.8 ReplicaSet

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

Scales the replicaset to 5 `kubectl scale replicaset --replicas=5 replicaset-1`

List the replica sets with `kubectl get replicaset`

To change the number of replicas, edit the file and `kubectl replace -f replicaset-defn.yaml`

Delete multiple ReplicaSets and underlying pods with `kubectl delete replicaset myapp-replicaset-1 replicaset-2` 

* **selector** is optional field for ReplicationController but not for ReplicaSet. When omitted, k8s assumes the selector to be same as the template specified in the template field.

Note: To edit a RS which is running, you need to edit its config then delete all the pods created by it so it can re-create.

## 1.9 Deployments

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
 template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
     containers:
     - name: nginx-container
       image: nginx
 replicas: 3
 selector:
   matchLabels:
    type: front-end
```
* Same as ReplicaSet, but allows for rollback.
* Updates pods either via Recreate or Rolling update (default)

Rollback with `kubectl rollout undo deployment/myapp-deployment`

Update without modifying YAML with `kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`

Check status and history of deployment

```text
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```

## 1.10 Services

### 1.11 NodePort

* NodePort service maps a port on the node to a port on the pod in the node.
  * NodePort: port exposed on the node. Ranges 30,000 - 32767
  * Port: Port on the service itself
  * TargetPort: Port on the pod.

### 1.11.1 NodePort definition

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

## 1.12 ClusterIP

* Creates an endpoint for consolidating pods of a particular deployment so others can talk to it.

![srvc1](https://github.com/kodekloudhub/certified-kubernetes-administrator-course/raw/master/images/srvc1.PNG)

### 1.12.1 ClusterIP Definition

```yaml
apiVersion: v1
kind: Service
metadata:
 name: back-end
spec:
 types: ClusterIP
 ports:
 - targetPort: 80
   port: 80
 selector:
   app: myapp
   type: back-end
```

* ClusterIP is the default type; assumed if omitted.

## 1.13 LoadBalancer

* Same as ClusterIP except it works with cloud providers. Setting type to LoadBalancer will make it load balance and create an DNS endpoint which other services can access.

```yaml
apiVersion: v1
kind: Service
metadata:
 name: myapp-service
spec:
 types: LoadBalancer
 ports:
 - targetPort: 80
   port: 80
   nodePort: 30008
```

* This doesn't work on unsupported cloud platforms; it gets interpreted as NodePort instead.

### 1.14 Service commands

Expose a port on a pod with a service `kubectl expose pod redis-pod --port=6379 --type=ClusterIP --name=redis-service`

## 1.15 Namespaces

* Within a namespace, you can refer to a service with just its hostname `mysql.connect("db-service")`

* When connecting to a service in another namespace with the same name you need to specify its "FQDN" like `mysql.connect("db-service.dev.svc.cluster.local")`

* Default domain name for cluster is **cluster.local**
* Resource type is given by **svc**
* Subdomain is **dev**
* Name of service **db-service**

* When namespace not specified, assumed to be referring to default namespace

* Work with resources in other namespaces by specifying namespace with `--namespace=name`

* Or specify in the yaml under **metadata** so you don't need to specify in `kubectl create`

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    namespace: dev
    labels:
       app: myapp
       type: front-end
  spec:
    containers:
    - name: nginx-container
      image: nginx
  ```

* Create the namespace in CLI with `kubectl create namespace dev`

* Switch to **dev** namespace permanently with `kubectl config set-context $(kubectl config current-context) --namespace=dev`

* View pods in all namespace with `kubectl get pods --all-namespaces`

### 1.16 Resource Quota

* Possible to limit resources for namespaces with a ResourceQuota yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

## 1.17 Imperative vs Declarative

### 1.17.1 Imperative

* Quicker if you just need to create a resource from scratch
* Or edit object in place

### 1.17.2 Declarative

Can use kubectl to apply a whole group of yaml files `kubectl apply -f /path/to/yaml`

* **kubectl apply** actually stores the last known config as a inline JSON doc in the live object config. Because of this don't mix the declarative and imperative approach together or the last known config won't be reflected accurately.

# 2. Scheduling

* Scheduler assigns pods to nodes.
* Without it, pods get stuck in Pending status.
* Check for scheduler in namespace **kube-system**

## 2.1 Manual scheduling

* Node property can be specified only during pod creation time, not after.

* Delete and re-create pod to create it on specified node.

  ```yaml
  spec:
    nodeName: node02
  ```

* Alternative is to do a curl to POST this Binding object

  ```yaml
  apiVersion: v1
  kind: Binding
  metadata:
    name: nginx
  target:
    apiVersion: v1
    kind: Node
    name: node02
  ```

  ```bash
  curl --header "Content-Type: application/json" --request POST --data '{"apiVersion":"1", "kind": "Binding" ... } ' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
  ```

## 2.2 Labels and selectors

Select pods with specific labels with `kubectl get pods --selector app=App1`

Get all objects with the label **env=prod** 

### 2.2.1 Annotations

* Unlike labels, selectors, annotations are used to record other details for information

  ```yaml
  metadata:
    annotations:
      buildversion: 1.34
  ```

## 2.3 Taints and Tolerations

* Purpose is to allow/disallow pods from being placed in certain nodes.
* Taint - Applied on a node, only those pods which have tolerance for this taint can be created on it.
* Toleration - Applied on a pod so it can be placed on nodes with specific taints.

### 2.3.1 Taints

* Apply taint on node `kubectl taint nodes nodename key=value:taint-effect`
  * `NoSchedule` - No pods will be placed unless can tolerate the taint. Existing incompatible pods remain.
  * `PreferNoSchedule` - k8s tries not to schedule incompatible pods on the node
  * `NoExecute` No pod placement and evicts all incompatible pods
* Can also specify just the key without value, but must have taint-effect.
* Can only restrict non-tolerated pods from being scheduled on the node; tolerated pods can still be placed on untainted nodes.
* To make pods go to specific nodes, have to use *node affinity* instead.
* By default, k8s applies a taint on the master node to prevent any pod scheduling on it
  * View taint on master with `kubectl describe node kubemaster | grep Taint`
* Remove taint on node `kubectl taint nodes nodename key=value:NoSchedule-`

### 2.3.2 Tolerations

* Specified in pod definitions, must match the taints.

  ```yaml
  spec:
   tolerations:
   - key: "app"
     operator: "Equal"
     value: "blue"
     effect: "NoSchedule"
  ```

## 2.4 Node Selectors

* Allows you to label nodes with key and value so pods can be created on specific nodes.
* Label a node with `kubectl label nodes <node-name> <label-key>=<label-value>`

* Pod definition contains label of node to be scheduled on

  ```yaml
  spec:
   nodeSelector:
    size: Large
  ```

* Very simple selection only, can't say you want node A **or** B, nor **not** C

## 2.5 Node Affinity

* Allows for **or**, **not** Boolean operators for pod scheduling from POV of pod

* Included in *pod* definitions

  ```yml
  spec:
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In
              values: 
              - Large
              - Medium
              # Alternatively if just one value
              values: ["blue"]
  ```

Node affinity types - Considered only during scheduling, ignored during execution.

* `requiredDuringSchedulingIgnoredDuringExecution`
* `preferredDuringSchedulingIgnoredDuringExecution`

Planned

- `requiredDuringSchedulingRequiredDuringExecution`
- `preferredDuringSchedulingRequiredDuringExecution`

Valid operator types [[ref](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)]:

* `NotIn`
* `Exists`
* `DoesNotExist`
* `Gt`
* `Lt`

Limitations

* Doesn't guarantee that other pods won't be placed in those nodes. This only affects where pods get to go, from a pod POV.

## 2.6 Taints, Tolerations vs Node Affinity

* Taints, tolerations prevent unwanted pods from being scheduled on specified nodes.
* Node affinity makes pods prefer or strictly require to be scheduled on specified nodes.
* Both used together to ensure pods go where they are wanted, and unwanted pods don't get scheduled on certain nodes.

## 2.7 Resource Limits

* 1 vCPU equals

  * 1 AWS vCPU
  * 1 GCP Core
  * 1 Azure Core
  * 1 Hyperthread

* k8s limits each container to

  * 1 vCPU (can't exceed)
  * 512 Mi (can exceed, but will get terminated eventually)
  * 

* If your application within the pod requires more than the default resources, you need to set them in the pod definition file.

  ```yaml
  spec:
   containers:
     resources:
       requests:
        memory: "1Gi"
        cpu: "1"
  ```

* Can set resource limits in the pod definition file

  ```yaml
  spec:
   containers:
     resources:
       requests:
        memory: "1Gi"
        cpu: "1"
       limits:
         memory: "2Gi"
         cpu: "2"
  ```

## 2.8 Daemon Sets

* Keeps a pod on every node, automatically provisioned when joining cluster and destroyed when leaving cluster

### 2.8.1 Use cases

* Ideal for running monitoring software on each node eg. networking agent.
* Deploy **kube-proxy** on each node.

### 2.8.2 DaemonSet definition

* Similar to ReplicaSet, except for kind.

  ```yaml
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: monitoring-daemon
    labels:
      app: nginx
  spec:
    selector:
      matchLabels:
        app: monitoring-agent
    template:
      metadata:
       labels:
         app: monitoring-agent
      spec:
        containers:
        - name: monitoring-agent
          image: monitoring-agent
  ```

* reated with `kubectl create 0f daemon-set.yaml`

* Viewed with `kubectl get daemonsets` or `kubectl describe daemonsets`

* Best way to create a DS is to create a deploy, then remove fields like **replica**, **strategy** etc. and change kind to DS.s

## 2.9  Static Pods

* Nodes that exist without a master node, api-server or any controllers
* Nodes that have kubelet only.
* kubelet checks **/etc/kubernetes/manifests** occasionally for pod yaml definition files and creates them
* Will check files for changes and recreates pods for changes to take effect
* Limited to pods only, not any other resource
* Path can be configured with argument when starting kubelet
  * **--pod-manifest-path=/path/to/defn/files**
  * **--config=kubeconfig.yaml**
    * kubeconfig.yaml contains line `staticPodPath: /etc/kubernetes/manifest`
* Once created, view pods with `docker ps` command