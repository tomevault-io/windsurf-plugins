---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

Auto Claude is an autonomous multi-agent coding framework that plans, builds, and validates software for you. It's a TypeScript-first Electron desktop application with a self-contained AI agent layer (Vercel AI SDK v6). A lightweight Python sidecar provides the optional Graphiti memory system.

> **Deep-dive reference:** [ARCHITECTURE.md](shared_docs/ARCHITECTURE.md) | **Frontend contributing:** [apps/desktop/CONTRIBUTING.md](apps/desktop/CONTRIBUTING.md)

## Product Overview

Auto Claude is a desktop application (+ CLI) where users describe a goal and AI agents autonomously handle planning, implementation, and QA validation. All work happens in isolated git worktrees so the main branch stays safe.

**Core workflow:** User creates a task → Spec creation pipeline assesses complexity and writes a specification → Planner agent breaks it into subtasks → Coder agent implements (can spawn parallel subagents) → QA reviewer validates → QA fixer resolves issues → User reviews and merges.

**Main features:**

- **Autonomous Tasks** — Multi-agent pipeline (planner, coder, QA) that builds features end-to-end
- **Kanban Board** — Visual task management from planning through completion
- **Agent Terminals** — Up to 12 parallel AI-powered terminals with task context injection
- **Insights** — AI chat interface for exploring and understanding your codebase
- **Roadmap** — AI-assisted feature planning with strategic roadmap generation
- **Ideation** — Discover improvements, performance issues, and security vulnerabilities
- **GitHub/GitLab Integration** — Import issues, AI-powered investigation, PR/MR review and creation
- **Changelog** — Generate release notes from completed tasks
- **Memory System** — Graphiti-based knowledge graph retains insights across sessions
- **Isolated Workspaces** — Git worktree isolation for every build; AI-powered semantic merge
- **Flexible Authentication** — Use a Claude Code subscription (OAuth) or API profiles with any Anthropic-compatible endpoint (e.g., Anthropic API, z.ai for GLM models)
- **Multi-Account Swapping** — Register multiple Claude accounts; when one hits a rate limit, Auto Claude automatically switches to an available account
- **Cross-Platform** — Native desktop app for Windows, macOS, and Linux with auto-updates

## Critical Rules

**Vercel AI SDK only** — All AI interactions use the Vercel AI SDK v6 (`ai` package) via the TypeScript agent layer in `apps/desktop/src/main/ai/`. NEVER use `@anthropic-ai/sdk` or `anthropic.Anthropic()` directly. Use `createProvider()` from `ai/providers/factory.ts` and `streamText()`/`generateText()` from the `ai` package. Provider-specific adapters (e.g., `@ai-sdk/anthropic`, `@ai-sdk/openai`) are managed through the provider registry.

**i18n required** — All frontend user-facing text uses `react-i18next` translation keys. Hardcoded strings in JSX/TSX break localization for non-English users. Add keys to both `en/*.json` and `fr/*.json`.

**Platform abstraction** — Never use `process.platform` directly. Import from `apps/desktop/src/main/platform/`. CI tests all three platforms.

**No time estimates** — Provide priority-based ordering instead of duration predictions.

**PR target** — Always target the `develop` branch for PRs, not `main`. Main is reserved for releases.

**No console.log in production code** — `console.log` output is invisible in bundled Electron apps. Use Sentry for error tracking in production; reserve `console.log` for development only.

## Work Approach: Orchestrator-First

You are an orchestrator. Your primary role is to understand what needs to be done, break it into workstreams, and delegate execution to agent teams. This keeps your context window focused on coordination and decision-making rather than filling up with implementation details.

<orchestrator_pattern>
When given a task, follow this pattern:

1. **Investigate first** — Read the actual code before forming any hypothesis. Use targeted searches (Glob, Grep, Read) for simple lookups. For broader exploration, spawn an Explore agent.

2. **Plan the approach** — Identify what needs to change, which files are involved, and whether work can be parallelized. For multi-step tasks, create a task list to track workstreams.

3. **Delegate execution** — Spawn agent teams to do the implementation work. Each agent gets a clear, self-contained assignment with all the context it needs: relevant file paths, the specific change to make, and acceptance criteria. Run independent workstreams in parallel.

4. **Verify and integrate** — Review agent outputs, run tests, and ensure changes work together. Fix integration issues or spawn follow-up agents as needed.
</orchestrator_pattern>

**When to delegate vs. do directly:**
- Delegate: multi-file changes, research across the codebase, independent parallel workstreams, tasks that would consume significant context
- Do directly: single-file edits, simple bug fixes, quick lookups, tasks where you already have the context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndyMik90/Aperant](https://github.com/AndyMik90/Aperant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
