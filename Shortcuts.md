# Command shortcuts

| Short form     | Long form                 |
| -------------- | ------------------------- |
| -n             | --namespace               |
| -A             | --all-namespaces          |
| kubectl get cm | kubectl get configmap     |
| csr            | certificatesigningrequest |

Can view the short forms with `kubectl api-resoures --sort-by=name`

# Links to k8s docs

[Persistent Volume and PVC with Pod](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

[Apply weave CNI plugin](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node)

[Network plugins type](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

[Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)

[Configure pod for CM](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data)

[Using secrets in Pod](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets)

Services:

1. [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)

Setting up k8s cluster:

1. [Installing kubeadm and pre-reqs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
2. 

Troubleshooting:

1. [Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

2. [Applications](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)

Scheduling:

* [Manually schedule on specific node](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename)

RBAC:

* [Creating roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
