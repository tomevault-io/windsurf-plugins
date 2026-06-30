---
trigger: always_on
description: This document outlines Terraform-specific development guidelines for the terraform-aws-backup module, focusing on best practices for AWS infrastructure as code.
---

# Terraform AWS Backup Module - Development Guidelines

## Overview
This document outlines Terraform-specific development guidelines for the terraform-aws-backup module, focusing on best practices for AWS infrastructure as code.

## Module Structure & Organization

### File Organization
- **main.tf** - Primary resource definitions and locals
- **variables.tf** - Input variable definitions with validation
- **outputs.tf** - Output value definitions
- **versions.tf** - Provider version constraints
- **iam.tf** - IAM roles and policies
- **notifications.tf** - SNS and notification configurations
- **organizations.tf** - AWS Organizations backup policies
- **selection.tf** - Resource selection logic
- **reports.tf** - Backup reporting configurations
- **audit_manager.tf** - Audit framework configurations

### Code Organization Principles
- Group related resources logically in separate files
- Use descriptive locals for complex expressions
- Maintain backward compatibility with existing variable names
- Keep validation logic close to variable definitions

## MCP-Powered Validation Workflow

### Development Lifecycle with MCP Integration

**Before Implementation:**
```bash
# Validate resource documentation
Terraform MCP: "Look up aws_backup_vault resource documentation"
Context7: "Get AWS Backup best practices"
```

**During Development:**
```bash
# Validate syntax and arguments
Terraform MCP: "Validate aws_backup_plan arguments"
Terraform MCP: "Check aws_backup_selection resource requirements"
```

**Validation Phase:**
```bash
# AI-powered validation
Task Agent (general-purpose): "Validate implementation comprehensively"
Task Agent (Explore - thorough): "Analyze code patterns and dependencies"
```

**Security Review:**
```bash
# Validate security configurations
Context7: "AWS Backup encryption best practices"
Terraform MCP: "Look up aws_kms_key for backup vault"
```

## Terraform Best Practices

### Resource Creation Patterns
**Favor `for_each` over `count`** for resource creation:

```hcl
# Preferred: Using for_each
resource "aws_backup_plan" "this" {
  for_each = var.enabled ? var.plans : {}

  name = each.value.name
  # ...
}

# Validate with: Terraform MCP "Check aws_backup_plan for_each patterns"
```

### Variables & Validation
Use validation blocks for critical inputs:

```hcl
variable "vault_name" {
  description = "Name of the backup vault to create"
  type        = string
  default     = null

  validation {
    condition     = var.vault_name == null ? true : can(regex("^[0-9A-Za-z-_]{2,50}$", var.vault_name))
    error_message = "The vault_name must be between 2 and 50 characters, contain only alphanumeric characters, hyphens, and underscores."
  }
}
# Validate naming rules with: Terraform MCP "aws_backup_vault naming constraints"
```

### Locals Organization
Structure locals for clarity and reusability:

```hcl
locals {
  # Resource creation conditions
  should_create_vault = var.enabled && var.vault_name != null
  should_create_lock  = local.should_create_vault && var.locked

  # Data processing
  rules = concat(local.rule, var.rules)

  # Validation helpers
  vault_lock_requirements_met = var.min_retention_days != null && var.max_retention_days != null
}
```

## AI-Powered Validation & Analysis

This module uses AI-powered validation instead of traditional automated tests. Claude AI with specialized subagents provides comprehensive code analysis, validation, and quality assurance.

### Validation Philosophy

Traditional testing frameworks require maintenance, can become outdated, and may not catch semantic issues or best practice violations. AI-powered validation provides:

- **Contextual Understanding**: Analyzes code semantically, not just syntactically
- **Best Practices Enforcement**: Validates against AWS and Terraform best practices
- **Security Analysis**: Identifies potential security vulnerabilities
- **Documentation Consistency**: Ensures examples match documentation
- **Performance Optimization**: Suggests improvements based on AWS Backup patterns

### Specialized Subagents for Validation

#### 1. Explore Agent - Code Understanding & Discovery
Use the Explore agent to understand codebase structure and relationships:

```bash
# Quick exploration (basic searches)
Task Agent (Explore - quick): "Find all backup vault configurations"
Task Agent (Explore - quick): "Locate IAM role definitions for backup service"

# Medium exploration (moderate depth)
Task Agent (Explore - medium): "Analyze backup selection patterns across the module"
Task Agent (Explore - medium): "Map all cross-region backup configurations"

# Thorough exploration (comprehensive analysis)
Task Agent (Explore - very thorough): "Examine all security configurations and encryption patterns"
Task Agent (Explore - very thorough): "Analyze complete audit framework implementation"
```

#### 2. Plan Agent - Implementation Strategy & Validation
Use the Plan agent for validating implementation approaches:

```bash
# Validate new feature implementation
Task Agent (Plan - medium): "Review the implementation strategy for VSS backup support"

# Analyze refactoring impact

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lgallard/terraform-aws-backup](https://github.com/lgallard/terraform-aws-backup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
