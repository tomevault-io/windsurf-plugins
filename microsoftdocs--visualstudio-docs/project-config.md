---
trigger: always_on
description: When working in this repository, apply the following Visual Studio–specific
---


# Visual Studio Product Context — Override Instructions

When working in this repository, apply the following Visual Studio–specific
overrides to any doc-kit agent, skill, or prompt. These rules take precedence
over Foundry-specific defaults shipped with doc-kit.

---

## 1. Product Identity

| Default (Foundry)                  | Override (Visual Studio)                          |
|------------------------------------|---------------------------------------------------|
| Product name: "Azure AI Foundry"   | **Visual Studio**                             |
| Docs base URL                      | `https://learn.microsoft.com/visualstudio/`        |
| Release-notes URL                  | `https://learn.microsoft.com/visualstudio/releases/2026/release-notes` |
| Download page                      | `https://visualstudio.microsoft.com/downloads/`    |
| Feedback mechanism                 | Developer Community (`developercommunity.visualstudio.com`) |
| Product hub                        | `https://aka.ms/vshub`                             |
| `ms.service`                       | `visual-studio-windows`                            |
| `ms.subservice` values             | {FILL: e.g., `ide`, `debugger`, `install`, `test`, `extensibility`, `copilot`} |
| Moniker / version selector         | `visualstudio` (current), `vs-2022` (previous)          |

## 2. Repository & Content Structure

| Aspect               | Value                                                              |
|-----------------------|--------------------------------------------------------------------|
| Docs repo             | `MicrosoftDocs/visualstudio-docs`                                  |
| Docs repo path        | `docs/`                                                            |
| Media folder          | `docs/ide/media/`, `docs/debugger/media/`, etc.                    |
| TOC files             | One `TOC.yml` per feature area under `docs/`                       |
| Breadcrumb file       | `docs/_breadcrumb/toc.yml`                                               |
| Includes folder       | `docs/includes/`                                                   |

## 3. Versioning & Moniker Rules

- Visual Studio docs use **moniker-based versioning**: `visualstudio`, `vs-2022`, `vs-2019`.
- The current GA version is referred to as **Visual Studio** (versionless), even though the underlying version increments monthly (18.0, 18.1, etc.)
- When writing content, target `visualstudio` unless explicitly asked for an older version.
- Use `:::moniker range="visualstudio"` / `:::moniker-end` fences for version-specific content.
- Do NOT mix moniker-fenced blocks with un-fenced content in the same section.

## 4. Metadata Defaults

Every Markdown article must include this YAML front matter (adjust values per article):

```yaml
---
title: "{Article Title}"
description: "{140-character max description}"
ms.date: {MM/DD/YYYY}
ms.topic: {how-to | conceptual | overview | reference | quickstart | tutorial}
author: {GitHub-username}
ms.author: {Microsoft-alias}
---
```

## 5. Key Technology Areas

Visual Studio 2026 documentation covers these primary areas:

| Area                     | Path prefix          | Notes                                              |
|--------------------------|----------------------|----------------------------------------------------|
| IDE & Editor             | `docs/ide/`          | Core editing, settings, themes, project system      |
| Debugging & Diagnostics  | `docs/debugger/`     | Breakpoints, profiling, diagnostics, visualizers    |
| Testing                  | `docs/test/`         | Unit testing, Live Unit Testing, IntelliTest        |
| Deployment               | `docs/deployment/`   | ClickOnce, installers, containers                   |
| Extensibility            | `docs/extensibility/`| VisualStudio.Extensibility (new model), VSSDK (legacy) |
| .NET / C#                | Cross-references to `learn.microsoft.com/dotnet/` |                           |
| Git tooling              | `docs/version-control/` | PR reviews, inline comments, Git integration     |
| Cloud                    | `docs/azure/`        | Azure integration, Container Tools, Service Fabric  |
| Install & Update         | `docs/install/`      | Workloads, offline install, admin layouts            |

## 6. Terminology & Style

| Instead of …                            | Use …                                          |
|-----------------------------------------|------------------------------------------------|
| "project" (AI/ML context)              | "solution" or "project" (VS solution/project)  |
| "deployment" (model deployment)        | "deployment" (app publish/deploy)              |
| "agent" (AI agent)                     | "agent" (Copilot agent, or avoid ambiguity)    |
| "playground"                           | Not applicable — omit                          |
| "hub" / "project" (Foundry hierarchy)  | Not applicable — omit                          |
| "SDK" (Azure AI SDK)                   | "Visual Studio SDK"  |
| "endpoint"                             | "endpoint" (debugging/web) — context-dependent |

- Follow the [Microsoft Writing Style Guide](https://learn.microsoft.com/style-guide/welcome/).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/visualstudio-docs](https://github.com/MicrosoftDocs/visualstudio-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
