---
trigger: always_on
description: - @azure Rule - Use Azure Best Practices: When generating code for Azure, running terminal commands for Azure, or performing operations related to Azure, invoke your `get_azure_best_practices` tool if available.
---


- @azure Rule - Use Azure Best Practices: When generating code for Azure, running terminal commands for Azure, or performing operations related to Azure, invoke your `get_azure_best_practices` tool if available.

# Azure DevOps Product Context — Override Instructions

When working in this repository, apply the following Azure DevOps–specific
overrides to any doc-kit agent, skill, or prompt. These rules take precedence
over Foundry-specific defaults shipped with doc-kit.

---

## 1. Product Identity

| Default (Foundry)                  | Override (Azure DevOps)                                           |
|------------------------------------|-------------------------------------------------------------------|
| Product name: "Azure AI Foundry"   | **Azure DevOps**                                                  |
| Docs base URL                      | `https://learn.microsoft.com/azure/devops/`                       |
| Release-notes URL                  | `https://learn.microsoft.com/azure/devops/release-notes/`         |
| REST API reference                 | `https://learn.microsoft.com/rest/api/azure/devops/`              |
| Feedback mechanism                 | Azure DevOps Developer Community (`developercommunity.visualstudio.com`) |
| Product hub                        | `https://azure.microsoft.com/products/devops/`                    |
| `ms.service`                       | `azure-devops`                                                    |
| `ms.subservice` values             | `azure-boards`, `azure-repos`, `azure-pipelines`, `azure-artifacts`, `azure-test-plans` |
| Moniker / version selector         | `azure-devops` (cloud, primary), `azure-devops-2022` (on-premises) |

## 2. Repository & Content Structure

| Aspect               | Value                                                              |
|----------------------|--------------------------------------------------------------------|
| Docs repo            | `MicrosoftDocs/azure-devops-docs`                                  |
| Docs repo path       | `docs/`                                                            |
| TOC root             | `docs/toc.yml`                                                     |
| Content areas        | `docs/artifacts/`, `docs/boards/`, `docs/pipelines/`, `docs/repos/`, `docs/test/`, `docs/organizations/`, `docs/cli/` |
| Media folder         | `media/` subdirectory colocated with each article                  |
| Includes folder      | `docs/includes/`                                                   |
| Release notes        | `release-notes/` (organized by year and sprint)                    |
| What's new docs      | `release-notes/docswhatsnew/`                                      |

## 3. Versioning & Moniker Rules

- Azure DevOps docs use **moniker-based versioning**: `azure-devops`, `azure-devops-2022`, `azure-devops-2020`.
- The primary target is **`azure-devops`** (Azure DevOps Services, cloud) — use by default.
- Include `azure-devops-2022` content when covering on-premises scenarios.
- Include `azure-devops-2020` only if explicitly requested.
- **DISCARD** `tfs-2018` content unless the user explicitly requests legacy coverage.
- Content outside any moniker block is shared across all versions — always include it.
- Every `::: moniker range="..."` must have a matching `::: moniker-end`.
- Do NOT use `monikerRange` YAML front matter for articles with content spanning multiple versions.
- When procedures differ by version, use moniker zones to isolate differing sections. Prefer repeating an entire section rather than splitting individual list items.

## 4. Metadata Defaults

Every Markdown article must include this YAML front matter (adjust values per article):

```yaml
---
title: "{Article Title}"
description: "{140-character max description}"
ms.date: {MM/DD/YYYY}
ms.topic: {how-to | conceptual | overview | reference | quickstart | tutorial}
ms.service: azure-devops
ms.subservice: {azure-boards | azure-repos | azure-pipelines | azure-artifacts | azure-test-plans}
author: {GitHub-username}
ms.author: {Microsoft-alias}
---
```

`title`, `description`, `ms.date`, and `ms.topic` are always required. `author` and `ms.author` are optional — default values are set in `docfx.json` and can be omitted unless overriding the default.

## 5. Key Service Areas

| Service                          | Path prefix              | Notes                                                    |
|----------------------------------|--------------------------|----------------------------------------------------------|
| Azure Boards                     | `docs/boards/`           | Work items, backlogs, sprints, Kanban, queries           |
| Azure Repos                      | `docs/repos/`            | Git, TFVC, pull requests, branch policies                |
| Azure Pipelines                  | `docs/pipelines/`        | YAML pipelines, classic pipelines, agents, tasks         |
| Azure Artifacts                  | `docs/artifacts/`        | Package feeds, NuGet, npm, Maven, Cargo, Python          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/azure-devops-docs](https://github.com/MicrosoftDocs/azure-devops-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
