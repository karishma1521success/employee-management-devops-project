# Project Diary

## Milestone 1 - Cluster Foundation

### Objective

Build a production-like Kubernetes cluster using KIND.

### Completed

- Created a multi-node KIND cluster
- Installed Calico CNI
- Created namespaces
- Labeled nodes
- Applied taints
- Labeled namespaces

### Verification

- All nodes are Ready
- Calico is healthy
- Namespaces created successfully
- Labels and taints verified

### Lessons Learned

- Labels provide identity and scheduling metadata.
- Taints protect dedicated nodes from unwanted workloads.
- Namespace labels can be used later by NetworkPolicies, RBAC, and automation.
