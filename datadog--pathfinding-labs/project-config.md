---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Pathfinding Labs** is a modular platform for deploying intentionally vulnerable AWS configurations to validate Cloud Security Posture Management (CSPM) tools and train security teams. Think of it as "Stratus Red Team for CSPM validation."

### Purpose
- **Validate CSPM Detection**: Does your security tooling detect all vulnerable configurations?
- **Train Security Teams**: Provide hands-on experience with real attack scenarios
- **Answer Critical Questions**: Who has access to my most sensitive S3 bucket? If an attacker compromises one employee, what's the likelihood they reach critical resources?
- **Measure Coverage**: Identify gaps in security monitoring
- **Practice IAM Exploitation**: Sharpen privilege escalation skills with real scenarios
- **Build Attack Chains**: Learn complex multi-hop and cross-account techniques

### Key Features
- **Single-Account Support**: Works with just ONE AWS account (prod) for most scenarios
- **Multi-Account Support**: Optional dev/ops accounts for cross-account scenarios
- **Modular Architecture**: Enable/disable individual scenarios via boolean flags
- **Granular Control**: Each scenario is independently deployable
- **100+ Scenarios Available**: Run `find modules/scenarios -name scenario.yaml | wc -l` for the current count. Full catalog with descriptions at [pathfinding.cloud/labs](https://pathfinding.cloud/labs)

## Architecture

### Directory Structure

```
pathfinding-labs/
├── cmd/plabs/                # Go CLI entry point
│   └── main.go
├── internal/                 # Go CLI internal packages
│   ├── cmd/                  # Cobra commands
│   ├── tui/                  # Bubble Tea TUI components
│   ├── config/               # Configuration management
│   ├── scenarios/            # Scenario discovery
│   ├── terraform/            # Terraform orchestration
│   ├── repo/                 # Repository management
│   └── demo/                 # Demo script execution
│
├── modules/
│   ├── environments/         # Base infrastructure (always deployed)
│   │   ├── prod/             # Production environment base resources
│   │   ├── dev/              # Development environment base resources (optional)
│   │   └── operations/       # Operations environment base resources (optional)
│   │
│   └── scenarios/            # Attack scenarios (opt-in via boolean flags)
│       ├── single-account/       # Single-account scenarios (PRIMARY)
│       │   ├── privesc-self-escalation/
│       │   │   ├── to-admin/    # Principal modifies itself to gain admin
│       │   │   └── to-bucket/   # Principal modifies itself for S3 access
│       │   ├── privesc-one-hop/
│       │   │   ├── to-admin/    # Single principal traversal to admin
│       │   │   └── to-bucket/   # Single principal traversal to S3 access
│       │   ├── privesc-multi-hop/
│       │   │   ├── to-admin/    # Multiple principal traversals to admin
│       │   │   └── to-bucket/   # Multiple principal traversals to S3 access
│       │   ├── cspm-misconfig/  # Single-condition security misconfigurations
│       │   └── cspm-toxic-combo/ # Multiple compounding misconfigurations
│       ├── tool-testing/         # Edge cases for testing detection engines
│       ├── ctf/                  # Capture-the-flag challenges (no demo scripts)
│       ├── attack-simulation/    # Recreations of real-world cloud breaches
│       ├── end-of-life-privesc-paths/ # Deprecated paths (AWS services retired/changed)
│       └── cross-account/
│           ├── dev-to-prod/     # Dev → Prod attack paths
│           │   ├── one-hop/
│           │   └── multi-hop/
│           └── ops-to-prod/     # Ops → Prod attack paths
│               └── one-hop/
│
├── main.tf                   # Root module with conditional instantiation
├── variables.tf              # Boolean flags for each scenario
├── outputs.tf                # Credential outputs for testing
├── terraform.tfvars          # Your configuration (gitignored)
└── go.mod / go.sum           # Go module dependencies
```

### Scenario Taxonomy

**One-Hop Privilege Escalation**
- Single principal traversal (regardless of action complexity)
- Pattern: `Principal A → [IAM actions] → Principal B (admin/bucket access)`
- Examples: `iam:PutRolePolicy`, `iam:PassRole + lambda:CreateFunction + lambda:InvokeFunction`
- Both role-based and user-based scenarios
- Deploy to: **prod account only**

**Multi-Hop Privilege Escalation**
- Multiple principal traversals (chaining 2+ one-hop paths)
- Pattern: `Principal A → Principal B → Principal C → Target`
- Examples: Role chains, multiple privilege escalation steps
- Deploy to: **prod account only** (for single-account) or **cross-account**

**CSPM Misconfig**
- Single-condition security misconfigurations
- Examples: EC2 with admin role, S3 bucket publicly accessible
- Focus on CSPM detection of individual misconfigurations
- Deploy to: **prod account only**

**CSPM Toxic Combinations**
- Multiple compounding security misconfigurations
- Examples: Public Lambda + Admin Role, Public S3 + Sensitive Data
- Focus on CSPM detection of combined risk scenarios

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataDog/pathfinding-labs](https://github.com/DataDog/pathfinding-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
