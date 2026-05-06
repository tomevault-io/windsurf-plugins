---
trigger: always_on
description: Guidelines for Docker, Kubernetes, Terraform, and deployment
---


# Infrastructure and DevOps Rules

## Docker

### Dockerfile Best Practices
- Use multi-stage builds to minimize image size
- Use specific base image versions (not `latest`)
- Run as non-root user when possible
- Minimize number of layers
- Use `.dockerignore` to exclude unnecessary files
- Pin dependency versions
- Scan images for vulnerabilities regularly

### Docker Compose
- Use version 3.x format
- Define health checks for services
- Use named volumes for persistence
- Set resource limits
- Use environment files for configuration
- Document required environment variables

### Image Tagging
- Tag with semantic versions (`1.0.0`)
- Use `latest` tag for most recent stable
- Include git commit SHA in CI builds
- Tag images with architecture (`amd64`, `arm64`)

## Kubernetes

### Manifest Organization
- Separate manifests by resource type
- Use namespaces for logical separation
- Apply consistent labels across resources
- Use resource requests and limits
- Implement readiness and liveness probes

### Resource Naming
- Use kebab-case for resource names
- Include component/tier in names
- Keep names descriptive but concise
- Use consistent naming across environments

### ConfigMaps and Secrets
- Store configuration in ConfigMaps
- Use Secrets for sensitive data
- Never commit secrets to version control
- Use external secret management in production
- Rotate secrets regularly

### Security
- Use Pod Security Standards/Policies
- Run containers as non-root
- Use read-only root filesystems when possible
- Implement network policies
- Scan images before deployment
- Keep Kubernetes version up to date

### Best Practices
- Set resource requests and limits for all containers
- Use rolling updates with proper health checks
- Implement horizontal pod autoscaling
- Use persistent volumes for stateful data
- Configure pod disruption budgets
- Enable RBAC and follow principle of least privilege

## Helm

### Chart Structure
```
helm/package-repo/
├── Chart.yaml           # Chart metadata
├── values.yaml          # Default values
├── values-dev.yaml      # Environment-specific
├── values-prod.yaml
├── templates/           # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl
└── README.md
```

### Chart Best Practices
- Use semantic versioning for charts
- Include comprehensive values.yaml documentation
- Validate templates with `helm lint`
- Test chart installation in clean environment
- Use template functions for consistent formatting
- Implement proper NOTES.txt for post-install info
- Use `.helmignore` to exclude unnecessary files

### Values Files
- Provide sensible defaults in values.yaml
- Document all values with comments
- Use nested structure for organization
- Support multiple environments
- Make resource limits configurable

### Templates
- Use `_helpers.tpl` for common template functions
- Keep templates DRY (Don't Repeat Yourself)
- Use proper indentation
- Add comments for complex logic
- Use helm hooks for lifecycle management

## Terraform

### Code Organization
```
terraform/
├── aws/                # Provider-specific modules
├── gcp/
├── azure/
├── digitalocean/
├── vultr/
└── modules/            # Reusable modules
    ├── networking/
    ├── compute/
    └── storage/
```

### Terraform Best Practices
- **Always run `terraform fmt` before committing**
- Use modules for reusable components
- Pin provider versions
- Use remote state with locking
- Never commit `.tfstate` files
- Use `.tfvars` files for environment configs
- Run `terraform validate` before applying
- Use meaningful resource names
- Tag all resources consistently

### Variables
- Define all variables in `variables.tf`
- Provide descriptions for all variables
- Set appropriate types (string, number, bool, list, map)
- Use validation rules when appropriate
- Provide sensible defaults when possible
- Document required vs optional variables

### Outputs
- Export important resource IDs and endpoints
- Document output values
- Use outputs to chain modules
- Include helpful information for next steps

### State Management
- Use remote backend (S3, GCS, Azure Blob)
- Enable state locking
- Use separate state files per environment
- Back up state files regularly
- Use workspaces for environment separation

### Security
- Use variables for sensitive data (API keys, passwords)
- Never commit sensitive data in `.tf` files
- Use secrets management services
- Enable encryption for state files
- Implement proper IAM/RBAC policies
- Use security scanning tools (tfsec, checkov)

### Cloud Provider Best Practices

#### AWS
- Use AWS Organizations for multi-account setup
- Implement proper VPC design
- Use security groups restrictively
- Enable CloudWatch logging
- Use IAM roles instead of access keys
- Enable encryption at rest and in transit

#### GCP
- Use GCP Projects for resource isolation
- Implement proper VPC design
- Use service accounts with minimal permissions
- Enable Cloud Logging and Monitoring
- Use encryption at rest and in transit

#### Azure
- Use Resource Groups for organization
- Implement proper VNet design
- Use Managed Identities
- Enable Azure Monitor
- Use encryption at rest and in transit

#### DigitalOcean
- Use VPC for network isolation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/package-repository-server](https://github.com/quinnjr/package-repository-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
