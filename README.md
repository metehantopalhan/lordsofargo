# Lords of Argo - Kubernetes Infrastructure as Code

This repository contains Kubernetes infrastructure configurations managed through ArgoCD, implementing a comprehensive monitoring, logging, and management stack.

## Components

### Core Infrastructure
- **NGINX Ingress Controller**: Manages external access to services
- **Cert-Manager**: Handles SSL/TLS certificate management
- **Rancher**: Kubernetes management platform

### Logging Stack (ELK)
- **Elasticsearch**:
  - Master Nodes: Manage cluster state and indices
  - Data Nodes: Store and process data
  - Coordinator Nodes: Handle client requests and search operations
- **Kibana**: Visualization and management interface for Elasticsearch
- **Fluent Bit**: Log collection and forwarding

### Monitoring Stack
- **Prometheus**: Metrics collection and storage
- **Grafana**: Metrics visualization and dashboarding
  - Pre-configured with Kubernetes cluster and Node Exporter dashboards

## Architecture

### Elasticsearch Cluster
- 3 Master nodes (dedicated control plane)
- 3 Data nodes with 100GB storage each
- 3 Coordinator nodes for load balancing

### Ingress Configuration
- NGINX Ingress Controller with LoadBalancer service
- Configured endpoints:
  - Kibana: kibana.example.com
  - Grafana: grafana.9.163.80.51.sslip.io
  - Prometheus: prometheus.9.163.80.51.sslip.io
  - Alertmanager: alertmanager.9.163.80.51.sslip.io
  - Rancher: rancher.9.163.80.51.sslip.io

## Prerequisites
- Kubernetes cluster
- ArgoCD installed and configured
- Helm 3.x
- kubectl configured with cluster access

## Deployment

All applications are deployed and managed through ArgoCD. Each component is defined in its own directory with corresponding Kubernetes manifests:

```
├── elk-coordinator/     # Elasticsearch coordinator nodes
├── elk-data/           # Elasticsearch data nodes
├── elkmaster/          # Elasticsearch master nodes
├── fluentbit/          # Log collection
├── grafana/            # Metrics visualization
├── kibana/             # Elasticsearch UI
├── nginx-ingress/      # Ingress controller
├── prometheus/         # Metrics collection
└── rancher/            # Cluster management
```

## Configuration

### Resource Allocation
- **Elasticsearch**:
  - Master: 500m-1000m CPU, 1Gi-2Gi Memory
  - Data: 1000m-2000m CPU, 2Gi-4Gi Memory
  - Coordinator: 500m-1000m CPU, 1Gi-2Gi Memory
- **Kibana**: 500m-1000m CPU, 1Gi-2Gi Memory
- **Grafana**: Persistent storage of 10Gi
- **Prometheus**: Server storage of 50Gi

### Security
- Default admin credentials for Grafana: admin/admin123
- Default admin credentials for Rancher: admin/admin123
- X-Pack security is disabled in Elasticsearch (not recommended for production)

## Maintenance

### Scaling
- Elasticsearch nodes can be scaled by modifying the `replicas` field in respective YAML files
- Coordinator nodes are set to scale horizontally for load balancing

### Persistence
- Data nodes: 100GB per node
- Master nodes: 10GB per node
- Grafana: 10GB
- Prometheus: 50GB

## Notes

- This setup uses soft anti-affinity for data and coordinator nodes, and hard anti-affinity for master nodes
- All components are configured with automated sync and self-healing through ArgoCD
- Namespace creation is automated for all components

## Production Considerations

1. Enable X-Pack security in Elasticsearch
2. Configure proper TLS certificates for all ingress endpoints
3. Review and adjust resource allocations based on actual usage
4. Implement proper backup strategies for persistent data
5. Configure alerting rules in Prometheus/Alertmanager

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a new Pull Request