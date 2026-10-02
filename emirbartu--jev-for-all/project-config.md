---
trigger: always_on
description: Jev for every agentic development workflow. This repo is the home for using Jev, TypeSafe's
---

# AGENTS.md

Jev for every agentic development workflow. This repo is the home for using Jev, TypeSafe's
System One decision model, wherever an agent codes. It targets OpenCode, Claude Code, Hermes and
pi today, so every harness gets the same decisions from one shared contract. Shipped today: an
OpenCode V2 plugin that routes skill and tool choices through Jev, skill-selection adapters for
Claude Code, Hermes and pi, and a `browser_task` server for any MCP client. Plugin id:
`jev-for-all`.

## What Jev is

Read this before touching decision logic.

Jev is not an LLM. It generates no text, writes no code, and calls no tools. It is the first
System One model: unstructured `state` in, typed answers out. The whole design of this repo
follows from three properties. Answers are typed by construction, every answer carries a
calibrated confidence, and each call returns in roughly 70 to 500 ms.

- Transport here: `@openrouter/sdk` -> `client.alpha.decisions.create` -> `POST
  https://openrouter.ai/api/alpha/decisions` with `{ model, state, questions }`.
- Question and answer shapes (only the first two are used in this repo):
  - `choice` `{ type, instructions, criteria }` -> `{ type: "choice", choice, probabilities:
    Record<option, number>, confidence }`
  - `noul` `{ type, instructions }` -> `{ type: "noul", noul: 0..1 }`, the probability that the
    statement is true.
  - `score` -> `{ score, probabilities, confidence }`, available from Jev but unused here.
- `state` is a string, JSON object or array. The keys of `criteria` are the only values a
  `choice` can return, so Jev cannot invent a tool or skill name. The plugin still re-validates
  ids against the live catalog before using an answer.
- Default model `~typesafe/jev-latest`. The alias resolves to `jev-1.13.0`, and the response's
  `model` reports the versioned id. OpenRouter also serves a stable, TypeSafe-SDK-compatible
  endpoint at `/api/v1/systemone` (bare ids like `jev-1.13` map to the `typesafe/` namespace).
  That matters only if the alpha transport needs replacing.
- Limits: 64k tokens per request, 32k for `state` plus the longest question, about 1,200 req/min,
  $0.042 per Mtok input, output free.
- TypeSafe's own docs state Jev is not a drop-in coding-agent model. This repo is the intended
  pattern: keep the LLM coding agent, and let Jev make the decisions.

Sources (fetch the `.md` variants for clean text; do not re-derive from the web):
- https://docs.typesafe.ai/introduction/coding-agents.md
- https://docs.typesafe.ai/primitives.md
- https://docs.typesafe.ai/cookbooks/skill_suggestion.md, the gate, rank and rerank design in
  `src/skills.ts`
- https://docs.typesafe.ai/cookbooks/function_calling.md, the Choice-over-tools design in
  `src/tools.ts`
- https://openrouter.ai/docs/api/api-reference/alphadecisions/submit-a-decisions-request.md

## What this plugin does and why

A coding agent sees its whole skill catalog and tool catalog every model step. It burns output
tokens deliberating, mis-loads skills, and the catalogs eat context. This plugin changes that:
Jev decides, code enforces, the agent acts.

- `prompt` hook (once per user message) -> Jev skill decision -> pushes at most one `{ id }` into
  `event.prompt.skills`, so OpenCode loads that skill at admission. `null` (no skill) is a
  decision, not a failure.
- `context` hook (every model dispatch, including tool-driven continuations) -> Jev tool decision
  -> filters `event.tools` in place to top-N plus `alwaysVisible`, and appends a
  `<system_one_routing>` hint to `event.system`. A low `needs_tool` value produces the hint only
  ("answer directly"), with no filtering.

Contract: fail open. A timeout, non-2xx response, malformed answer, low confidence or unknown id
means the plugin changes nothing in the request. It never throws out of a hook, and it warns once
per session (`createWarnOnce`).

Data egress: the `context` hook sends the conversation tail, including tool-result bodies (file
contents, shell output), rendered up to `tools.stateBudget` (6000 chars), plus tool names and
descriptions, to OpenRouter. The README "Privacy and fail-open behavior" section is the
user-facing disclosure. Update it whenever a change alters what leaves the machine. Excluding
tool-result bodies is not an option today.

## Commands

| Command | Notes |
| --- | --- |
| `bun run typecheck` | `tsc --noEmit`, strict. There is no build step; OpenCode runs `index.ts` with Bun. |
| `bun scripts/conformance.ts` | Shared 9-case conformance against the TS core and the Claude Code adapter. |
| `OPENROUTER_API_KEY=... bun scripts/jev-probe.ts decisions` | Live Jev smoke test (noul and choice). |
| `OPENROUTER_API_KEY=... bun scripts/jev-probe.ts catalog tools.json [task]` | Live tool-routing probe against a tool catalog. |
| `bun scripts/eval.ts` | Headless baseline-vs-routed usage eval. Needs `opencode` on PATH, provider auth, and optionally `EVAL_MODEL`. |

Install for use is a local path (`{ "package": "/abs/path/to/repo" }`); the package is not
published to npm.

Commits are authored as the repo's configured identity, `emirbartu <bartuekinci42@gmail.com>`,
and never overridden with an agent identity.

## Releases


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emirbartu/jev-for-all](https://github.com/emirbartu/jev-for-all) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
