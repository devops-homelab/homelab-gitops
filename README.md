
# Homelab GitOps Configuration

This repository contains GitOps configurations and Helm chart values for the MediTrack homelab environment. It supports both traditional Ingress and modern Gateway API implementations, showcasing advanced deployment patterns including blue-green deployments with preview environments.

## Repository Structure

```
charts/
├── apps/
│   └── meditrack/
│       ├── aggregator-service/
│       ├── appointment-scheduling-service/
│       ├── doctor-record-service/
│       │   ├── values-dev.yaml              # Traditional ingress implementation
│       │   └── values-dev-gateway.yaml      # Gateway API implementation
│       ├── notification-service/
│       └── patient-record-service/
├── infra/
│   └── bootstrap/
│       ├── actions-runner-controller/
│       ├── external-secrets/
│       ├── kyverno/
│       ├── reloader/
│       └── sealed-secrets/
└── local-use-only/
    ├── db-creds.yaml
    └── github_arc_pat.yaml
```

## Deployment Strategies

### Traditional Ingress Implementation
Most services use battle-tested Kubernetes Ingress with Kong:
```yaml
ingress:
  enabled: true
  ingressClassName: kong
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hostname: service.apps.meditrack-app.me
```

### Gateway API Implementation (Showcase)
The `doctor-record-service` demonstrates modern Gateway API patterns:

**Production & Preview Routing**:
- Production: `https://dev.doctor-records.apps.meditrack-app.me`
- Preview: `https://preview.dev.doctor-records.apps.meditrack-app.me`

**Advanced Features**:
- Blue-green deployments with Argo Rollouts
- Automatic TLS certificates for both domains
- Traffic splitting and preview environments
- Kong Gateway controller integration

## Service Architecture

### MediTrack Microservices

#### `aggregator-service/`
Data aggregation service with CronJob scheduling:
- **Pattern**: Batch processing with Kubernetes CronJobs
- **Scaling**: Resource-based autoscaling
- **Data Sources**: Multiple microservice APIs

#### `appointment-scheduling-service/`
Appointment management with traditional ingress:
- **Pattern**: REST API with PostgreSQL backend
- **Deployment**: Blue-green with Argo Rollouts
- **Features**: Real-time appointment booking

#### `doctor-record-service/`
Dual implementation showcase:
- **Traditional**: `values-dev.yaml` - Standard ingress implementation
- **Gateway API**: `values-dev-gateway.yaml` - Modern networking with preview
- **Database**: PostgreSQL with connection pooling
- **Security**: Environment-based secret injection

#### `notification-service/`
Event-driven notification system:
- **Pattern**: Pub/Sub with message queues
- **Channels**: Email, SMS, push notifications
- **Reliability**: Dead letter queues and retries

#### `patient-record-service/`
Patient data management with enhanced security:
- **Compliance**: HIPAA-compliant data handling
- **Encryption**: Data at rest and in transit
- **Audit**: Complete audit trail logging

### Infrastructure Components

#### `bootstrap/actions-runner-controller/`
Self-hosted GitHub Actions runners:
- **Scalability**: Automatic runner provisioning
- **Security**: Ephemeral runners for each job
- **Cost Optimization**: Scale-to-zero capabilities

#### `bootstrap/external-secrets/`
External secret management:
- **Providers**: Support for various secret stores
- **Sync**: Automatic secret synchronization
- **Security**: Least-privilege access patterns

#### `bootstrap/kyverno/`
Policy-as-code security enforcement:
- **Policies**: Security, compliance, and best practices
- **Validation**: Resource validation at admission
- **Mutation**: Automatic security improvements

## Configuration Management

### Environment-Specific Values

#### Development Environment
```yaml
# values-dev.yaml
environment: development
replicas: 1
resources:
  requests:
    cpu: 100m
    memory: 128Mi
autoscaling:
  enabled: false
```

#### Gateway API Configuration
```yaml
# values-dev-gateway.yaml
gateway:
  enabled: true
  gatewayClassName: kong
  listeners:
    - name: https
      port: 443
      protocol: HTTPS
      hostname: dev.doctor-records.apps.meditrack-app.me

httpRoute:
  preview:
    enabled: true
    hostnames:
      - preview.dev.doctor-records.apps.meditrack-app.me

strategy:
  type: blueGreen
  blueGreen:
    autoPromotionEnabled: false
```

### Secret Management

#### Database Credentials
```yaml
# Sealed secret for database access
envFromSecrets:
  DB_USER:
    secretName: db-credentials
    key: username
  DB_PASSWORD:
    secretName: db-credentials
    key: password
```

#### GitHub Integration
```yaml
# GitHub Actions runner authentication
github_arc_pat: <sealed-secret>
```

## GitOps Workflow

### Deployment Pipeline
1. **Code Change**: Developer pushes to application repository
2. **CI Pipeline**: GitHub Actions builds and pushes container image
3. **GitOps Update**: CI updates image tag in this repository
4. **ArgoCD Sync**: ArgoCD detects change and deploys to cluster
5. **Health Check**: Kubernetes validates deployment health
6. **Preview Available**: New version accessible via preview URL (Gateway API)
7. **Manual Promotion**: Operator promotes to production traffic

### Branch Strategy
- **`main`**: Production-ready configurations
- **Feature branches**: Testing new configurations
- **Pull requests**: Code review for GitOps changes

### Monitoring & Observability
- **ArgoCD Dashboard**: GitOps deployment status
- **Rollout Dashboard**: Blue-green deployment progress
- **Grafana**: Application and infrastructure metrics
- **Jaeger**: Distributed tracing across services

## Migration Examples

### From Traditional to Gateway API

1. **Current State** (Traditional Ingress):
```yaml
# values-dev.yaml
ingress:
  enabled: true
  hostname: dev.doctor-records.apps.meditrack-app.me
```

2. **Transition State** (Both Enabled):
```yaml
# values-dev-gateway.yaml
ingress:
  enabled: true  # Keep existing traffic
gateway:
  enabled: true  # Add Gateway API
httpRoute:
  enabled: true
```

3. **Future State** (Gateway API Only):
```yaml
# values-dev-gateway.yaml
ingress:
  enabled: false
gateway:
  enabled: true
httpRoute:
  preview:
    enabled: true
```

## Local Development

### Prerequisites
- Kubernetes cluster with Gateway API support
- ArgoCD for GitOps
- Kong ingress controller
- cert-manager for TLS automation

### Quick Start
```bash
# Apply local configurations
kubectl apply -f local-use-only/db-creds.yaml

# Deploy parent ArgoCD application
kubectl apply -f local-use-only/argocd-parent-app.yaml

# Watch deployments
kubectl get applications -n argocd
```

### Testing Gateway API Features
```bash
# Test production endpoint
curl https://dev.doctor-records.apps.meditrack-app.me/health

# Test preview endpoint
curl https://preview.dev.doctor-records.apps.meditrack-app.me/health

# Check rollout status
kubectl argo rollouts status doctor-record-service-gateway
```

## Contributing

### Adding New Services
1. Create service directory under `charts/apps/meditrack/`
2. Add both traditional and Gateway API value files
3. Configure appropriate secrets and environment variables
4. Test deployment in preview environment
5. Update this README with service documentation

### Best Practices
- Always include health checks and readiness probes
- Use sealed secrets for sensitive data
- Configure resource limits and requests
- Enable monitoring and observability
- Document service dependencies

## Security Considerations

- **Secret Management**: All secrets encrypted with sealed-secrets
- **Network Policies**: Micro-segmentation between services
- **TLS Everywhere**: Automatic HTTPS with Let's Encrypt
- **RBAC**: Least-privilege service accounts
- **Policy Enforcement**: Kyverno security policies

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

**Cloud Native • GitOps Driven • Production Ready**
