# Notes for CKAD

These are notes taken specifically for CKAD which are not included in the [CKA notes](./CKA.md).

# 1. Observability

## 1.1 Health Probes

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

* kubelet monitors whether containers are ready. Three types:
  * `livenessProbe`: Indicates whether the container is running. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its [restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). If a Container does not provide a liveness probe, the default state is `Success`.
  * `readinessProbe`: Indicates whether the container is ready to respond to requests. If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is `Failure`. If a Container does not provide a readiness probe, the default state is `Success`.
  * `startupProbe`: Indicates whether the application within the container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the container, and the container is subjected to its [restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). If a Container does not provide a startup probe, the default state is `Success`.

* Startup probes are used for containers that take a long time to start up, where using the liveness probe may indicate false negative results
* Liveness probes are used if a container needs to be restarted without its running processes getting killed. Configure a health check to a running endpoint within the container for example.
* Readiness probes can be used to stop traffic from being sent without restarting the container when it fails. Used for maintenance or checks on backend services used by the container, where the running process in the container is still functional.

## 1.2 Fields

Taken from docs

- `initialDelaySeconds`: Number of seconds after the container has started before liveness or readiness probes are initiated. Defaults to 0 seconds. Minimum value is 0.
- `periodSeconds`: How often (in seconds) to perform the probe. Default to 10 seconds. Minimum value is 1.
- `timeoutSeconds`: Number of seconds after which the probe times out. Defaults to 1 second. Minimum value is 1.
- `successThreshold`: Minimum consecutive successes for the probe to be considered successful after having failed. Defaults to 1. Must be 1 for liveness and startup Probes. Minimum value is 1.
- `failureThreshold`: When a probe fails, Kubernetes will try `failureThreshold` times before giving up. Giving up in case of liveness probe means restarting the container. In case of readiness probe the Pod will be marked Unready. Defaults to 3. Minimum value is 1.

## 1.3 Examples

Pod definition

```yaml
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

HTTP checks

```yaml
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

## 1.4 Liveness Probes

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

## 3.2 Admission Controllers

* Used to set finer grain rules eg. no images from unauthorised registry, without tags etc.

* Check enabled admission controllers with `k -n kube-system exec kube-apiserver-master -- kube-apiserver -h | grep enable-admission-plugins` if kubeadm is used to set up cluster
  * Also can check `ps aux | grep kube-apiserver` on master node to see plugins enabled/disabled.

* Enable admission controllers with `kube-apiserver --enable-admission-plugins=NodeRestriction,other-plugin-names`
  * Edit the kube-apiserver static pod definition file

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
  * **Preferred**: Version used when retrieving information through API with kube commands
  * **Storage**: Version stored in etcd regardless of what is specified in YAML definition

* Check preferredVersion with curl to `master:8001/apis/batch` for example, after `kubectl proxy`
* Check storage version with command `etcd get "/registry/deployments/default/blue" --print-value-only`
* Enable API versions not enabled by default by adding argument `kube-apiserver --runtime-config=batch/v2alpha1`

## 3.5 API Deprecations

* Convert the apiVersion with `kubectl convert -f deploy.yaml --output-version apps/v1` with [link here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin)
* kubectl convert is a plugin to be installed separately.

* Ref [here](https://kubernetes.io/docs/reference/using-api/deprecation-policy/)

* To check which API version is enabled, do `kubectl proxy -h` to see command to proxy all the k8s API then `curl localhost:8001/apis/api-group` to check which is enabled

## 3.6 Custom Resource Definitions

* Custom resources require custom controllers to monitor resource request creation so they can execute the necessary changes
* Need to define custom resource definition before kubectl can accept resource requests.

For example suppose we want a custom resource FlightTicket

```yaml
# flight-ticket.yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```

We first create a custom resource definition (CRD) with CRD definition file

```yaml
# flight-ticket-custom-definition.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced # Specifies if custom resource is namespaced or not
  group: flights.com # API group CR belongs to
  names:
    kind: FlightTicket # Name of CR
    singular: flightticket # Specify both singular and plural terms for custom resource
    plural: flighttickets # Seen in k api-resources output
    shortnames: # So you can run k get ft
    - ft 
  versions:
  - name: v1 # State current version of CR
    served: true
    storage: true # API group storage version
  schema:
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
            properties:
              from:
                type: string
              to:
                type: string
              number:
                type: integer
                minimum: 1
                maximum: 10
```

Then run `kubectl create -f flight-ticket-custom-definition.yaml`, now you can start creating the custom resource object

## 3.7 Custom Controllers

* Creating CRDs only allows CR resource objects to be created, but nothing happens once created.
* Start with this [sample custom controller repo](https://github.com/kubernetes/sample-controller)

## 3.8 Operator Framework

* CRDs and custom controllers are created separately, but they can actually be created and deployed together as an operator framework.

  Can create both with `kubectl create -f flight-operator.yaml`

* etcd operator is an example of an operator
* Operators are available at operatorhub.io, eg [etcd](https://operatorhub.io/operator/etcd)

## 3.9 Deployment Strategy

* **Blue-green**: Replace existing blue deployment by slowly rolling out green but no switch over. When done, switch over to green all at once.
  * Implementation: Create the green Deployment with a different label, then edit the service selector to point to it instead of blue.
* **Canary**: Route a small % of traffic to new deployment for testing, then redirect all traffic to new deployment
  * Implemention: Use two labels, one common to both deployments. Service should select just one label. It'll route just half the traffic to canary deployment. To decrease this, reduce canary replicas to minimal number of pods. Alternatively use Istio.

## 3.10 Helm

* Helm brings together the different resource objects of an application, simplifying deployment which native k8s doesn't do well.
* k8s doesn't know or relate the various Secrets, Deployments, PVs, PVCs together. These are different objects.
  * Helm package just needs a yaml file (**values.yaml**) where we can customise things like size of PV, username, passwords etc.
* Commands:
  * `helm install wordpress` or `helm uninstall wordpress`
  * `helm upgrade wordpress`
  * `helm rollback wordpress`

### 3.10.1 Helm commands

Search for charts on Artifact hub (artifacthub.io) with `helm search hub wordpress`

Add other charts repo with `helm repo add bitnami https://charts.bitnami.com/bitnami`

Search the added repo with `helm search repo wordpress`

List existing repos with `helm repo list`

Install helm charts with `helm install release-name chart-name` eg. `helm install release-2 bitnami/wordpress`

List installed packages `helm list`

Uninstall packages with `helm uninstall my-release`

Download but not install package with `helm pull --untar bitnami/wordpress` to working dir.

Install package with path to dir `helm install release-4 ./wordpress`





