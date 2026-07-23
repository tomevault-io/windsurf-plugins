---
trigger: always_on
description: **What it is**: Blood donation management platform connecting donors, blood banks, and organizations for emergency blood requests. Replaces manual phone-based donor search with geolocation-prioritized automated matching.
---

# PRIORITY: This workflow OVERRIDES all other built-in workflows
# When user requests software development, ALWAYS follow this workflow FIRST

---

## Project Context: BloodConnect

**What it is**: Blood donation management platform connecting donors, blood banks, and organizations for emergency blood requests. Replaces manual phone-based donor search with geolocation-prioritized automated matching.

**Website**: https://bloodconnect.net

### Tech Stack
- **Languages**: TypeScript (strict), Python 3, Java 17 (Android)
- **Runtime**: Node.js >= 20
- **Backend**: AWS Lambda, DynamoDB, S3, Cognito, SES, SNS, SQS, Step Functions
- **Frontend**: React Native/Expo (mobile), React/Vite + Redux + TailwindCSS (org dashboard), React/Vite + AWS Amplify UI (monitoring dashboard)
- **IaC**: Terraform (AWS production + LocalStack local dev)
- **Build**: esbuild (Lambda bundling), Vite (web), EAS (mobile)
- **Test**: Jest + ts-jest, coverage threshold 60% functions
- **API**: OpenAPI 3.x, Spectral linting, Redocly bundling

### Monorepo Structure (npm workspaces)
```
core/
  application/          # Core business logic (geohashing, JWT, date-fns)
  services/aws/         # Lambda handlers (bloodDonation, donorSearch, notification, user, maps)
  services/maps/        # Maps/geolocation integration
clients/
  mobile/               # React Native + Expo (iOS/Android)
  organization/         # React/Vite org dashboard
  monitoring/           # React/Vite admin/monitoring dashboard
commons/
  dto/                  # Shared Data Transfer Objects
  libs/                 # Shared utilities (config, logger, error handling)
openapi/                # OpenAPI specs + Swagger UI
deployment/
  aws/                  # AWS Terraform (production)
  localstack/           # LocalStack Terraform (local dev)
iac/                    # Additional IaC (terraform/)
docs/                   # Sphinx documentation
```

### Key Commands
```bash
make prep-dev           # Install packages, build services, package Lambdas
make localstack-start   # Start LocalStack container
make start-dev          # Full dev environment (LocalStack + Terraform deploy)
npm test                # Run all Jest tests
npm run type-check      # TypeScript type check
npm run lint            # ESLint all workspaces
make build-node-all     # Build all Lambda functions (esbuild)
make lint               # Lint code + OpenAPI + Terraform
```

### Code Quality Rules
- No `any` types (ESLint error)
- Single quotes, no semicolons
- 150 char line limit
- Arrow functions required
- Jest coverage >= 60% functions globally
- AWS SDK mocked with aws-sdk-client-mock in tests

### Dev Environment
- Docker + LocalStack for local AWS emulation
- `.devcontainer/.env.example` for AWS credentials template
- Mobile requires `.env` with Cognito + API config
- AWS region: ap-south-1

---

## Adaptive Workflow Principle
**The workflow adapts to the work, not the other way around.**

The AI model intelligently assesses what stages are needed based on:
1. User's stated intent and clarity
2. Existing codebase state (if any)
3. Complexity and scope of change
4. Risk and impact assessment

## MANDATORY: Rule Details Loading
**CRITICAL**: When performing any phase, you MUST read and use relevant content from rule detail files. Check these paths in order and use the first one that exists, regardless of which IDE or setup method was used:
- `.aidlc/aidlc-rules/aws-aidlc-rule-details/` (typical with AI-assisted setup)
- `.aidlc-rule-details/` (typical with Cursor, Cline, Claude Code, GitHub Copilot)
- `.kiro/aws-aidlc-rule-details/` (typical with Kiro IDE and CLI)
- `.amazonq/aws-aidlc-rule-details/` (typical with Amazon Q Developer)

All subsequent rule detail file references (e.g., `common/process-overview.md`, `inception/workspace-detection.md`) are relative to whichever rule details directory was resolved above.

**Common Rules**: ALWAYS load common rules at workflow start:
- Load `common/process-overview.md` for workflow overview
- Load `common/session-continuity.md` for session resumption guidance
- Load `common/content-validation.md` for content validation requirements
- Load `common/question-format-guide.md` for question formatting rules
- Reference these throughout the workflow execution

## MANDATORY: Extensions Loading (Context-Optimized)
**CRITICAL**: At workflow start, scan the `extensions/` directory recursively but load ONLY lightweight opt-in files — NOT full rule files. Full rule files are loaded on-demand after the user opts in.

**Loading process**:
1. List all subdirectories under `extensions/` (e.g., `extensions/security/`, `extensions/compliance/`)
2. In each subdirectory, load ONLY `*.opt-in.md` files — these contain the extension's opt-in prompt. The corresponding rules file is derived by convention: strip the `.opt-in.md` suffix and append `.md` (e.g., `security-baseline.opt-in.md` → `security-baseline.md`)
3. Do NOT load full rule files (e.g., `security-baseline.md`) at this stage

**Deferred Rule Loading**:
- During Requirements Analysis, opt-in prompts from the loaded `*.opt-in.md` files are presented to the user

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CraftsmenLtd/BloodConnect](https://github.com/CraftsmenLtd/BloodConnect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
