# RBAC (Role-Based Access Control)

## Overview

Role-Based Access Control (RBAC) is Kubernetes' authorization mechanism used to control which identities can perform specific actions on Kubernetes resources.

In this project, RBAC is implemented using:

- ServiceAccounts
- Roles
- RoleBindings
- ClusterRoles
- ClusterRoleBindings

The objective is to follow the Principle of Least Privilege by granting only the permissions required for each user or automation component.

---

## Service Accounts

The following ServiceAccounts have been created:

| ServiceAccount | Namespace | Purpose |
|---------------|-----------|---------|
| developer-sa | dev | Simulates a developer |
| qa-sa | qa | Simulates a QA engineer |
| devops-sa | prod | Deployment automation |
| jenkins-sa | jenkins | Jenkins CI/CD |
| argocd-sa | argocd | Argo CD GitOps |
| monitoring-sa | monitoring | Monitoring components |

> **Note:** From Kubernetes v1.24 onward, ServiceAccount tokens are no longer automatically stored as Secrets. Tokens are generated dynamically using the TokenRequest API.


## Developer Role

The `developer-role` grants permissions required by application developers working in the `dev` namespace.

### Allowed Operations

- Create, update, patch, and delete Pods
- Manage Deployments
- Manage Services
- Manage ConfigMaps
- View Pod logs
- Create and manage Jobs and CronJobs
- Read PersistentVolumeClaims
- View ReplicaSets

### Restricted Operations

- Access Kubernetes Secrets
- Modify Nodes
- Manage Namespaces
- Modify RBAC resources
- Execute commands inside running containers (`kubectl exec`)

### RoleBinding

The `developer-role` is assigned to the `developer-sa` ServiceAccount using the `developer-rolebinding`. This follows the Principle of Least Privilege by granting only the permissions required for development activities.


## QA Role

The `qa-role` provides read-only access to application resources in the `qa` namespace, with the ability to restart Deployments.

### Allowed Operations

- View Pods, Services, ConfigMaps, Deployments, ReplicaSets, Jobs, and CronJobs
- View Pod logs
- View PersistentVolumeClaims
- Restart Deployments using `kubectl rollout restart`

### Restricted Operations

- Create, update, or delete workloads
- Access Secrets
- Modify RBAC resources
- Manage cluster-scoped resources

### Design Rationale

QA engineers validate application behavior and troubleshoot issues without having permissions to change application definitions or delete resources. The role grants only the permissions required for testing activities.


## DevOps ClusterRole

The `devops-role` is implemented as a ClusterRole to provide a reusable set of permissions across multiple namespaces.

### Why ClusterRole?

Instead of creating identical Roles in `dev`, `qa`, and `prod`, a single ClusterRole is reused through namespace-specific RoleBindings. This reduces duplication and simplifies maintenance.

### Allowed Operations

- Manage Pods, Deployments, StatefulSets, DaemonSets, Services, ConfigMaps, Secrets, PVCs, Jobs, CronJobs, Ingresses, and NetworkPolicies.
- View Pod logs.
- Execute commands inside Pods (`pods/exec`).

### Restricted Operations

- Manage Nodes.
- Modify ClusterRoles or RoleBindings.
- Create or delete Namespaces.
- Manage CustomResourceDefinitions (CRDs).

This design provides operational flexibility while limiting access to cluster-wide infrastructure resources.
