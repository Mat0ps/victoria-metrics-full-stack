# VictoriaMetrics Full Stack

<div align="center">

![Monitoring](https://img.shields.io/badge/Monitoring-VictoriaMetrics-orange?style=for-the-badge&logo=prometheus&logoColor=white)
![Logs](https://img.shields.io/badge/Logs-VictoriaLogs-blue?style=for-the-badge&logo=elasticsearch&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-blue?style=for-the-badge&logo=kubernetes&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Visualization-orange?style=for-the-badge&logo=grafana&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-Package--Manager-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)

</div>

<div align="center">
  <img src="https://media.licdn.com/dms/image/v2/D4E3DAQGcuAkqsJDmyg/image-scale_191_1128/image-scale_191_1128/0/1721059570179/victoriametrics_cover?e=1750312800&v=beta&t=r14oA2NBDeWbB7Q5ceelZ960__SfJVC3RkNrrkKfBvM" alt="VictoriaMetrics" width="900"/>

</div>

---

## 🏗️ Architecture

This repository contains two main components that work together to provide complete observability:

### 📊 **VictoriaMetrics K8s Stack** (`victoria-metrics-k8s-stack/`)
Complete Kubernetes monitoring solution including:
- **VictoriaMetrics Operator**: Manages VictoriaMetrics components via Kubernetes CRDs
- **VMSingle**: High-performance metrics storage (TSDB)
- **VMAgent**: Metrics collection and scraping
- **VMAlert**: Alerting rules engine
- **Alertmanager**: Alert routing and notifications
- **Grafana**: Visualization dashboards with VictoriaLogs plugin
- **Node Exporter**: Host and hardware metrics
- **Kube State Metrics**: Kubernetes cluster state metrics

### 📝 **VictoriaLogs Cluster** (`victoria-logs-cluster/`)
Scalable log management solution including:
- **VLSelect**: Query processing component
- **VLInsert**: Log ingestion component  
- **VLStorage**: Log storage with configurable retention
- **Vector**: Log collection and forwarding agent
- **VMAuth**: Authentication and authorization proxy (optional)

### 🚀 **ArgoCD GitOps** (`argocd/`)
GitOps deployment configurations:
- **ApplicationSet**: Automated deployment of both metrics and logs stacks
- **Multi-source Applications**: Values from Git repository with Helm charts
- **Sync Policies**: Automated pruning and self-healing

## 🚀 Quick Start

### Prerequisites

**For Helm Deployment:**
- Kubernetes cluster (>=1.25.0)
- Helm 3.x
- `kubectl` configured to access your cluster

**For GitOps Deployment:**
- Kubernetes cluster (>=1.25.0)
- ArgoCD installed and configured
- Git repository access for configurations
- `kubectl` configured to access your cluster

### Option 1: Helm Installation

1. **Add VictoriaMetrics Helm repository:**
   ```bash
   helm repo add vm https://victoriametrics.github.io/helm-charts/
   helm repo update
   ```

2. **Install the Metrics Stack:**
   ```bash
   helm install victoria-metrics-stack ./victoria-metrics-k8s-stack \
     --create-namespace \
     --namespace victoria-metrics \
     --values ./victoria-metrics-k8s-stack/values.yaml
   ```

3. **Install the Logs Cluster:**
   ```bash
   helm install victoria-logs ./victoria-logs-cluster \
     --create-namespace \
     --namespace victoria-metrics \
     --values ./victoria-logs-cluster/values.yaml
   ```

### Option 2: GitOps with ArgoCD

1. **Apply the ApplicationSet:**
   ```bash
   kubectl apply -f argocd/appsets/victoria-metrics-k8s-stack-appset.yaml
   ```

2. **Verify Applications are Created:**
   ```bash
   kubectl get applications -n argocd -l app.kubernetes.io/part-of=victoria-metrics-stack
   ```

3. **Monitor Sync Status:**
   ```bash
   argocd app list | grep victoria-metrics
   # or
   kubectl get applications -n argocd -o wide
   ```

**GitOps Features:**
- ✅ **Automated Sync**: Self-healing and automatic pruning enabled
- ✅ **Multi-Source**: Combines Helm charts with Git-based configuration
- ✅ **Namespace Management**: Automatic namespace creation
- ✅ **Server-Side Apply**: Enhanced resource management

## 🌐 Access Services

After installation, access the web interfaces:

```bash
# Grafana (metrics visualization)
kubectl port-forward -n victoria-metrics svc/victoria-metrics-k8s-stack-grafana 3000:80

# VictoriaMetrics (metrics API)
kubectl port-forward -n victoria-metrics svc/vmsingle-victoria-metrics-k8s-stack 8429:8429

# VictoriaLogs (logs query API)
kubectl port-forward -n victoria-logs svc/vlselect-victoria-logs-cluster 9471:9471
```

**Service URLs:**
- **Grafana**: http://localhost:3000 (admin/admin)
- **VictoriaMetrics UI**: http://localhost:8429/vmui
- **VictoriaLogs UI**: http://localhost:9471/select/logsql/

## ⚙️ Configuration

### Metrics Stack Configuration

Key configuration options in `victoria-metrics-k8s-stack/values.yaml`:

```yaml
# Storage retention
vmsingle:
  spec:
    retentionPeriod: "1"  # months
    storage:
      resources:
        requests:
          storage: 20Gi

# Scraping settings
vmagent:
  spec:
    scrapeInterval: 20s
    selectAllByDefault: true

# Grafana settings
grafana:
  enabled: true
  sidecar:
    dashboards:
      enabled: true
```

### Logs Cluster Configuration

Key configuration options in `victoria-logs-cluster/values.yaml`:

```yaml
# Log retention
vlstorage:
  retentionPeriod: 7d
  persistentVolume:
    size: 10Gi

# Resource limits
vlselect:
  resources:
    requests:
      cpu: 100m
      memory: 200Mi

# Vector log collection
vector:
  enabled: true
```

## 📋 Features

### Metrics Monitoring
- ✅ **Comprehensive Metrics Collection**: Kubernetes, Node, and Application metrics
- ✅ **Built-in Dashboards**: Pre-configured Grafana dashboards for Kubernetes monitoring
- ✅ **Alerting Rules**: Default alerting rules for common issues
- ✅ **High Performance**: Optimized for large-scale environments
- ✅ **Low Resource Usage**: Efficient resource utilization
- ✅ **Long-term Storage**: Configurable retention periods

### Logs Management  
- ✅ **Scalable Architecture**: Distributed components for high throughput
- ✅ **LogsQL**: Powerful query language for log analysis
- ✅ **Vector Integration**: Flexible log collection and processing
- ✅ **Grafana Integration**: Unified observability interface
- ✅ **High Compression**: Efficient storage utilization

## 🔧 Customization

### Adding Custom Dashboards

Place Grafana dashboard JSON files in:
```
victoria-metrics-k8s-stack/files/dashboards/
```

### Custom Alerting Rules

Add VMRule manifests in:
```
victoria-metrics-k8s-stack/templates/
```

### Log Collection Configuration

Modify Vector configuration in:
```
victoria-logs-cluster/templates/vector-config.yaml
```

## 📊 Default Dashboards

The metrics stack includes dashboards for:
- **Kubernetes Cluster Overview**: Cluster-level metrics and health
- **Node Exporter**: Host system metrics  
- **Kubernetes Workloads**: Pod, Deployment, StatefulSet metrics
- **VictoriaMetrics**: Database performance metrics
- **VictoriaLogs**: Log ingestion and query performance

## 🔍 Monitoring Targets

### Automatic Discovery
- Kubernetes API Server
- Kubelet metrics
- cAdvisor metrics  
- CoreDNS
- Node Exporter
- Kube State Metrics

### Custom ServiceMonitors
Use `VMServiceScrape` CRDs to add custom monitoring targets:

```yaml
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMServiceScrape
metadata:
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-application
  endpoints:
  - port: metrics
```

## 🚨 Alerting

### Default Alert Rules
- Node down
- High CPU/Memory usage
- Disk space low
- Pod crash looping
- Kubernetes API server issues

### Custom Alerts
Create `VMRule` resources:

```yaml
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMRule
metadata:
  name: my-alerts
spec:
  groups:
  - name: my-app.rules
    rules:
    - alert: MyAppDown
      expr: up{job="my-app"} == 0
```

## 🔧 Troubleshooting

### Common Issues

#### Helm Deployment Issues
1. **Pods not starting**: Check resource limits and node capacity
2. **Metrics not appearing**: Verify ServiceScrape configurations
3. **High memory usage**: Adjust retention periods and storage limits
4. **Dashboard not loading**: Check Grafana datasource configuration

#### GitOps/ArgoCD Issues
1. **Application stuck in progressing**: Check ArgoCD server logs and sync policies
2. **Values not being applied**: Verify Git repository access and values file paths
3. **CRD conflicts**: Ensure VictoriaMetrics operator is deployed first
4. **Namespace issues**: Check if CreateNamespace=true is in sync options

### Useful Commands

#### General Kubernetes
```bash
# Check operator status
kubectl get vmoperator -n victoria-metrics

# View metrics targets
kubectl get vmservicescrape -n victoria-metrics

# Check log ingestion
kubectl logs -n victoria-logs -l app.kubernetes.io/name=vlinsert

# Monitor resource usage
kubectl top pods -n victoria-metrics
```

#### ArgoCD Specific
```bash
# Check application status
argocd app get victoria-metrics-k8s-stack

# Force sync application
argocd app sync victoria-metrics-k8s-stack

# View application logs
argocd app logs victoria-metrics-k8s-stack

# Check repository connection
argocd repo list
```

#### Debug Commands
```bash
# Check all VictoriaMetrics resources
kubectl get vm* -A

# Verify Helm releases
helm list -A | grep victoria

# Check persistent volumes
kubectl get pv,pvc -n victoria-metrics

# Network connectivity test
kubectl run test-pod --image=curlimages/curl --rm -it -- /bin/sh
```

## 📚 Documentation

### Official Documentation
- [VictoriaMetrics K8s Stack](https://docs.victoriametrics.com/helm/victoriametrics-k8s-stack/)
- [VictoriaLogs Cluster](https://docs.victoriametrics.com/helm/victorialogs-cluster/)
- [VictoriaMetrics Documentation](https://docs.victoriametrics.com/)
- [VictoriaLogs Documentation](https://docs.victoriametrics.com/victorialogs/)
- [LogsQL Query Language](https://docs.victoriametrics.com/victorialogs/logsql/)

### GitOps Resources
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [ApplicationSets Guide](https://argocd-applicationset.readthedocs.io/)
- [Helm Charts with ArgoCD](https://argo-cd.readthedocs.io/en/stable/user-guide/helm/)

### Local Configuration Files
- [Metrics Stack Values](./victoria-metrics-k8s-stack/values.yaml)
- [Logs Cluster Values](./victoria-logs-cluster/values.yaml)
- [ArgoCD ApplicationSet](./argocd/appsets/victoria-metrics-k8s-stack-appset.yaml)
