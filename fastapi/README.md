# FastAPI Helm Chart

A comprehensive Helm chart for deploying FastAPI applications on Kubernetes with support for:
- Background task processing (Taskiq workers)
- Scheduled tasks (Taskiq scheduler)
- Database migrations
- Static file serving
- Horizontal Pod Autoscaling
- High availability

## Features

### Core Components

- **FastAPI Application**: Main web application deployment with configurable replicas and autoscaling
- **Taskiq Workers**: Background task processing with independent scaling
- **Taskiq Scheduler**: Cron-like scheduled task execution (single replica)
- **Database Migration**: Pre-install/upgrade job for database schema updates
- **Static File Serving**: Optional nginx sidecar for efficient static asset delivery

### Kubernetes Resources

- Deployments (API, Worker, Scheduler)
- Services and Ingress
- HorizontalPodAutoscalers (API and Worker)
- PodDisruptionBudget
- ServiceAccount
- Pre-install/upgrade migration Jobs

## Installation

```bash
helm install my-fastapi ./fastapi
```

With custom values:

```bash
helm install my-fastapi ./fastapi -f my-values.yaml
```

## Configuration

### Basic Application Configuration

```yaml
image:
  repository: gcr.io/my-project/my-app
  tag: "1.0.0"
  pullPolicy: IfNotPresent

replicaCount: 3

# Environment variables
config:
  extra:
    DATABASE_URL: "postgresql://user:pass@host/db"
    REDIS_URL: "redis://redis:6379/0"
  # Or use existing ConfigMap/Secret
  existingConfig: "my-configmap"
  existingSecret: "my-secret"
```

### Taskiq Worker Configuration

Enable background task processing:

```yaml
worker:
  enabled: true
  replicaCount: 2
  command: ["taskiq", "worker", "app.tasks:broker"]
  
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi
  
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 70
  
  terminationGracePeriodSeconds: 30
```

### Taskiq Scheduler Configuration

Enable scheduled tasks (always runs as single replica):

```yaml
scheduler:
  enabled: true
  command: ["taskiq", "scheduler", "app.tasks:scheduler"]
  
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi
```

### Database Migrations

Run migrations before deployment:

```yaml
migration:
  enabled: true
  command: ["alembic", "upgrade", "head"]
  resources:
    limits:
      memory: 256Mi
```

### Static File Serving

Serve static files via nginx sidecar:

```yaml
static:
  enabled: true
  source: "/app/static/"
  root: "/var/www/public/static/"
  prefix: "/static/"
  
  service:
    port: 8001
  
  resources:
    limits:
      cpu: 100m
      memory: 64Mi
```

### Ingress Configuration

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.example.com
```

### Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /.well-known/health
    port: http
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /.well-known/health
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3

startupProbe:
  httpGet:
    path: /.well-known/health
    port: http
  initialDelaySeconds: 0
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 30
```

### Autoscaling

API autoscaling:

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 20
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

### High Availability

```yaml
podDisruptionBudget:
  enabled: true
  minAvailable: 1
  # Or use maxUnavailable: 1
```

### Resource Limits

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

## Architecture

### Component Overview

```
┌─────────────────┐
│     Ingress     │
└────────┬────────┘
         │
    ┌────┴────┐
    │ Service │
    └────┬────┘
         │
    ┌────┴─────────────────┐
    │                      │
┌───┴────┐          ┌──────┴──────┐
│  API   │          │   Static    │
│  Pod   │          │  (nginx)    │
└───┬────┘          └─────────────┘
    │
    │ (shared config/secrets)
    │
    ├────────────────┬────────────────┐
    │                │                │
┌───┴────┐      ┌────┴─────┐    ┌────┴────────┐
│ Worker │      │ Worker   │    │  Scheduler  │
│  Pod   │      │   Pod    │    │     Pod     │
└────────┘      └──────────┘    └─────────────┘
```

### Labels and Selectors

All components use consistent labeling:

- API: `app.kubernetes.io/component: api`
- Worker: `app.kubernetes.io/component: worker`
- Scheduler: `app.kubernetes.io/component: scheduler`
- Migration: `app.kubernetes.io/component: migration`

## Examples

### Full Production Deployment

```yaml
image:
  repository: gcr.io/my-project/my-api
  tag: "v1.2.3"

replicaCount: 3

config:
  existingSecret: api-secrets

migration:
  enabled: true
  command: ["alembic", "upgrade", "head"]

worker:
  enabled: true
  replicaCount: 2
  command: ["taskiq", "worker", "app.tasks:broker"]
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 70

scheduler:
  enabled: true
  command: ["taskiq", "scheduler", "app.tasks:scheduler"]

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 80

podDisruptionBudget:
  enabled: true
  minAvailable: 2

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.example.com
```

### Development Deployment

```yaml
image:
  repository: my-api
  tag: latest
  pullPolicy: Always

replicaCount: 1

config:
  extra:
    DEBUG: "true"
    DATABASE_URL: "postgresql://postgres:postgres@postgres/dev"

ingress:
  enabled: true
  hosts:
    - host: api.local
      paths:
        - path: /
          pathType: Prefix
```

## Upgrading

```bash
helm upgrade my-fastapi ./fastapi -f my-values.yaml
```

## Uninstalling

```bash
helm uninstall my-fastapi
```

## Values Reference

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Container image repository | `gcr.io/my-project/my-image` |
| `image.tag` | Container image tag | `""` (uses appVersion) |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `replicaCount` | Number of API replicas | `1` |
| `worker.enabled` | Enable worker deployment | `false` |
| `worker.replicaCount` | Number of worker replicas | `1` |
| `worker.command` | Worker startup command | `["taskiq", "worker", "app.tasks:broker"]` |
| `scheduler.enabled` | Enable scheduler deployment | `false` |
| `scheduler.command` | Scheduler startup command | `["taskiq", "scheduler", "app.tasks:scheduler"]` |
| `migration.enabled` | Enable migration job | `false` |
| `migration.command` | Migration command | `["alembic", "upgrade", "head"]` |
| `static.enabled` | Enable static file serving | `false` |
| `autoscaling.enabled` | Enable HPA for API | `false` |
| `podDisruptionBudget.enabled` | Enable PDB | `false` |

See `values.yaml` for complete documentation of all parameters.

## License

See LICENSE file.
