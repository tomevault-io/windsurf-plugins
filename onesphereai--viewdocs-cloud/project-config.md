---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Viewdocs Cloud Migration** is a multi-tenant, serverless document management system migrating from on-premise Java/Spring/Tomcat/Oracle stack to AWS serverless architecture. The system provides document viewing, search, download, and management capabilities across multiple archive systems (IESC, IES, CMOD).

### Key Characteristics
- **Multi-tenant**: Pool model with logical isolation (5-500 tenants)
- **Serverless**: AWS Lambda, API Gateway, DynamoDB, S3, CloudFront
- **Hybrid**: Integrates with on-premise systems (IES, CMOD, FRS, HUB) via Direct Connect
- **Multi-region**: Primary in ap-southeast-2, DR in ap-southeast-4 (Active-Passive failover)
- **Scale**: 500 concurrent users, 10-1000 users per tenant

## Technology Stack

### Backend
- **Language**: TypeScript
- **Runtime**: Node.js on AWS Lambda
- **API**: API Gateway (REST)
- **Database**: DynamoDB with Global Tables
- **Storage**: S3 (temporary document storage, bulk downloads)
- **Authentication**: AWS Cognito with SAML 2.0 federation
- **Event Processing**: EventBridge, Step Functions, SQS

### Frontend
- **Framework**: Angular
- **Hosting**: S3 + CloudFront

### Infrastructure as Code
- **Tool**: AWS CDK (TypeScript)
- **Deployment**: CDK Pipelines

### Integrations
- **IESC**: REST API (AWS-hosted, per-tenant stack)
- **IES**: SOAP API (on-premise via Direct Connect)
- **CMOD**: SOAP API (IBM on-premise via Direct Connect)
- **FRS Proxy**: SOAP API (AWS proxy to on-premise FRS/IBM MQ via Direct Connect)
- **IDM**: SAML 2.0 IdP (AWS-hosted) + support for external IdPs
- **Email**: IDM Email Service (current), Email Platform REST (future)
- **MailRoom Backend**: REST API (AWS-hosted, independent document routing and assignment microservice)

## Architecture Principles

### Multi-Tenancy (Pool Model)
- **Logical Isolation**: Single shared infrastructure with tenant_id partitioning
- **Tenant Identification**: Subdomain-based routing (tenant1.viewdocs.example.com)
- **Data Isolation**: DynamoDB partition key design with tenant_id prefix
- **No Cross-Tenant Access**: Strict authorization checks on every request

### Serverless & Non-VPC (ADR-012)
- **Lambda WITHOUT VPC**: All Lambda functions deployed without VPC for cost savings ($1,992/year) and performance (faster cold starts)
- **Public Endpoints**: Lambda connects to AWS services (DynamoDB, S3, Secrets Manager) via public endpoints with IAM authentication
- **Direct Connect**: Public VIF with IPsec VPN tunnel to on-premise systems (IES, CMOD, FRS)
- **No NAT Gateways**: No VPC infrastructure to manage (no subnets, route tables, security groups)
- **Equivalent Security**: TLS 1.2+ encryption + IPsec VPN + IAM roles + KMS encryption

### Security
- **Data Residency**: All data in Australia (ap-southeast-2, ap-southeast-4)
- **Encryption at Rest**: DynamoDB, S3 with KMS
- **Encryption in Transit**: TLS 1.2+ for AWS services, IPsec VPN for Direct Connect
- **Authentication**: Cognito with SAML 2.0, support for external IdPs
- **Authorization**: Role-based access control (RBAC) with ACLs stored in DynamoDB
- **Audit**: All document operations logged to DynamoDB with TTL (6mo prod, 1mo UAT, 1wk dev)

### Performance & Resilience
- **Caching**: CloudFront for static assets, DynamoDB for configuration/ACLs
- **Async Processing**: Step Functions for bulk downloads (up to 5GB)
- **Retry Logic**: Built into FRS Proxy for on-premise integrations
- **Concurrency**: Lambda concurrency controls to prevent noisy neighbor

### Observability
- **Logging**: CloudWatch Logs (centralized per tenant)
- **Metrics**: CloudWatch Metrics
- **Tracing**: X-Ray for distributed tracing
- **Events**: EventBridge for real-time events to HUB

## Common Development Commands

### Source Control (Bitbucket)
```bash
# Create feature branch
git checkout -b feature/JIRA-123-description

# Commit changes
git add .
git commit -m "feat: add bulk download feature"

# Push to Bitbucket
git push origin feature/JIRA-123-description

# Create Pull Request via Bitbucket UI
# After PR approval, merge to develop triggers Jenkins pipeline
```

### CDK Infrastructure
```bash
# Install dependencies
npm install

# Bootstrap CDK (first time only per account/region)
cdk bootstrap aws://ACCOUNT-ID/ap-southeast-2
cdk bootstrap aws://ACCOUNT-ID/ap-southeast-4

# Synthesize CloudFormation templates
cdk synth

# Deploy to dev environment (via Jenkins or manual)
cdk deploy --all --context env=dev

# Deploy to UAT environment (via Jenkins or manual)
cdk deploy --all --context env=uat

# Deploy to prod environment (via Jenkins only - requires Heat approval)
cdk deploy --all --context env=prod

# Destroy stack (dev/uat only)
cdk destroy --all --context env=dev
```

### Backend Lambda Development
```bash
# Install dependencies
cd backend
npm install

# Run unit tests
npm test

# Run single test file
npm test -- --testPathPattern=document-service.test.ts

# Run tests with coverage
npm test -- --coverage

# Lint code
npm run lint

# Fix linting issues
npm run lint:fix

# Build for deployment
npm run build

# Run locally with SAM (if configured)
sam local start-api
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onesphereai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
