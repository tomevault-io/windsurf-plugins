---
trigger: always_on
description: This file defines the Terraform style conventions for this repository, based on the [HashiCorp Terraform Style Guide](https://developer.hashicorp.com/terraform/language/style). All Terraform code in this workspace MUST conform to these rules. When reviewing or generating Terraform code, enforce every rule below.
---

# Terraform Style Guide

This file defines the Terraform style conventions for this repository, based on the [HashiCorp Terraform Style Guide](https://developer.hashicorp.com/terraform/language/style). All Terraform code in this workspace MUST conform to these rules. When reviewing or generating Terraform code, enforce every rule below.

---

## Code Formatting

- Indent **two spaces** per nesting level.
- Align `=` signs when multiple single-line arguments appear on consecutive lines at the same nesting level:
  ```hcl
  ami           = "abc123"
  instance_type = "t2.micro"
  ```
- Place all arguments at the **top** of a block, then nested blocks **below**, separated by one blank line.
- Use **empty lines** to separate logical groups of arguments within a block.
- **Meta-arguments first**: list meta-arguments (`count`, `for_each`) at the top of a resource block, separated from other arguments by one blank line.
- **Meta-argument blocks last**: place `lifecycle`, `depends_on` blocks at the bottom, separated from other blocks by one blank line.
  ```hcl
  resource "aws_instance" "example" {
    # meta-argument first
    count = 2

    ami           = "abc123"
    instance_type = "t2.micro"

    network_interface {
      # ...
    }

    # meta-argument block last
    lifecycle {
      create_before_destroy = true
    }
  }
  ```
- Separate **top-level blocks** with exactly one blank line.
- Separate **nested blocks** with blank lines, except when grouping related blocks of the same type.
- Run `terraform fmt` before committing. Use `-recursive` to format subdirectories.

## Code Validation

- Run `terraform validate` before committing to check syntax and internal consistency.

## Comments

- Use `#` for **all** comments (single-line and multi-line). Do NOT use `//` or `/* */`.
- Write self-explanatory code; only add comments when necessary to clarify complexity.

## Resource Naming

- Use a **descriptive noun** for every resource name.
- Separate words with **underscores** (`_`), not hyphens or camelCase.
- Do NOT include the resource type in the resource name (the address already contains it).
- Wrap both resource type and name in **double quotes**.

  **Bad:**
  ```hcl
  resource aws_instance webAPI-aws-instance {...}
  ```
  **Good:**
  ```hcl
  resource "aws_instance" "web_api" {...}
  ```

## Resource Order

- Define **data sources before** the resources that reference them so code "builds on itself".
- Within a resource block, order parameters as follows:
  1. `count` or `for_each` meta-argument
  2. Resource-specific non-block parameters
  3. Resource-specific block parameters
  4. `lifecycle` block (if required)
  5. `depends_on` (if required)

## Variables

- Every variable MUST have a `type` and a `description`.
- Provide a `default` for optional variables.
- Set `sensitive = true` for passwords, private keys, and other secrets.
- Use `validation` blocks only when values have uniquely restrictive requirements.
- Order variable parameters:
  1. `type`
  2. `description`
  3. `default` (optional)
  4. `sensitive` (optional)
  5. `validation` blocks

  ```hcl
  variable "db_disk_size" {
    type        = number
    description = "Disk size for the API database"
    default     = 100
  }

  variable "db_password" {
    type        = string
    description = "Database password"
    sensitive   = true
  }
  ```

## Outputs

- Every output MUST have a `description`.
- Order output parameters:
  1. `description`
  2. `value`
  3. `sensitive` (optional)

  ```hcl
  output "web_public_ip" {
    description = "Public IP of the web instance"
    value       = aws_instance.web.public_ip
  }
  ```

## Local Values

- Use local values **sparingly**; overuse makes code harder to understand.
- If referenced in multiple files, define locals in a `locals.tf` file.
- If specific to one file, define locals at the **top** of that file.
- Use descriptive nouns with underscores for local value names.

## Provider Configuration

- Always include a **default provider configuration** (without `alias`).
- Define **all providers** in the same file.
- If multiple instances of a provider exist, define the **default first**.
- For non-default providers, the `alias` parameter must be the **first** parameter in the block.

## Dynamic Resource Count (`count` / `for_each`)

- Use `count` and `for_each` **sparingly**; they add complexity.
- Use `count` when resources are almost identical.
- Use `for_each` when arguments need distinct values not derivable from an integer.
- A common pattern for conditional resources: `count = var.condition ? 1 : 0`.
- If the effect of a meta-argument is not immediately obvious, add a comment.

## File Naming Conventions

- `main.tf` — resource and data source blocks (or split by logical group as the codebase grows).
- `variables.tf` — all variable blocks, in **alphabetical order**.
- `outputs.tf` — all output blocks, in **alphabetical order**.
- `providers.tf` — all `provider` blocks and configuration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stackitcloud/stackit-landing-zone](https://github.com/stackitcloud/stackit-landing-zone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
