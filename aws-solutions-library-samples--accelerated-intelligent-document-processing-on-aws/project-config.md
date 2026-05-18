---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GenAI Intelligent Document Processing (GenAIIDP) is a serverless solution for automated document processing using AWS services. It combines OCR with generative AI to extract structured data from unstructured documents at scale.

The system uses a modular architecture with nested CloudFormation stacks supporting multiple document processing patterns while maintaining common infrastructure for queueing, tracking, and monitoring.

## Build & Development Commands

### Building and Publishing

Build and publish deployment artifacts to S3:

```bash
# Primary build script (recommended)
python3 publish.py <cfn_bucket_basename> <cfn_prefix> <region> [--verbose]

# Example
python3 publish.py idp-1234567890 idp us-east-1

# With verbose output for debugging build failures
python3 publish.py idp-1234567890 idp us-east-1 --verbose

```

The build process:
- Checks system dependencies (AWS CLI, SAM CLI, Docker, Python 3.12+, Node.js 22.12+)
- Builds CloudFormation templates and assets using SAM
- All pattern functions are built within the unified pattern directory
- Uploads artifacts to S3 bucket named `<cfn_bucket_basename>-<region>`

### Code Quality & Linting

```bash
# Run all linting and formatting (includes UI)
make lint

# Fast lint (skips UI lint if unchanged via checksum)
make fastlint

# Python linting only
make ruff-lint

# Python formatting only
make format

# Type checking with basedpyright
make typecheck
make typecheck-stats

# UI linting (checks for changes via checksum)
make ui-lint

# UI build verification
make ui-build

# CI/CD linting (check-only, no modifications)
make lint-cicd
```

### Testing

```bash
# Run all tests (idp_common_pkg + idp_cli + srt security scan)
make test

# Run tests in idp_common_pkg only
cd lib/idp_common_pkg && make test

# Run unit tests only
cd lib/idp_common_pkg && make test-unit

# Run integration tests (requires AWS resources)
cd lib/idp_common_pkg && make test-integration

# Run idp_cli tests
cd idp_cli && python -m pytest -v

# Run specific test markers
pytest -m "unit"
pytest -m "integration"
```

### Security Scanning

The project includes automated security scanning with the [Sample Security Review Tool (SRT)](https://github.com/aws-samples/sample-security-review-tool):

```bash
# Run full SRT workflow (setup → scan → optional fix)
make srt

# Or run individual steps:
make srt-setup     # Download and configure SRT
make srt-scan      # Run security assessment
make srt-fix       # Interactive fix mode
```

**CI/CD Integration:**
- SRT automatically runs on merge requests targeting `develop` branch (GitLab CI `security_review` stage)
- Does not run on feature branch pushes to avoid blocking development
- Pipeline fails if high-priority security findings are detected
- Provides security gate before code is merged to `develop`

### IDP CLI Commands

The IDP CLI is used for programmatic deployment and batch processing:

```bash
# Install all packages into current Python environment
make setup
# Or create an isolated .venv first
make setup-venv

# Deploy a new stack
idp-cli deploy \
    --stack-name my-idp-stack \
    --pattern pattern-2 \
    --admin-email your.email@example.com \
    --max-concurrent 100 \
    --wait

# Deploy with custom configuration
idp-cli deploy \
    --stack-name my-idp-stack \
    --pattern pattern-2 \
    --custom-config ./config_library/unified/bank-statement-sample/config.yaml \
    --wait

# Process documents in batch
idp-cli run-inference \
    --stack-name <your-stack-name> \
    --dir ./samples/ \
    --monitor

# Download results
idp-cli download-results \
    --stack-name <your-stack-name> \
    --batch-id <batch-id> \
    --output-dir ./results/
```

### Local Lambda Testing

```bash
cd patterns/unified/
sam build
sam local invoke OCRFunction -e ../../testing/OCRFunction-event.json --env-vars ../../testing/env.json
```

### Development Setup

```bash
# Install idp_common library in edit mode with all dependencies
cd lib/idp_common_pkg && make dev
```

## Architecture Overview

### Nested Stack Architecture

The solution uses a modular architecture with the main template (`template.yaml`) and nested pattern stacks:

**Main Stack** (`template.yaml`) - Pattern-agnostic resources:
- S3 Buckets (Input, Output, Working, Configuration, Evaluation Baseline)
- SQS Queues and Dead Letter Queues
- DynamoDB Tables (Execution Tracking, Concurrency, Configuration)
- Lambda Functions (Queue Processing, Queue Sending, Workflow Tracking, Document Status Lookup, Evaluation, UI Integration)
- CloudWatch Alarms and Dashboard
- Web UI Infrastructure (CloudFront, S3 for static assets, CodeBuild)
- Authentication (Cognito User Pool, Identity Pool)
- AppSync GraphQL API (for UI-backend communication)

**Unified Pattern Stack** (`patterns/unified/template.yaml`) - Processing resources:
- Step Functions State Machine (BDA branch + Pipeline branch + shared tail)
- Lambda Functions (OCR, Classification, Extraction, Assessment, Summarization, Evaluation, etc.)
- CloudWatch Dashboard

### Processing Modes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-solutions-library-samples/accelerated-intelligent-document-processing-on-aws](https://github.com/aws-solutions-library-samples/accelerated-intelligent-document-processing-on-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
