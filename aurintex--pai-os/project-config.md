---
trigger: always_on
description: > **Start here**: This file serves as the primary index for AI agents (Cursor, Windsurf, Copilot) to understand the paiOS project structure, conventions, and active tasks.
---

# AI Agent Context Map

> **Start here**: This file serves as the primary index for AI agents (Cursor, Windsurf, Copilot) to understand the paiOS project structure, conventions, and active tasks.

## 1. Project Identity
**paiOS** is an open-source operating system for Personal AI Hardware (paiBox, paiScribe).
- **Core Principle**: Privacy-first, local-only inference.
- **Architecture**: Hexagonal, Monorepo, Rust-based Engine.

## 2. Critical Context Files
Before writing code, **always** check these files for the latest standards:

| Topic | Source of Truth |
|-------|-----------------|
| **Architecture** | `docs/src/content/docs/architecture/adr/index.mdx` (ADRs) |
| **Workspace & Build** | `docs/src/content/docs/architecture/workspace-and-build.mdx` (engine/ layout, feature flags, crates) |
| **Coding Style** | `docs/src/content/docs/guides/contributing/standards.mdx` |
| **Rust Style & Best Practices** | `docs/src/content/docs/guides/contributing/rust-style.mdx` (stack vs heap, generics vs `Box<dyn Trait>`, embedded) |
| **Workflow** | `docs/src/content/docs/guides/contributing/workflow.mdx` |
| **Glossary** | `docs/src/content/docs/glossary.mdx` (link here when using defined terms) |
| **Task Status** | `.taskmaster/tasks/tasks.json` (via `task-master` CLI) |

## 3. Tool Usage Rules
- **Search first**: Use `grep` / `glob` to find existing patterns before inventing new ones.
- **No hallucinations**: If you need a library, check `Cargo.toml` or `package.json` first.
- **Tests**: All new features require unit tests. Run `cargo test` (Rust) or `npm test` (JS/TS).
- **GitHub issues**: When creating issues (e.g. via MCP or CLI), use **Conventional Commits** for the title: `type(scope): short description` (e.g. `feat(common): add config format detection`). See [Workflow](docs/src/content/docs/guides/contributing/workflow.mdx) for types, scope, and branch naming.
- **Documentation**: When editing docs, link to the [Glossary](docs/src/content/docs/glossary.mdx) for terms that have an entry (e.g. IPC, gRPC, HITL, UDS). Use `[Term](/glossary/#letter)`.
- **Rust code**: When writing or reviewing Rust in `engine/`, follow [Rust Style and Best Practices](docs/src/content/docs/guides/contributing/rust-style.mdx) (stack vs heap, generics vs `Box<dyn Trait>`, formatting).
- **Rust formatting (CI parity)**: Before you commit or open a PR for Rust changes, run the same check as CI from the repo root: `cd engine && cargo fmt --all -- --check`. If it fails, run `cargo fmt --all` in `engine/` and re-check. Optional: install [pre-commit](https://pre-commit.com/) and run `pre-commit install` once so commits are blocked locally when fmt would fail (see `.pre-commit-config.yaml`). With the repo `.vscode/settings.json`, Rust files format on save when using rust-analyzer in Cursor/VS Code.

## 4. Architecture Summary
- **Engine**: Rust daemon (`engine/`) running distinct threads for inference (NPU/CPU/GPU).
- **IPC**: gRPC over Unix Domain Sockets.
- **Docs**: Astro Starlight site (`docs/`).

## 5. Active Focus
Check the Task Master for the current priority:
`task-master next`

## 6. Cursor MCP (optional)
The repo ships with **Taskmaster** in `.cursor/mcp.json` so contributors can try it without setup (“ah, this could be useful”). Task data stays local (see [ADR-007](docs/src/content/docs/architecture/adr/007-project-management-strategy.mdx) and [ADR-009](docs/src/content/docs/architecture/adr/009-ai-context-strategy.mdx)); GitHub issues remain the source of truth. For MCP AI features (parse-prd, expand, research, etc.), add at least one provider API key: replace the placeholder values in the `env` section of `.cursor/mcp.json` (e.g. `ANTHROPIC_API_KEY`, `PERPLEXITY_API_KEY`) with your keys. For CLI-only use, `.env` is enough. See [Task Master configuration](https://docs.task-master.dev/getting-started/quick-start/configuration-quick).

## 7. Excalidraw MCP (optional)
**Not all contributors have this installed.** Architecture diagrams are `.excalidraw` files in `docs/public/images/Architecture/`. They can be edited in the browser (e.g. [Excalidraw](https://excalidraw.com)) or via the optional [yctimlin/mcp_excalidraw](https://github.com/yctimlin/mcp_excalidraw) MCP for AI-assisted read/edit/iterate.

- **If you don’t use the MCP**: Edit `.excalidraw` files by opening them in Excalidraw or the VS Code Excalidraw extension. Docs embed from `docs/public/images/Architecture/`; no MCP required.
- **If you use the MCP**: Clone the repo to a location of your choice, run `npm ci && npm run build`, then start the canvas (e.g. `HOST=0.0.0.0 PORT=3000 npm run canvas` in that clone). Add the server to `.cursor/mcp.json` with `command`/`args` pointing at your `dist/index.js` and `env.EXPRESS_SERVER_URL` set to your canvas URL (default `http://localhost:3000`). See the project README for Cursor config examples. Tools: `import_scene` (load file), `describe_scene` / `get_canvas_screenshot` (inspect), `export_scene` (save).
- **When giving instructions**: Prefer “open/edit the `.excalidraw` file in Excalidraw” unless the user has confirmed they use the Excalidraw MCP.

## Learned User Preferences

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aurintex/pai-os](https://github.com/aurintex/pai-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
