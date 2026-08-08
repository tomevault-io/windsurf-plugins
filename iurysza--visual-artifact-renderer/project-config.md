---
trigger: always_on
description: > Map, not manual. Start here, then follow pointers. Keep this file short so the task stays in context.
---

# AGENTS.md — Visualizer

> Map, not manual. Start here, then follow pointers. Keep this file short so the task stays in context.

Visualizer is a **JSON-to-UI runtime**: agents emit a constrained artifact spec, the Pi extension delegates validation/storage to the CLI, and a Next.js renderer maps each node to a trusted adapter. The LLM never writes React, routes, JSX, imports, CSS, or full HTML for the renderer.

## Where knowledge lives

| Need | Read |
|---|---|
| Map of all docs | [`ai-artifacts/docs/index.md`](./ai-artifacts/docs/index.md) |
| Agent-first principles | [`ai-artifacts/docs/CORE_BELIEFS.md`](./ai-artifacts/docs/CORE_BELIEFS.md) |
| Senior-engineer handoff | [`ai-artifacts/AGENT_ONBOARDING.md`](./ai-artifacts/AGENT_ONBOARDING.md) |
| System architecture and tradeoffs | [`ai-artifacts/ARCHITECTURE.md`](./ai-artifacts/ARCHITECTURE.md) |
| Domain concepts, layers, data flow | [`ai-artifacts/SEMANTIC_MAP.md`](./ai-artifacts/SEMANTIC_MAP.md) |
| Design system and node philosophy | [`ai-artifacts/docs/DESIGN.md`](./ai-artifacts/docs/DESIGN.md) |
| Frontend/renderer guide | [`ai-artifacts/docs/FRONTEND.md`](./ai-artifacts/docs/FRONTEND.md) |
| Testing and verification | [`ai-artifacts/docs/RELIABILITY.md`](./ai-artifacts/docs/RELIABILITY.md) |
| User-facing setup and usage | [`README.md`](./README.md) |
| Model-facing routing and tool usage | [`skill/SKILL.md`](./skill/SKILL.md) |
| Node type reference | [`ai-artifacts/docs/nodes.md`](./ai-artifacts/docs/nodes.md) |

## Core principles

1. **JSON, not code.** The agent surface is the exported contract. Run `visual-artifact contract` to see it. Never generate React, routes, JSX, imports, CSS, or full HTML for the renderer.
2. **The contract is the handshake.** `shared/src/contract.ts` is the shared source of truth; `app/src/lib/contract/artifact-schema.ts` + `app/src/lib/contract/artifact-manifest.ts` consume it and `pnpm export:contract` writes `cli/assets/contract.json`. Run `pnpm export:contract` after schema or manifest changes.
3. **Validate at boundaries.** CLI/Pi tool validates before writing; renderer parses with Zod before rendering.
4. **Single sources of truth.** URL/path math lives in `app/src/lib/artifacts/paths.ts`. Source and installed runtimes share `~/.agents/skills/visual-artifact/artifacts/<project>/<slug>/artifact.json` by default.
5. **Small, well-named files.** Prefer scoped adapter files over monolithic registries.
6. **Types are documentation.** Push semantic meaning into names. Use Zod to make illegal specs unrepresentable.
7. **OpenSpec changes are local-only.** Keep `openspec/changes/` planning packages out of git. They can exist locally for discovery/implementation, but PRs must not add change specs or task markdown.

## Before committing renderer/schema changes

```bash
cd app
pnpm lint
pnpm export:contract
pnpm verify:artifacts
pnpm build
```

Run `pnpm visual:qa` if you touched adapters or styling.

## Fast dev environment

### Server roles

| Port | Server | When to use |
|---|---|---|
| `:9999` | `pnpm dev` (Next dev, HMR) | **Default dev server.** Editing renderer code, live mode, viewing artifacts with the latest source. |
| `:9998` | `visual-artifact serve` (CLI static server) | **Static-export preview** of the built `out/`, and what `create` auto-starts to open a created artifact. |
| `:8400` | Impeccable live helper | Live-mode control plane (bar + `/poll`). Independent of the app server. |

`pnpm dev` and `visual-artifact serve` used to share `:9999` and collide. They are now split: dev/HMR on `:9999`, static preview on `:9998`. Live mode needs HMR, so it always targets `:9999` (`pnpm dev`).

```bash
cd app
pnpm dev          # http://localhost:9999/  (dev + HMR + live mode)

cd ../cli
bun run src/main.ts serve --no-open  # http://localhost:9998/  (static preview + live JSON)
```

The CLI is self-contained under `cli/`. `visual-artifact bootstrap` builds `app/`, compiles the CLI, and symlinks the binary into `~/.pi/bin/`.

## After landing new features

New components, schema changes, or CLI behavior changes need to be rebuilt and reinstalled before the Pi extension and CLI reflect them. After the build checks pass, prompt the user to run:

```bash
cd app
pnpm lint
pnpm export:contract
pnpm verify:artifacts
pnpm build
cd ..
visual-artifact bootstrap
```

Then tell them to run `/reload` in Pi or restart Pi to load the updated extension.

## Live visual iteration with Impeccable

Use the Impeccable `live` command to iterate on UI in the browser. The helper runs in the background and injects a global bar into the page so the user can pick an element, request a design action, and get generated variants without leaving the browser.

### When to use

Use live mode only when the user wants to iterate on the renderer **in the browser** — pick a specific element they are looking at, request a design action, and get generated variants without leaving the page.

Trigger it on phrases like:
- "live iterate on this UI" / "use live mode" / "let me pick an element and redesign it"
- "make *this* bolder/better/quieter" (pointing at an element they can see but will not describe in text)
- "shape this element" / "redesign the thing I'm clicking"

Do **not** use it for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iurysza/visual-artifact-renderer](https://github.com/iurysza/visual-artifact-renderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
