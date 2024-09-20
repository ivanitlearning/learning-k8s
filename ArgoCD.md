# ArgoCD

## 1. Basics

GitOps approach to k8s.

### Terminology

Target state - Desired state of application in Git
Live state - Actual state of application
Sync status - Whether live matches target state
Sync operation status - Whether sync succeeded
Refresh - Shows diff between **HEAD** of Git and live state
Health - Health checks for k8s resources

### Features

- SSO integration
- Audit trail for application events and API calls
- Webhook integration with Git repo
- Rollback to any specific Git commit
- Provides Prometheus metrics
- Supports Kustomize, Helm, Jsonnet
- Presync, Sync, Postsync application, blue-green, canary rollouts
- Multi-tenancy and RBAC policies for authorization
- Deploy to multiple clusters

### Architecture

- ArgoCD is gRPC webserver with REST API

### Install and config

- Possible to use `argocd` binary to login to remote ArgoCD server

### ArgoCD application

- Connecting to a git repository adds a k8s secret to the argocd namespace where it's installed with metadata (link to repo) of the git repo to be deployed

CLI:

Deploy via CLI

```text
argocd app create helm-guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path helm-guestbook --dest-server https://kubernetes.default.svc --dest-namespace default
```

Sync with CLI: `argocd app sync solar-system-app-2`

### Reconciliation Loop

- Default timeout period is 3 min; ArgoCD will check git repo for changes this often (ie. polling frequency)
- Can also use git webhook to trigger ArgoCD checks
- Edit ConfigMap `argocd-cm` to update the time to check eg. `60s`
  ```text
  data:
    timeout.reconciliation: 60s
  ```

### Application Health

ArgoCD can check if various k8s resource types are healthy

- Service - Check that it has hostname or has status for ingress
- Ingress - Has hostname or IP
- Deployment, StatefulSet, DaemonSet - Replicas match desired
- PVC - Status.phase is Bound

Possible to write [custom health check in Lua](https://argo-cd.readthedocs.io/en/stable/operator-manual/health/#way-1-define-a-custom-health-check-in-argocd-cm-configmap), and defined in ConfigMap `argocd-cm`. It can be used to monitor configmaps for undesirable values.

### Sync strategies

- Manual or automatic sync applies changes due to changes in Git
- **Auto-prune** decides what happens when files are deleted from Git. Disabled means nothing is deleted.
- **Self-heal** restores k8s resources back to Git state when `kubectl` edits are made to cluster

### ArgoCD Applications

### Multiple clusters

- ArgoCD stores cluster info in k8s secrets

### ArgoCD RBAC

- RBAC policies in CM `argocd-rbac-cm`
- CLI commands to test rbac perms
  - `argocd account can-i create clusters '*'`
- ArgoCD supports local users via `argocd-cm`.