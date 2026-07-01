---
trigger: always_on
description: This document outlines Terraform-specific development guidelines for the terraform-aws-cognito-user-pool module, focusing on best practices for AWS Cognito identity and access management infrastructure as code.
---

# Terraform AWS Cognito User Pool Module - Development Guidelines

## Overview
This document outlines Terraform-specific development guidelines for the terraform-aws-cognito-user-pool module, focusing on best practices for AWS Cognito identity and access management infrastructure as code.

## Module Structure & Organization

### File Organization
- **main.tf** - Primary Cognito User Pool resource definitions and locals
- **variables.tf** - Input variable definitions with validation
- **outputs.tf** - Output value definitions
- **versions.tf** - Provider version constraints
- **client.tf** - Cognito User Pool client configurations
- **domain.tf** - User pool domain configurations
- **identity-provider.tf** - Identity provider configurations
- **managed-login-branding.tf** - Managed login branding configurations
- **resource-server.tf** - Resource server configurations
- **ui-customization.tf** - UI customization configurations
- **user-group.tf** - User group configurations

### Code Organization Principles
- Group related resources logically in separate files
- Use descriptive locals for complex expressions
- Maintain backward compatibility with existing variable names
- Keep validation logic close to variable definitions

## Terraform Best Practices

### Resource Creation Patterns
**Favor `for_each` over `count`** for resource creation:

```hcl
# Preferred: Using for_each
resource "aws_cognito_user_pool_client" "this" {
  for_each = var.enabled ? var.clients : {}

  name         = each.value.name
  user_pool_id = aws_cognito_user_pool.pool[0].id
  # ...
}

# Avoid: Using count when for_each is more appropriate
resource "aws_cognito_user_pool_client" "this" {
  count = var.enabled ? length(var.clients) : 0
  # ...
}
```

### Variables & Validation
Use validation blocks for critical inputs where appropriate:

```hcl
# Example: Basic validation for naming conventions
variable "user_pool_name" {
  description = "Name of the Cognito User Pool to create"
  type        = string
  default     = null

  validation {
    condition     = var.user_pool_name == null ? true : can(regex("^[0-9A-Za-z-_\\s]{1,128}$", var.user_pool_name))
    error_message = "The user_pool_name must be between 1 and 128 characters, contain only alphanumeric characters, spaces, hyphens, and underscores."
  }
}
```

### Locals Organization
Structure locals for clarity and reusability:

```hcl
locals {
  # Resource creation conditions
  should_create_pool   = var.enabled && var.user_pool_name != null
  should_create_domain = local.should_create_pool && var.domain != null

  # Data processing
  clients = concat(local.default_client, var.clients)

  # Validation helpers
  mfa_requirements_met = var.mfa_configuration != null && var.software_token_mfa_configuration != null
}
```

## AI-Powered Validation Requirements

This module uses AI-powered validation through specialized Claude Code subagents instead of traditional automated tests. This approach provides comprehensive analysis of code quality, security, Terraform best practices, and module functionality.

### Validation Strategy

#### Specialized Subagents Available
The module is configured with the following specialized validation agents:

1. **terraform-cognito** - AWS Cognito User Pool specialist for Terraform infrastructure development
   - Validates Cognito resource configurations
   - Ensures AWS best practices for identity management
   - Reviews authentication and authorization patterns

2. **cognito-migration** - AWS Cognito upgrade and migration specialist
   - Validates backward compatibility
   - Reviews migration paths for breaking changes
   - Ensures smooth version upgrades

3. **module-documentation** - Documentation and example specialist
   - Validates example configurations
   - Reviews documentation completeness
   - Ensures examples follow best practices

4. **terraform-security** - Security analysis and hardening specialist
   - Performs security analysis of configurations
   - Reviews password policies, MFA settings
   - Validates access controls and encryption settings
   - Checks for security vulnerabilities and misconfigurations

### When to Request AI Validation

#### For New Features
**Request validation when adding new features:**
- Request analysis from `terraform-cognito` agent for resource implementation review
- Use `module-documentation` agent to validate examples and documentation
- Invoke `terraform-security` agent for security implications analysis
- Ask for comprehensive review covering functionality, security, and best practices

**Example request:**
```
@claude Please use the terraform-cognito and terraform-security agents to review
the new advanced security mode feature I just implemented. Validate the
implementation follows AWS best practices and check for any security concerns.
```

#### For Modifications
**Request validation when modifying existing functionality:**
- Use `cognito-migration` agent to verify backward compatibility
- Invoke `terraform-security` agent if changes affect security configurations
- Request `terraform-cognito` agent review for resource configuration changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lgallard/terraform-aws-cognito-user-pool](https://github.com/lgallard/terraform-aws-cognito-user-pool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
