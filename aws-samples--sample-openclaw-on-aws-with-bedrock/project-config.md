---
trigger: always_on
description: This repository provides **AWS-native CloudFormation templates** for deploying [OpenClaw](https://www.npmjs.com/package/openclaw) (formerly Clawdbot/Moltbot), an open-source personal AI assistant. It integrates with Amazon Bedrock for model access and supports messaging platforms including WhatsApp, Telegram, Discord, Slack, Microsoft Teams, iMessage, and Google Chat.
---

# CLAUDE.md

## Project Overview

This repository provides **AWS-native CloudFormation templates** for deploying [OpenClaw](https://www.npmjs.com/package/openclaw) (formerly Clawdbot/Moltbot), an open-source personal AI assistant. It integrates with Amazon Bedrock for model access and supports messaging platforms including WhatsApp, Telegram, Discord, Slack, Microsoft Teams, iMessage, and Google Chat.

This is **not** an application codebase -- it is infrastructure-as-code (CloudFormation YAML) plus documentation. There are no Lambda functions, no package.json, no build system, and no automated tests.

## Repository Structure

```
.
├── clawdbot-bedrock.yaml          # Main CloudFormation template (Linux/Graviton)
├── clawdbot-bedrock-mac.yaml      # macOS CloudFormation template (Apple Silicon/Intel)
├── .kiro/
│   └── steering/
│       └── deploy-moltbot-conversationally.md  # Kiro AI deployment guide (~1300 lines)
├── images/                        # Screenshots for documentation
├── README.md                      # Primary documentation (English)
├── README_CN.md                   # Chinese documentation
├── DEPLOYMENT.md                  # Step-by-step deployment guide
├── SECURITY.md                    # Security architecture and best practices
├── TROUBLESHOOTING.md             # Common issues and resolution steps
├── QUICK_START_KIRO.md            # Kiro AI-guided deployment quickstart
├── CONTRIBUTING.md                # Contribution guidelines
├── CODE_OF_CONDUCT.md             # Amazon Open Source Code of Conduct
└── LICENSE                        # MIT No Attribution
```

## Key Files

### CloudFormation Templates

- **`clawdbot-bedrock.yaml`** (~660 lines): The primary deployment template for Linux (Ubuntu 24.04 on Graviton ARM or x86). Creates a full VPC, IAM roles, security groups, optional VPC endpoints, and an EC2 instance that bootstraps OpenClaw via a UserData script.

- **`clawdbot-bedrock-mac.yaml`** (~760 lines): macOS variant requiring a Dedicated Host. Supports mac1.metal (Intel), mac2.metal (M1), mac2-m2.metal (M2), and mac2-m2pro.metal (M2 Pro). Includes region-specific AMI mappings.

### Template Parameters

Both templates share these key parameters:
- `OpenClawModel` -- Bedrock model ID (default: `global.amazon.nova-2-lite-v1:0`; 9 models available)
- `InstanceType` -- EC2 instance type (default: `c7g.large` Graviton)
- `KeyPairName` -- EC2 key pair for emergency SSH
- `AllowedSSHCIDR` -- SSH access CIDR (set to `127.0.0.1/32` to disable)
- `CreateVPCEndpoints` -- Private networking toggle (adds ~$22/month)
- `EnableSandbox` -- Docker sandbox isolation for non-main sessions

### Template Conditions

- `CreateEndpoints` -- Controls VPC endpoint creation
- `AllowSSH` -- Controls SSH security group rules
- `UseGraviton` -- Detects ARM instance types (t4g, c7g, m7g) for AMI selection

### Resources Created (Linux Template)

| Category | Resources |
|----------|-----------|
| Networking | VPC, Internet Gateway, public/private subnets, route tables |
| VPC Endpoints | Bedrock Runtime, Bedrock Mantle, SSM, SSM Messages, EC2 Messages (conditional) |
| IAM | Role (SSM + CloudWatch + Bedrock + SSM Parameter Store), Instance Profile |
| Security | EC2 security group, VPC endpoint security group (conditional) |
| Compute | EC2 instance with ~200-line UserData bootstrap script |
| Orchestration | CloudFormation WaitCondition with 900s timeout |

### UserData Bootstrap Sequence

The EC2 UserData script installs and configures:
1. System packages (apt-get update/upgrade)
2. AWS CLI v2
3. SSM Agent
4. Docker engine
5. Node.js 22 via NVM (with retry logic)
6. OpenClaw npm package (global install with retry logic)
7. AWS region detection and configuration
8. OpenClaw config (`~/.openclaw/openclaw.json`) with generated gateway token
9. systemd user service registration

## Supported Models

| Model | ID |
|-------|----|
| Nova 2 Lite (default) | `global.amazon.nova-2-lite-v1:0` |
| Claude Sonnet 4.5 | `global.anthropic.claude-sonnet-4-5-20250929-v1:0` |
| Nova Pro | `us.amazon.nova-pro-v1:0` |
| Claude Opus 4.6 | `global.anthropic.claude-opus-4-6-v1` |
| Claude Opus 4.5 | `global.anthropic.claude-opus-4-5-20251101-v1:0` |
| Claude Haiku 4.5 | `global.anthropic.claude-haiku-4-5-20251001-v1:0` |
| Claude Sonnet 4 | `global.anthropic.claude-sonnet-4-20250514-v1:0` |
| DeepSeek R1 | `us.deepseek.r1-v1:0` |
| Llama 3.3 70B | `us.meta.llama3-3-70b-instruct-v1:0` |
| Kimi K2.5 | `moonshotai.kimi-k2.5` |

**Note on Project Mantle models:** Kimi K2.5 is a Project Mantle model that uses the `bedrock-mantle` endpoint (`https://bedrock-mantle.REGION.api.aws/v1`) for OpenAI-compatible API access. While it also works through the standard Converse API (`bedrock-runtime`), there are known issues with tool-call parsing. The templates include both `bedrock-runtime` and `bedrock-mantle` VPC endpoints to support all models.

## Supported Regions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-OpenClaw-on-AWS-with-Bedrock](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
