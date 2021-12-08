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
