---
trigger: always_on
description: Analyzes Infrastructure as Code files (Terraform, CloudFormation, Kubernetes, Docker Compose) and generates visual architecture diagrams. Use when analyzing infrastructure code, designing cloud architectures, or when the user requests architecture diagrams from IaC.
---


# IaC Architecture Diagram Generator

Analyzes Infrastructure as Code repositories and generates professional architecture diagrams using Nano Banana Pro. Supports Terraform, CloudFormation, Kubernetes, Docker Compose, Pulumi, and other common IaC formats.

## Core Philosophy

Infrastructure diagrams should accurately represent the logical architecture, resource relationships, and security boundaries defined in your IaC. This skill parses IaC files to extract resources, dependencies, and hierarchical structures, then generates diagrams that follow cloud architecture best practices.

## Workflow

When a user requests an architecture diagram from IaC files, follow these steps:

### Step 1: Discover IaC Files

Use Glob to identify IaC files in the target directory:

- **Terraform**: `*.tf`, `*.tfvars`
- **CloudFormation**: `*.yaml`, `*.yml`, `*.json`, `*.template`
- **Kubernetes**: `*.yaml`, `*.yml` (in manifests/, k8s/, kube/ directories)
- **Docker Compose**: `docker-compose.yaml`, `docker-compose.yml`
- **Pulumi**: `*.ts`, `*.py`, `*.go` (with Pulumi imports)
- **Azure ARM**: `*.json` (with ARM schema)
- **GCP Deployment Manager**: `*.yaml`, `*.jinja`, `*.py`

If no specific file is mentioned, search the current directory recursively.

### Step 2: Validate and Parse IaC Files

Run the appropriate parser script based on file type. The parser accepts **local paths or GitHub repository URLs**.

**Local files/directories:**
```bash
# Terraform
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform path/to/terraform/dir

# CloudFormation
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py cloudformation path/to/template.yaml

# Kubernetes
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py kubernetes path/to/manifests/

# Docker Compose
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py docker-compose path/to/docker-compose.yaml
```

**GitHub repositories (cloned automatically):**
```bash
# Clone entire repo and parse
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform https://github.com/user/repo

# Clone and parse specific subdirectory
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform https://github.com/user/repo/tree/main/infrastructure

# Short format also works
python ~/.claude/skills/iac-diagram-generator/scripts/parse_iac.py terraform github.com/user/repo
```

**Supported GitHub URL formats:**
- `https://github.com/user/repo`
- `https://github.com/user/repo/tree/branch/path/to/dir`
- `github.com/user/repo`
- `git@github.com:user/repo`

The parser automatically clones the repo to a temp directory, parses the files, and cleans up after.

The parser will return a JSON structure containing:
- Resources (compute, networking, storage, security)
- Dependencies and relationships
- Hierarchical organization (VPCs, subnets, namespaces)
- Connection types (public internet, private, managed services)

### Step 3: Analyze the Resource Graph

Review the parsed structure to understand:
- **Hierarchy**: VPC > Availability Zones > Subnets > Resources
- **Resource Types**: Compute (EC2, Lambda), Networking (VPC, Load Balancers), Storage (S3, RDS), Security (IAM, Security Groups)
- **Dependencies**: Which resources depend on others (explicit and implicit)
- **Connections**: How resources communicate (HTTP, database connections, message queues)
- **Security Boundaries**: VPCs, subnets, security groups, network ACLs

### Step 4: Generate Nano Banana Pro Diagram Prompt

Create a detailed, structured prompt for Nano Banana Pro that describes the architecture diagram using natural language. Follow these guidelines carefully to ensure **consistent, visually stunning results**.

#### Visual Design System

Every diagram MUST follow this standardized visual template for consistency:

**Canvas & Outer Margins:**
- "A professional 16:9 landscape architecture diagram"
- "The canvas has a clean white outer margin (at least 60 pixels on all sides) creating breathing room before the page edge"
- "This outer margin ensures no content touches or approaches the canvas boundaries"

**Border Frame (Inside the outer margin):**
- "A subtle rounded-corner border with a thin dark gray stroke frames all diagram content"
- "Everything - header, zones, connections, legend, and logos - is contained INSIDE this border"
- "An inner padding of 30 pixels separates all content from the border edge"

**Header Section (Inside the frame, top 12%):**
- "A gradient header bar spans the full width inside the frame, transitioning from deep navy blue (#1a365d) on the left to teal (#0d9488) on the right"
- "The title '[ARCHITECTURE NAME]' appears in large, bold white sans-serif text (like Inter or SF Pro) centered in the header"
- "A subtitle below reads '[Brief Description]' in smaller light gray text"

**Main Canvas (Inside the frame, middle 78%):**
- "The main area has a very light cool gray background (#f8fafc)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnpsasser/iac-diagram-generator](https://github.com/johnpsasser/iac-diagram-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
