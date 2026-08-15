---
trigger: always_on
description: This is the **tool-only** repository. It has no bundled application, backend, assets,
---

# Agentic Ship

This is the **tool-only** repository. It has no bundled application, backend, assets,
or deployment. The product-stack rules below are guidance for a downstream product
workspace that adopts Agentic Ship; do not expect `src/`, `convex/`, or a local website
to exist here.

This file is where every rule is **declared**. `CLAUDE.md` imports it with one line;
`.claude/skills` links to `.agents/skills`. Skills elaborate these rules into
procedure — they must never restate one differently, and never introduce a rule that is
not declared here. When a skill and this file disagree, this file is right and the skill
is a bug.

Works with any agentic tool. Codex, Cursor, Windsurf, Cline, Copilot, Gemini CLI, and
OpenClaw (with this repository as its agent workspace) read this file natively; skills
are plain markdown any agent can follow. Canonical role
briefs live in `.agents/agents/`; `pnpm sync:agents` generates the Claude plugin's
top-level `agents/`, project-native Codex and Cursor agents, and non-secret Hermes and
OpenClaw profiles. Cursor gets MCP through the committed
`.cursor/mcp.json` mirror; Codex gets project-scoped TOML generated from the same pinned
catalog.
The repo is also an **installable plugin** for Claude Code and Codex — manifests in
`.claude-plugin/` and `.codex-plugin/` point at the same `.agents/skills/`, so the
plugin adds a second delivery path, never a second copy of a rule.
Per-tool matrix and sync rules: `.agents/skills/agent-compatibility/SKILL.md`.
Skills with a `references/` folder keep their deep material there — load it only when
the task needs it.

Instructions live here. Procedures live in `.agents/skills/`. Tool wiring lives in
`.mcp.json`. Product, feature, and human-input contracts live in `.agents/contracts/`.
Safe runtime coordination lives under gitignored `.agent-state/`; credentials never do.
Plugin wiring lives in `.claude/settings.json` (the official `nextjs` plugin from the
vercel/next.js repo is declared there). Provenance for all of it lives in
`skills.lock.json`. One rule, one home: it is declared here, applied there.

## Stack

Node 20+ · pnpm · TypeScript parser · Playwright capture runtime. Downstream product
stacks are selected by their own product contract.

Delivery and tracking ride on two services: **GitHub** carries the repository, pull
requests, and CI through the authenticated `gh` CLI (`pnpm provider:login github`), and
**Linear** is the optional development-tracking mirror through its hosted MCP — both
cataloged in `.agents/connections/providers.json`, both revocable, neither required for
the core workflow.

Version pins live in `skills.lock.json`. Do not hand-edit versions — run the
`upstream-sync` skill.

## Commands

**Node is the only runtime this repo assumes.** Every Agentic Ship operation is a Node script
in `scripts/` behind a `pnpm` name, so it behaves identically on macOS, Linux and
Windows. The buyer may be on any of the three.

| Command | Does |
| --- | --- |
| `pnpm verify` | **the offline definition of done** — tool health + agent/MCP/UI contracts + unit gates (no product lint or build lives in this tool repo) |
| `pnpm verify:full` | verify + the fail-closed production dependency audit; use before a PR or deploy |
| `pnpm test` | gate G2 — deterministic tool contracts (vitest, in-memory). The G3 e2e gate (`pnpm test:e2e`, Playwright against the product build) runs in a downstream product workspace, not this tool repo |
| `pnpm heal` | tier-1 deterministic repairs (links, mirrors, env, lockfile, MCP server boot), then health as proof |
| `pnpm preflight [--prod]` | **the go-live gate** — live keys, email flips, no seed backdoor |
| `pnpm health` | offline tool-repo health — required skill/contract/adapter/MCP assets present, no bundled application residue, Node 20+ runtime. (Product-workspace checks — pins, SSOT, tokens, env leaks, backend status — run downstream where `src/`/`convex/` exist) |
| `pnpm onboard [provider] --host <host>` | provider-selective status or the next resumable human step |
| `pnpm connect` | begin, inspect, resume, or cancel safe service-connection receipts |
| `pnpm provider:login <cli>` | install + browser-OAuth pair a vendor's official CLI (stripe, netlify, github, 21st) |
| `pnpm stripe:provision` | webhook endpoint and plan prices through the paired CLI; secrets flow straight into Convex env, never printed. `--test-key` copies the CLI's TEST key so a sandbox can take a 4242 payment; it refuses anything that is not `sk_test`/`rk_test`, and refuses `--prod` |
| `pnpm secret:set NAME` | hidden-input prompt in the user's terminal, piped into Convex env — no chat, no history, no files |
| `pnpm setup:auth [site-url]` | generate `BETTER_AUTH_SECRET` and set it plus `SITE_URL` straight into a downstream Convex env; the secret is never printed |
| `pnpm agent:work` | durable dependency-aware work queue shared across supported AI hosts |
| `pnpm ui:plan <init\|check>` | create or validate the product's visual-direction contract before authored UI |
| `pnpm component:list [query]` | list reusable local components before catalog discovery |
| `pnpm ui:review capture --base-url <local-url>` | capture the declared route/state/theme/viewport matrix with browser audits |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moasq/agentic-ship](https://github.com/moasq/agentic-ship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
