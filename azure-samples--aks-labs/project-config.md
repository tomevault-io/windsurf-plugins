---
trigger: always_on
description: > Scope: Repository-wide standards and writing guidance for hands-on labs, workshops, and teaching content.
---

# AKS Labs - GitHub Copilot Instructions

> Scope: Repository-wide standards and writing guidance for hands-on labs, workshops, and teaching content.
> Module-specific guidance may appear in subfolders (for example, README files under docs/).
> File-specific guidance lives in .github/instructions/*.instructions.md.

---

## Part 1: Repository standards

### Repository overview

This repo hosts a Docusaurus site with hands-on AKS labs and workshop content. Content lives in docs/, blog/, and pages, with supporting assets under docs/\*\*/assets/.

### Primary content types

- Labs and workshops: docs/**.md or docs/**.mdx
- Guides and reference docs: docs/**.md or docs/**.mdx
- Blog posts: blog/\*\*
- React components: src/components/\*\*

### Content principles (workshops and labs)

- Goal-first: Start with the lab outcome and what the learner will build.
- Prerequisites: List required tools, versions, subscriptions, and access.
- Time estimates: Include an estimated duration per section.
- Step clarity: Use numbered steps with imperative verbs.
- Expected results: Include validation steps and sample outputs.
- Recap learning: End each lab with a short summary that restates what the learner achieved and learned.
- Troubleshooting: Add a short troubleshooting section with common errors.
- Safety: Call out cost, cleanup steps, and permissions.

### Docusaurus conventions

- Front matter required for new docs (id, title, sidebar_position when needed).
- Use sentence-style headings.
- Keep sections short and scannable.
- Prefer MDX only when components are required.

### File naming

| Type            | Convention                             | Example                      |
| :-------------- | :------------------------------------- | :--------------------------- |
| Markdown        | kebab-case.md                          | getting-started.md           |
| MDX             | kebab-case.mdx                         | aks-automatic.mdx            |
| React component | PascalCase.tsx                         | LandingPage.tsx              |
| TS utility      | camelCase.ts                           | analytics.ts                 |
| CSS             | kebab-case.css or Component.module.css | custom.css, Index.module.css |
| YAML            | kebab-case.yaml or .yml                | deployment.yaml              |
| Shell scripts   | kebab-case.sh                          | setup-cluster.sh             |

### Code style

- Markdown: CommonMark; keep lines readable and wrap long paragraphs.
- YAML: 2 spaces, no tabs.
- Shell: bash with set -euo pipefail; add prerequisite checks.
- TypeScript: ESLint defaults; functional React components with hooks.
- Code samples: Use fenced code blocks with triple backticks and a language identifier, such as bash, python, or typescript.
- Inline code: Use single backticks only for code terms embedded in normal sentences (for example, `kubectl`).

### Links and images

- Use descriptive link text (avoid “click here”).
- Prefer relative links within the repo.
- Provide alt text for all images.
- Store images under the nearest docs/\*\*/assets/ folder.

### Kubernetes and Azure examples

- Never use :latest images.
- Always include resource requests and limits in manifests.
- Include labels and namespaces where relevant.
- Avoid embedding secrets; use placeholders and Key Vault references.

### Security and privacy

- Never commit secrets, tokens, or credentials.
- Avoid customer-specific data or personal email addresses.
- Use generic sample values (example-resource-group, example-cluster).

### Build and development

This is a Docusaurus site. Typical commands:

- npm install
- npm start
- npm run build
- npm run typecheck

### Git workflow

- Use Conventional Commits for messages and PR titles.
- Keep PRs focused and include validation notes for doc changes.

### Azure tooling rule

- @azure Rule - Use Azure Best Practices: When generating code for Azure, running terminal commands for Azure, or performing operations related to Azure, invoke your get_azure_best_practices tool if available.
- Agents may call MCP clients when needed and when allowed by repo rules.

---

## Part 2: Lab writing guide (summary)

### Voice and tone

- Crisp, clear, and friendly.
- Use second person and imperative verbs.
- Prefer input-neutral verbs (select, enter, choose).

### Lab structure

1. Overview
2. Objectives
3. Prerequisites
4. Architecture or concept brief
5. Steps
6. Validation
7. Summary (recap what the learner learned)
8. Cleanup
9. Troubleshooting

### Steps formatting

- Use numbered lists for procedures.
- Keep steps short and focused.
- Use bold for UI elements.
- Provide copy-pasteable commands.

### Troubleshooting guidance

- Include error message snippets when possible.
- Provide the most likely fix first.
- Link to official docs for deeper dives.

### Accessibility

- Provide alt text for images.
- Avoid directional-only guidance (for example, “click the button on the right”).
- Use descriptive link text.

---

## Part 3: Terminology reminders

- Azure Kubernetes Service (AKS) on first mention, then AKS.
- kubectl and kubeconfig are lowercase.
- Cluster, node, pod, and namespace are lowercase as common nouns.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/aks-labs](https://github.com/Azure-Samples/aks-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
