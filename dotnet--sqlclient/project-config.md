---
trigger: always_on
description: This document provides guidance for AI coding agents working with the Microsoft.Data.SqlClient repository.
---

# AI Agent Guidelines for Microsoft.Data.SqlClient

This document provides guidance for AI coding agents working with the Microsoft.Data.SqlClient repository.

## Quick Start

### Essential Context Files

Before making changes, agents should be aware of:

| File | Purpose |
|------|---------|
| [README.md](README.md) | Project overview |
| [BUILDGUIDE.md](BUILDGUIDE.md) | Build and test instructions |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution guidelines |
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | Copilot-specific instructions |

### Detailed Technical Instructions

The `.github/instructions/` directory contains comprehensive guides:

| Guide | Coverage |
|-------|----------|
| [architecture.instructions.md](.github/instructions/architecture.instructions.md) | Project structure, unified project model |
| [tds-protocol.instructions.md](.github/instructions/tds-protocol.instructions.md) | TDS protocol, packet handling |
| [ado-pipelines.instructions.md](.github/instructions/ado-pipelines.instructions.md) | Azure DevOps CI/CD pipelines |
| [onebranch-pipeline-design.instructions.md](.github/instructions/onebranch-pipeline-design.instructions.md) | OneBranch multi-product pipeline design spec |
| [testing.instructions.md](.github/instructions/testing.instructions.md) | Test framework, running tests |
| [connection-pooling.instructions.md](.github/instructions/connection-pooling.instructions.md) | Connection pool internals |
| [api-design.instructions.md](.github/instructions/api-design.instructions.md) | Public API design principles |
| [features.instructions.md](.github/instructions/features.instructions.md) | Feature reference, keywords |
| [documentation.instructions.md](.github/instructions/documentation.instructions.md) | Documentation and samples |
| [external-resources.instructions.md](.github/instructions/external-resources.instructions.md) | Docs links, version matrix, external references |
| [ado-work-items-markdown.instructions.md](.github/instructions/ado-work-items-markdown.instructions.md) | Ensure Azure DevOps work item descriptions are Markdown and preserve newlines |

## Workflow Prompts

This repository provides reusable prompts in `.github/prompts/` for common maintainer workflows. Use these to guide agents through multi-step operations.

| Prompt | Purpose |
|--------|---------|
| [fix-bug.prompt.md](.github/prompts/fix-bug.prompt.md) | Diagnose and fix a bug with tests and documentation |
| [implement-feature.prompt.md](.github/prompts/implement-feature.prompt.md) | Plan and implement a new feature end-to-end |
| [triage-issue.prompt.md](.github/prompts/triage-issue.prompt.md) | Triage a new GitHub issue with labeling and categorization |
| [code-review.prompt.md](.github/prompts/code-review.prompt.md) | AI-assisted code review for a pull request |
| [perf-optimization.prompt.md](.github/prompts/perf-optimization.prompt.md) | Investigate and implement performance improvements |
| [release-notes.prompt.md](.github/prompts/release-notes.prompt.md) | Generate release notes for a specific milestone |
| [update-build-pipelines.prompt.md](.github/prompts/update-build-pipelines.prompt.md) | Modify Azure DevOps CI/CD pipeline configuration |

## Core Principles

1. **Cross-Platform Compatibility**: Code must work on .NET Framework 4.6.2+ and .NET 8.0+
2. **Backward Compatibility**: No breaking changes without proper deprecation
3. **Test-First Development**: All changes require tests
4. **Security by Default**: Secure defaults, no credential logging
5. **Protocol Compliance**: Follow MS-TDS specifications
6. **Performance Optimization**: Use pooling, async, efficient allocations
7. **Observability**: EventSource tracing, meaningful errors

## Terminal Reliability Rules

When using shell/terminal tools, follow these rules strictly:

1. Treat non-zero terminal exit codes as immediate failures to investigate; do not continue as if the command succeeded.
2. If a bash session exits, assume it is dead and start a new command/session; do not wait for additional output from that session.
3. After any command expected to gather data, verify output was actually returned before proceeding.
4. If command execution failed, report the failure clearly and retry with a corrected command instead of waiting.
5. Avoid `set -e` in this automation context; prefer single-purpose commands with explicit follow-up checks so failures are visible without killing the shell unexpectedly.
6. Prefer shorter command batches over long chained scripts when collecting evidence; this makes bash exits easier to detect and recover from.

## Branch Naming

All branches created by AI agents **must** live under the `dev/automation/` prefix. Use a descriptive suffix, for example:

- `dev/automation/fix-connection-timeout`
- `dev/automation/add-json-type-tests`

Do **not** create branches directly under `main`, `dev/`, or any other top-level prefix.

## Common Tasks

### Bug Fix Workflow

1. Understand the issue from the bug report
2. Locate relevant code in `src/Microsoft.Data.SqlClient/src/` (do NOT modify legacy `netcore/src/` or `netfx/src/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/SqlClient](https://github.com/dotnet/SqlClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
