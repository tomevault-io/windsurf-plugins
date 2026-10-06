---
trigger: always_on
description: This document defines how AI coding assistants—including coding agents, IDE assistants, and automation agents—should work in the Argus repository. The goal is to make AI-assisted development reviewable, verifiable, and reversible, while protecting SOC data, user permissions, and security automation capabilities.
---

# AGENTS.md — Argus AI Coding Guidelines

This document defines how AI coding assistants—including coding agents, IDE assistants, and automation agents—should work in the Argus repository. The goal is to make AI-assisted development reviewable, verifiable, and reversible, while protecting SOC data, user permissions, and security automation capabilities.

## 1. Project Context and Working Principles

Argus is an AI-native Agentic SOC platform involving alerts, incident tickets, assets, detection rules, workflows, AI assistants, MCP, and external integrations.

Follow these principles:

1. **Understand before changing.** Before making changes, inspect the relevant source code, tests, documentation, configuration, and Git status. Do not infer existing behavior from filenames or the user’s description alone.
2. **Keep changes small and focused.** Only modify what is needed for the task. Avoid unrelated refactoring, large-scale formatting, dependency upgrades, or mass renaming.
3. **Prioritize security.** Permissions, authentication, credentials, MCP, workflows, external integrations, and production configuration are high-risk areas and require explicit security review.
4. **Use evidence.** When uncertain about an API, dependency, command, or business rule, check the repository’s implementation, documentation, and configuration. Do not present assumptions as facts.
5. **Never claim unperformed checks.** Report only checks that were actually run and their results. If a check could not be run, state why and identify any remaining risks.

## 2. Before Starting a Task

Before coding:

- Read the root `README.md`, relevant documentation, and code in the target module.
- Check `git status` and existing diffs. Do not overwrite or discard changes made by the user.
- Identify the affected area: `backend/`, `frontend/`, `docs/`, `k8s/`, Docker configuration, or another directory.
- Find relevant tests, nearby implementations, API contracts, and configuration. Follow existing project patterns.
- Confirm available commands from `makefile`, `package.json`, Python project configuration, CI workflows, or documentation. **Do not guess commands or test frameworks.**
- If the request is ambiguous and could affect permissions, data, API compatibility, or production behavior, ask for clarification. Do not expand the scope of authorization on your own.

## 3. AI Coding Workflow

Follow this process for each task:

1. **Summarize your understanding:** Briefly describe the goal, affected modules, and main risks.
2. **Outline the approach:** For cross-cutting or high-risk changes, list the files you plan to modify and how you will validate the changes.
3. **Make the smallest reasonable change:** Preserve the existing architecture, naming, error handling, and coding style.
4. **Add verification:** For behavior changes, add or update tests, type checks, or documentation as appropriate.
5. **Review the diff:** Check the scope of changes, sensitive information, permission bypasses, error handling, and compatibility.
6. **Report the results:** Summarize what changed, which checks were run, and which checks were not run and why.

Do not rewrite an entire module solely to make it “look cleaner” unless requested.

## 4. General Coding Rules

- Prefer existing libraries, abstractions, components, and tools. Explain why a new dependency is necessary before introducing one.
- Do not submit fake implementations, placeholder behavior, fabricated data, or code that silently swallows exceptions.
- Do not convert errors into success responses. Error messages should help with troubleshooting without revealing secrets, tokens, personal data, or sensitive internal configuration.
- When changing externally visible behavior, review and update related APIs, UI, documentation, and tests.
- Preserve compatibility. If a breaking change is necessary, clearly describe its impact, migration plan, and rollback approach.
- Do not modify lock files, generated files, migration files, or configuration formats unless the task requires it.
- Do not “fix” failures by disabling linting, type checks, tests, or security checks.
- Separate frontend and backend responsibilities; keep modules loosely coupled.
- Maintain a clear boundary between the frontend and backend, communicating through well-defined APIs rather than relying on each other’s internal implementation details. 
- Keep modules focused on their responsibilities, minimize cross-module dependencies, and avoid circular dependencies.

## 5. Backend: Django / Django REST Framework

When modifying `backend/`:

- Inspect the target Django app’s models, serializers, views, permissions, URLs, services, and tests. Follow the organization used in that module.
- For API changes, review authentication, authorization, input validation, pagination/filtering, error responses, and the API contract.
- Do not rely on hiding UI controls in the frontend for access control. The backend must enforce authorization.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sec-Link/Argus-Agentic-SOC-Platform](https://github.com/Sec-Link/Argus-Agentic-SOC-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
