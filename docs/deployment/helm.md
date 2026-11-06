# Helm Chart Deployment

Deploy KubeSight using the official Helm chart.

## Installation

### From Local Chart

```bash
helm install kubesight ./charts/kubesight \
  -n kubesight --create-namespace
```

### With Custom Values

```bash
helm install kubesight ./charts/kubesight \
  -n kubesight --create-namespace \
  -f values-production.yaml
```

### From Repository (Coming Soon)

```bash
helm repo add kubesight https://gauravtayade11.github.io/kubesight/charts
helm install kubesight kubesight/kubesight -n kubesight --create-namespace
```

## Configuration

### values.yaml

```yaml
# Backend configuration
backend:
  replicaCount: 1
  image:
    repository: kubesight-backend
    tag: latest
    pullPolicy: IfNotPresent
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi
    requests:
      cpu: 200m
      memory: 256Mi
  extraEnv:
    - name: GEMINI_API_KEY
      valueFrom:
        secretKeyRef:
          name: kubesight-secrets
          key: gemini-api-key

# Frontend configuration
frontend:
  replicaCount: 1
  image:
    repository: kubesight-frontend
    tag: latest
    pullPolicy: IfNotPresent
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
    requests:
      cpu: 50m
      memory: 64Mi

# Ingress configuration
ingress:
  enabled: false
  className: nginx
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
  hosts:
    - host: kubesight.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

# Service account
serviceAccount:
  create: true
  name: kubesight

# RBAC
rbac:
  create: true
```

## Common Configurations

### Enable Ingress

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: kubesight.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: kubesight-tls
      hosts:
        - kubesight.example.com
```

### High Availability

```yaml
backend:
  replicaCount: 3
  resources:
    limits:
      cpu: 2000m
      memory: 2Gi

frontend:
  replicaCount: 2
```

### External Database (Future)

```yaml
database:
  enabled: false
  external:
    host: postgres.example.com
    port: 5432
    database: kubesight
```

## Operations

### Upgrade

```bash
helm upgrade kubesight ./charts/kubesight -n kubesight
```

### Rollback

```bash
helm rollback kubesight 1 -n kubesight
```

### Uninstall

```bash
helm uninstall kubesight -n kubesight
```

### View Values

```bash
helm get values kubesight -n kubesight
```

## Troubleshooting

### Check Release Status

```bash
helm status kubesight -n kubesight
```

### View Manifest

```bash
helm get manifest kubesight -n kubesight
```

### Debug Installation

```bash
helm install kubesight ./charts/kubesight \
  -n kubesight --create-namespace \
  --debug --dry-run
```
