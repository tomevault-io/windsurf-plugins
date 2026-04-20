---
trigger: always_on
description: > VS Code Copilot-specific orchestration instructions.
---

# APEX - Copilot Instructions

> VS Code Copilot-specific orchestration instructions.
> For general project conventions, build commands, and code style, see the root `AGENTS.md`.

## Quick Start

1. Enable subagents: `"github.copilot.chat": { "customAgentInSubagent": { "enabled": true } }`
2. Open Chat (`Ctrl+Shift+I`) → Select **Orchestrator** → Describe your project
3. The Orchestrator guides you through all steps with approval gates

## Multi-Step Workflow

| Step | Agent                                                                      | Output                                                                                       | Review                           | Gate       |
| ---- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------------------- | ---------- |
| 1    | Requirements                                                               | `01-requirements.md`                                                                         | 1×                               | Approval   |
| 2    | Architect                                                                  | `02-architecture-assessment.md` + cost estimate                                              | 1× + 1 cost (opt-in: multi-pass) | Approval   |
| 3    | Design (opt)                                                               | `03-des-*.{py,png,md}` diagrams and ADRs                                                     | —                                | —          |
| 3.5  | Governance (`04g-Governance`)                                              | `04-governance-constraints.md/.json`                                                         | 1×                               | Approval   |
| 4    | IaC Plan (`05-IaC Planner`)                                                | `04-implementation-plan.md` + `04-dependency-diagram.py/.png` + `04-runtime-diagram.py/.png` | opt-in (default: skip)           | Approval   |
| 5    | IaC Code (Bicep: `06b-Bicep CodeGen` / Terraform: `06t-Terraform CodeGen`) | `infra/bicep/{project}/` or `infra/terraform/{project}/`                                     | opt-in (default: skip)           | Validation |
| 6    | Deploy (Bicep: `07b-Bicep Deploy` / Terraform: `07t-Terraform Deploy`)     | `06-deployment-summary.md`                                                                   | —                                | Approval   |
| 7    | As-Built                                                                   | `07-*.md` documentation suite                                                                | —                                | —          |
| Post | Lessons (Orchestrator)                                                     | `09-lessons-learned.json/.md`                                                                | —                                | —          |

All outputs → `agent-output/{project}/`. Context flows via artifact files + handoffs.
Review column = adversarial passes by challenger subagents; 1-pass default, multi-pass opt-in
Single-pass comprehensive review is the default; multi-pass rotating lens is opt-in for complex projects.
Reviews target AI-generated creative decisions only (Steps 1, 2, 3.5, 4, 5).

## Skills (Auto-Invoked by Agents)

| Skill                         | Purpose                                                                                           |
| ----------------------------- | ------------------------------------------------------------------------------------------------- |
| `appinsights-instrumentation` | Application Insights telemetry patterns, SDK setup, APM best practices                            |
| `azure-adr`                   | Architecture Decision Records                                                                     |
| `azure-ai`                    | Azure AI services (Search, Speech, OpenAI, Document Intelligence)                                 |
| `azure-aigateway`             | Azure API Management as AI Gateway for models, MCP tools, agents                                  |
| `azure-artifacts`             | Template H2 structures, styling, generation rules                                                 |
| `azure-bicep-patterns`        | Reusable Bicep patterns (hub-spoke, PE, diagnostics)                                              |
| `azure-cloud-migrate`         | Cross-cloud migration assessment and code conversion to Azure                                     |
| `azure-compliance`            | Compliance scanning, security auditing, Key Vault expiration monitoring                           |
| `azure-compute`               | VM size recommendations, VMSS, configuration guidance                                             |
| `azure-cost-optimization`     | Cost savings analysis, utilization metrics, optimization recommendations                          |
| `azure-defaults`              | Regions, tags, naming, AVM, security, governance, pricing                                         |
| `azure-deploy`                | Execute Azure deployments (azd up, terraform apply, az deployment)                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonathan-vella/azure-agentic-infraops](https://github.com/jonathan-vella/azure-agentic-infraops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
