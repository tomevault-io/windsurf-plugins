---
trigger: always_on
description: - @azure Rule - Use Azure Best Practices: When generating code for Azure, running terminal commands for Azure, or performing operations related to Azure, invoke your `get_azure_best_practices` tool if available.
---


- @azure Rule - Use Azure Best Practices: When generating code for Azure, running terminal commands for Azure, or performing operations related to Azure, invoke your `get_azure_best_practices` tool if available.

# Azure DevOps YAML Schema & Task Reference — Override Instructions

When working in this repository, apply the following overrides to any doc-kit
agent, skill, or prompt. These rules take precedence over Foundry-specific
defaults shipped with doc-kit.

---

## 1. Product Identity

| Default (Foundry)                  | Override (Azure DevOps)                                           |
|------------------------------------|-------------------------------------------------------------------|
| Product name: "Azure AI Foundry"   | **Azure Pipelines** (YAML schema), **Azure DevOps** (tasks)      |
| Docs base URL                      | `https://learn.microsoft.com/azure/devops/pipelines/yaml-schema/` (schema), `https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/` (tasks) |
| REST API reference                 | `https://learn.microsoft.com/rest/api/azure/devops/`              |
| Feedback mechanism                 | Azure DevOps Developer Community (`developercommunity.visualstudio.com`) |
| Tasks source repo                  | `https://github.com/microsoft/azure-pipelines-tasks`              |
| YAML validation schema source      | `https://github.com/Microsoft/azure-pipelines-vscode`  There is additional information about how developers release the code at https://github.com/microsoft/azure-pipelines-vscode/blob/main/RELEASE.md#bumping-service-schema           |
| `ms.service`                       | `azure-devops-pipelines`                                          |
| `ms.topic`                         | `reference` (set globally in `docfx.json`)                        |

## 2. Repository & Content Structure

| Aspect               | Value                                                              |
|----------------------|--------------------------------------------------------------------|
| Docs repo            | `MicrosoftDocs/azure-devops-yaml-schema` (this repo)               |
| YAML schema articles | `content/` — one `.md` file per YAML schema definition             |
| Task reference articles | `task-reference/` — one `.md` file per task version             |
| TOC                  | `content/TOC.yml`                                                  |
| Breadcrumb           | `content/breadcrumb/toc.yml`                                       |
| Includes             | `content/includes/`                                                |
| docfx config         | `content/docfx.json`                                               |
| Build output dest    | `azure-devops-yaml-schema`                                         |
| Auto-publish         | `auto-publish.yml` (pipeline definition at repo root)              |

## 3. Auto-Generated vs. Editable Content

**Critical:** Most article content in this repo is auto-generated from task
metadata and YAML schema definitions. Unless a change is being made by the
repository's approved automation workflow described below, contributions must
only touch **editable content regions** or article metadata values such as
`ms.date`.

These instructions are for doc-kit agents, skills, and prompts. The exception
below is intended only for automation-generated pull requests that carry the
label `task-automation` or `yaml-schema-automation`; it does not change the
default guidance for normal contributor-authored PRs.

Pull requests with the label `task-automation` or `yaml-schema-automation`
are produced by the tooling that generates the task reference documentation,
so those automation PRs may create new articles or update non-editable
content regions.

### Editable content tags

Editable regions are delimited by HTML comments:

```html
<!-- :::editable-content name="{section}"::: -->
Your editable text here.
<!-- :::editable-content-end::: -->
```

Valid `name` values include: `description`, `helpMarkDown`, `remarks`,
`examples`, `seeAlso`.

### Rules

- **DO** edit text inside `<!-- :::editable-content ...::: -->` … `<!-- :::editable-content-end::: -->` tags.
- **DO NOT** edit any text outside of `editable-content` tags — those sections are overwritten by the automated generation process.
- **DO NOT** create new task articles or new YAML schema definition articles manually — they are auto-generated when Azure DevOps sprints deploy.
- **DO NOT** modify auto-generated YAML syntax blocks, input tables, allowed values, defaults, or aliases.
- Monikers may appear inside `editable-content` tags for sections like remarks, examples, and see also.

### Editable sections in task articles

| Section              | Editable? | Notes                                     |
|---------------------|-----------|-------------------------------------------|
| Task description     | Yes       | Inside `editable-content` tags            |
| Input descriptions   | Yes       | Inside `editable-content` tags            |
| Output descriptions  | Yes       | Inside `editable-content` tags            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/azure-devops-yaml-schema](https://github.com/MicrosoftDocs/azure-devops-yaml-schema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
