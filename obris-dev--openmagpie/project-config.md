---
trigger: always_on
description: Conventions for AI coding agents (Claude Code, Codex, Cursor, etc.) and human contributors.
---

# AGENTS.md

Conventions for AI coding agents (Claude Code, Codex, Cursor, etc.) and human contributors.

This file is cross-cutting only. Each top-level app owns its own conventions:

- [apps/core/AGENTS.md](apps/core/AGENTS.md): Django backend (models, services, auth, plugins)
- [apps/cli/AGENTS.md](apps/cli/AGENTS.md): `magpie` CLI (Typer + httpx + Pydantic)
- [web/AGENTS.md](web/AGENTS.md): pnpm workspace (Next.js + shared packages)

When working in `apps/core/`, `apps/cli/`, or `web/`, load the matching `AGENTS.md` alongside this one.

## What is OpenMagpie

An open-source semantic listener. Tell it what to listen for; it picks out what matters from any stream and learns over time.

Three things stay pluggable across the codebase:
- **Connectors** (Reddit, GitHub, GDocs, Slack, ...): yield typed `SourcePayload` subclasses from each source
- **Engines** (any OpenAI-compatible `/v1` LLM: Ollama, vLLM, llama.cpp, LM Studio, OpenAI, ...; future keyword/other): BYO LLM that judges a `SourcePayload` for a semantic-filter action
- **Action kinds** (semantic_filter, extract, webhook, log, future keyword/Slack/email): the steps a Watch runs over each feed item -- three families: FILTER (gate the chain), EXTRACT (hydrate structured fields onto the run), DELIVER

The product is **only** a listener: watches, judges, learns, notifies. It does NOT auto-reply, post back to sources, run workflows, or generate new reports/analysis as a product surface. Exporting your OWN activity is in scope though -- e.g. `magpie activity export` dumping an action's runs (incl. extracted fields) to CSV is observability over data you already produced, not report generation. Scope test: if a feature isn't listening / learning / notifying (or surfacing what it already did), it's out.

## Repo layout

```
apps/core/                  Django backend (see apps/core/AGENTS.md)
apps/cli/                   magpie CLI (see apps/cli/AGENTS.md)
packages/openmagpie-schema/ pure Pydantic models shared by core + cli
web/                        pnpm workspace, Next.js (see web/AGENTS.md)
make/                       Per-concern Makefile targets
scripts/                    quickstart installer (quickstart/{bootstrap,preflight,run,seed,tick}.sh) + dev tooling (check-docker, hooks, lint/whitespace/branch checks, make-help)
tools/                      Python dev tooling (schema_sync/: generate + guard packages/openmagpie-schema/schema.json)
pyproject.toml + uv.lock    uv workspace root (one lock for all members)
```

## Naming (cross-cutting domain vocabulary)

- The unit of attention is a **`Watch`**: a subscription over a set of feeds plus an ordered chain of actions. "Listener" survives as the product pitch ("a Watch is a listener"), not a code-level node name.
- A polled item is a **`FeedItem`** (persisted Django row). The in-memory typed version a connector produces is a **`SourcePayload`** (Pydantic).
- A single action executing against one feed item is a **`WatchActionRun`** (the audit row), surfaced publicly (CLI + REST) as **`activity`** (the model name stays `WatchActionRun`; the unit is an "activity entry"). There is no `Event` / hit model; a successful filter is just a `WatchActionRun` that advanced the chain.
- **Resource names qualify by parent only when dependent.** A first-class entity, a hub other resources are addressed relative to, is named bare (`Watch`, `Feed`, `WatchAction`). A dependent record or component, meaningless apart from its parent, is qualified by it (a feed's items / sources, an action's activity / deliveries). This drives both REST route names ([apps/core/AGENTS.md](apps/core/AGENTS.md)) and CLI command nouns ([apps/cli/AGENTS.md](apps/cli/AGENTS.md)).
- Source connectors are named for the variant: **`RedditSubRedditConnector`** (kind=`"reddit_subreddit"`). Future Reddit variants get their own connector + kind.
- Payloads from sources are named for *what happened*: **`NewRedditPostPayload`** (`PAYLOAD_KIND="new_post"`).
- An action's typed result is a per-kind model: **`SemanticFilterResult`** (`{passed, score, reason}`), `WebhookResult`, `LogResult`.

## Cross-cutting code rules

- **State-machine values get a const object + derived type from the start.** No bare string literals in match arms or status checks. Python: `class Status(Enum): ...`. TypeScript: `const PHASE = {...} as const; type Phase = typeof PHASE[keyof typeof PHASE]`.
- **No em dashes, and no `--` as a stand-in for one.** Rewrite: commas, parens, or two sentences. Applies to UI text, comments, docs.
- **Shell scripts are POSIX `sh`.** `#!/bin/sh`, no bashisms (`[[ ]]`, `=~`, arrays, `local`, `set -o pipefail`, `${BASH_SOURCE}`, `< <(...)`). Everything in `scripts/` must pass `shellcheck -s sh` (enforced in pre-commit + CI).
- **Convention docs describe what to do.** No justifications, no historical context, no "we chose X because of Y." Forward-looking constraints are fine; past-decision narratives are not.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obris-dev/openmagpie](https://github.com/obris-dev/openmagpie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
