---
trigger: always_on
description: Deployment and DevOps guidelines for Docker Compose and Kubernetes
---

# Deployment & DevOps Guidelines

HyperFix supports multiple deployment methods: Docker Compose for development/testing and Kubernetes for production. This document outlines deployment patterns and best practices.

## Docker Compose

### Configuration

The main Docker Compose configuration is in `compose.yml` at the project root.

### Service Structure

```yaml
services:
  postgres:
    image: postgres:16-alpine
    env_file:
      - .env
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hyperfix -d hyperfix"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    image: ghcr.io/samalehzen/hyper-api:latest
    ports:
      - "1337:1337"
    env_file:
      - .env
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped

  web:
    image: ghcr.io/samalehzen/hyper-web:latest
    ports:
      - "5173:5173"
    env_file:
      - .env
    depends_on:
      - api
    restart: unless-stopped
```

### Best Practices

1. **Health Checks**: Always include health checks for databases
2. **Dependencies**: Use `depends_on` with `condition: service_healthy` for databases
3. **Volumes**: Use named volumes for persistent data
4. **Restart Policy**: Use `unless-stopped` for production-like behavior
5. **Environment Variables**: Use `.env` file, never hardcode secrets

```yaml
# Good: Proper health check and dependency
services:
  api:
    depends_on:
      postgres:
        condition: service_healthy

  postgres:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hyperfix -d hyperfix"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## Kubernetes (Helm Charts)

### Chart Structure

Helm charts are located in `charts/hyperfix/`:

```
charts/hyperfix/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default values
├── README.md               # Chart documentation
└── templates/
    ├── deployment.yaml     # API and Web deployments
    ├── services.yaml       # Service definitions
    ├── ingress.yaml        # Ingress configuration
    ├── postgresql-deployment.yaml  # PostgreSQL deployment
    ├── pvc.yaml            # Persistent volume claims
    ├── hpa.yaml            # Horizontal Pod Autoscaler
    └── serviceaccount.yaml # Service accounts
```

### Deployment Patterns

#### API Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "hyperfix.fullname" . }}-api
spec:
  replicas: {{ .Values.api.replicaCount }}
  template:
    spec:
      containers:
      - name: api
        image: "{{ .Values.api.image.repository }}:{{ .Values.api.image.tag }}"
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: {{ include "hyperfix.fullname" . }}-secrets
              key: database-url
```

#### Web Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "hyperfix.fullname" . }}-web
spec:
  replicas: {{ .Values.web.replicaCount }}
  template:
    spec:
      containers:
      - name: web
        image: "{{ .Values.web.image.repository }}:{{ .Values.web.image.tag }}"
        env:
        - name: VITE_API_URL
          value: {{ .Values.web.apiUrl | quote }}
```

### Best Practices

1. **Secrets Management**: Use Kubernetes Secrets, never hardcode
2. **ConfigMaps**: Use ConfigMaps for non-sensitive configuration
3. **Resource Limits**: Always set resource requests and limits
4. **Health Checks**: Include liveness and readiness probes
5. **Replicas**: Configure appropriate replica counts for high availability

```yaml
# Good: Proper resource limits and health checks
containers:
- name: api
  resources:
    requests:
      memory: "256Mi"
      cpu: "100m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  livenessProbe:
    httpGet:
      path: /api/health
      port: 1337
    initialDelaySeconds: 30
    periodSeconds: 10
  readinessProbe:
    httpGet:
      path: /api/health
      port: 1337
    initialDelaySeconds: 5
    periodSeconds: 5
```

## Environment Variables

### Development

All environment variables in a single `.env` file at project root:

```bash
# Database
DATABASE_URL=postgresql://hyperfix_user:hyperfix_password@localhost:5432/hyperfix
POSTGRES_DB=hyperfix
POSTGRES_USER=hyperfix_user
POSTGRES_PASSWORD=hyperfix_password

# API
KANEO_API_URL=http://localhost:1337
AUTH_SECRET=your-secret-key

# Web
KANEO_CLIENT_URL=http://localhost:5173
VITE_API_URL=http://localhost:1337
```

### Production

Use Kubernetes Secrets and ConfigMaps:

```yaml
# Secret for sensitive data
apiVersion: v1
kind: Secret
metadata:
  name: hyperfix-secrets
type: Opaque
stringData:
  database-url: postgresql://user:pass@postgres:5432/hyperfix
  auth-secret: your-secret-key

# ConfigMap for non-sensitive data
apiVersion: v1
kind: ConfigMap
metadata:
  name: hyperfix-config
data:
  api-url: https://api.example.com
  client-url: https://app.example.com
```

## Database Migrations

### Automatic Migrations

Migrations run automatically on API startup:

```typescript
// apps/api/src/index.ts
await migrate(db, {
  migrationsFolder: `${process.cwd()}/drizzle`,
});
```

### Manual Migrations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SamalehZen/HyperFix-Kanban](https://github.com/SamalehZen/HyperFix-Kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
