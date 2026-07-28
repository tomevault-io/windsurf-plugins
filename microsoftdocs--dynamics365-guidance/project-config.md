---
trigger: always_on
description: This is a Microsoft Learn documentation repository for **Dynamics 365 guidance content**, published via the Open Publishing System (OPS/DocFX). It contains implementation guidance, business process catalog articles, reference architectures, and design patterns for Dynamics 365 apps and services.
---

# Copilot Instructions for dynamics365-guidance-pr

## Repository Overview

This is a Microsoft Learn documentation repository for **Dynamics 365 guidance content**, published via the Open Publishing System (OPS/DocFX). It contains implementation guidance, business process catalog articles, reference architectures, and design patterns for Dynamics 365 apps and services.

The public-facing repo is `MicrosoftDocs/dynamics365-guidance`; this is the private working repo (`-pr` suffix).

## Architecture

- **`guidance/`** — The docset root (defined in `.openpublishing.publish.config.json`)
  - `business-processes/` — End-to-end business process catalog (the largest content area)
  - `implementation-guide/` — Success by Design framework guidance
  - `reference-architectures/` — Architecture patterns with Azure and Dynamics 365
  - `patterns/` — Reusable implementation and business process patterns
  - `agent-templates/` — Copilot Studio / AI agent scenario templates
  - `develop/` — UI/UX design guidance for Power Apps and Dynamics 365 Sales
  - `migrate/` — Migration/upgrade guidance (AX, CRM, GP, NAV, AIM)
  - `organizational-strategy/` — Organizational design and strategy content
  - `roles/` — Audience/role definitions and career guidance
  - `techtalks/` — TechTalk video summaries and landing pages
  - `resources/` — Implementation tools and sample solutions
  - `fasttrack/` — FastTrack-specific content
  - `isv/` — Guidance for independent software vendors
  - `includes/` — Reusable Markdown snippets (excluded from build output)
  - `media/` — Images (PNG, JPG, SVG)
- **`shared-content/`** — Pulled from `MicrosoftDocs/powerapps-docs-pr` as a dependent repository (referenced via `~/../shared-content/`)
- **`.openpublishing.redirection.json`** — URL redirections (add entries here when renaming/moving articles)

## Content Conventions

### Article metadata (YAML front matter)

Every `.md` file must start with YAML metadata:

```yaml
---
title: Short descriptive title
description: A 1-2 sentence summary for SEO (max ~160 chars).
#customer intent: As a <role>, I want to <goal> so that I can <outcome>.
author: githubusername
ms.author: microsoftalias
ms.topic: concept-article  # or reference-architecture, overview, how-to-guide, hub-page, landing-page
ms.date: MM/DD/YYYY
ms.custom: bap-template  # Required for most articles
---
```

- `ms.date` uses **MM/DD/YYYY** format and must reflect the last meaningful update.
- The `#Required; don't change.` comment appears after `bap-template` in many files — preserve it.
- `#customer intent` is a commented-out YAML field used in newer articles. Preserve it when present.
- Reference architecture articles use `ms.topic: reference-architecture`.

### Headings and structure

- The first line after front matter must be an H1 (`#`) matching the `title` metadata.
- Use sentence case for all headings.
- Two trailing spaces at end of paragraphs are used intentionally for line breaks in some articles.

### Include files

- Local includes: `[!INCLUDE [name](../includes/filename.md)]`
- Cross-repo includes: `[!INCLUDE [name](~/../shared-content/shared/guidance-includes/filename.md)]`

### Cross-references and links

- Internal links use relative paths: `[link text](../implementation-guide/article.md)`
- Links to other Microsoft Learn docsets use absolute paths without domain: `[text](/dynamics365/guidance/business-processes/)`
- TOC cross-references: `href: /dynamics365/path?toc=/dynamics365/guidance/toc.json`

### Images

- Store images in `media/` subdirectories adjacent to content.
- Use descriptive alt text.
- Supported formats: PNG, JPG, SVG.
- Use `:::image type="content" source="media/filename.ext" alt-text="Description." lightbox="media/filename.ext":::` — the `lightbox` attribute enables click-to-zoom and is used for all image types (not just SVGs).

### Alerts and callouts

Use Microsoft Learn alert syntax for notes, tips, warnings, and important callouts:

```markdown
> [!NOTE]
> Content for the note.

> [!TIP]
> Content for the tip.

> [!IMPORTANT]
> Content for the important callout.
```

### Contributors section

Articles with external contributors include a `## Contributors` section at the bottom following the pattern in `guidance/includes/daf-contributors-md.md` (with LinkedIn links and job titles).

## File Naming

- Use lowercase kebab-case: `area-topic-subtopic.md`
- Business process articles follow the pattern: `{end-to-end-scenario}-{process-name}-{qualifier}.md`
  - Example: `order-to-cash-monitor-customer-credit-collections-overview.md`
  - Example: `project-to-profit-pattern-enter-project-expenses-expense-management.md`

## Quality Checks

- **Acrolinx** is configured (`.acrolinx-config.edn`) with a minimum quality score of 80. It checks spelling, grammar, style, and Microsoft terminology.
- The VS Code extension `ms-learns.documentor` is recommended (`.vscode/extensions.json`).

## Redirections

When renaming or deleting an article, add an entry to `.openpublishing.redirection.json`:

```json
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/dynamics365-guidance](https://github.com/MicrosoftDocs/dynamics365-guidance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
