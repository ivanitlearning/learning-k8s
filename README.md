# Learning k8s
Notes on learning Kubernetes, starting with CKA. Based on KodeKloud's course.

# 1. Core Concepts

## 1.1 k8s architecture

### 1.1.1 Master node

* etcd cluster stores info about the cluster
* kube-scheduler responsible for scheduling applications or containers on nodes
* controllers take care of different functions like node controller, replication controller
* kube-apiserver responsible for orchestrating all operations within the cluster

### 1.1.2 Worker node

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

Set environment variable **KUBE_EDITOR** to specify editor when calling `kubectl edit` eg. `export KUBE_EDITOR=vim`

Start the nginx container using the default command, but use custom arguments (arg1 .. argN) for that command.

`kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>`

Start the nginx container using a different command and custom arguments.

`kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>`

Exec bash shell into container within pod `kubectl exec -it pod-name --container main-app -- /bin/bash`

Or just execute command and exit `kubectl exec -it pod-name --container main-app -- cat /var/log/syslog`

Delete pods quicker with `--force`

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

* Similar to Deployment, except for kind.

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

* Created with `kubectl create 0f daemon-set.yaml`

* Viewed with `kubectl get daemonsets` or `kubectl describe daemonsets`

* Best way to create a DS is to create a deploy, then remove fields like **replica**, **strategy** etc. and change kind to DS.s

## 2.9  Static Pods

* Nodes that exist without a master node, api-server or any controllers.
* Nodes that have kubelet only.
* kubelet checks **/etc/kubernetes/manifests** occasionally for pod yaml definition files and creates them
  * Will check files for changes and recreates pods for changes to take effect
* Static pods have names which the node name is appended to theirs.
* Limited to pods only, not any other resource
* Path can be configured with argument when starting kubelet (check with `ps aux | grep kubelet`)
  * **--pod-manifest-path=/path/to/defn/files**
  * **--config=kubeconfig.yaml**
    * kubeconfig.yaml contains line `staticPodPath: /etc/kubernetes/manifest`
* Once created, view pods with `docker ps` command
* Pods created by kubelet will also appear with `kubectl get pods`

### 2.9.1 Use cases

* Used for deploying control-plane components as static pods on master nodes with kubelet only. Put these files in the pod definition folders.
  * controller-manager.yaml
  * apiserver.yaml
  * etcd.yaml

* kubeadm uses this to set up a k8s cluster

### 2.9.2 Tips and commands

* Identify static pods by identifying which pods have $nodename as part of their names [[ref](https://stackoverflow.com/questions/65657808/how-to-identify-static-pods-via-kubectl-command#comment116095266_65658362)]

	```text
	root@controlplane:~# kubectl get nodes
	NAME           STATUS   ROLES                  AGE     VERSION
	controlplane   Ready    control-plane,master   4m1s    v1.20.0
	node01         Ready    <none>                 2m56s   v1.20.0
	root@controlplane:~# kubectl get pods -A | grep -i controlplane
	kube-system   etcd-controlplane                      1/1     Running   0          8m34s
	kube-system   kube-apiserver-controlplane            1/1     Running   0          8m34s
	kube-system   kube-controller-manager-controlplane   1/1     Running   0          8m34s
	kube-system   kube-scheduler-controlplane            1/1     Running   0          8m34s
	```

## 2.10 Multiple Schedulers

* Allows your own custom scheduler for pods
* Your custom scheduler can be used for certain apps, while the rest goes through default
* Scheduler itself is a pod

### 2.10.1 Custom scheduler

* Get scheduler binary from https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler

* Created with a pod definition file with `kubectl create -f custom-scheduler.conf`

* Default k8s scheduler definition in **/etc/kubernetes/manifests/kube-scheduler.conf**

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-custom-scheduler
    namespace: kube-system
  spec:
    containers:
    -  command:
       - kube-scheduler
       - --address:127.0.0.1
       - kubeconfig=/etc/kubernetes/scheduler.conf
       # For HA setup where you have multiple master nodes with k-scheduler process on both of them
       # Set --leader-elect=false so the custom scheduler can move on to the main loop
       - --leader-elect=true
       # Custom name for scheduler
       - --scheduler-name=my-custom-scheduler
       - --lock-object-name-mycustom-scheduler
       image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
       name: kube-scheduler
  ```

* Specify custom scheduler to use for a given pod in the pod definition file

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - image: nginx
      name: nginx
    schedulerName: my-custom-scheduler
  ```

### 2.10.2 Commands

* Check which node the scheduler gets assigned with `kubectl get events`
* View logs of custom scheduler with `kubectl logs my-custom-scheduler --namespace=kube-system`

# 3. Logging & Monitoring

* Tools for monitoring pods and nodes
  * metrics server

## 3.1 Metrics Server

* Installation

```text
git clone https://github.com/kubernetes-incubator/metrics-server.git
kubectl create -f metric-server/deploy/1.8+/
```

* View cluster performance

```text
kubectl top node
kubectl top pod
```

## 3.2 Viewing application logs

* View pod logs with `kubectl logs -f podname`
* For multi-container pods, to view container logs within a pod do `kubectl logs -f podname containername`

# 4. Application Lifecycle Management

## 4.1 Rolling Updates and Rollbacks

* Some commands to help check deployment or rollout history

* Check status of deployment with `kubectl rollout status deployment/myapp-deployment`

* Check rollout history with `kubectl rollout history deployment/myapp-deployment`

* Update with 

  * `kubectl apply -f deployment.yaml` or 
  * `kubectl set image deployment/myapp-deployment <container-name>=nginx:1.9.1`

* Rollback with `kubectl rollout undo deployment/myapp-deployment`

* Check rollout strategy with `kubectl describe deploy deploy-name` and look for **StrategyType**

* Change rollout strategy with `kubectl edit deploy deploy-name` 

  ```yaml
  spec:
    strategy:
      type: Recreate
  ```

### 4.1.1 Update strategy

* Recreate - Replace all deployed pods at once
* Rolling update - Replace in batches (DEFAULT)
  * **RollingUpdateStrategy** tells you how many pods can go down at one time during rolling updates.

## 4.2 Commands and Arguments in k8s

In dockerfile we can have

```yaml
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["10"]
```

Similar to specifying `CMD` and `ENTRYPOINT` in docker

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
 containers:
 - name: ubuntu-sleeper
   image: ubuntu-sleeper
   command: ["sleep2.0"] # overwrites ENTRYPOINT ["sleep"]
   args: ["15"] # = overwrites CMD ["10"]
```

Alternatively we can specify as an array with

```yaml
spec:
  command:
  - "sleep"
  - "10"
```

## 4.3 Configure environmental variables in apps

Specify env variables in pod definition files

```yaml
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
   - containerPort: 8080
   env:
   - name: APP_COLOR
     value: pink
```

### 4.3.1 Via ConfigMap

Pod definition file references the ConfigMap

```yaml
spec:
  containers:
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

ConfigMap app-config:

```yaml
APP_COLOR: blue
APP_MODE: prod
```

#### 4.3.1.1 Imperative

Creating configmap by specifying key-value pairs `kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod`

By specifying files `kubectl create configmap app-config --from-file=app_config.properties`

#### 4.3.1.2 Declarative

configmap.yaml, create with `kubectl apply -f configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
 name: app-config
data:
 APP_COLOR: blue
 APP_MODE: prod
```

#### 4.3.1.3 ConfigMap commands

View/describe configmaps `kubectl get configmaps cfgmap`  and `kubectl describe configmaps cfgmap`

#### 4.3.1.4 Troubleshooting with ConfigMap

Some pods may mount config within volumes with ConfigMap. The following shows kube-proxy using a config file specified at **/var/lib/kube-proxy/configuration.conf** which is on mount path **/var/lib/kube-proxy**

```text
root@controlplane:~# k -n kube-system describe pods kube-proxy
Name:                 kube-proxy-44d5s
Namespace:            kube-system   
...
Controlled By:  DaemonSet/kube-proxy
Containers:                                                                         
  kube-proxy:
    Container ID:  docker://78da095b5b318d197cdb427d0962d0cbdebefe85349a724cf54bb78b1dad7b96
    Image:         k8s.gcr.io/kube-proxy:v1.18.0
    Image ID:      docker-pullable://k8s.gcr.io/kube-proxy@sha256:9e858386d52d0abaf936c1d10a763648ab7d85c8eb0af08a50a64238146e5571
    Port:          <none>
    Host Port:     <none>
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/configuration.conf
      --hostname-override=$(NODE_NAME)
...
    Mounts:
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
      /var/lib/kube-proxy from kube-proxy (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-proxy-token-rxfjp (ro)
...
Volumes:
  kube-proxy:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-proxy
    Optional:  false
```

But a check with the ConfigMap shows the config file isn't **configuration.conf** but rather **config.conf**

```text
root@controlplane:~# k -n kube-system describe cm kube-proxy
Name:         kube-proxy
Namespace:    kube-system
Labels:       app=kube-proxy
Annotations:  kubeadm.kubernetes.io/component-config.hash: sha256:3c1c57a3d54be93d94daaa656d9428e3fbaf8d0cbb4d2221339c2d450391ee35
                      
Data                 
====               
config.conf:    
---- 
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
bindAddressHardFail: false
```

So to fix this we can edit the kube-proxy pod to load config from config.conf instead. Since this pod is created by a DaemonSet, we should kubectl edit DaemonSet to correct this. Once fixed, we can verify with kubectl exec that the mount path indeed contains config.conf

```text
root@controlplane:~# k -n kube-system exec -it kube-proxy-bjzfc -- ls -lah /var/lib/kube-proxy
total 16K
drwxrwxrwx 3 root root 4.0K Nov 18 16:43 .
drwxr-xr-x 1 root root 4.0K Nov 18 16:43 ..
drwxr-xr-x 2 root root 4.0K Nov 18 16:43 ..2021_11_18_16_43_54.821752807
lrwxrwxrwx 1 root root   31 Nov 18 16:43 ..data -> ..2021_11_18_16_43_54.821752807
lrwxrwxrwx 1 root root   18 Nov 18 16:43 config.conf -> ..data/config.conf
lrwxrwxrwx 1 root root   22 Nov 18 16:43 kubeconfig.conf -> ..data/kubeconfig.conf
```

### 4.3.2 Via Secrets

Pod definition referencing secrets

```yaml
spec:
  containers:
    env:
    - name: APP_COLOR
      valueFrom:
        secretKeyRef:
          name: mysecret # Name of Secret
          key: appcolor
```

app_secret.properties

```yaml
DB_Host: mysql
DB_User: root
DB_Password: passwrd
```

Alternatively use **envFrom** to import all Secret's data as environment variables instead of specifying each

```yaml
spec:
  containers:
      envFrom:
      - secretRef:
          name: mysecret # Name of Secret
```

Secrets in Pods as volumes

```yaml
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

Inside container, stored in **/opt/app-secret-volumes**, files as keys

#### 4.3.2.1 Imperative

```text
# Via command line
kubectl create secret generic secret-name --from-literal=DB_Password=passwrd
# Via file
kubectl create secret generic app-secret --from-file=app_secret.properties 
```

#### 4.3.2.2 Declarative

* Values must be stored in base64, then use `kubectl create/apply`

secret-data.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: app-secret
data:
  DB_Host: bX1zcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

* Alternatively use **stringData** to avoid base64

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: app-secret
stringData:
  DB_Host: mysql
  DB_User: user
  DB_Password: Password1
```

#### 4.3.2.3 Commands

View/describe secrets with `kubectl get/describe secrets`

To view values, do `kubectl get secret app-secret -o yaml`

## 4.4 Multi-container Pods

* Allow multiple containers to run together
* Here there's an array of two member containers in pod definition.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - ContainerPort: 8080
  - name: log-agent
    image: log-agent
```

## 4.5 Init Containers

* When pod is created processes containers named initContainers have commands which are run to completion (just once) before the other containers start.
* Specified in pod definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

* kubelet runs the Pod's init containers in the order they appear in the Pod's spec (so only one init container is run at a time until they are all complete) [[ref](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)]

* If command in init container fails, it'll enter a CrashLoopBackoff state and containers won't execute.

# 5. Cluster Maintenance

## 5.1 Node maintenance

* Node down for > 5 min (default), pods are terminated on that node. 
* If part of RS or deployment, will be re-created on other nodes. Otherwise lost forever.
* Pod eviction timeout - Time taken for pods to be recreated when nodes go down. Set on controller manager. Default 5 min.
* If you need < 5 min (default) to take the node offline, you can go ahead. However, if you can't, drain the nodes with `kubectl drain node-name` to remove the pods on it.
  * Pods on node terminated and re-created elsewhere
  * Node marked as unschedulable, no scheduling until unmarked.
  * Re-enable node with `kubectl uncordon node-name`
  * Pods will *not* be moved back to the uncordoned node automatically.
* To mark node un-schedulable without terminating pods `kubectl cordon node-name`

## 5.2 Kubernetes Versions

* Check installed k8s version on nodes with `kubectl get nodes`
* Format: v1.10.3 - {major}.{minor}.{patch}

## 5.3 Cluster Upgrade

* kube-apiserver needs to be the highest version in the cluster (note the minor version number)
* Components versions can be
  * x-1 versions lower than kube-apiserver
    * controller-manager
    * kube-scheduler
  * x-2 versions lower than kube-apiserver
    * kubelet
    * kube-proxy

* kubectl can be one version higher or lower than kube-apiserver

### 5.3.1 Upgrading methods

1. Easily upgraded if on cloud services
2. kubeadm can upgrade with `kubeadm upgrade plan` and `kubeadm upgrade apply`
3. kubeadm doesn't install or upgrade kubelet

All-at-once:

* Upgrade all the worker nodes at once with disruption

Rolling:

* Pods get moved to other nodes, then down for upgrading

New nodes (AWS immutable):

* New nodes with higher k8s version get added to cluster
* Pods get moved to the new nodes, and removed from old ones

### 5.3.2 Upgrading sequence

1. Master node gets upgraded first, then worker nodes.
2. While upgrading, control plane components eg. api-server, scheduler and controller-manager go down
3. Worker nodes and apps still functioning. 
4. All management functions are down ie. no `kubectl`, no deployment or modification allowed.
5. Now upgrade worker node kubeadm then kubelet

### 5.3.3. Upgrade instructions in detail

#### 5.3.3.1 Upgrade master node

1. Check what you can upgrade components to with `kubeadm upgrade plan`, 
   * Note this is the highest you can upgrade the cluster to if kubeadm is not upgraded
2. Drain the node with `kubectl drain masternode`
   1. Verify the master node has unschedulable taint, and has no running pods

Upgrade the kubeadm package if needed (to support higher cluster versions)

1. Check available package versions for kubeadm to upgrade to with `apt list -a kubeadm`, say **1.20.0-00**
2. Upgrade the kubeadm on master node with `apt upgrade kubeadm=1.20.0-00`
   1. Verify kubeadm has been upgraded to version `kubeadm version`
3. Verify there are now new cluster versions to upgrade to with `kubeadm upgrade plan`, say **v1.20.00**
4. Upgrade the cluster with `kube upgrade apply v1.20.0`

Now upgrade the kubelet package version

1. Upgrade the kubelet version, check available package versions with `apt list -a kubelet` 
   1. Check current installed kubelet package version with `dpkg -l | grep kubelet`
   2. Upgrade kubelet with `apt upgrade kubelet=1.20.0-00`

When complete, uncordon the node

1. Uncordon the master node with `kubectl uncordon masternode`
   1. Verify the taint is gone

#### 5.3.3.2 Upgrade worker nodes

Next upgrade the worker nodes (same steps above)

1. Drain worker node (from master node)
2. Install package kubeadm with target version on worker node
3. Upgrade the node config to the same version with `kubeadm upgrade node`
   * Note: This step is not present above for master node.
4. Install package kubelet with target version on worker node
5. Exit, go back to master node, check with `kubectl get nodes` that kubelet is upgraded on worker node.

Tips

1. Use `apt-mark showhold` to see which packages are held back from upgrading

## 5.4 Backup and Restore

* Save all resource config in all namespaces to yaml `kubectl get all --all-namespaces -o yaml`
* Alternative is to backup etcd

### 5.4.1 Backup etcd

* etcd runs as a pod on the master node.

* Check the arguments for running etcd by describing the pod

  ```text
  etcd
  --cert-file=/etc/kubernetes/pki/etcd/server.crt	# ETCDCTL_CERT
  --data-dir=/var/lib/etcd 						# etcd datadir
  --key-file=/etc/kubernetes/pki/etcd/server.key	# ETCDCTL_KEY
  --listen-client-urls=https://127.0.0.1:2379,https://10.32.62.3:2379 # etcd endpoint
  --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt # ETCDCTL_CACERT
  ```

* First export the environmental variables so you don't need to specify them [[ref](https://stackoverflow.com/questions/52695573/why-do-i-need-to-put-etcdctl-api-3-in-front-of-etcdctl-for-etcdctl-snapshot-save)]

  ```bash
  export ETCDCTL_API=3 # Specifies etcdctl ver 3
  export ETCDCTL_CACERT=/etc/etcd/ca.pem
  export ETCDCTL_CERT=/etc/etcd/kubernetes.pem
  export ETCDCTL_KEY=/etc/etcd/kubernetes-key.pem
  etcdctl member list --endpoints=https://127.0.0.1:2379 
  ```

* The CACERT, CERT and KEY are needed if the etcd DB is TLS protected.

  Otherwise you need to preface all commands with 

  ```text
  ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/etcd-server.crt \
  --key=/etc/kubernetes/pki/etcd/etcd-server.key snapshot save /tmp/snapshot.db
  ```

* Check that you can list members if all the env variables correct or arguments passed correctly with `etcdctl member list`

  ```text
  root@controlplane:~# env | grep ETCDCTL
  ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
  ETCDCTL_API=3
  ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
  ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
  root@controlplane:~# etcdctl member list --endpoints=127.0.0.1:2379
  c94fae94143096ba, started, controlplane, https://10.3.84.3:2380, https://10.3.84.3:2379
  ```

* Save snapshot of etcd with `etcdctl snapshot save snapshot.db`

* Check status of snapshot `etcdctl snapshot status snapshot.db -w=table`

* Check with `ps aux` where the etcd **--data-dir** is

* To restore etcd from backup
  1. Stop kube-apiserver service `service kube-apiserver stop` (if k8s controller was manually installed)
  2. Restore snapshot with `etcdctl snapshot restore snapshot.db --data-dir /path/to/new/data/dir/`
  3. Update the `--data-dir` by checking the staticPodPath of kubelet for etcd.yaml then edit the yaml file for kubelet to re-create the etcd pod
  4. Start the kube-apiserver service `service kube-apiserver start` and restart `service etcd restart` (not required)

# 6. Security

## 6.1 k8s Security Primitives

* Disable password authentication, allow private keys only.
* Authentication options to kube-apiserver
  * Files - Username, passwords
  * Files - Usernames, tokens
  * Certs
  * External authentication - LDAP, AD
  * Service accounts

* Authorization - What permissions they have?
  * RBAC/ABAC

etc (skip notes)

## 6.2 Authentication

* Users are managed by kube-apiserver
* Authentication via
  * Static pw file
  * Static token file
  * Certs
  * ID services eg. Kerberos

### 6.2.1 Static password and token file - Basic

#### 6.2.1.1  Static password

user-details.csv (note group column may be absent)

```csv
password123,user1,u0001,group1
password321,user2,u0002,group2
```

kube-apiserver.service

```text
ExecStart=/usr/local/bin/kube-apiserver
  --basic-auth-file=user-details.csv
```

Restart kube-apiserver to take effect

If kubeadm was used to set up cluster, modify pod definition file for kube-apiserver (default: /etc/kubernetes/manifests/kube-apiserver.yaml) to add

```yaml
spec:
  containers:
  - command:
  - kube-apiserver
  - --basic-auth-file=user-details.csv
```

and kubeadm will automatically recreate kube-apiserver pod

* Check you can authenticate with username, pw with `curl -v -k https://master-node:6443/api/v1/pods -u "user:password"`

#### 6.2.1.2 Static token file

user-token-details.csv

```csv
<token1>,user1,u0001,group1
<token2>,user2,u0002,group2
```

Similar to password file, used with `--token-auth-file=user-token-details.csv` instead. curl command now is `curl -v -k https://master-node:6443/api/v1/pods --header"Authorization: Bearer <token>`

Notes:

1. Generally not recommended to use static password/token files because creds in plaintext
2. Deprecated since 1.19  

## 6.3 TLS in k8s

* Certificates or public keys come in *.crt, *.pem, doesn't have word "key" in them
  * server.crt
  * server.pem
  * client.crt
  * client.pem
* Private keys have the word "key" in them
  * server.key
  * server-key.pem
  * client.key
  * client-key.pem

* Server certs for servers. These servers have public certs and private keys associated
  * kube-apiserver
    * Clients (each have their own key pairs): kubectl, kube-scheduler, kube-controller-manager, kube-proxy
  * etcd server
  * kubelet server


### 6.3.1



## 6.4 Certificates API

* CA server is just a pair of key files, typically the master node.
* Users create CertificateSigningRequest object
  * Admins can review requests and approve them

### Steps

1. User first creates key jane.key and sends it to admin

   ```bash
   openssl genrsa -out jane.key 2048
   ```

2. Admin takes the key and generates a CSR file

   ```bash
   openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr 
   ```

3. Then uses the CSR file to generate CSR resource

   ```yaml
   apiVersion: certificates.k8s.io/v1beta1
   kind: CertificateSigningRequest
   metadata:
     name: jane
   spec:
     groups:
     - system:authenticated
     usages:
     - digital signature
     - key encipherment
     - server auth
     request:
       {cat jane.csr | base64}
   ```

4. When generated view all CSRs with `kubectl get csr` and approve with `kubectl certificate approve csr-name`

* You can view the certificate with `kubectl get csr csr-name -o yaml`
  * The certificate itself is base64 encoded
* kube-controller-manager contains the paths of cluster signing cert file and signing key

## 6.5 KubeConfig

* File holds config that `kubectl` will need as arguments

* Default stored in **$HOME/.kube/config**

* File consists of three sections: Clusters, Users, Contexts (eg. $user@$cluster)

  * Contexts link the users to cluster
  * Use `--current-context` with kubectl to see current context

* View the kubeconfig file with `kubectl config view`, add `--kubeconfig=/config/file` for some other config file, or to switch context

* Switch context with `kubectl config use-context produser@production`

* Under **cluster** can specify CA

  ```yaml
  clusters:
  - name: production
    cluster:
      certificate-authority: /etc/kubernetes/pki/ca.crt
      # or use base64 -w0 ca.crt and paste below
      certificate-authority-data: <base64 data>    
  ```

## 6.6. API Groups

* Use curl to enumerate with

  * Check k8s version with `curl https://kube-master:6443/version`
  * Check pods with `curl https://kube-master:6443/api/v1/pods`
  * The APIs here are /version and /api

* /core and /\<named> groups

* Will get error if you list API groups with `curl https://localhost:6443 -k`, need to specify these arguments for authentication

  ```text
  --key admin.key
  --cert admin.crt
  --cacert ca.crt
  ```

* Alternatively start `kubectl proxy` by default on TCP 8001, this will automatically specify the certs for you so you can just `curl http://localhost:8001 -k`
  * Not the same as kube proxy

## 6.7 Authorization

* Allow only certain k8s users to perform authorized tasks like enumerating resources but not creating/destroying them

* Types:

  1. Node authorization
  2. ABAC - Attribute-based
  3. RBAC - Role-based
  4. Webhook - 3rd party authorization

* When multiple authorization nodes specified, it is handled in order of specification

  ```text
  # kube-apiserver definition
  --authorization-mode=Node,RBAC,Webhook
  ```

  1. Node handles only node requests, denies everything else
  2. RBAC performs checks and grants users permissions, will not go on to Webhook

* Each time a request is denied, it gets passed to next node for checks. If it grants, it exits and doesn't pass on.

## 6.8 Role-based Access Control (RBAC)

* Create roles with yaml. This is a role definition

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer
    namespace: blue
  rules:
  - apiGroups: [""] # "" indicates the core API group if blank
    resources: ["pods"]
    verbs: ["get", "list", "update", "delete", "create"]
  - apiGroups: [""] # Can specify multiple API groups
    resources: ["ConfigMap"]
    verbs: ["create"]
    resourceNames: ["blue","orange"] # Allows access only to blue, orange pods
  ```

* To check which API groups each resource belongs to, do `kubectl api-resources -o wide | grep resource-type`. Verify with `kubectl explain resource-type` that the Version field has the same API group [[ref](https://stackoverflow.com/questions/57821065/which-api-group-in-k8s)]

  * Eg. for deployments we can see that the apiGroup is **apps** with the allowed verbs
  
  ```text
    root@controlplane:~# k api-resources -o wide | grep -i deployment
  deployments                       deploy       apps/v1                                true         Deployment                       [create delete deletecollection get list patch update watch]
  ```
  
* Bind a user to the role with a rolebinding definition. Here **dev-user** is bound to **developer** role

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: devuser-developer-binding
  subjects:
  - kind: User
    name: dev-user # "name" is case sensitive
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: developer
    apiGroup: rbac.authorization.k8s.io
  ```

* List roles with `kubectl get roles`

* List rolebindings with `kubectl get rolebindings`

* List details about resources and permissions for each resource with `kubectl describe role role-name`, same describe for rolebindings

* Check for permissions for current user (returns yes/no)

  * `kubectl auth can-i create deployments`
  * `kubectl auth can-i delete nodes`

* Specify `--as username` to view permissions as that user in specific namespace

  * `kubectl auth can-i create deployments --as dev-user --namespace ns-1`

* Note: Both the role and rolebindings must specify the same namespace

## 6.9 Cluster Roles

* Some resources like nodes can't be grouped by namespace (ie. grouped under a namespace) because they are cluster wide. 

* Resources are either namespace-wide or cluster-wide (eg. namespace)

* View resources which are namespace wide `kubectl api-resources --namespaced=true` or False

* ClusterRole and ClusterRoleBinding are like Roles and RoleBindings, except cluster-wide.

* ClusterRole example:

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: cluster-administrator
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["nodes"]
    verbs: ["get", "list", "delete", "create"]
  ```

* ClusterRoleBinding binds the user to the role

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: cluster-admin-role-binding
  subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: cluster-administrator
    apiGroup: rbac.authorization.k8s.io
  ```

* Although pods are namespace-wide, you can use a ClusterRole to allow the user access to all pods across the cluster.

* To check which request verbs are available for a resource, say persistentvolumes do `kubectl api-resources -o wide | grep persistentvolumes`

  * [Documentation](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#review-your-request-attributes) (not too helpful)

## 6.10 Service Account

* Different from user accounts which are used by humans

* Used by applications to authorise access
  * Prometheus using service account to monitor cluster
  * Jenkins/Gitlab using service accounts to deploy applications

* Create service account with `kubectl create serviceaccount dashboard-sa`
  * Token is automatically created (view with `describe`) as secret
  
* View service account with `kubectl get serviceaccount`

* Now can use RESTful API to view `curl https://kube-master:6443/api -k --header "Authorization: Bearer <base64-secret>`

* If the app which uses the secret is itself a pod on the cluster, it is mounted as a volume mount in **/var/run/secrets/kubernetes.io/serviceaccount**. View with `describe pod`, this one shows default token which is automatically mounted if nothing is specified.

  ```yaml
  Mounts:
    /var/run/secrets/kubernetes.io/serviceaccount from default-token-j4hkv (ro)
  ```

* Can view it with `kubectl exec -it pod-name cat /var/run/secrets/kubernetes.io/serviceaccount`

* To specify service account in pod definition (delete and re-create pod). 

  ```yaml
  # Pod definition file
  spec:
    serviceAccountName: dashboard-sa
    automountServiceAccountToken: false # Specify this to avoid adopting service account token
  ```

* Usage: Configure a deployment or pod to use a serviceaccount, which in turn has a secret

## 6.11 Image Security

* To use an image from private repository first login then do a run

  * `docker login private-registry.io`
  * `docker run private-registry.io/apps/internal-app`

* Create a secret to store the creds

  ```text
  kubectl create secret docker-registry regcred \
  --docker-server=private-registr \
  --docker-username=user
  --docker-password=password
  --docker-email=user@mail.com
  ```

* Specify in pod definition with

  ```yaml
  spec:
    containers:
    - name: nginx
      image: private-registry.io/apps/internal-app
    imagePullSecrets:
    - name: regcred
  ```

## 6.12 Security Contexts

* Like in Docker, we can specify run-as for containers and Linux capabilities 

  ```yaml
  # Pod definition
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities: 
          add: ["MAC_ADMIN"]
  ```

* Supported only at the container, not pod level

* Check what user is running in the container with `kubectl exec -it podname -- <cmd>`

## 6.13 Network Policies

* Similar to NACLs, except default is allow all implicit

  * Seen in lab

* Created as k8s resource. This one applies 3306 ingress for pod api-pod.

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
   name: db-policy
  spec:
    podSelector:
      matchLabels:
        role: db # Selects DB pod via label to apply the policy to
    policyTypes:
    - Ingress
    ingress:
    - from:
      - ipBlock: # Works as OR with ns and pod selector if the "-" specified
          cidr: 172.17.0.0/16 # Can also specify /32 for one IP
          except:
          - 172.17.1.0/24
      - namespaceSelector:
          matchLabels:
            name: prod # Select other namespaces to apply to
      - podSelector: # If omitted, assumes that all pods in NS will be allowed
          matchLabels:
            role: api-pod # Selects the pod to allow traffic FROM
      ports:
      - protocol: TCP
        port: 3306
  ```

* Note that policies are stateful (replies to ingress automatically allowed)

# 7. Storage

## 7.1 Volumes

* Can be mounted on pods [[ref](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)]

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: test-pd
  spec:
    containers:
    - image: k8s.gcr.io/test-webserver
      name: test-container
      volumeMounts:
      - mountPath: /test-pd # Where the mount is found inside the pod
        name: test-volume
    volumes:
    - name: test-volume # This must match with name in volumeMounts
      hostPath:
        path: /data # directory location on host
        type: Directory # this field is optional
  ```

## 7.2 Persistent Volumes

* Cluster-wide storage volumes, users can select storage from pool using PVC. 

* `kubectl get persistentvolume`

* Definition file:

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-vol1
  spec:
    accessModes: [ "ReadWriteOnce" ]
    capacity:
     storage: 1Gi
    hostPath:
     path: /tmp/data
  ```


## 7.3 Persistent Volume Claims

* Every PVC is bound to a single PV.

* One-to-one PVC to PV. No other claims can use the same volume

* If no volumes, PVC remains pending until PV available

* Note: Access Modes on PVC, PV **must** match.

* Can use labels and selectors to bind to preferred volume if multiple available

* PVC definition file

  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: myclaim
  spec:
    accessModes: 
      - ReadWriteOnce
    selector:
      matchLabels:
        release: "stable"
    resources:
     requests:
       storage: 1Gi
  ```
  
* Query with `kubectl get persistentvolumeclaim`

* Once PVC deleted, PV by default is **Retain** or not available for reuse until manually reclaimed.

  * **Delete** setting will delete the PV once PVC is deleted; frees up storage.
  * **Recycle** will delete all data stored and made available to other PVCs

* Pod usage. Once created, a PVC can be specified in the pod definition

  ```yaml
  kind: Pod
  metadata:
    name: mypod
  spec:
    containers:
      - name: myfrontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
          name: mypd
    volumes:
      - name: mypd
        persistentVolumeClaim:
          claimName: myclaim
  ```

## 7.4 Storage Class

* Static provisioning is when you need to create the storage on the cloud provider, then create the PV then finally the PVC and then include the PVC in the pod definition

* Dynamic provisioning automatically creates storage on the cloud provider so you only need to create PVC to reference SC directly.

  * PV is automatically created when storage class is created

* For Google cloud, SC definition is

  ```yaml
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
     name: google-storage
  provisioner: kubernetes.io/gce-pd
  ```

* If the storage provider supports it can specify different types of storage via `type`. Check [this](https://kubernetes.io/docs/concepts/storage/storage-classes/)

  * AWS: io1, gp2, st1 etc
  * GCE: pd-standard, pd-ssd

# 8. Networking

Lectures explain Linux namespaces and how you can use it manually to create isolated regions within a host, and how to connect them together. How docker connects the containers using bridge network and forwards the port to the host using iptables is also explained.

## 8.1 Linux namespaces

* Create new network NS `ip netns add red`

* List NS `ip netns`

* View interfaces `ip link`

* Exec command in NS `ip netns exec red <cmd>`

* To connect two NS together

  1. Create the virtual cable  with interfaces (**veth-red** and **veth-blue**) on both ends `ip link add veth-red type veth peer name veth-blue`

  2. Attach each interface to NS

     ```bash
     ip link set veth-red netns red
     ip link set veth-blue netns blue
     ```

  3. Give each interface an IP address (here both are in the same subnet)

     ```bash
     ip netns exec red ip addr add 192.168.15.1/24 dev veth-red
     ip netns exec blue addr add 192.168.15.2/24 dev veth-blue
     ```

  4. Bring up the NS interfaces

     ```bash
     ip netns exec red ip link set veth-red up
     ip netns exec blue ip link set veth-blue up
     ```

  5. Now you can ping blue veth from red and vice-versa

     ```bash
     ip netns exec red ping 192.168.15.2
     ```

* Create a virtual switch in Linux 

  1. `ip link add v-net-0 type bridge` of name **v-net-0**

  2. Check interface of v-switch with `ip link`

  3. Bring interface up with `ip link set dev v-net-0 up`

  4. Create the virtual cables, each end for the NS the other for bridge

     ```bash
     ip link add veth-red type veth peer name veth-red-br
     ip link add veth-blue type veth peer name veth-blue-br
     ```

  5. Set the interfaces to the NS and bridge

     ```bash
     # For red
     ip link set veth-red netns red
     ip link set veth-red-br master v-net-0
     # For blue
     ip link set veth-blue netns blue
     ip link set veth-blue-br master v-net-0
     ```

  6. Assign the virtual interfaces with IP address

     ```bash
     ip netns exec red ip addr add 192.168.15.1/24 dev veth-red
     ip netns exec blue ip addr add 192.168.15.2/24 dev veth-blue
     ```

  7. Bring up the interfaces

     ```bash
     ip netns exec red link set veth-red up
     ip netns exec blue link set veth-blue up
     ```

  8. Assign an IP to the bridge for connectivity with the host

     ```bash
     ip addr add 192.168.15.5/24 dev v-net-0
     ```

  9. Now you can ping either red or blue veth `ping 192.168.15.1`

* The NS still can't reach any IPs other than the host. Check each NS routing table with `ip netns exec blue route`

* To allow connectivity from inside NS to an external network **192.168.1.0/24**

  1. Add a route to 192.168.1.0/24

     ```bash
     ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
     # Verify with
     ip netns exec blue route
     ```

  2. Enable NAT on the host so external networks can reply to traffic originating from NS (for virtual network 192.168.15.0/24)

     ```bash
     iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
     ```

  3. Add a default route to the NS route table

     ```bash
     ip netns exec blue ip route add default via 192.168.15.5
     ```

  4. Now you can ping Google `ip netns exec blue ping 8.8.8.8`

* To port forward any traffic hitting port 80 on host to NS blue configure this on iptables

  ```bash
  iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
  ```

  Verify with `iptables -nvL -t nat`

## 8.2 CNI in k8s

* Check the running kubelet pod or service, look for these arguments then
  * **--network-plugin**
  * **--cni-bin-dir** (default **/opt/cni/bin**)
  * **--cni-conf-dir** (default **/etc/cni/net.d**)
* Default CNI path is **/opt/cni/bin**
* Deploy weave to k8s cluster `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`
  * Specify `&env.<options>=setting` to apply if required. [[Ref](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-changing-configuration-options)]
  * Only need to apply the CNI config to master node.
* Check that the weave pods have been deployed `kubectl get pods -n kube-system`

* Pod IP allocation range can be found by checking logs or config for weave pod or other network plugin

## 8.3 IP address management weave

To be filled up

## 8.4 Service networking

* Pods are hosted on nodes, while services are hosted across cluster-wide ie. ClusterIP
* When service is created, it is assigned IP from a pre-defined range.
* **kube-proxy** on each node creates forwarding rule that forwards traffic meant for the service IP to the pod IP via
  * userspace
  * iptables
  * ipvs
* Specify the service cluster IP range with `kube-apiserver --service-cluster-ip-range cidr-range` (default 10.0.0/24)
* Both the pod IP range and service cluster IP range should not overlap
* Check network forwarding rules created by kube-proxy with `iptables -L -t nat | grep local-cluster`
  * Or in the logs **/var/log/kube-proxy.log** (dir may vary)

* Tip: Can use `ipcalc` to check IP range of given CIDR

## 8.5 DNS in k8s

* DNS works on a cluster, not node level; IP addressing is handled by the network CNI plugin

* Created services automatically assigned a DNS name and reachable.

* In the same namespace, you can just specify the service name eg. web-service to reach it outside NS, if not append the namespace (app) to it eg. **web-service.app** to make domain.

  | Hostname    | Namespace | Type | Root          | IP address    |
  | ----------- | --------- | ---- | ------------- | ------------- |
  | web-service | apps      | svc  | cluster.local | 10.107.37.188 |
  | 10-244-2-5  | apps      | pod  | cluster.local | 10.244.2.5    |

  Service FQDN: web-service.apps.svc.cluster.local

  Pod FQDN: 10-244-2-5.apps.pod.cluster.local

* Can lookup FQDN and IP of service with `host service-name` (doesn't work for pods except with FQDN)

* DNS records for pods are disabled by default (enable below)

### 8.5.1 CoreDNS in k8s

* Deployed as pod in **kube-system** NS, via a RS in a deployment

* Config file: /etc/coredns/Corefile

  * **proxy** setting in Corefile specifies whether or not pods get DNS name
  * Corefile initalised as ConfigMap object; edit ConfigMap object to modify config

* CoreDNS available as service **kube-dns** 

* kubelet config file specifies DNS server IP and domain (**clusterDomain** and **clusterDNS** setting)

  * Also found in CoreDNS configmap

* Tip: Check the deployment for the DNS tool used to see whether the path containing the config file is mounted as a volume and how the config is passed in

  ```text
  root@controlplane:~# k -n kube-system describe deploy coredns
  ...
      Args:
        -conf
        /etc/coredns/Corefile
      Limits:
        memory:  170Mi
      Requests:
        cpu:        100m
        memory:     70Mi
      Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
      Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
      Environment:  <none>
      Mounts:
        /etc/coredns from config-volume (ro)
    Volumes:
     config-volume:
      Type:               ConfigMap (a volume populated by a ConfigMap)
      Name:               coredns
      Optional:           false
    Priority Class Name:  system-cluster-critical
  ...
  ```

## 8.6 Ingress

* Service NodePorts can only allocated high number ports >30k

* Alternative is to use LoadBalancer type instead of NodePort which gives you a DNS name + 80 but works for cloud providers only.

* Ingress works like a k8s load balancer, 

  * Provides DNS domain name (rules for each sub-domain)
  * SSL cert
  * Route to different pods via web paths (rules for different paths for each sub-domain)

* Two controllers: GCE and Nginx ingress maintained by k8s project

* Ingress definition file (nginx eg)

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: ingress-controller
  spec:
    replicas: 1
    selector:
      matchLabels:
        name: nginx-ingress
    template:
      metadata:
        labels:
          name: nginx-ingress
      spec:
        serviceAccountName: ingress-serviceaccount # Needed for ingress controller to monitor cluster
        containers:
          - name: nginx-ingress-controller
            image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
            args: # Just like regular nginx arguments
              - /nginx-ingress-controller
              - --configmap=$(POD_NAMESPACE)/nginx-configuration
            env: # Need to specify pod names and namespace in environmental variables (required by nginx pod)
              - name: POD_NAME
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              - name: POD_NAMESPACE
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.namespace
            ports: # Used by nginx controller
              - name: http
                containerPort: 80
              - name: https
                containerPort: 443
  ```

  Also specify a CM so you can edit config in future.

  ```yaml
  kind: ConfigMap
  apiVersion: v1
  metadata:
    name: nginx-configuration
  ```

* Specify a service to expose the ingress controller to the node

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: ingress
  spec:
    type: NodePort
    ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
    selector:
      name: nginx-ingress
  ```

* Ingress service needs a service account for permissions to monitor k8s cluster for ingress resources detecting changes

  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: nginx-ingress-serviceaccount
  ```

### 8.6.1 Ingress Resources

* Used to route users based on hostname or web paths

* This one routes to **wear-service**

  ```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: ingress-wear
  spec:
    backend:
      serviceName: wear-service
      servicePort: 80
  ```

* This has 1 rule (1 vhost) but two paths

  ```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: ingress-wear-watch
  spec:
    rules:
    - http:
        paths:
        - path: /wear
          backend:
            serviceName: wear-service
            servicePort: 80
        - path: /watch
          backend:
            serviceName: watch-service
            servicePort: 80
  ```

* This has two rules (sub-domains or vhosts) and 1 path each

  ```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: ingress-wear-watch
  spec:
    rules:
    - host: wear.my-online-store.com
      http:
        paths:
        - backend:
            serviceName: wear-service
            servicePort: 80
    - host: watch.my-online-store.com
      http:
        paths:
        - backend:
            serviceName: watch-service
            servicePort: 80 
  ```

* To get ingress resources `kubectl get ingress` or `kubectl describe ingress ingress-name`

* Create in imperative way `kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear\*=wear-service:80"`

* To direct to a service in another NS specify it in the ingress definition file

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
    name: ingress-wear-watch
    namespace: another-NS
  ```

  The NS are distinct so remove any directs to services in other NS.

# 9. Design and Install k8s cluster

* Use kubeadm for on-prem
* etcd can be put on another node for large clusters to enable HA.

## 9.1 Configure HA

* kubectl should map to kube-apiserver with load balancer to avoid duplication

* Scheduler and controller manager runs active/standby mode because they may duplicate resources if both act at same time

  * kube-controller-manager elected as active with

    ```text
    kube-controller-manager --leader-elect true
    						--leader-elect-lease-duration 15s # Non-leader holds lease for 15s
    						--leader-elect-renew-deadline 10s # Active renews lease every 10s
    						--leader-elect-retry-period 2s # Both processes try become leader every 2s
    ```

    First node to update the endpoint becomes leader. 

  * Similar process for scheduler

* etcd can be run outside of master node
  * kube-apiserver must know where etcd is

* HA will need a LB to talk to two master nodes, two or three etcd nodes.

### 9.1.1 Configure etcd for HA

* For multi-node etcd, one is leader by election RAFT

* When non-leader etcd node receives write request, it ensures copies of the write are distributed to other etcd instances

* To be HA, etcd must be present in at least 3 nodes so you can lose 1 and still maintain quorum

* etcd needs to know where its peers are

  ```text
  etcd \
  --initial-cluster peer-1=https://${PEER_1}:2380,peer-2=https://${PEER_2}:2380
  ```

* Prefer odd over even number of nodes because HA allows one to fail

# 10. Deployment with kubeadm

## 10.1 General steps

1. Install docker, kubadm, kubectl, kubelet on the nodes
   1. Be sure the versions don't conflict eg. kubelet cannot be higher than kube-apiserver
2. Initialise master node with `kubeadm init`
   1. Copy token for worker nodes to join or generate new one with `kubeadm token create`
3. Join worker nodes to cluster with command
   1. Nodes reflect as **NotReady** until CNI plugin is installed
4. Run the `sysctl` command on all nodes 
5. Apply the CNI plugin on the master node

## 10.2 Notes on setting up kubeadm

Install exact version of kubeadm with `apt install kubeadm=1.21.0-00`

Check with either `dpkg -l` or `apt list kubeadm`

# 11. Troubleshooting

## 11.1 Application failure

Troubleshooting service failure

1. Check whether you can access the web page with `curl`
2. Check that cluster endpoint of service is correct.
3. Describe service and the pod/deployment it maps and check if it matches
4. Check the pod to see if it restarted multiple times
   1. Check the logs as well including previous logs `--previous`

## 11.2 Control Plane failure

Steps

1. Check nodes are healthy
2. Check k8s pods on control plane are running or running as services
3. Check logs of control plane pods
   1. Use `journalctl -u kube-apiserver` to check logs if not a static pod

* If `kubectl` can't work, run `docker ps -a | grep Exited` to see which control plane container failed and check its logs

## 11.3 Worker Node failure

* Check nodes and describe to see why NotReady
* **Unknown** means the worker node stopped communicating with master
* SSH to node and check for CPU, disk space problems as indicated by `describe`
* Check kubelet service logs on worker nodes 
* Check certificates are issued by correct CA
  * Check the dir it's currently configured in

* Note: systemd service files are typically in 

  ```
  /usr/lib/systemd/system/: units provided by installed packages
  /etc/systemd/system/: units installed by the system administrator
  ```

  Or just check service for path

* [Troubleshooting clusters doc](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

* Display info on running endpoints for master node `kubectl cluster-info`

## 11.4 Network troubleshooting

1. Check whether network plugin is installed if pods can't start
   1. kubectl apply CNI plugin
2. kube-proxy troubleshooting
   1. Check logs
   2. Check config.conf mounted on configmap is correct
   3. Check kube-proxy is running in container

# 12. kubectl JSONPath

* Append `-o jsonp` to kubectl queries to return info in JSON

* Append `-o=jsonpath=` encapsulate in **'{}'**

* Get image name used by pods in cluster

  ```text
  kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
  ```

* Add `{"\n"}` for newline and `{"\t"}`

* Pipe to jq and less with colours

  ```text
  k get nodes -o=jsonpath='{.items[*].status}' | jq -C . | less -R
  ```

* Sort by propert

  ```text
  kubectl get nodes --sort-by=.metadata.name
  ```

  
