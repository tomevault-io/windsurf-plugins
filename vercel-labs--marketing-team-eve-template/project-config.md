---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

This repository holds a team of marketing agents built on the [eve](https://eve.dev) agent framework. The root **lead** agent holds the shared product picture (brand context in Vercel Blob) and routes each request to one specialist: `product-marketer` for positioning and messaging, `content-marketer` for long-form pieces, `social-media-coordinator` for short-form posts and the Typefully queue, `seo` for organic search work, or `email` for adapting copy into mail and running Resend. The five specialists have no subagents of their own: each does its own web research and its own review pass inline. The lead's workflow lives in `agent/instructions.md`; each specialist's lives in `agent/subagents/<id>/instructions.md`.

The lead picks a specialist by reading `description` in each `agent.ts`, so adding a specialist means adding a directory. Nothing in `agent/instructions.md` enumerates them, and nothing should.

The whole agent is defined under `agent/`. eve discovers capabilities from the filesystem. See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for the component map, data flow, and boundaries.

## Setup & commands

```bash
pnpm install        # install dependencies (Node 24.x)
pnpm dev            # eve dev — local TUI; run /model once to link a model provider
pnpm typecheck      # tsc (TypeScript, no emit)
pnpm check          # ultracite (Biome) lint + format check
pnpm fix            # ultracite (Biome) auto-fix
pnpm build          # eve build
eve deploy          # deploy to Vercel production (use this, not raw `vercel deploy`)
npx eve info        # print the discovered surface + discovery diagnostics
pnpm validate       # check + typecheck + eve info in one command
```

There is no unit-test suite. **Verify changes with `pnpm validate` (lint, typecheck, and discovery diagnostics must all report 0 errors / 0 warnings), then exercise the agent in the `pnpm dev` TUI.**

`npx eve info` is the fastest way to confirm a change landed: it prints every discovered tool, skill, connection, and subagent. When a file you added doesn't show up there, discovery didn't classify it as an authored slot, and `.eve/discovery/diagnostics.json` says why.

## eve conventions

- **Read the relevant guide in the installed eve package's `docs/` before writing code.** Don't invent framework APIs; confirm them against the docs. Under pnpm the real path is `node_modules/.pnpm/eve@<version>_<hash>/node_modules/eve/docs/` — resolve it with `ls -d node_modules/.pnpm/eve@*/node_modules/eve | head -1`, because plain `node_modules/eve/docs/` globs don't resolve.
- **Identity comes from the filesystem, never a `name` field.** The tool at `agent/tools/save_brand_context.ts` is the tool `save_brand_context`; the subagent at `agent/subagents/content-marketer/` is the tool `content-marketer`.
- Authored slots: `agent/agent.ts` (model), `agent/instructions.md` (system prompt), `agent/tools/*.ts` (`defineTool`), `agent/connections/*.ts`, `agent/channels/*.ts`, `agent/skills/<name>/SKILL.md`, `agent/subagents/<id>/agent.ts` (`defineAgent`), `agent/sandbox.ts`. `agent/lib/` holds plain modules, not a slot. The same slots nest: every subagent directory can carry its own `instructions.md`, `tools/`, `connections/`, `skills/`, `sandbox.ts`, and `subagents/`.
- **Subagents inherit nothing.** Every declared subagent runs in a fresh child session with none of the parent's skills, connections, tools, or sandbox, so the caller packs everything into the `message`. This is why all five specialists have their own `get_brand_context` tool, their own `sandbox.ts`, and their own `connections/notion.ts`. Those six Notion copies (five specialists plus the root) are identical, so edit them together; `md5 -q $(find agent -name notion.ts -path '*connections*') | sort -u | wc -l` should print 1.
- **Tools** run in the app runtime (full `process.env`), one default export per file. Gate destructive tools with `approval` from `eve/tools/approval`. **Connections** accept the same `approval` field: `notion.ts` substring-matches an `APPROVAL_REQUIRED_TOOLS` list, and `typefully.ts` goes further, gating deletes unconditionally but gating `create_draft`/`edit_draft` only when the call actually schedules (`requestBody.publish_at` is set), so saving a plain draft stays friction-free. `resend.ts` is the one connection that also narrows which tools the model can discover at all, with `tools.allow`; see [The email agent's two boundaries](#the-email-agents-two-boundaries).
- **Skills** are load-on-demand. Every packaged skill (`<name>/SKILL.md`) requires `description` frontmatter; that description is the routing hint and the only thing the model sees before loading. The `references/` files under a skill require a sandbox to materialize, so an agent with reference files needs its own `sandbox.ts`. The frontmatter is YAML, so an unquoted `description` containing a colon followed by a space fails to parse and the skill is dropped with a `discover/skill-frontmatter-invalid` error. Rephrase around the colon rather than quoting, to match the rest of the descriptions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/marketing-team-eve-template](https://github.com/vercel-labs/marketing-team-eve-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
