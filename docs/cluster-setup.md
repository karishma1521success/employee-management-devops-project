kPerfect. Let's build this like a real DevOps engineer would.

# Phase 1: Build the Foundation (Cluster Design)

Before deploying any application, let's create a production-like Kubernetes platform.

## Target Architecture

```text
Kind Cluster

├── Control Plane
│
├── Worker-1 (Platform Services)
│   ├── Jenkins
│   ├── ArgoCD
│   ├── Prometheus
│   └── Grafana
│
├── Worker-2 (Production)
│   └── Employee Management App (prod)
│
└── Worker-3 (Non-Prod)
    ├── Employee Management App (dev)
    └── Employee Management App (qa)
```

---

# Step 1: Recreate Kind Cluster

Create:

```yaml
# kind-config.yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

networking:
  disableDefaultCNI: true

nodes:
- role: control-plane

- role: worker

- role: worker

- role: worker
```

Create cluster:

```bash
kind create cluster \
--name employee-management \
--config kind-config.yaml
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
employee-management-control-plane

employee-management-worker

employee-management-worker2

employee-management-worker3
```

---

# Step 2: Install Calico

Install Calico and verify:

```bash
kubectl get pods -n kube-system
```

All nodes should become:

```text
Ready
```

---

# Step 3: Create Namespaces

Create:

```yaml
# kubernetes/namespaces/namespaces.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: qa
---
apiVersion: v1
kind: Namespace
metadata:
  name: prod
---
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins
---
apiVersion: v1
kind: Namespace
metadata:
  name: argocd
---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```

Apply:

```bash
kubectl apply -f kubernetes/namespaces/
```

Verify:

```bash
kubectl get ns
```

---

# Step 4: Label Nodes

Let's assign business meaning to worker nodes.

Check nodes:

```bash
kubectl get nodes
```

Example:

```text
employee-management-worker
employee-management-worker2
employee-management-worker3
```

Label them:

```bash
kubectl label node employee-management-worker role=platform

kubectl label node employee-management-worker2 role=production

kubectl label node employee-management-worker3 role=nonprod
```

Verify:

```bash
kubectl get nodes --show-labels
```

---

# Step 5: Reserve Worker-1 for Platform Services

Apply taint:

```bash
kubectl taint node employee-management-worker \
dedicated=platform:NoSchedule
```

Verify:

```bash
kubectl describe node employee-management-worker
```

Expected:

```text
Taints:
dedicated=platform:NoSchedule
```

---

# Why Are We Doing This?

In real companies:

| Node            | Purpose                     |
| --------------- | --------------------------- |
| Platform Node   | Jenkins, ArgoCD, Monitoring |
| Production Node | Business applications       |
| NonProd Node    | Dev, QA                     |

Reason:

* Jenkins builds should not compete with production workloads.
* Monitoring should stay available even during app spikes.
* Production gets predictable resources.

---

# Interview Question

**Q: Why use taints instead of only labels?**

Answer:

> Labels attract workloads. Taints repel workloads. By tainting the platform node, we ensure only pods with the correct toleration can run there, preventing accidental scheduling of application workloads.

---

# Step 6: Create Documentation

Create:

```text
docs/
├── architecture.md
├── cluster-setup.md
├── scheduling.md
├── interview-notes.md
```

---

# cluster-setup.md

```md
# Cluster Setup

Cluster Type: Kind

Control Plane Nodes: 1

Worker Nodes: 3

CNI: Calico

Namespaces:
- dev
- qa
- prod
- jenkins
- argocd
- monitoring

Node Allocation:

Worker-1:
Role: Platform
Taint: dedicated=platform:NoSchedule

Worker-2:
Role: Production

Worker-3:
Role: NonProd
```

---

# Git Commit

```bash
git add .
git commit -m "feat: create production-like cluster foundation"
```

---

# Milestone 1 Complete

Once you've recreated the cluster and verified:

* 1 control plane
* 3 workers
* Calico working
* Namespaces created
* Labels applied
* Platform node tainted

tell me **"Milestone 1 complete"** and we'll move to **Phase 2: Storage, Ingress, Resource Quotas, LimitRanges, and Production Namespace Design**, which is where the cluster starts looking much more like a real-world environment.

