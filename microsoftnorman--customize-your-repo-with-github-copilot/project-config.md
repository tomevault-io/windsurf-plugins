---
trigger: always_on
description: This repository contains "The Definitive Guide to Customizing Your Repo for GitHub Copilot" — a comprehensive documentation guide covering GitHub Copilot's eight customization primitives and two platform extensions. This is a documentation project, not a code project.
---

# Copilot Instructions for customize-your-repo

## Project Overview

This repository contains "The Definitive Guide to Customizing Your Repo for GitHub Copilot" — a comprehensive documentation guide covering GitHub Copilot's eight customization primitives and two platform extensions. This is a documentation project, not a code project.

## Content Guidelines

### Tone and Voice
- Use professional, third-person tone throughout
- Write for an audience of experienced developers and team leads
- Be direct and concise — avoid filler phrases
- Match the existing document's voice and structure

### Product Naming
- Always refer to the product as **GitHub Copilot** — never shorten it to just "Copilot" in isolation, marketing-style references, or product names (e.g., "GitHub Copilot CLI", not "Copilot CLI" as a standalone brand)
- After the full name "GitHub Copilot" has been introduced in a section, subsequent prose mentions within that section may use "Copilot" for readability
- Headings, tables of contents, page titles, and the first mention in any standalone section must use the full name "GitHub Copilot"
- Never use "Github Copilot", "github copilot", "GH Copilot", or other stylistic variants — the canonical form is "GitHub Copilot" with that exact capitalization

### Accuracy Requirements
- All technical claims must align with official documentation:
  - https://code.visualstudio.com/docs/copilot
  - https://code.visualstudio.com/docs/copilot/customization/overview (VS Code customization hub — agents, skills, hooks, MCP, plugins)
  - https://code.visualstudio.com/docs/copilot/customization/agent-plugins (Agent Plugins — bundled customization packages)
  - https://docs.github.com/en/copilot
  - https://docs.github.com/en/copilot/reference/custom-agents-configuration (Custom agents frontmatter reference)
  - https://github.com/github/copilot-cli
  - https://github.com/features/copilot/cli/
  - https://github.blog/changelog/label/copilot/
  - https://github.blog/ (announcements, feature deep-dives, and engineering posts)
  - https://github.blog/ai-and-ml/automate-repository-tasks-with-github-agentic-workflows/ (GitHub Agentic Workflows — Continuous AI via coding agents in GitHub Actions)
  - https://github.github.com/gh-aw/ (Agentic Workflows documentation site — reference, patterns, and setup)
  - https://docs.github.com/en/copilot/concepts/agents/copilot-memory (Copilot Memory — automatic repository-level learning)
  - https://github.com/github/copilot-sdk (Copilot SDK — embed agent runtime in custom applications)
  - https://agentskills.io (Agent Skills specification — open standard for portable agent capabilities)
  - https://modelcontextprotocol.io (MCP specification — standard for connecting AI agents to external tools)
  - https://learn.microsoft.com/en-us/training/modules/configure-customize-github-copilot-visual-studio-code/ (Microsoft Learn training module for Copilot customization)
  - https://github.com/github/awesome-copilot (Curated community plugins, skills, and agent examples)
  - https://github.com/github/CopilotForXcode (GitHub Copilot for Xcode — extension source, releases, and documentation)
  - https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-extension?tool=xcode (GitHub Copilot for Xcode setup and installation)
  - https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-extension?tool=eclipse (GitHub Copilot for Eclipse setup and installation)
  - https://docs.github.com/en/copilot/reference/copilot-feature-matrix?tool=eclipse (Copilot feature matrix for Eclipse — per-version support)
  - https://marketplace.eclipse.org/content/github-copilot (GitHub Copilot on Eclipse Marketplace — versions, compatibility, reviews)
  - https://github.com/microsoft/copilot-eclipse-feedback (Eclipse plugin feedback and issue tracker)
  - https://github.com/microsoft/copilot-eclipse-feedback/releases (Eclipse plugin release notes)
  - https://devblogs.microsoft.com/java/ghc-eclipse-is-going-open-source/ (Microsoft for Java Dev Blog — Copilot for Eclipse open source announcement)
  - https://devblogs.microsoft.com/java/ (Microsoft for Java Dev Blogs — Eclipse Copilot updates and deep dives)
  - https://github.com/orgs/community/discussions/151288 (GitHub Copilot for Eclipse community discussions)
  - https://www.youtube.com/@code (VS Code YouTube channel — official demos, livestreams, and feature deep-dives; use with the `check-video-sources` skill to fetch transcripts and timestamps)
  - https://www.youtube.com/@GitHub (GitHub YouTube channel — official product announcements, Copilot walkthroughs, and GitHub Universe sessions; use with the `check-video-sources` skill to fetch transcripts and timestamps)
- **Always fetch the latest documentation before answering questions about Copilot features** — your training data may be outdated
- Use the Microsoft docs tools to search and fetch from code.visualstudio.com
- Use the fetch_webpage tool for docs.github.com/en/copilot, github.com/github/copilot-cli, github.com/features/copilot/cli, and github.blog pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoftnorman/customize-your-repo-with-github-copilot](https://github.com/microsoftnorman/customize-your-repo-with-github-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
