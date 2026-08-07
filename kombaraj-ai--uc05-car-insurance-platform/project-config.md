---
trigger: always_on
description: This repo contains ALL infrastructure code for deploying the Car Insurance microservices to AWS.
---

# Car Insurance Platform — Claude Code Instructions

This repo contains ALL infrastructure code for deploying the Car Insurance microservices to AWS.
The application repo (car-insurance-microservices) is READ-ONLY — never modify it.

## Directory Layout

```
terraform/environments/{dev,prod}/   # Root modules (one per environment)
terraform/modules/{vpc,eks,ecr,rds,dns,secrets,karpenter}/
helm/carinsurance-service/              # Generic Helm chart (shared by all 9 services)
helm-values/                         # Per-service YAML + per-env (dev.yaml, prod.yaml)
k8s/base/                            # Namespaces, external-secrets CRs
k8s/argocd/install/                  # ArgoCD installation manifests
k8s/argocd/applications/{dev,prod}/  # ArgoCD Application CRDs
.github/workflows/                    # CI pipelines (build + push only, ArgoCD handles CD)
scripts/                             # Operational scripts
docs/                                # Architecture docs, runbooks, ADRs
```

## Terraform Conventions

- **Provider:** AWS provider ~> 5.0, region eu-central-1
- **ECR:** Uses `aws_ecr_repository` in eu-central-1 with lifecycle policies, scan-on-push, and configurable tag immutability
- **State:** S3 + DynamoDB locking, key pattern: `carinsurance/{env}/terraform.tfstate`
- **Modules:** All reusable modules in `terraform/modules/`. Environments call modules.
- **Naming:** `carinsurance-{env}-{resource}` (e.g., `carinsurance-dev-vpc`, `carinsurance-prod-eks`)
- **Tagging:** Every resource MUST have tags: `Project=carinsurance`, `Environment={dev|prod}`, `ManagedBy=terraform`
- **Variables:** Use `variable` blocks with `description`, `type`, and `default` where sensible
- **Outputs:** Export IDs, ARNs, and endpoints needed by downstream modules
- **Sensitive values:** Never hardcode secrets. Use `sensitive = true` for secret outputs.
- **Formatting:** Run `terraform fmt` before committing. Use `terraform validate` after edits.
- **Files per module:** `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf` (provider constraints)

## Kubernetes Conventions

- **Namespaces:** `carinsurance-dev`, `carinsurance-prod` (one namespace per environment)
- **Labels:** Every resource: `app.kubernetes.io/name`, `app.kubernetes.io/part-of=carinsurance`, `app.kubernetes.io/managed-by=Helm`
- **Probes:** Every Deployment MUST have readinessProbe and livenessProbe using `/actuator/health/{readiness,liveness}`
- **Resources:** Every container MUST have requests and limits (memory: 128Mi request / 512Mi limit)
- **Image tags:** Use commit SHA tags, never `latest` in production
- **Secrets:** Use ExternalSecret CRs pointing to AWS Secrets Manager — never store secrets in YAML
- **Service startup order:** Config Server → Discovery Server → all others (use init containers)
- **Packaging:** Helm chart (`helm/carinsurance-service/`), per-service + per-env values in `helm-values/`
- **Deployment:** ArgoCD GitOps — CI commits image tags to Git, ArgoCD syncs to cluster

## Helm Conventions

- **Single generic chart** in `helm/carinsurance-service/` shared by all 9 services
- **Per-service config** in `helm-values/{service}.yaml` (ports, env vars, init containers)
- **Per-env config** in `helm-values/{dev,prod}.yaml` (replicas, HPA, PDB, resource quotas)
- **ArgoCD merges values:** service file + env file when deploying
- **Template outputs** validated with `helm template` before commit

## ArgoCD Conventions

- **CI pushes images**, ArgoCD deploys. GitHub Actions NEVER runs `kubectl apply`.
- **Dev:** auto-sync enabled (prune + self-heal)
- **Prod:** manual sync required (approval via ArgoCD UI/CLI)
- **Application CRDs** in `k8s/argocd/applications/{dev,prod}/`
- **One Application per service per environment** (18 total: 9 services × 2 envs)

## Security Rules (NON-NEGOTIABLE)

1. **No secrets in code** — use AWS Secrets Manager + External Secrets Operator
2. **No public S3 buckets** — block public access on all buckets
3. **No open security groups** — no 0.0.0.0/0 ingress except ALB on 80/443
4. **Encryption everywhere** — RDS encryption at rest, S3 SSE, EBS encryption
5. **Least privilege IAM** — specific actions on specific resources, never `*/*`
6. **Security groups are the perimeter** — all resources in public subnets (cost optimization for learning), SGs enforce access control
7. **No terraform destroy without approval** — hooks block this command
8. **No *.tfvars or .env files committed** — .gitignore enforces this

## AWS Environment Details

| Setting | Dev | Prod |
|---------|-----|------|
| Region | eu-central-1 | eu-central-1 |
| K8s namespace | carinsurance-dev | carinsurance-prod |
| State key | carinsurance/dev/terraform.tfstate | carinsurance/prod/terraform.tfstate |
| RDS instance | db.t4g.micro, single-AZ (free tier) | db.t4g.micro, single-AZ (free tier) |
| EKS nodes | 2x t4g.small ARM (Graviton free trial) | 2x t4g.small ARM (Graviton free trial) |
| Deploy mode | ArgoCD auto-sync | ArgoCD manual sync |
| Replicas | 1 per service | 2+ per service, HPA |

## Application Services (9 total)

| Service | Port | Needs MySQL | Notes |
|---------|------|-------------|-------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kombaraj-ai/uc05-car-insurance-platform](https://github.com/kombaraj-ai/uc05-car-insurance-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
