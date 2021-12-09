# Notes for CKAD

These are notes taken specifically for CKAD which are not included in the [CKA notes](./CKA.md).

# 1. Observability

## 1.1 Readiness Probes

Run `kubctl describe pod` and check **Conditions**. These are all either True or False

* `PodScheduled`: the Pod has been scheduled to a node.
* `ContainersReady`: all containers in the Pod are ready.
* `Initialized`: all [init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) have completed successfully.
* `Ready`: the Pod is able to serve requests and should be added to the load balancing pools of all matching Services.

k8s service assumes pods in Ready state can start receiving traffic, which may not be the case hence the need for Readiness Probes

Types of probes

- httpGet
- tcpSocket
- exec command

Specify `initialDelaySeconds` to wait before checking probes

Generally readiness probes are meant to test if a pod is ready to start receiving traffic overall. Failed health checks result in pods removed from service endpoints.

## 1.2 Liveness Probes

Meant to check if pod's internal app is ready. Failed health checks subject the pod to its restart policy.

# 2. Pod Design

## 2.1 Jobs

Unlike pods, containers jobs are run until completion, not indefinitely. Tasks include batch file, sending email etc.

* By default k8s restarts pods that are completed unless RestartPolicy is set to Never.

Job definition

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3 # Job will repeat until it gets 3 successful jobs
  parallelism: 3 # Create all the jobs concurrently instead of one at a time, will create further jobs one by one until termination
  template:
    spec:
      containers:
      - name: math-add
        image: ubuntu
        command: ["expr","3","+","2"]
      restartPolicy: Never
```

* Only Never or OnFailure allowed as RestartPolicy
* Specify **backoffLimit** number to specify number of retries before regarding job as failed. Default: 6

View jobs with `kubectl get jobs` and get output of job with `kubectl logs`

## 2.2 Cron Jobs

Run jobs regularly with CronJobs

Move everything under spec of Job resource to under **jobTemplate**. This job runs every minute

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3 # Job will repeat until it gets 3 successful jobs
      parallelism: 3 # Create all the jobs concurrently instead of one at a time, will create further jobs one by one until termination
      template:
        spec:
          containers:
          - name: math-add
            image: ubuntu
            command: ["expr","3","+","2"]
          restartPolicy: Never
```

Manually run a job from a cronjob with `kubectl create job --from=cronjob/<name of cronjob> <name of job>`

# 3. Services and Networking

Kodekloud note: Topics here not on exam

## 3.1 Stateful Sets

* Unlike deployments, we can configure master/slave configuration such that master pods come up first then slaves.
  * Master pod must be running state before the other pods deployed.
* Pods get names derived from STS name and in sequence eg. mysql-0.
  * First pod will always be named \<sts-name>-0
* Pods will be scaled in order; last will be removed first.

Definition

* Create as Deployment then rename Kind to StatefulSet.
* Provide a **serviceName** in the STS definition.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
...
```

## 3.2 Headless Services

* This gives a DNS name to pods in the format `podname.headless-svcname.namespace.svc.cluster.domain`
  * Eg. `mysql-0.mysql-h.default.svc.cluster.local`
* Headless service have `clusterIP: None` set.
* Pod definition must have **subdomain** set to name of headless service, and **hostname** field in pod definition.
* This example sets the subdomain and hostname of a pod (no STS here)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: mysql
spec:
  containers:
  - name: mysql
    image: mysql
  subdomain: mysql-h # This must match name of headless service
  hostname: mysql-pod # Required to create A record for pod.
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
```

However, note that as part of a Deployment this means every pod created will the exact same DNS name `mysql-pod.mysql-h.default.svc.cluster.local`

With a StatefulSet, we can omit the **subdomain** and **hostname**

* Note that serviceName must be specified to link back to headless service so it knows what subdomain to assign to pod.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  serviceName: mysql-h # This MUST match the headless service name
  replicas: 3
  matchLabels:
    app: mysql
  template:
    metadata:
      name: myapp-pod
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
```

STS pods have DNS names `mysql-0.mysql-h.default.svc.cluster.local`. Now all pods will have separate DNS created.

# 3. Sep 2021 Changes

Named in Kodekloud for lack of a better term

## 3.1 Define, build Docker images

* If an image layer fails when building, Docker uses a cached copy of the successful image to continue building
* Build docker image `docker build -f Dockerfile -t name:tag .`
* Run container from image with `docker run -d -p 8282:8080/tcp name:tag`
* 

## 3.2 Admission Controllers

* Used to set finer grain rules eg. no images from unauthorised registry, without tags etc.

* Check enabled admission controllers with `k -n kube-system exec kube-apiserver-master -- kube-apiserver -h | grep enable-admission-plugins` if kubeadm is used to set up cluster
  * Also can check `ps aux | grep kube-apiserver` to see plugins enabled/disabled.

* Enable admission controllers with `kube-apiserver --enable-admission-plugins=NodeRestriction,other-plugin-names`
  * Edit the kube-apiserver static pod definition file
* 

View list of [admission controllers here](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do)

## 3.3 Custom admission controllers

### 3.3.1 Mutating admission controllers

* These change or mutate objects as they are created to add attributes (eg. default storageclass where none specified)
* Generally invoked before validating admission controllers because mutating ac's may prevent validating ac's from being invoked.

### 3.3.2 Validation admission controllers

* Validates resource creation requests to see if requested resources exist and rejects if not eg. create resource in non-existent namespace

### 3.3.3 Admission webhooks

* When request to create resource is made, MutatingAdmissionWebhook sends AdmissionReview to Admission Webhook server containing object details.
* Reply is sent to mutate and/or validate the resource request.

* A ValidatingWebhookConfiguration resource is created to determine the rules under which the webhook is called.

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration # Can be MutatingWebhookConfiguration also
metadata:
  name: pod-policy.example.com
webhooks:
- name: pod-policy.example.com
  rules: # These determine when the admission webhook will be called ie. which resource creation trigger it
  - apiGroups:   [""] # This example is triggered when pods are created.
    apiVersions: ["v1"]
    operations:  ["CREATE"]
    resources:   ["pods"]
    scope:       Namespaced
  clientConfig:
    service: # This assumes the webhook service is in this NS with this name
      namespace: webhook-namespace
      name: webhook-service
    caBundle: "Ci0tLS0tQk...<`caBundle` is a PEM encoded CA bundle which will be used to validate the webhook's server certificate.>...tLS0K"
```

## 3.4 API Versions

* API groups like /apps/v1 are stable/GA versions
* Alternatives are /v1alpha1, /v1beta1
* Order of versions

|                     | Alpha                     | Beta                    | GA (Stable) |
| ------------------- | ------------------------- | ----------------------- | ----------- |
| Version name        | vX**alpha**Y eg. v1alpha1 | vX**beta**Y eg. v1beta1 | vX eg. v1   |
| Enabled by default? | No                        | Yes                     | Yes         |

* GA/stable versions can be Preferred or Storage. These can differ
  * Preferred: Version used when retrieving information through API with kube commands
  * Storage: Version stored in etcd regardless of what is specified in YAML definition

* Check preferredVersion with curl to `master:8001/apis/batch` for example
* Check storage version with command `etcd get "/registry/deployments/default/blue" --print-value-only`
* Enable API versions not enabled by default by adding argument `kube-apiserver --runtime-config=batch/v2alpha1`

## 3.5 API Deprecations

* Convert the apiVersion with `kubectl convert -f deploy.yaml --output-version apps/v1` with [link here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin)
* kubectl convert is a plugin to be installed separately.

* Ref [here](https://kubernetes.io/docs/reference/using-api/deprecation-policy/)

* To check which API version is enabled, do `kubectl proxy -h` to see command to proxy all the k8s API then `curl localhost:8001/apis/api-group` to check which is enabled
