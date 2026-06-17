---
trigger: always_on
description: This repository is the public source package for Balaur, a local-first personal agent shipped as a Bun/Ink CLI named `balaur`.
---

# Balaur public project instructions

This repository is the public source package for Balaur, a local-first personal agent shipped as a Bun/Ink CLI named `balaur`.

Balaur follows a small-core, local-first design: a transparent runtime with capability pushed into composable TypeScript modules, Markdown skills, CLI tools, and client adapters. This file is injected into agent context, so keep it lean and high-signal — add a rule only when it changes a real decision.

## Working style

- Prefer direct, practical implementation steps.
- Keep solutions KISS, inspectable, and reversible.
- Use TypeScript/JavaScript for Balaur product runtime code.
- Use Bun as the default command surface: `bun install`, `bun run <script>`, and `bun <file>`. Do not use `npm` or `node` for routine Balaur work unless a concrete dependency forces it; if so, say why.
- If Bun is installed but not on `PATH`, prefer the explicit local path (`~/.bun/bin/bun`) over falling back to npm/node.
- Keep the private vault, generated context, journals, OAuth tokens, local model artifacts, and runtime credentials out of git.
- Prefer Balaur runtime modules, Markdown skills, package manifest settings, and documented hooks over wrapper scripts or framework patches.
- Keep host operating-system setup outside this repository; document only portable environment variables and extension-level configuration here.
- On Windows, install every Balaur host dependency through `winget` when a winget package exists; ask before using Chocolatey, Scoop, manual downloads, or ad-hoc installers.

## Product shape: Balaur CLI only

- Balaur ships as a standalone CLI named `balaur`, with a Bun/Ink interface and a Bun single-file build target.
- Use only `@earendil-works/pi-ai` and `@earendil-works/pi-agent-core` from the Pi ecosystem. Do not depend on `@earendil-works/pi-coding-agent`, Pi extensions, Pi themes, Pi TUI, Pi SDK sessions, or Pi package manifests.
- **No MCP.** Expose capability as Balaur runtime tools, CLI commands, or Markdown skills — not as MCP servers.
- **No sub-agent frameworks, no permission gates, no bespoke plan/todo engines** baked into Balaur. Assemble from primitives only when a concrete need exists.
- Keep context transparent: prefer durable state in on-disk files (Johnny Decimal vault entries, master conversation, compacted sub-conversations, skills, setup config) over hidden in-session state.

## Architecture patterns

- **Terminal avatar identity is native ANSI-only.** Balaur's daily terminal UI uses one portable renderer fed by canonical PNG sprite sheets. Keep only sextant and octant renderers; do not carry forward half-block output. Do not add terminal-specific image-protocol backends, runtime graphics dependencies, cache builders, or hand-maintained terminal art.
- **23×11 cells is the default avatar review target.** Design and judge avatars at 23 columns × 11 rows (`BALAUR_AVATAR_ROWS=11`, the default). Keep source art as 3×3, 9-frame PNG sheets with 256×256 frames; review generated ANSI instead of editing renderer output.
- **Focused runtime entry points.** CLI entrypoints live in `src/`. Keep entrypoints focused on orchestration; move reusable or testable logic into `lib/*.ts`.
- **Small, single-purpose modules.** Split features into modules such as `paths.ts`, `sqlite.ts`, `vault.ts`, runtime modules, or avatar/design helpers. Pure/format/parse helpers belong in `lib/` so they are unit-testable without Pi or I/O. Treat a module past ~500 lines as a smell to decompose, not extend.
- **Shared helpers live in `lib/`.** Reuse `lib/paths.ts`, `lib/sqlite.ts`, runtime helpers, and small focused modules instead of re-implementing path resolution, SQLite loading, or UI rendering per feature.
- **Runtime events:** keep `lib/runtime/events.ts` small and typed. Long-lived resources must have explicit cleanup through the runtime `close()` path.
- **Tools:** use `AgentTool` from `@earendil-works/pi-agent-core` with clear `name`, `label`, `description`, TypeBox `parameters`, and an `execute` that returns inspectable `content`/`details`. Keep tool output bounded and avoid leaking private paths or secrets.
- **Commands and UI:** implement Balaur slash commands in the runtime/Ink layer. Headless and piped-input paths must still be safe.
- **Persistent state:** survive restarts via on-disk vault/config/model files resolved lazily from `lib/paths.ts`, not module globals. Avoid module-level singletons except for trivial same-process coupling that is documented and not persisted.
- **OS-agnostic by construction.** Centralize platform/`env` branching and inject it so resolution is unit-testable per OS. Resolve Windows/macOS/Linux paths explicitly; gate host commands behind the right platform.

## Coding style

- **Erasable TypeScript only.** Do not use `enum`, `namespace`, constructor parameter properties, or other syntax that needs emit. Use `type`/`interface`, `as const`, and TypeBox/string literal unions instead.
- Prefer pure functions, early returns, and plain data over classes and inheritance. No premature abstraction or config "frameworks."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexradunet/nazar-studio](https://github.com/alexradunet/nazar-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
