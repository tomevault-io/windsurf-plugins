---
trigger: always_on
description: SDAF is Microsoft's open-source framework for deploying and configuring SAP
---

# Copilot Instructions — SAP Deployment Automation Framework (SDAF)

## Project overview

SDAF is Microsoft's open-source framework for deploying and configuring SAP
landscapes on Azure. It provisions infrastructure with **Terraform** and
configures the OS/database/SAP application layers with **Ansible**, driven
from Azure DevOps/GitHub Actions pipelines, from a **.NET web app**
(`Webapp/`) that manages deployment configuration, or via **localized script
execution** — running the `deploy/scripts/` entry points (e.g.
`install_deployer.sh`, `installer.sh`, `deploy_controlplane.sh`) directly from
a shell, without any pipeline or the Webapp involved. Consumers are SAP
Basis/infra teams standing up dev/QA/prod SAP systems (HANA and AnyDB) across
any Azure region. See `docs/repository_overview.md` for the full architecture,
module map, and data flow.

The Webapp subdirectory has its own
`Webapp/.github/copilot-instructions.md` for Azure-tool usage rules — those
apply only when working under `Webapp/`; this file covers the rest of the repo
and general cross-cutting rules.

## Strict requirement: follow official best practices

For every technology in this repo, changes must follow the **official best
practices from that technology's own documentation** — not conventions
inferred from blog posts, Stack Overflow, or general training knowledge.
Concretely:

- **Terraform / HashiCorp**: follow the official
  [Terraform documentation](https://developer.hashicorp.com/terraform/docs)
  and [Terraform language style guide](https://developer.hashicorp.com/terraform/language/style)
  for module structure, resource design, and testing (`terraform test`)
  patterns, reconciled with this repo's documented conventions below.
- **AzureRM / AzAPI providers**: follow the
  [AzureRM provider docs](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
  and [AzAPI provider docs](https://registry.terraform.io/providers/Azure/azapi/latest/docs)
  for resource arguments and upgrade guidance.
- **Ansible**: follow the official
  [Ansible best practices guide](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)
  and [ansible-lint documentation](https://ansible.readthedocs.io/projects/lint/).
- **Python**: follow [PEP 8](https://peps.python.org/pep-0008/) and the
  official [Black documentation](https://black.readthedocs.io/) for
  formatting, and official [pytest](https://docs.pytest.org/) /
  [pytest-cov](https://pytest-cov.readthedocs.io/) docs for test/coverage
  conventions.
- **.NET / ASP.NET Core**: follow official
  [Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/) guidance.
- **Azure services**: follow official Azure best practices — when Azure MCP
  tools are available (see `Webapp/.github/copilot-instructions.md`), invoke
  `azmcp_bestpractices_get` before generating Azure-related code.

When a claim about "what the docs say" matters, verify it by fetching and
reading the actual current documentation page rather than relying on memory
or a search-result snippet — do not assert a documented value or practice
without a direct quote from the source.

## Tech stack

- **Terraform** (`deploy/terraform/`) — Azure infrastructure. Providers:
  `azurerm`, `azapi`. Pinned versions live in each module's `providers.tf`;
  check that file rather than assuming a version here, and update it (not
  this doc) when bumping providers.
- **Ansible** (`deploy/ansible/`) — OS, database, and SAP application
  configuration, driven by numbered playbooks.
- **ASP.NET Core / .NET 9.0** (`Webapp/`) — configuration UI backed by Azure
  Table Storage.
- **Bash / PowerShell / Python** (`deploy/scripts/`) — deployment orchestration
  scripts invoked by pipelines or run manually.
- **Azure DevOps pipelines** (`deploy/pipelines/`) and **GitHub Actions**
  (`.github/workflows/`) for CI/CD and linting/testing.

## Critical rule: never run `terraform fmt`

This repo intentionally uses **wide, column-aligned `=` signs** in all `.tf`
files (not the default tight alignment `terraform fmt` produces). Do **not**
run `terraform fmt` or any auto-formatter on Terraform files — it will
destroy the existing alignment and produce massive unrelated diffs. When
editing `.tf` files by hand, match the surrounding alignment style.

## Terraform conventions (`deploy/terraform/`)

- Root (entry-point) modules live under `run/{sap_deployer,sap_library,
  sap_landscape,sap_system}` and `bootstrap/{sap_deployer,sap_library}`.
  Reusable child modules live under `terraform-units/modules/` and are
  referenced by relative path from root modules.
- Each root module follows a consistent file layout: `module.tf`,
  `providers.tf`, `backend.tf`, `variables_global.tf`, `variables_local.tf`,
  `tfvar_variables.tf`, `transform.tf`, `output.tf`, `imports.tf`. Put
  complex/derived logic in `variables_local.tf`, not inline in resources.
- Use underscores, not hyphens, in Terraform identifiers (hyphens are fine
  inside actual Azure resource names/strings).
- Known provider aliases: `azurerm.main`, `azurerm.dnsmanagement`,
  `azurerm.privatelinkdnsmanagement`, `azapi.restapi`.
- Validate changes with (no state, no fmt):
  ```bash
  cd deploy/terraform/run/<module>   # or deploy/terraform/bootstrap/<module>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/sap-automation](https://github.com/Azure/sap-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
