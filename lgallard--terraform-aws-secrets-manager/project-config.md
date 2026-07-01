---
trigger: always_on
description: This document outlines Terraform-specific development guidelines for the terraform-aws-secrets-manager module, focusing on best practices for AWS infrastructure as code.
---

# Terraform AWS Secrets Manager Module - Development Guidelines

## Overview
This document outlines Terraform-specific development guidelines for the terraform-aws-secrets-manager module, focusing on best practices for AWS infrastructure as code.

## Module Structure & Organization

### File Organization
- **main.tf** - Primary resource definitions and locals
- **variables.tf** - Input variable definitions with validation
- **outputs.tf** - Output value definitions
- **versions.tf** - Provider version constraints

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
resource "aws_secretsmanager_secret" "this" {
  for_each = var.enabled ? var.secrets : {}

  name = each.value.name
  # ...
}

# Avoid: Using count when for_each is more appropriate
resource "aws_secretsmanager_secret" "this" {
  count = var.enabled ? length(var.secrets) : 0
  # ...
}
```

### Variables & Validation
Use validation blocks for critical inputs where appropriate:

```hcl
# Example: Basic validation for naming conventions
variable "secret_name" {
  description = "Name of the secret to create"
  type        = string
  default     = null

  validation {
    condition     = var.secret_name == null ? true : can(regex("^[0-9A-Za-z-_/.]{1,512}$", var.secret_name))
    error_message = "The secret_name must be between 1 and 512 characters, contain only alphanumeric characters, hyphens, underscores, periods, and forward slashes."
  }
}
```

### Locals Organization
Structure locals for clarity and reusability:

```hcl
locals {
  # Resource creation conditions
  should_create_secret = var.enabled && var.secret_name != null
  should_create_replica = local.should_create_secret && length(var.replica_regions) > 0

  # Data processing
  secrets = concat(local.secret, var.secrets)

  # Validation helpers
  rotation_requirements_met = var.rotation_enabled && var.rotation_lambda_arn != null
}
```

## Security Considerations

### General Security Practices
- Consider encryption requirements (KMS keys, etc.)
- Follow principle of least privilege for IAM
- Implement proper access controls
- Use secure defaults where possible

### Example Security Patterns
```hcl
# Example: KMS key validation (optional)
variable "secret_kms_key_arn" {
  description = "The server-side encryption key for secrets"
  type        = string
  default     = null

  validation {
    condition     = var.secret_kms_key_arn == null ? true : can(regex("^(arn:aws:kms:[a-z0-9-]+:[0-9]{12}:key/[a-f0-9-]{36}|alias/[a-zA-Z0-9/_-]+|[a-f0-9-]{36})$", var.secret_kms_key_arn))
    error_message = "KMS key ID must be a valid KMS key ID, key ARN, or alias ARN."
  }
}
```

## Ephemeral Password Support

### Overview
The module supports ephemeral mode to prevent sensitive data from being stored in Terraform state files. This security feature uses write-only arguments (`_wo` parameters) and requires Terraform 1.11 or later.

### When to Use Ephemeral Mode

**Use ephemeral mode when:**
- Working with highly sensitive credentials (database passwords, API keys, certificates)
- Security compliance requires that secrets never appear in state files
- Working in environments where state files might be exposed or audited
- Integrating with ephemeral resources (e.g., `random_password`)

**Consider standard mode when:**
- Security requirements are less stringent
- State file security is already ensured through other means
- Working with Terraform versions < 1.11
- Need maximum compatibility with existing workflows

### Configuration Patterns

#### Standard vs Ephemeral Mode Comparison

**Standard Mode (Default):**
```hcl
module "secrets_manager" {
  source = "lgallard/secrets-manager/aws"

  secrets = {
    database_password = {
      description   = "Database password"
      secret_string = var.db_password
    }
  }
}
```

**Ephemeral Mode:**
```hcl
module "secrets_manager" {
  source = "lgallard/secrets-manager/aws"

  # Enable ephemeral mode
  ephemeral = true

  secrets = {
    database_password = {
      description              = "Database password (ephemeral)"
      secret_string            = var.db_password
      secret_string_wo_version = 1  # Required for ephemeral mode
    }
  }
}
```

#### Secret Types with Ephemeral Mode

**String Secrets:**
```hcl
secrets = {
  api_token = {
    description              = "API authentication token"
    secret_string            = var.api_token
    secret_string_wo_version = 1
  }
}
```

**Key-Value Secrets:**
```hcl
secrets = {
  database_credentials = {
    description = "Database connection details"
    secret_key_value = {
      username = "admin"
      password = var.db_password
      host     = "db.example.com"
      port     = "5432"
    }
    secret_string_wo_version = 1
  }
}
```

**Binary Secrets:**
```hcl
secrets = {
  ssl_certificate = {
    description              = "SSL private key"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lgallard/terraform-aws-secrets-manager](https://github.com/lgallard/terraform-aws-secrets-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
