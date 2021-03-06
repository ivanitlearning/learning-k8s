# Note *warning*

These contain spoilers from killer.sh. Don't read this unless you want to spoil yourself.

# CKA

## Q1

You have access to multiple clusters from your main terminal through `kubectl` contexts. Write all those context names into `/opt/course/1/contexts`.

Next write a command to display the current context into `/opt/course/1/context_default_kubectl.sh`, the command should use `kubectl`.

Finally write a second command doing the same thing into `/opt/course/1/context_default_no_kubectl.sh`, but without the use of `kubectl`.

__________

How to do it without kubectl? curl to what?

_____________

## Q12

Use *Namespace* `project-tiger` for the following. Create a *Deployment* named `deploy-important` with label `id=very-important` (the `Pods` should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image `nginx:1.17.6-alpine` and the second one named container2 with image `kubernetes/pause`.

There should be only ever **one** *Pod* of that *Deployment* running on **one** worker node. We have two worker nodes: cluster1-worker1 and cluster1-worker2. Because the *Deployment* has three replicas the result should be that on both nodes **one** *Pod* is running. The third *Pod* won't be scheduled, unless a new worker node will be added.

In a way we kind of simulate the behaviour of a *DaemonSet* here, but using a *Deployment* and a fixed number of replicas.

_____________

How to use pod anti-affnity to make sure just one pod runs on each node? I tried this and it didn't work

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    id: very-important
  name: deploy-important
  namespace: project-tiger
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important
  template:
    metadata:
      labels:
        id: very-important
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
               labelSelector:
                matchExpressions:
                - key: id
                  operator: In
                  values:
                  - very-important
               topologyKey: topology.kubernetes.io/hostname
      containers:
      - image: nginx:1.17.6-alpine
        name: container1
      - image: kubernetes/pause
        name: container2
```

_____________

## Q13

Create a *Pod* named `multi-container-playground` in *Namespace* `default` with three containers, named `c1`, `c2` and `c3`. There should be a volume attached to that *Pod* and mounted into every container, but the volume shouldn't be persisted or shared with other *Pods*.

Container `c1` should be of image `nginx:1.17.6-alpine` and have the name of the node where its *Pod* is running on value available as environment variable MY_NODE_NAME.

Container `c2` should be of image `busybox:1.31.1` and write the output of the `date` command every second in the shared volume into file `date.log`. You can use `while true; do date >> /your/vol/path/date.log; sleep 1; done` for this.

Container `c3` should be of image `busybox:1.31.1` and constantly write the content of file `date.log` from the shared volume to stdout. You can use `tail -f /your/vol/path/date.log` for this.

Check the logs of container `c3` to confirm correct setup.

_________________

Need to create pv, pvc? Created pv, pvc but can't mount on all 3 containers. Just c1 works.

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: /tmp/Q13
---
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
---
# multi.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi-container-playground
  name: multi-container-playground
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
  - image: nginx:1.17.6-alpine
    name: c1
    volumeMounts:
    - mountPath: /vol/log
      name: task-pv-storage
  - image: busybox:1.31.1
    name: c2
    volumeMounts:
    - mountPath: /vol/log
      name: task-pv-storage
    command:
    - "while true; do date >> /vol/log/date.log; sleep 1; done"
  - image: busybox:1.31.1
    name: c3
    volumeMounts:
    - mountPath: /vol/log
      name: task-pv-storage
    command:
    - "tail -f /vol/log/date.log"
```

Using describe

```text
  c2:  
    Container ID:  containerd://4ced9b4fdde96b343af07bdfa886dc916701ba8d8dbc0aa65bc161837df020ff        
    Image:         busybox:1.31.1
    Image ID:      docker.io/library/busybox@sha256:95cf004f559831017cdf4628aaf1bb30133677be8702a8c5f2994629f637a209
    Port:          <none>                                                       
    Host Port:     <none>
    Command:          
      while true; do date >> /vol/log/date.log; sleep 1; done
    State:          Waiting
      Reason:       CrashLoopBackOff                                                                                                                            
    Last State:     Terminated                                                  
      Reason:       StartError
      Message:      failed to create containerd task: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process cause
d: exec: "while true; do date >> /vol/log/date.log; sleep 1; done": stat while true; do date >> /vol/log/date.log; sleep 1; done: no such file or directory: unk
nown                      
      Exit Code:    128   
      Started:      Thu, 01 Jan 1970 00:00:00 +0000
      Finished:     Sun, 28 Nov 2021 11:14:59 +0000
    Ready:          False
    Restart Count:  42                                                                                                                                          
    Environment:    <none>   
    Mounts:          
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ldnzv (ro)
      /vol/log from task-pv-storage (rw)
  c3:                                      
    Container ID:  containerd://0c239843cf252eb20ad2ae1ac2d1724f78c2397eacab207c8ee8ec7a5939b25a             
    Image:         busybox:1.31.1                                                                           
    Image ID: docker.io/library/busybox@sha256:95cf004f559831017cdf4628aaf1bb30133677be8702a8c5f2994629f637a209           
    Port:          <none>                                                                                                                                 
    Host Port:     <none>                                                                                                                                       
    Command:                                                                                                                                                    
      tail -f /vol/log/date.log                                                                                                                                 
    State:          Waiting                                                                                                                                     
      Reason:       CrashLoopBackOff                                                                                                                            
    Last State:     Terminated
      Reason:       StartError
      Message:      failed to create containerd task: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process cause
d: exec: "tail -f /vol/log/date.log": stat tail -f /vol/log/date.log: no such file or directory: unknown
      Exit Code:    128
      Started:      Thu, 01 Jan 1970 00:00:00 +0000
      Finished:     Sun, 28 Nov 2021 11:15:00 +0000
    Ready:          False
    Restart Count:  42
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ldnzv (ro)
      /vol/log from task-pv-storage (rw)
```

Almost there. Just note that when you run commands you can do

```yaml
command:
  - ash
  - -c
  - whatever command and argument to run here.
```

Don't have to include each parameter in its own array if the entire thing is passed to the shell.

## EQ 1

Check all available *Pods* in the *Namespace* `project-c13` and find the names of those that would probably be terminated first if the *Nodes* run out of resources (cpu or memory) to schedule all *Pods*. Write the *Pod* names into `/opt/course/e1/pods-not-stable.txt`.

________

What is the priority of pod termination? I tried metrics API server to check but it wasn't installed.

```text
k8s@terminal:~/Q13$ k top pods
error: Metrics API not available
k8s@terminal:~/Q13$ k get nodes
NAME               STATUS   ROLES                  AGE   VERSION
cluster1-master1   Ready    control-plane,master   73d   v1.22.1
cluster1-worker1   Ready    <none>                 73d   v1.22.1
cluster1-worker2   Ready    <none>                 73d   v1.22.1
k8s@terminal:~/Q13$ ssh cluster1-master1
Last login: Sun Nov 28 10:53:53 2021 from 192.168.100.2
root@cluster1-master1:~# k top pods
error: Metrics API not available
```

## EQ 2

Exec into the *Pod* and use `curl` to access the Kubernetes Api of that cluster manually, listing all available secrets. You can ignore insecure https connection. Write the command(s) for this into file `/opt/course/e4/list-secrets.sh`.

___________

What's the curl command for this?

## PQ 2

You're asked to confirm that kube-proxy is running correctly on all nodes. For this perform the following in *Namespace* `project-hamster`:

Create a new *Pod* named `p2-pod` with two containers, one of image `nginx:1.21.3-alpine` and one of image `busybox:1.31`. Make sure the busybox container keeps running for some time.

Create a new *Service* named `p2-service` which exposes that *Pod* internally in the cluster on port 3000->80.

Find the kube-proxy container on all nodes `cluster1-master1`, `cluster1-worker1` and `cluster1-worker2` and make sure that it's using iptables. Use command `crictl` for this.

Write the iptables rules of all nodes belonging the created *Service* `p2-service` into file `/opt/course/p2/iptables.txt`.

Finally delete the *Service* and confirm that the iptables rules are gone from all nodes.

___________________

1. How to find the iptables rules belonging to the service?
2. How to expose port for multi-container pods?

_________

## Q7

The metrics-server hasn't been installed yet in the cluster, but it's something that should be done soon. Your college would already like to know the kubectl commands to:

1. show *node* resource usage
2. show *Pod* and their containers resource usage

Please write the commands into `/opt/course/7/node.sh` and `/opt/course/7/pod.sh`.

_____________

The ans for 2 isn't `kubectl top pods`

# Qns to review

1, 7, 9, 12, 13, 14

# 2nd run- through

Q3

```text
k8s@terminal:~/Q1$ k -n project-c13 scale sts/o3db --replicas=1 --record=true
Flag --record has been deprecated, --record will be removed in the future
statefulset.apps/o3db scaled
```

What's the non-deprecated ans for this?

Ans: There's no non-deprecated ans for this.

Qn 9
Check if the ans if to use nodename for pod.yaml instead of static pods

Ans: Yes it does.

Q11
Does the pod DS itself need the specific labels or just the pods?

Ans: Ans puts labels on the DS and pods.

Q14 
Is there a better way to get the path of the config file other than guessing cni dir under /etc?

Ans: No, the ans looks for the cni dir under /etc

Q16

Is this how to solve?

```text
k8s@terminal:~$ k -n project-c14 get roles --no-headers | wc -l
300
k8s@terminal:~$ echo "project-c14 300" > /opt/course/16/crowded-namespace.txt	
```

Ans: Yes

Q20

Why is /var/lib/kubelet non-existent even after installation?

```text
root@cluster3-worker2:~# ls -lah /var/lib/kubelet
ls: cannot access '/var/lib/kubelet': No such file or directory
```

Notes: C3 broken. Finish everything else first then restart session.

Q24

Does the egress policy need include two entries for db1 and db2 pods?

Ans: Yes, if not backend pod will be able to access db2 on port 1111 which is not allowed.

## Redo cluster3

Q19,20,21 - Done

# CKAD

## Q2

Your manager would like to run a command manually on occasion to output the status of that exact *Pod*. Please write a command that does this into `/opt/course/2/pod1-status-command.sh`. The command should use `kubectl`.

Can use `awk` here?

## Q4

4. There seems to be a broken release, stuck in `pending-upgrade` state. Find it and delete it

I don't see it?

```text
k8s@terminal:~/Q4$ helm -n mercury list -o yaml
- app_version: 2.4.49
  chart: apache-8.6.5
  name: internal-issue-report-apache
  namespace: mercury
  revision: "1"
  status: deployed
  updated: 2021-12-18 07:49:57.769708159 +0000 UTC
- app_version: 1.21.3
  chart: nginx-9.5.4
  name: internal-issue-report-apiv2
  namespace: mercury
  revision: "2"
  status: deployed
  updated: 2021-12-18 07:45:27.890130721 +0000 UTC
- app_version: 1.21.1
  chart: nginx-9.5.0
  name: internal-issue-report-app
  namespace: mercury
  revision: "1"
  status: deployed
  updated: 2021-09-16 12:11:41.417558951 +0000 UTC
k8s@terminal:~/Q4$ helm -n mercury list
NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
internal-issue-report-apache    mercury         1               2021-12-18 07:49:57.769708159 +0000 UTC deployed        apache-8.6.5    2.4.49     
internal-issue-report-apiv2     mercury         2               2021-12-18 07:45:27.890130721 +0000 UTC deployed        nginx-9.5.4     1.21.3     
internal-issue-report-app       mercury         1               2021-09-16 12:11:41.417558951 +0000 UTC deployed        nginx-9.5.0     1.21.1
```

## Q8 

There is an existing *Deployment* named `api-new-c32` in *Namespace* `neptune`. A developer did make an update to the *Deployment* but the updated version never came online. Check the *Deployment* history and find a revision that works, then rollback to it. Could you tell Team Neptune what the error was so it doesn't happen again?

I rolled it to the revision with the highest image version number. Is that correct?

## Q11

2. Build the image using Docker, named `registry.killer.sh:5000/sun-cipher`, tagged as `latest` and `v1-docker`, push these to the registry

Possible to have multiple tags?

Skipped 3,4,5, requiring use of Podman?

3. Build the image using Podman, named `registry.killer.sh:5000/sun-cipher`, tagged as `v1-podman`, push it to the registry
4. Run a container using Podman, which keeps running in the background, named `sun-cipher` using image `registry.killer.sh:5000/sun-cipher:v1-podman`. Run the container from `k8s@terminal` and not `root@terminal`
5. Write the logs your container `sun-cipher` produced into `/opt/course/11/logs`. Then write a list of all running Podman containers into `/opt/course/11/containers`

## Scoring

Initial: 99/112

After doing everything once, past 2 hrs: 102/112

