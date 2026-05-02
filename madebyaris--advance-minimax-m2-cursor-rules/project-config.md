---
trigger: always_on
description: DevOps and infrastructure: Docker, Kubernetes, Terraform, CI/CD pipelines, and cloud deployment patterns
---


# DevOps & Infrastructure Patterns

Containerization, orchestration, infrastructure as code, and CI/CD best practices.

## DevOps Workflow

Before changing infrastructure or deployment code:

```text
1. Read the existing Docker, Kubernetes, Terraform, or CI files first
2. Understand the current state and provider/tool versions in use
3. For version-sensitive work, verify current versions with the actual current date
4. Validate configurations before recommending apply or deploy steps
```

### CLI-First DevOps Workflow

Prefer CLI validation and dry-run workflows:
```bash
# Docker
docker build -t test:latest .
docker-compose config  # Validate compose file
docker-compose up --dry-run  # Test without running

# Kubernetes
kubectl apply --dry-run=client -f manifest.yaml
kubectl diff -f manifest.yaml  # See changes before applying
kubeval manifest.yaml  # Validate against schema

# Terraform
terraform init
terraform fmt -recursive
terraform validate
terraform plan -out=tfplan  # Prefer planning before apply

# Helm
helm lint ./my-chart
helm template ./my-chart  # Render templates locally
helm install --dry-run --debug my-release ./my-chart
```

### Post-Edit Verification

After meaningful infrastructure changes, run the smallest useful validation for the files you touched:

```bash
# Docker
docker build -t test:latest .

# Terraform
terraform fmt -check -recursive
terraform validate
terraform plan

# Kubernetes
kubectl apply --dry-run=client -f manifest.yaml
```

Use broader checks only when the change warrants them.

### Common DevOps Syntax Traps (Avoid These!)

```yaml
# WRONG: YAML indentation with tabs
services:
	app:      # Tab character - YAML error!
		image: nginx

# CORRECT: Always use spaces (2 spaces standard)
services:
  app:
    image: nginx

# WRONG: Missing quotes for special values
environment:
  - VERSION=1.0      # Might be parsed as number
  - ENABLED=true     # Might be parsed as boolean

# CORRECT: Quote string values
environment:
  - VERSION="1.0"
  - ENABLED="true"

# WRONG: Hardcoded secrets in config
env:
  - name: DB_PASSWORD
    value: "supersecret123"  # NEVER do this!

# CORRECT: Use secrets
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secrets
        key: password
```

### Infrastructure Version Pinning

Always pin versions explicitly:

```dockerfile
# WRONG
FROM node:latest
FROM python

# CORRECT - Pin major.minor at minimum
FROM node:20-alpine
FROM python:3.12-slim
```

```hcl
# WRONG
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

# CORRECT - Pin provider versions
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

---

## Docker

### Dockerfile Best Practices

```dockerfile
# Use specific version tags
FROM node:20-alpine AS builder

# Set working directory
WORKDIR /app

# Copy dependency files first (better caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:20-alpine AS production

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy built assets from builder
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Run application
CMD ["node", "dist/main.js"]
```

### Multi-Stage Builds

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server ./cmd/server

# Final stage
FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /app
COPY --from=builder /app/server .
EXPOSE 8080
ENTRYPOINT ["./server"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - backend

volumes:
  postgres_data:

networks:
  backend:
    driver: bridge
```

---

## Kubernetes

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
