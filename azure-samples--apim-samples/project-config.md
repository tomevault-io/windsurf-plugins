---
trigger: always_on
description: This instructions file is designed to guide GitHub Copilot's behavior specifically for this repository. It is intended to provide clear, general, and maintainable guidelines for code generation, style, and collaboration.
---


# GitHub Copilot Instructions for this Repository

## Purpose

This instructions file is designed to guide GitHub Copilot's behavior specifically for this repository. It is intended to provide clear, general, and maintainable guidelines for code generation, style, and collaboration.

**In case of any conflict, instructions from other individualized or project-specific files (such as `my-copilot.instructions.md`) take precedence over this file.**

## Repository Context

- This repository provides a playground to safely experiment with and learn Azure API Management (APIM) policies in various architectures.
- The primary technologies are Python, Bicep, Jupyter notebooks, Azure CLI, APIM policy XML, and Markdown.
- The technical audience includes developers, architects, and DevOps engineers who want to understand and implement APIM policies effectively.
- The less technical audience includes decision makers and stakeholders who need to understand the value and capabilities of APIM policies without deep technical details.

## Instruction Hierarchy

- When the user asks about **Python** or a Python file is referenced in the chat context, prefer guidance and examples from `./python.instructions.md`.
- When the user asks about **Bicep** or a Bicep file is referenced in the chat context, prefer guidance and examples from `./bicep.instructions.md`.
- When the user asks about **JSON** or a JSON file is referenced in the chat context, prefer guidance and examples from `./json.instructions.md`.
- When the user asks about **GitHub Workflows** or workflow files (`.github/workflows/*.yml`) are referenced in the chat context, prefer guidance and examples from `./github-workflows.instructions.md`.
- When other languages are used, look for a relevant instructions file to be included. The format is `./[language].instructions.md` where `[language]` acts as a placeholder. Also consider synonyms
  such as `JavaScript`, `JScript`, etc.

In case of any conflicting instructions, the following hierarchy shall apply. If a conflict cannot be resolved by this hierarchy, please prompt the user and ask for their situational preference.

  1. Individualized instructions (e.g. a developer's or an organization's instruction file(s)), if present
  2. This repository's `.github/.copilot-instructions.md`
  3. General best practices and guidelines from sources such as [Microsoft Learn](https://learn.microsoft.com/docs/)
    This includes the [Microsoft Cloud Adoption Framework](https://learn.microsoft.com/azure/cloud-adoption-framework/).
  4. Official [GitHub Copilot best practices documentation](https://docs.github.com/enterprise-cloud@latest/copilot/using-github-copilot/coding-agent/best-practices-for-using-copilot-to-work-on-tasks)

## Copilot Personality Behavior

- Never be rude, dismissive, condescending, threatening, aggressive, or otherwise negative.
- Emphasise friendly, supportive, and collaborative interactions.
- Be concise and to the point, but adjust the level of detail based on the user's technical expertise that you can infer from the conversation.

## General Principles

- Write concise, efficient, and well-documented code for a global audience.
- Consider non-native English speakers in code comments and documentation, using clear and simple language.
- Treat accessibility as a default quality requirement across the entire repository, not only for presentations.
- New or updated user-facing experiences (docs, webpages, notebooks, dashboards/workbooks, and slide content) must target WCAG 2.0 AA contrast and non-color-only communication as the baseline.

## Consistency & Uniformity

Uniformity, clarity, and ease of use are paramount across all infrastructures and samples. Every infrastructure and every sample should look and feel as alike as possible so that users maintain familiarity as they move between them. A user who has completed one sample should never feel like they are viewing something entirely new when they open the next.

- **Follow the established templates.** New infrastructures must follow the structure of existing infrastructures. New samples must follow `samples/_TEMPLATE`. Deviations are permitted only when a sample has genuinely unique requirements, and those deviations should be minimal.
- **Use consistent naming, headings, and cell order.** Markdown headings, variable names, section labels (e.g. `USER CONFIGURATION`, `SYSTEM CONFIGURATION`), emoji usage, and code cell ordering must match the patterns established by the template and existing artefacts.
- **Keep README structure uniform.** Infrastructure READMEs and sample READMEs each follow their own standard layout (see the guidelines below). Readers should be able to predict where to find objectives, configuration steps, and execution instructions.
- **Reuse shared utilities.** Use `NotebookHelper`, `InfrastructureNotebookHelper`, `ApimRequests`, `ApimTesting`, and shared Bicep modules rather than inventing ad-hoc alternatives. Shared code is the single best tool for enforcing uniformity.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/Apim-Samples](https://github.com/Azure-Samples/Apim-Samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
