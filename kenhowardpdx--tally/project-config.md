---
trigger: always_on
description: This file provides context and guidelines for GitHub Copilot when working on the Tally project.
---

# GitHub Copilot Instructions

This file provides context and guidelines for GitHub Copilot when working on the Tally project.

## Project Overview

Tally is a financial application for managing recurring bills and forecasting bank account balances. The architecture consists of:

- **Backend**: Python FastAPI application deployed on AWS Lambda
- **Frontend**: Svelte application (planned) served via AWS CloudFront and S3
- **Database**: PostgreSQL on AWS RDS
- **Authentication**: Auth0 integration
- **Infrastructure**: Terraform for AWS resource management

## Cost Management Philosophy

**This project prioritizes cost optimization for solo developers without VC funding.**

### Cost-First Design Principles

- **Zero unnecessary AWS costs**: Eliminate NAT Gateways, unused Elastic IPs, and expensive services
- **Leverage AWS Free Tier**: Maximize usage of free services and resource limits
- **Public subnet architecture**: Lambda functions in public subnets (with security groups) vs private + NAT Gateway
- **Conservative resource sizing**: Use minimal CIDR ranges, single-AZ deployments where appropriate
- **Cost-aware alternatives**: Choose managed services only when they provide clear value over self-managed options

### Infrastructure Cost Guidelines

- **VPC Architecture**: Public subnets + security groups instead of private subnets + NAT Gateway (saves $588/year)
- **Database**: Consider RDS Free Tier limits, use db.t3.micro or similar cost-effective instances
- **Lambda**: Stay within generous free tier limits (1M requests, 400,000 GB-seconds per month)
- **Storage**: Use S3 Standard-IA or Glacier for infrequent access, lifecycle policies for cost optimization
- **Monitoring**: Use CloudWatch Free Tier, avoid unnecessary custom metrics and dashboards

## Code Style and Conventions

### Python (Backend)

- Use Python 3.13+ features
- Follow PEP 8 style guidelines
- Use type hints for all function signatures
- Prefer Poetry for dependency management
- Use pytest for testing
- Follow FastAPI best practices for API development
- Use Pydantic models for request/response validation

### Infrastructure (Terraform)

- Use Terraform >= 1.0.0
- Organize code into reusable modules
- Follow HashiCorp Configuration Language (HCL) best practices
- Use variables.tf and outputs.tf for module interfaces
- Include comprehensive resource tagging
- Use data sources for existing resources when possible
- **Cost optimization first**: Always consider monthly costs before adding resources
- **Favor free-tier resources**: VPC subnets, security groups, route tables are free
- **Avoid cost generators**: NAT Gateways ($45/month), Elastic IPs ($3.65/month), excessive data transfer

### File Formatting

- **Always end files with a single empty line** - This is a POSIX standard and helps with clean diffs
- Use consistent indentation (2 spaces for YAML/HCL, 4 spaces for Python)
- Remove trailing whitespace
- Use Unix line endings (LF)
- Keep line lengths reasonable (80-120 characters)

### Git Workflow

- Use conventional commits format (feat:, fix:, docs:, etc.)
- Create feature branches from main
- Use descriptive branch names (feature/description, fix/bug-name)
- Include tests for new features
- Update documentation when adding new functionality
- **Prefer rebase over merge commits**: Use `git rebase origin/main` instead of `git merge` to maintain clean history
- **Avoid merge commits**: When resolving conflicts, always use rebase (`git rebase origin/main`) instead of merge commits

#### Pull Request Creation

- **For complex PR descriptions**: Create `pr-body.md` file and use `gh pr create --body-file pr-body.md`
- **For simple PRs**: Use inline `--body` with GitHub CLI
- **Always clean up**: Remove `pr-body.md` after PR creation (it's temporary)
- **Use emojis and formatting**: Make PR descriptions clear and scannable with sections, checkboxes, and context

### GitHub Actions Version Management

- All GitHub Actions versions are centrally managed in `.github/action-versions.conf`
- Use `make validate-actions` to check that all workflows use approved versions
- Install git hooks with `make install-git-hooks` to automatically validate on commits
- Update `.github/action-versions.conf` when approving new action versions
- The validation script at `scripts/validate-actions.sh` enforces version consistency

#### Branch Management & Cleanup

- **Auto-prune setup**: Configure `git config --global fetch.prune true` for automatic cleanup
- **Regular pruning**: Use `git remote prune origin` to remove stale remote references
- **Local cleanup**: Delete merged branches with `git branch -d branch-name`
- **Force delete unmerged**: Use `git branch -D branch-name` for branches with open PRs
- **Verify cleanup**: Use `git branch -a` to check remaining branches

## Development Practices

### Local Development

- Use Docker Compose for local development environment
- **Use `.secrets` file** for local sensitive configuration (never commit this file)
- Test GitHub Actions locally using `act` (use Makefile targets like `make github_workflow_terraform-pr`)
- Validate Terraform changes locally before committing
- Run tests before pushing changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kenhowardpdx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
