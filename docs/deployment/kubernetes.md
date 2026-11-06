# Kubernetes Deployment

Deploy KubeSight to your Kubernetes cluster.

## Prerequisites

- Kubernetes 1.24+
- kubectl configured
- Cluster admin access (for RBAC setup)

## Quick Deploy

```bash
# Apply manifests
kubectl apply -f k8s/

# Check status
kubectl get pods -n kubesight

# Port forward
kubectl port-forward -n kubesight svc/kubesight-frontend 3000:80
```

## Manifests

### Namespace

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kubesight
```

### RBAC

KubeSight needs cluster-wide read access:

```yaml
# k8s/rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kubesight
  namespace: kubesight
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kubesight-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "nodes", "namespaces", "events", "configmaps"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "statefulsets", "daemonsets", "replicasets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods/log", "pods/exec"]
    verbs: ["get", "create"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubesight-reader
subjects:
  - kind: ServiceAccount
    name: kubesight
    namespace: kubesight
roleRef:
  kind: ClusterRole
  name: kubesight-reader
  apiGroup: rbac.authorization.k8s.io
```

### Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubesight-backend
  namespace: kubesight
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubesight-backend
  template:
    metadata:
      labels:
        app: kubesight-backend
    spec:
      serviceAccountName: kubesight
      containers:
        - name: backend
          image: kubesight-backend:latest
          ports:
            - containerPort: 8000
          env:
            - name: K8S_IN_CLUSTER
              value: "true"
            - name: GEMINI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: kubesight-secrets
                  key: gemini-api-key
```

### Service

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: kubesight-frontend
  namespace: kubesight
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: kubesight-frontend
```

### Ingress

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubesight
  namespace: kubesight
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
spec:
  ingressClassName: nginx
  rules:
    - host: kubesight.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubesight-frontend
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: kubesight-backend
                port:
                  number: 8000
```

## Secrets

Create secrets for sensitive data:

```bash
kubectl create secret generic kubesight-secrets \
  --from-literal=gemini-api-key=YOUR_API_KEY \
  -n kubesight
```

## Verification

```bash
# Check pods
kubectl get pods -n kubesight

# Check logs
kubectl logs -f deployment/kubesight-backend -n kubesight

# Test connectivity
kubectl port-forward svc/kubesight-frontend 3000:80 -n kubesight
```

## Updating

```bash
# Update images
kubectl set image deployment/kubesight-backend \
  backend=kubesight-backend:v1.4.0 -n kubesight

# Or apply updated manifests
kubectl apply -f k8s/
```
