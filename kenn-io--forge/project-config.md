---
trigger: always_on
description: kenn-forge is a local-first maintainer console. A Go/Huma server syncs
---

# kenn-forge Agent Instructions

## Project Model

kenn-forge is a local-first maintainer console. A Go/Huma server syncs
provider-backed pull requests, issues, and activity into SQLite and serves an
embedded Svelte 5 SPA. Kata and Docs are separate first-class modes whose data
remains owned by their external or filesystem domains. The project builds
without CGO; use the README and Makefile for discoverable setup, build, and
development commands.

## Provider Support

kenn-forge supports GitHub, GitLab, Forgejo, and Gitea. This is the single
canonical provider list; `gitealike` is the shared Forgejo/Gitea adapter.
Provider-backed features must work across every supported provider within its
declared capabilities and preserve provider-verified stable repository identity;
owner/name remains a mutable route. The routing table below owns the detailed rules.

## Non-Provider Modes

Kata and Docs are first-class modes, not platform providers. Do not force them
through `internal/platform`: their data remains owned respectively by Kata
daemons and configured filesystem folders.

## Context Routing

Read the smallest relevant set of topic documents before changing or reviewing
the corresponding area. These documents own the detailed invariants; this file
only routes to them.

| When working on | Read |
| --- | --- |
| Provider interfaces or package boundaries | `context/provider-architecture.md` |
| Provider identity, sync, import, routes, or settings | `context/platform-sync-invariants.md` |
| GitHub-specific sync or notifications | `context/github-sync-invariants.md`, `context/notifications-in-activity.md` |
| Config fields that persist to TOML | `context/config-persistence.md` |
| Database schema migrations | `context/db-migrations.md` |
| Deferred merge behavior | `context/deferred-merge.md` |
| Embed routes or host bridges | `context/embeds.md` |
| Daemon startup, discovery, host/origin validation, or SSE replay | `context/server-runtime.md` |
| Fleet settings, snapshots, host routing, or peer transports | `context/fleet-architecture.md` |
| API failures or frontend error branching | `context/error-handling.md` |
| Retries, rate limits, scheduling, or single-flight work | `context/retries-and-backoffs.md` |
| Go test commands, assertions, fixtures, or shell tests | `context/testing-basics.md` |
| Test lanes, provider tests, API contracts, or HTTP tests | `context/testing.md` |
| Repository-controlled session bootstrap or dependency installation | `context/agent-bootstrap.md` |
| User documentation, screenshots, or the Zensical site | `context/docs-authoring.md` |
| Pushing, opening a pull request, or changing PR metadata, comments, or review threads | `context/pull-request-workflow.md` |
| Frontend visual design or component conventions | `context/ui-design-system.md` |
| Frontend Effect workflows, services, layers, errors, or async ownership | `context/frontend-effect.md` |
| Frontend interaction, route state, persistence, or input semantics | `context/ui-interaction-contracts.md` |
| Phone routes, narrow layouts, or touch UX | `context/mobile-ux.md` |
| Workflow or terminal panel interaction models | `context/vscode-workflow-panel-interaction-spec.md` |
| Workspace APIs, creation, item identity, lifecycle hooks, or generated launch context | `context/workspace-apis.md` |
| Workspace deletion, runtime sessions, tmux, or terminal UI | `context/workspace-runtime-lifecycle.md` |
| Repository source-browser routes, clones, refs, or previews | `context/repository-source-browser.md` |
| Inline diff review drafts, comments, or threads | `context/inline-review-comments.md` |
| Kata task authority, daemon integration, task UI, or Kata workspaces | `context/kata-mode.md`, `context/workspace-apis.md` |
| Markdown folders, Docs APIs, or git publishing | `context/docs-mode.md` |

## Conventions

- Prefer stdlib over external dependencies
- The `kenn-forge` binary has one Cobra root command. Register every public command on that tree; do not add a second parser, manual dispatcher, or command-facing `flag.FlagSet`. (`cmd/kenn-forge/cli.go::newRootCommand`)
- CLI flags must affect execution or fail validation; reject shared persistent flags outside the commands that consume them instead of silently ignoring user input. (`internal/cli/ctl/ctl.go::installControlFlagValidation`)
- Do the task requested, not the task imagined. Do not widen scope without explicitly confirming with the user first
- When a backwards compatibility adapter, shim, alias, fallback wrapper, or legacy translation layer seems useful, ask the user for EXPRESS permission before introducing it. These shims carry very high maintenance cost because they preserve old paths, multiply edge cases, and make future changes harder to reason about; explain the compatibility benefit and why direct migration or removal is not the better choice.
- Use `huma` for the web framework and OpenAPI generation
- Regenerate API artifacts with `make api-generate`; the Go client also supports `go generate ./internal/apiclient/generated`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenn-io/forge](https://github.com/kenn-io/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
