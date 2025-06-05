# Victoria Metrics Custom Stack Helm Chart

This is a custom Helm chart based on the official [Victoria Metrics k8s stack](https://github.com/VictoriaMetrics/helm-charts/tree/master/charts/victoria-metrics-k8s-stack) with enhanced custom dashboard support.

## 🚀 Features

- **Deployment Mode Switching**: Easy switching between single-instance and cluster modes
- **Backup & Disaster Recovery**: Comprehensive backup solutions with multiple storage backends
- **Custom Dashboard Support**: Built-in support for custom VictoriaMetrics dashboards
- **Production Ready**: Resource limits, monitoring, and alerting pre-configured
- **Multi-Cloud Support**: Backup to S3, GCS, Azure, or local storage

## 📦 Deployment Modes

### Single Mode (Default)
Best for development, testing, and small-scale production environments:

```yaml
deploymentMode:
  mode: "single"  # Uses VMSingle
```

**Characteristics:**
- Single VictoriaMetrics instance
- Lower resource requirements
- Simpler management
- No high availability

### Cluster Mode
Best for high-availability production environments:

```yaml
deploymentMode:
  mode: "cluster"  # Uses VMCluster
```

**Characteristics:**
- Multiple VictoriaMetrics components (VMStorage, VMSelect, VMInsert)
- High availability and horizontal scaling
- Data replication
- Higher resource requirements

### Switching Between Modes

**⚠️ Important Migration Notes:**

1. **Single to Cluster Migration:**
   ```bash
   # 1. Backup current data
   helm upgrade victoria-metrics-stack ./victoria-metrics-custom-stack \
     --set backup.enabled=true \
     --set backup.strategy=vmbackup
   
   # 2. Wait for backup to complete, then switch mode
   helm upgrade victoria-metrics-stack ./victoria-metrics-custom-stack \
     --set deploymentMode.mode=cluster \
     --set backup.restore.enabled=true
   ```

2. **Cluster to Single Migration:**
   ```bash
   # Similar process - backup first, then restore to single mode
   helm upgrade victoria-metrics-stack ./victoria-metrics-custom-stack \
     --set deploymentMode.mode=single \
     --set backup.restore.enabled=true
   ```

## 🔄 Backup & Disaster Recovery

### Quick Start - S3 Backup

```yaml
backup:
  enabled: true
  strategy: "vmbackup"
  vmbackup:
    schedule: "0 2 * * *"  # Daily at 2 AM
    storage:
      type: "s3"
      s3:
        endpoint: "s3.amazonaws.com"
        bucket: "my-vm-backups"
        region: "us-east-1"
        existingSecret: "s3-credentials"
        accessKeyIdKey: "access-key-id"
        secretAccessKeyKey: "secret-access-key"
```

### Storage Backend Options

#### 1. Amazon S3
```yaml
backup:
  vmbackup:
    storage:
      type: "s3"
      s3:
        endpoint: "s3.amazonaws.com"
        bucket: "vm-backups"
        region: "us-east-1"
        # Option 1: Use existing secret (recommended)
        existingSecret: "s3-credentials"
        accessKeyIdKey: "access-key-id"
        secretAccessKeyKey: "secret-access-key"
        # Option 2: Direct credentials (not recommended for production)
        accessKeyId: "AKIAIOSFODNN7EXAMPLE"
        secretAccessKey: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

#### 2. Google Cloud Storage
```yaml
backup:
  vmbackup:
    storage:
      type: "gcs"
      gcs:
        bucket: "vm-backups"
        existingSecret: "gcs-credentials"
        credentialsKey: "credentials.json"
```

#### 3. Azure Blob Storage
```yaml
backup:
  vmbackup:
    storage:
      type: "azure"
      azure:
        storageAccount: "mystorageaccount"
        container: "vm-backups"
        existingSecret: "azure-credentials"
        storageAccountKey: "storage-account-key"
```

#### 4. Filesystem (Local/NFS)
```yaml
backup:
  vmbackup:
    storage:
      type: "filesystem"
      filesystem:
        mountPath: "/backup"
        persistentVolume:
          enabled: true
          size: "500Gi"
```

### Backup Retention Policies

```yaml
backup:
  vmbackup:
    retention:
      daily: "7d"    # Keep daily backups for 7 days
      weekly: "4w"   # Keep weekly backups for 4 weeks  
      monthly: "12m" # Keep monthly backups for 12 months
```

### Disaster Recovery

#### Restore from Backup
```yaml
backup:
  restore:
    enabled: true
    source:
      vmbackup:
        storage:
          type: "s3"
          s3:
            endpoint: "s3.amazonaws.com"
            bucket: "vm-backups"
            region: "us-east-1"
            backupPath: "backups/2024-01-01"  # Specific backup
        restoreDate: "2024-01-01T10:00:00Z"   # Optional: specific timestamp
```

#### Manual Restore Process
```bash
# 1. Scale down VictoriaMetrics
kubectl scale deployment vmsingle-victoria-metrics-stack --replicas=0

# 2. Run restore job
helm upgrade victoria-metrics-stack ./victoria-metrics-custom-stack \
  --set backup.restore.enabled=true \
  --set backup.restore.source.vmbackup.storage.s3.backupPath="backups/2024-01-01"

# 3. Scale back up
kubectl scale deployment vmsingle-victoria-metrics-stack --replicas=1
```

### Creating Backup Secrets

#### S3 Credentials
```bash
kubectl create secret generic s3-credentials \
  --from-literal=access-key-id="AKIAIOSFODNN7EXAMPLE" \
  --from-literal=secret-access-key="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  -n monitoring
```

#### GCS Credentials
```bash
kubectl create secret generic gcs-credentials \
  --from-file=credentials.json="path/to/service-account.json" \
  -n monitoring
```

#### Azure Credentials
```bash
kubectl create secret generic azure-credentials \
  --from-literal=storage-account-key="your-storage-account-key" \
  -n monitoring
```

## 🔧 Production Configuration Examples

### High Availability Setup
```yaml
deploymentMode:
  mode: "cluster"

vmcluster:
  spec:
    replicationFactor: 2
    vmstorage:
      replicaCount: 3
      resources:
        requests:
          cpu: 1000m
          memory: 4Gi
        limits:
          cpu: 2000m
          memory: 8Gi

backup:
  enabled: true
  vmbackup:
    schedule: "0 2 * * *"
    storage:
      type: "s3"
      s3:
        bucket: "production-vm-backups"
        existingSecret: "s3-backup-credentials"
```

### Development Setup
```yaml
deploymentMode:
  mode: "single"

vmsingle:
  spec:
    retentionPeriod: "7d"
    resources:
      requests:
        cpu: 200m
        memory: 512Mi

backup:
  enabled: true
  vmbackup:
    schedule: "0 3 * * 0"  # Weekly on Sunday
    storage:
      type: "filesystem"
      filesystem:
        mountPath: "/backup"
        persistentVolume:
          enabled: true
          size: "50Gi"
```

## Components Included

- **VictoriaMetrics Single** - Time series database
- **VMAgent** - Metrics collection agent
- **VMAlert** - Alerting rules engine
- **Alertmanager** - Alert handling
- **Grafana** - Visualization and dashboards
- **Victoria Metrics Operator** - Kubernetes operator for VM components
- **Node Exporter** - Node metrics collection
- **Kube State Metrics** - Kubernetes cluster metrics
- **Custom Dashboards** - Pre-built and custom dashboards

## Custom Dashboards

This chart includes:

### Standard Dashboards
- VictoriaMetrics Single Node
- VMAgent monitoring
- VMAlert monitoring 
- Node Exporter Full
- Kubernetes Views (Global, Nodes, Pods, Namespaces)
- Kubernetes System (API Server, CoreDNS, Kubelet)
- Grafana Overview
- Alertmanager Overview

### Custom Dashboards
- **VictoriaLogs Explorer** - Custom log exploration dashboard
- **VictoriaLogs Dashboard** - Enhanced logs visualization

## Installation

### Prerequisites
- Kubernetes cluster (>= 1.25.0)
- Helm 3.x
- Sufficient cluster resources

### Install the Chart

```bash
# Add Victoria Metrics helm repository
helm repo add vm https://victoriametrics.github.io/helm-charts/
helm repo update

# Install the chart
helm install victoria-metrics-stack ./victoria-metrics-custom-stack \
  --namespace monitoring \
  --create-namespace
```

### Upgrade

```bash
helm upgrade victoria-metrics-stack ./victoria-metrics-custom-stack \
  --namespace monitoring
```

### Uninstall

```bash
helm uninstall victoria-metrics-stack --namespace monitoring
```

## Configuration

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `vmsingle.enabled` | Enable VictoriaMetrics Single | `true` |
| `vmsingle.spec.retentionPeriod` | Data retention period | `"30d"` |
| `vmsingle.spec.storage.resources.requests.storage` | Storage size | `100Gi` |
| `grafana.enabled` | Enable Grafana | `true` |
| `grafana.persistence.size` | Grafana storage size | `10Gi` |
| `defaultDashboards.custom.enabled` | Enable custom dashboards | `true` |
| `vmagent.enabled` | Enable VMAgent | `true` |
| `vmalert.enabled` | Enable VMAlert | `true` |
| `alertmanager.enabled` | Enable Alertmanager | `true` |

### Custom Dashboard Configuration

To add your own dashboards:

1. Place JSON dashboard files in `files/dashboards/custom/`
2. Update `values.yaml` to include your dashboard:

```yaml
defaultDashboards:
  custom:
    enabled: true
    your-dashboard-name:
      enabled: true
      datasource: VictoriaMetrics
```

### Resource Requirements

**Minimum Resources:**
- CPU: 4 cores
- Memory: 8GB RAM
- Storage: 120GB

**Recommended Resources:**
- CPU: 8 cores
- Memory: 16GB RAM  
- Storage: 500GB+

## Accessing Services

### Grafana
- **Service**: `grafana`
- **Port**: `3000`
- **Default credentials**: `admin/prom-operator`

### VictoriaMetrics
- **Service**: `vmsingle-victoria-metrics-custom-stack-vm-stack`
- **Port**: `8429`
- **Endpoint**: `/api/v1/`

### VMAgent
- **Service**: `vmagent-victoria-metrics-custom-stack-vm-stack`
- **Port**: `8429`

### Alertmanager
- **Service**: `alertmanager-victoria-metrics-custom-stack-vm-stack`
- **Port**: `9093`

## Port Forwarding Examples

```bash
# Access Grafana
kubectl port-forward -n monitoring svc/grafana 3000:80

# Access VictoriaMetrics
kubectl port-forward -n monitoring svc/vmsingle-victoria-metrics-custom-stack-vm-stack 8429:8429

# Access Alertmanager
kubectl port-forward -n monitoring svc/alertmanager-victoria-metrics-custom-stack-vm-stack 9093:9093
```

## Troubleshooting

### Common Issues

1. **Pods not starting**: Check resource availability and storage class
2. **Dashboards not loading**: Verify custom dashboard ConfigMaps are created
3. **Metrics not appearing**: Check VMAgent scrape configurations
4. **Storage issues**: Ensure persistent volumes are available

### Helpful Commands

```bash
# Check all pods status
kubectl get pods -n monitoring

# Check Victoria Metrics logs
kubectl logs -n monitoring deployment/vmsingle-victoria-metrics-custom-stack-vm-stack

# Check VMAgent logs
kubectl logs -n monitoring deployment/vmagent-victoria-metrics-custom-stack-vm-stack

# Check custom dashboard ConfigMap
kubectl get configmap -n monitoring victoria-metrics-custom-dashboards -o yaml
```

## Customization

### Adding New Dashboards

1. Create dashboard JSON files in `files/dashboards/custom/`
2. Update the ConfigMap template if needed
3. Upgrade the chart

### Modifying Default Configuration

Edit `values.yaml` and run:

```bash
helm upgrade victoria-metrics-stack ./victoria-metrics-custom-stack \
  --namespace monitoring \
  --values custom-values.yaml
```

## Support

This chart is based on the official Victoria Metrics helm charts. For:
- **Chart issues**: Check the official [Victoria Metrics helm charts repository](https://github.com/VictoriaMetrics/helm-charts)
- **Victoria Metrics documentation**: [docs.victoriametrics.com](https://docs.victoriametrics.com)
- **Custom modifications**: Review this chart's templates and values

## License

This chart follows the same Apache 2.0 license as the original Victoria Metrics helm charts. 