---
trigger: always_on
description: > Steering document for Claude Code when working within the `documentation/` directory.
---

# CLAUDE.md — VAMS Documentation

> Steering document for Claude Code when working within the `documentation/` directory.
> Auto-loaded when the working context is within `documentation/`.

---

## Project Overview

VAMS documentation is built with **Docusaurus** (React-based static site generator) and lives in `documentation/docusaurus-site/`. The source Markdown files are in `documentation/docusaurus-site/docs/`.

-   **Docusaurus config**: `documentation/docusaurus-site/docusaurus.config.ts`
-   **Sidebar config**: `documentation/docusaurus-site/sidebars.ts`
-   **Source pages**: `documentation/docusaurus-site/docs/` (78 Markdown files)
-   **Custom CSS**: `documentation/docusaurus-site/src/css/custom.css`
-   **Static images**: `documentation/docusaurus-site/static/img/`
-   **Architecture diagrams**: `documentation/diagrams/` (source PNGs, JPEGs, draw.io files)
-   **OpenAPI spec**: `documentation/VAMS_API.yaml`
-   **Build output**: `documentation/docusaurus-site/build/`

---

## Documentation Structure

```
docusaurus-site/docs/
├── index.md                    # Landing page
├── overview/                   # Solution overview, benefits, use cases, features, costs
├── concepts/                   # Core concepts: databases, assets, files, pipelines, metadata, permissions
├── architecture/               # Architecture overview, details, AWS resources, security, networking, data model
├── deployment/                 # Prerequisites, deploy, config reference, external S3, update, uninstall
├── user-guide/                 # Getting started, web UI, upload tutorial, asset mgmt, search, metadata, permissions
├── cli/                        # CLI getting started, installation, command reference, automation
├── pipelines/                  # Pipeline overview + 10 individual pipeline docs + custom pipeline guide
├── developer/                  # Dev setup, backend, frontend, CDK, viewer plugins, audit logging
├── api/                        # API overview, auth, assets, files, metadata, search, pipelines, workflows, tags
├── troubleshooting/            # Common issues, known limitations, FAQ
└── additional/                 # Quotas, partner integrations, viewer plugins ref, notices, revisions
```

---

## Writing Style

Follow AWS documentation standards:

1. **Tone**: Professional, formal, solution-focused
2. **AWS service names**: Always fully qualified (e.g., "Amazon DynamoDB" not "DynamoDB")
3. **Paragraphs**: 2-4 sentences, concise
4. **Headings**: `##` for main sections, `###` for subsections
5. **Admonitions**: Use Docusaurus admonition syntax:
    - `:::note` — General information
    - `:::tip` — Helpful suggestions
    - `:::warning` — Caution needed
    - `:::danger` — Critical warnings
    - `:::info` — Supplementary information
    - With title: `:::warning[Custom Title]`
6. **Code blocks**: Always include language tags (`bash, `python, `typescript, `json)
7. **Tables**: For comparisons, feature lists, field references
8. **Mermaid diagrams**: Use ```mermaid code blocks (supported via @docusaurus/theme-mermaid)
9. **Cross-references**: Use relative links `[Page Title](../section/page.md)`
10. **Images**: Reference from `/img/` (maps to `static/img/`)
11. **Curly braces**: Escape `{variable}` as `\{variable\}` outside code blocks (MDX parses them as JSX)
12. **Never reference other AWS solutions** by name — VAMS documentation is standalone
13. **Never hardcode version numbers** — reference source of truth (`config.ts`)

---

## Build Commands

```bash
# Install dependencies
cd documentation/docusaurus-site
npm install

# Local preview (live reload)
npm run start
# Opens http://localhost:3000

# Build static site
npm run build
# Output in documentation/docusaurus-site/build/
```

---

## When to Update Documentation

| Change Type             | Documentation to Update                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| New API endpoint        | `api/` relevant page, `VAMS_API.yaml`, `cli/command-reference.md` (if CLI updated)             |
| New config option       | `deployment/configuration-reference.md`                                                        |
| New pipeline            | `pipelines/` new page + `pipelines/overview.md` table + `overview/features.md` + `sidebars.ts` |
| New viewer plugin       | `developer/viewer-plugins.md`, `additional/viewer-plugins.md`, `overview/features.md`          |
| New DynamoDB table      | `architecture/aws-resources.md`, `architecture/data-model.md`                                  |
| Permission model change | `concepts/permissions-model.md`, `user-guide/permissions.md`                                   |
| New CLI command         | `cli/command-reference.md`, `cli/automation.md` (if new patterns)                              |
| UI navigation change    | `user-guide/web-interface.md`, `user-guide/getting-started.md`                                 |
| Breaking change         | `additional/revisions.md`, `deployment/update-the-solution.md`                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/visual-asset-management-system](https://github.com/awslabs/visual-asset-management-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
