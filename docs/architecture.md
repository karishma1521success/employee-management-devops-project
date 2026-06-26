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
