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
