# Cluster Governance

## Overview

Cluster Governance ensures that Kubernetes resources are used securely, fairly, and predictably across multiple teams sharing the same cluster.

In this project, Cluster Governance is implemented using:

- ResourceQuota
- LimitRange (Upcoming)
- RBAC (Upcoming)
- Pod Security Admission (Upcoming)

The goal is to simulate how a production Kubernetes platform is managed in an organization.

---

# Why Cluster Governance?

Imagine three teams sharing the same Kubernetes cluster.

- Development Team
- QA Team
- Platform Team

Without governance:

- One team can consume all cluster resources.
- Developers can accidentally deploy hundreds of Pods.
- Critical production workloads may fail to schedule.
- Infrastructure services like Jenkins or Prometheus may become unavailable.

Cluster Governance prevents these issues.

---

# Current Cluster Architecture

```

                           Kubernetes Cluster

                     +---------------------------+
                     |     Control Plane         |
                     +---------------------------+

        +----------------+----------------+----------------+
        |                |                |                |
        ▼                ▼                ▼

+----------------+ +----------------+ +----------------+
| Worker Node 1  | | Worker Node 2  | | Worker Node 3  |
| Platform       | | Production     | | Non-Production |
+----------------+ +----------------+ +----------------+

Jenkins           Employee App       Employee App (Dev)
Argo CD           (Production)       Employee App (QA)
Prometheus
Grafana
Ingress Controller

Node Allocation
Node	Purpose	Labels	Taints
Control Plane	Kubernetes Control Plane	node-role.kubernetes.io/control-plane	NoSchedule
Worker 1	Platform Services	role=platform	role=platform:NoSchedule
Worker 2	Production Applications	role=production	role=production:NoSchedule
Worker 3	Development & QA	role=non-production	None
Namespace Design
Namespace	Purpose
dev	Development workloads
qa	QA workloads
prod	Production workloads
jenkins	CI Pipeline
argocd	GitOps
monitoring	Prometheus & Grafana
ResourceQuota
What is ResourceQuota?

A ResourceQuota limits the total amount of resources that a namespace can consume.

It prevents one namespace from exhausting cluster resources and affecting other workloads.

ResourceQuota is enforced by the Kubernetes API Server during the Admission Control phase.

Why ResourceQuota?

Without ResourceQuota:

Developer accidentally deploys:

replicas: 100

Kubernetes attempts to create all Pods until cluster resources are exhausted.

This can impact:

Production applications
Jenkins
Monitoring
Other development teams

With ResourceQuota:

The request is rejected before new Pods are admitted if the namespace exceeds its configured limits.

ResourceQuota Configuration
Namespace	Pods	CPU Requests	Memory Requests	CPU Limits	Memory Limits
dev	10	2 CPU	4 Gi	4 CPU	8 Gi
qa	10	2 CPU	4 Gi	3 CPU	6 Gi
prod	20	2 CPU	4 Gi	4 CPU	8 Gi
jenkins	10	2 CPU	4 Gi	4 CPU	8 Gi
argocd	10	2 CPU	4 Gi	4 CPU	8 Gi
monitoring	10	2 CPU	4 Gi	4 CPU	8 Gi

Note: These values are suitable for the current KIND cluster. They will be adjusted later based on the actual resource requirements of Jenkins, Argo CD, and the monitoring stack.

How ResourceQuota Works
Developer

    │

kubectl apply

    │

API Server

    │

Admission Controllers

    │
    ├── ResourceQuota
    ├── LimitRange
    ├── Pod Security

    │

Request Accepted

    │

etcd

    │

Scheduler

    │

Worker Node

ResourceQuota is evaluated before scheduling.

If the quota is exceeded, the Pod is rejected before the Scheduler attempts to place it on a node.

Verification

Verify ResourceQuota objects:

kubectl get resourcequota -A

Describe a specific ResourceQuota:

kubectl describe resourcequota dev-quota -n dev

Expected output:

Current Usage
Hard Limits
Resource Consumption





# LimitRange

## Overview

A **LimitRange** is a Kubernetes policy that defines default, minimum, and maximum CPU and memory values for containers, Pods, or PersistentVolumeClaims within a namespace.

In this project, LimitRange is used to:

- Automatically assign default CPU and memory requests/limits to containers.
- Enforce consistent resource allocation across namespaces.
- Work together with ResourceQuota to implement production-grade resource governance.

---

# Why LimitRange?

In a multi-team Kubernetes cluster, developers often forget to specify resource requests and limits.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev

spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

This Pod does not define:

- CPU Requests
- Memory Requests
- CPU Limits
- Memory Limits

Without these values:

- Kubernetes cannot accurately schedule workloads.
- ResourceQuota cannot properly account for resource usage.
- One application could consume excessive cluster resources.

LimitRange solves this problem by automatically assigning default values.

---

# How LimitRange Works

```
Developer

      │

kubectl apply

      │

API Server

      │

LimitRange Admission Controller

      │

Default Resources Added

      │

ResourceQuota Validation

      │

Scheduler

      │

Worker Node
```

LimitRange is evaluated **before** the Scheduler places the Pod on a node.

---

# Relationship Between LimitRange and ResourceQuota

ResourceQuota and LimitRange are complementary.

| ResourceQuota | LimitRange |
|---------------|------------|
| Limits total resources per namespace | Defines resources per container or Pod |
| Namespace-level policy | Container/Pod-level policy |
| Prevents one namespace from consuming all cluster resources | Ensures every container has appropriate resource values |
| Rejects workloads exceeding configured quotas | Automatically injects default resource values |

In production environments, these two resources are almost always used together.

---

# Namespace Configuration

## Development (dev)

| Setting | Value |
|----------|-------|
| Default Request CPU | 200m |
| Default Request Memory | 256Mi |
| Default Limit CPU | 500m |
| Default Limit Memory | 512Mi |

---

## QA (qa)

| Setting | Value |
|----------|-------|
| Default Request CPU | 200m |
| Default Request Memory | 256Mi |
| Default Limit CPU | 500m |
| Default Limit Memory | 512Mi |

---

## Production (prod)

| Setting | Value |
|----------|-------|
| Default Request CPU | 500m |
| Default Request Memory | 512Mi |
| Default Limit CPU | 800m |
| Default Limit Memory | 750Mi |

> These values are intentionally higher because production workloads require more predictable and stable resource allocation.

---

## Platform Namespaces

The following namespaces currently use the same defaults as the development environment:

- argocd
- jenkins
- monitoring

These values will be reviewed and adjusted after deploying Jenkins, Argo CD, Prometheus, and Grafana based on actual resource consumption.

---

# Why Different Values for Production?

This project follows a production-like cluster architecture.

```
Worker-1
Platform Services

• Jenkins
• Argo CD
• Prometheus
• Grafana
• Ingress Controller

----------------------------

Worker-2
Production Applications

• Employee Management Application

----------------------------

Worker-3
Development & QA

• Development
• QA
```

Production workloads are business-critical and therefore receive higher default resource allocations.

---

# Verification

List all LimitRanges.

```bash
kubectl get limitrange -A
```

or

```bash
kubectl get limits -A
```

Describe a LimitRange.

```bash
kubectl describe limitrange dev-limitrange -n dev
```

View the complete YAML.

```bash
kubectl get limitrange dev-limitrange -n dev -o yaml
```

---

# Validation Lab

Create a Pod without defining any resources.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-resource-test
  namespace: dev

spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

Apply the Pod.

```bash
kubectl apply -f default-resource-test.yaml
```

Inspect the Pod.

```bash
kubectl get pod default-resource-test \
-n dev \
-o yaml
```

Expected result:

Kubernetes automatically adds:

```yaml
resources:
  requests:
    cpu: 200m
    memory: 256Mi

  limits:
    cpu: 500m
    memory: 512Mi
```

This confirms that the LimitRange Admission Controller mutated the Pod before it was stored in etcd.

---

# Lessons Learned

- LimitRange is enforced during the Admission Control phase.
- LimitRange does not reserve CPU or memory.
- LimitRange automatically assigns default resource requests and limits.
- ResourceQuota depends on resource requests and limits to track namespace resource usage.
- Production clusters typically configure LimitRange for every application namespace.
- Developers do not need to define resources manually if appropriate defaults exist.

---

# Common Mistakes

### Assuming LimitRange reserves resources

Incorrect.

LimitRange only defines policies. Resources are reserved when Pods are scheduled.

---

### Using ResourceQuota without LimitRange

This often causes Pods without resource requests to be rejected because Kubernetes cannot calculate quota usage.

---

### Setting unrealistic default values

Assigning excessively high CPU or memory values can reduce cluster utilization and prevent Pods from scheduling.

---

### Assuming LimitRange changes existing Pods

LimitRange only affects newly created Pods.

Existing Pods are not modified.

---

# Interview Questions

## What is a LimitRange?

A LimitRange is a namespace-level policy that defines default, minimum, and maximum resource values for containers or Pods.

---

## Why is LimitRange used with ResourceQuota?

ResourceQuota limits the total resources a namespace can consume.

LimitRange ensures every Pod has resource requests and limits so ResourceQuota can properly enforce its policies.

---

## Does LimitRange reserve CPU or memory?

No.

It only defines default or allowed values.

Resources are allocated by the Scheduler when Pods are placed on worker nodes.

---

## When is LimitRange evaluated?

During the Admission Control phase by the Kubernetes API Server, before the Pod is stored in etcd and before scheduling occurs.

---

## How can you verify that LimitRange worked?

Create a Pod without resource requests or limits.

Then inspect it using:

```bash
kubectl get pod <pod-name> -o yaml
```

If the Pod contains automatically injected resource requests and limits, the LimitRange has been applied successfully.

---

# Production Best Practices

- Configure a LimitRange for every application namespace.
- Use LimitRange together with ResourceQuota.
- Define realistic default values based on workload requirements.
- Review and adjust resource defaults as applications evolve.
- Validate LimitRange behavior by inspecting the final Pod specification instead of relying only on successful Pod creation.

---

# Current Project Status

✅ Multi-node KIND Cluster

✅ Calico CNI

✅ Namespace Design

✅ Node Labels

✅ Node Taints

✅ Namespace Labels

✅ ResourceQuota

✅ LimitRange

⬜ RBAC

⬜ Pod Security Admission

⬜ Network Policies

⬜ Ingress Controller

⬜ Jenkins

⬜ Argo CD

⬜ Monitoring Stack

⬜ Employee Management Application

⬜ GitOps

⬜ CI/CD Pipeline
