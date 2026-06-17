---
trigger: always_on
description: Interactive CLI tool for provisioning ibl.ai platform infrastructure on AWS. Built with Python, Typer, Rich, and questionary. Uses Terraform for resource management.
---

# iblai-infra — Claude Development Guide

## Overview

Interactive CLI tool for provisioning ibl.ai platform infrastructure on AWS. Built with Python, Typer, Rich, and questionary. Uses Terraform for resource management.

**Command pattern:** `iblai infra <command>`

## Project Structure

```
iblai-infra/
├── pyproject.toml                          # uv/hatch config, dynamic version, entry point: iblai = iblai_infra.cli:app
├── src/iblai_infra/
│   ├── __init__.py                         # __version__ = "1.2.3"
│   ├── __main__.py                         # python -m iblai_infra support
│   ├── cli.py                              # Typer app: root `iblai` + `infra` subgroup + `ingress` subgroup + landing screen menu
│   ├── app.py                              # Wizard orchestrator (5-step flow)
│   ├── models.py                           # Pydantic models — contract between wizard & Terraform, ingress registry
│   ├── ui.py                               # Rich console, ibl.ai branding, progress helpers
│   ├── prompts/
│   │   ├── credentials.py                  # Step 1: AWS auth (profile/keys/env), show_step param
│   │   ├── infrastructure.py               # Steps 2-3: project, compute, network, SSH
│   │   ├── dns_certs.py                    # Step 4: domain, Route53, certificates
│   │   └── review.py                       # Step 5: summary + confirm
│   ├── providers/
│   │   └── aws.py                          # AWS helpers: STS validation, Route53, key pairs, IP detect, permission checks
│   ├── terraform/
│   │   ├── runner.py                       # TerraformRunner: setup/init/plan/apply/destroy with JSON streaming
│   │   ├── state.py                        # ProjectState + session + ingress registry + lock backends (~/.iblai-infra/)
│   │   ├── templates/aws/single-server/
│   │   │   ├── main.tf                     # VPC, subnets, ALB, EC2, S3, certs, DNS
│   │   │   ├── variables.tf                # All Terraform variables
│   │   │   ├── outputs.tf                  # IPs, ALB DNS, S3 buckets, SSH command
│   │   │   └── user_data.sh               # Docker, AWS CLI, UFW, systemd setup
│   │   └── templates/aws/multi-server/
│   │       ├── main.tf                     # VPC (4 subnet tiers), NAT, ALB, N×app EC2, 1×services EC2, optional RDS/Redis, EFS, S3, certs, DNS
│   │       ├── variables.tf                # All multi-server variables (compute, managed services, secrets marked sensitive)
│   │       ├── outputs.tf                  # App server IPs (list), services IP, RDS/Redis endpoints, backward-compat singular outputs
│   │       ├── user_data_app.sh            # App server bootstrap (Docker, AWS CLI, NFS, UFW)
│   │       └── user_data_services.sh       # Services server bootstrap (Docker, AWS CLI, internal UFW)
│   └── ansible/
│       ├── __init__.py
│       ├── runner.py                       # AnsibleRunner: preflight, SSH test, inventory, playbook execution
│       └── templates/single-server/        # Ansible playbook + roles (docker, awscli, python, ibl_cli_ops, ibl_platform, ibl_dm, ibl_edx, ibl_spa, final_steps)
├── tests/
│   ├── conftest.py                         # Shared fixtures (aws_credentials, infra_config, project_state, workspace_root)
│   ├── test_models.py                      # Pydantic model validation, all enum combos, edge cases
│   ├── test_state.py                       # State persistence, session save/load/clear
│   ├── test_cli.py                         # CLI commands, _run_setup branches, _resolve_credentials
│   ├── test_app.py                         # Wizard orchestrator (_show_workspace, _show_results, _offer_setup)
│   ├── test_ui.py                          # Rich UI helpers, banner, step_header, summary_panel
│   ├── providers/test_aws.py               # AWS helpers: sessions, credentials, hosted zones, key pairs, permissions
│   ├── terraform/test_runner.py            # TerraformRunner: tfvars generation, event parsing, labels
│   ├── ansible/test_runner.py              # AnsibleRunner: role extraction, failure detection, preflight, SSH test
│   └── prompts/
│       ├── test_validators.py              # IP, CIDR, domain validation
│       ├── test_review.py                  # Review prompt with all SSH × cert × env combinations
│       └── test_setup.py                   # Setup prompt flow, SSH key resolution, key permissions
```

## Architecture

### CLI Structure (Typer)

- **Root app** (`iblai`): `--version`, `--help`
- **Subgroup** (`iblai infra`): `provision`, `retry <name>`, `setup [name]`, `resetup <name>`, `launch`, `destroy <name>`, `status <name>`, `list`, `permissions`, `auth`
- **Nested subgroup** (`iblai infra ingress`): `add`, `remove`, `list`, `configure`, `status`, `claim`, `release`
- Running `iblai infra` with no arguments shows branded landing screen with interactive arrow-key menu
- The landing screen menu uses `questionary.select()` to dispatch to commands directly
- When launching provision from the menu, calls `run_provision_wizard(show_banner=False)` to avoid double banner
- Entry point in `pyproject.toml`: `iblai = "iblai_infra.cli:app"`

### Session Persistence


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iblai/infra-cli](https://github.com/iblai/infra-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
