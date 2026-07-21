---
trigger: always_on
description: **Alex** (Agentic Learning Equities eXplainer) is a multi-agent enterprise-grade SaaS financial planning platform. This is the capstone project for Weeks 3 and 4 of the "AI in Production" course taught by Ed Donner on Udemy that deploys Agent solutions to production.
---

# Alex - AI in Production Course Project Guide

## Project Overview

**Alex** (Agentic Learning Equities eXplainer) is a multi-agent enterprise-grade SaaS financial planning platform. This is the capstone project for Weeks 3 and 4 of the "AI in Production" course taught by Ed Donner on Udemy that deploys Agent solutions to production.

The user is a student on the course. You are working with the user to help them build Alex successfully. The user is working in Cursor (the VS Code fork), and they might be on a Windows PC, a Mac (intel or Apple silicon) or a Linux machine. All python code is run with uv and there are uv projects in every directory that needs it. The student is familiar with AWS services (Lambda, App Runner, Cloudfront) and has been introduced to Terraform, uv, NextJS and docker. They have budget alerts set, but they should still regularly check the billing screens in AWS console to keep a close watch on costs.

The student has an AWS root user, and also an IAM user called "aiengineer" with permissions. They have run `aws configure` and should be signed in as the aiengineer user with their default region.

### What Students Will Build

Students will deploy a complete production AI system featuring:
- **Multi-agent collaboration**: 5 specialized AI agents working together via orchestration
- **Serverless architecture**: Lambda, Aurora Serverless v2, App Runner, API Gateway, SQS
- **Cost-optimized vector storage**: S3 Vectors (90% cheaper than OpenSearch)
- **Real-time financial analysis**: Portfolio management, retirement projections, market research
- **Production-grade practices**: Observability, guardrails, security, monitoring
- **Full-stack application**: NextJS React frontend with Clerk authentication

### Learning Objectives

By completing this project, students will:
1. Deploy and manage production AI infrastructure on AWS
2. Implement multi-agent systems using the OpenAI Agents SDK
3. Integrate AWS Bedrock (with Nova Pro model) for LLM capabilities
4. Build cost-effective vector search with S3 Vectors and SageMaker embeddings
5. Create serverless agent orchestration with SQS and Lambda
6. Deploy a complete full-stack SaaS application
7. Implement enterprise features: monitoring, observability, guardrails, security

### Commercial Product

Alex is a SaaS product that provides insights on users' equity portfolios through reports and charts. Alex is integrated with Clerk for user management and the database architecture keeps user data separate.

---

## Directory Structure

```
alex/
├── guides/              # Step-by-step deployment guides (START HERE)
│   ├── 1_permissions.md
│   ├── 2_sagemaker.md
│   ├── 3_ingest.md
│   ├── 4_researcher.md
│   ├── 5_database.md
│   ├── 6_agents.md
│   ├── 7_frontend.md
│   ├── 8_enterprise.md
│   ├── architecture.md
│   └── agent_architecture.md
│
├── backend/             # Agent code and Lambda functions
│   ├── planner/         # Orchestrator agent
│   ├── tagger/          # Instrument classification agent
│   ├── reporter/        # Portfolio analysis agent
│   ├── charter/         # Visualization agent
│   ├── retirement/      # Retirement projection agent
│   ├── researcher/      # Market research agent (App Runner)
│   ├── ingest/          # Document ingestion Lambda
│   ├── database/        # Shared database library
│   └── api/             # FastAPI backend for frontend
│
├── frontend/            # NextJS React application
│   ├── pages/
│   ├── components/
│   └── lib/
│
├── terraform/           # Infrastructure as Code (IMPORTANT: Independent directories)
│   ├── 2_sagemaker/     # SageMaker embedding endpoint
│   ├── 3_ingestion/     # S3 Vectors and ingest Lambda
│   ├── 4_researcher/    # App Runner research service
│   ├── 5_database/      # Aurora Serverless v2
│   ├── 6_agents/        # Multi-agent Lambda functions
│   ├── 7_frontend/      # CloudFront, S3, API Gateway
│   └── 8_enterprise/    # CloudWatch dashboards and monitoring
│
└── scripts/             # Deployment and local development scripts
    ├── deploy.py        # Frontend deployment
    ├── run_local.py     # Local development
    └── destroy.py       # Cleanup script
```

---

## Course Structure: The 8 Guides

**IMPORTANT:** before working with the student, you MUST read all guides in the guides folder, in the correct order (1-8), to fully understand the project.

### Week 3: Research Infrastructure

**Day 3 - Foundations**
- **Guide 1: AWS Permissions** (1_permissions.md)
  - Set up IAM permissions for Alex project
  - Create AlexAccess group with required policies
  - Configure AWS CLI and credentials

- **Guide 2: SageMaker Deployment** (2_sagemaker.md)
  - Deploy SageMaker Serverless endpoint for embeddings
  - Use HuggingFace all-MiniLM-L6-v2 model
  - Test embedding generation
  - Understand serverless vs always-on endpoints

**Day 4 - Vector Storage**
- **Guide 3: Ingestion Pipeline** (3_ingest.md)
  - Create S3 Vectors bucket (90% cost savings!)
  - Deploy Lambda function for document ingestion
  - Set up API Gateway with API key auth

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ed-donner/alex](https://github.com/ed-donner/alex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
