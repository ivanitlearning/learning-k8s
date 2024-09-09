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