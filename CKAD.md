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

You can manually run a job from a cronjob with `kubectl create job --from=cronjob/<name of cronjob> <name of job>`

