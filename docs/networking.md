## NGINX Ingress Controller

### Purpose
Provides a single HTTP/HTTPS entry point into the Kubernetes cluster.

### Design Decisions
- Installed using Helm.
- Dedicated `ingress-nginx` namespace.
- Scheduled on the platform node using nodeSelector and tolerations.
- NodePort Service for the local Kind environment.

### Future Enhancements
- TLS termination
- Prometheus metrics
- High availability
- External DNS integration
