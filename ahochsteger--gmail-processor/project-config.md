---
trigger: always_on
description: This document provides essential information for AI agents to understand the project's structure, architecture, and conventions.
---

# AI Agent Guidelines for Gmail Processor

This document provides essential information for AI agents to understand the project's structure, architecture, and conventions.

> [!CAUTION]
> **Strict Safe Git Protocol**: AI agents are strictly PROHIBITED from modifying the Git database.
>
> - **Zero-Push/Commit Policy**: You MUST NOT run `git add`, `git commit`, `git push`, or any other state-modifying Git commands.
> - **Local Prep Only**: You are only permitted to prepare and preview local file changes (HTML, JS, CSS, TS, JSON, MD, etc.).
> - **Human-Decision Mandatory**: All Git write operations are exclusively reserved for human maintainers after manual review of your prepared changes.

## Agent Mission & Knowledge Continuity

As an AI agent, your primary mission is to maintain the **Gmail Processor** repository in a professional, high-quality, and consistent state.

### 1. Continuous Knowledge Capture

Sustainability depends on shared understanding. You MUST update this document whenever:

- A new architectural decision is made.
- A significant workflow or script is changed or introduced.
- A technical constraint or "gotcha" is discovered and resolved.
- A new tool is added to the maintenance pipeline.

### 2. Guarding the Standard

You are responsible for enforcing the standards defined in this manual. Do not allow regressions in:

- **Canonical Formatting**: Use `npm run lint-fix` regularly.
- **Alphabetical Ordering**: Maintain order in all lists and maintenance scripts.
- **Project Hygiene**: Proactively use pruning and validation tools (`npm run lint-prune`, `npm run lint:devbox:unused`, `npm run lint:scripts`) to keep the codebase lean and reference-stable.

## Project Overview

Gmail Processor is a TypeScript-based library designed to automate the processing of Gmail messages and attachments using Google Apps Script (GAS). It uses a rule-based configuration (JSON/YAML) to perform actions like storing attachments in Google Drive or logging information in spreadsheets.

## Architecture & Core Concepts

### 1. Context-Aware Hierarchy

The system uses a hierarchical context model to pass information through the processing layers:

- **EnvContext:** Global environment info (GAS services, run mode).
- **ProcessingContext:** Configuration, adapters, and action registry.
- **ThreadContext:** Info about the current GMail thread.
- **MessageContext:** Info about the current GMail message.
- **AttachmentContext:** Info about the current GMail attachment.

_File Reference:_ [Context.ts](src/lib/Context.ts)

### 2. Processing Flow

Processing is handled by specialized processor classes that follow the hierarchy:

1. `GmailProcessor` (Top-level)
2. `ThreadProcessor`
3. `MessageProcessor`
4. `AttachmentProcessor`

Base logic for matching and action execution is in the [BaseProcessor](src/lib/processors/BaseProcessor.ts) abstract class.

### 3. Action Registry & Providers

Actions (e.g., `attachment.store`, `thread.addLabel`) are organized into providers and registered in the `ActionRegistry`.

- **GlobalActions:** General actions.
- **ThreadActions / MessageActions / AttachmentActions:** Context-specific actions.

_Pattern:_ Actions are typically static methods in classes implementing `ActionProvider`.

### 4. Adapter Layer

To maintain testability and GAS compatibility, external service interactions are wrapped in adapters:

- `GmailAdapter`, `GDriveAdapter`, `SpreadsheetAdapter`, `LogAdapter`.

## Project Structure

- `src/lib/`: Core library logic.
  - `actions/`: Action provider implementations.
  - `adapter/`: GAS service abstractions.
  - `config/`: Configuration models and validation.
  - `processors/`: Hierarchical logic for processing GMail entities.
- `src/examples/`: Reference configurations and E2E tests.
- `scripts/`: Build and maintenance scripts.

## Maintenance Triggers

To keep the workspace in a sane state, specific commands MUST be executed after modifying certain types of files:

| Affected Area               | Required Action       | Command                                  |
| :-------------------------- | :-------------------- | :--------------------------------------- |
| `src/lib/`                  | Rebuild & Update Docs | `npm run all:update && npm run test:lib` |
| `src/examples/*.ts`         | Re-generate Examples  | `npm run update:examples`                |
| `package.json`              | Node.js Pruning       | `npm run lint-prune`                     |
| `devbox.json`               | Devbox Pruning        | `npm run lint:devbox:unused`             |
| Root `*.md` files           | Sync Docs             | `npm run update:docs`                    |
| `package.json`              | Total Reinstall       | `npm run all:reinstall`                  |
| Documentation CSS/Config    | Verify Build          | `npm run ci:docs`                        |
| Deployment / Release        | Update Release Notes  | `npm run release:update`                 |
| Release Preview             | Dry-run Release Notes | `npm run release:notes`                  |
| `package.json` / `scripts/` | Script Integrity      | `npm run lint:scripts`                   |

## Generated Artifacts & Automation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahochsteger/gmail-processor](https://github.com/ahochsteger/gmail-processor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
