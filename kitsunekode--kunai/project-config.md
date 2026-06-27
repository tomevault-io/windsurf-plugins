---
trigger: always_on
description: Kunai is a terminal-first Bun CLI that finds playable direct-provider video streams and hands them off to `mpv`.
---

# Kunai — Agent Entry Point

Kunai is a terminal-first Bun CLI that finds playable direct-provider video streams and hands them off to `mpv`.

## Documentation Philosophy

- Give agents routing, topology, and hard boundaries
- Keep process overhead low and let the model reason from code
- Document only the constraints that are expensive to rediscover
- Prefer pointers to deep docs over repeating the same facts everywhere

## Core Priorities

- Runtime: use `bun`, `bunx`, `bun run`
- Reliability first
- Performance matters, but not at the cost of correctness
- Keep behavior predictable during failure, recovery, and provider churn
- Make failures diagnosable with enough logging and context to reason about them
- Avoid leaving the terminal in a broken or confusing state
- If a tradeoff is required, choose correctness and robustness over short-term convenience

## Maintainability

- Long-term maintainability is a core priority
- Before adding new functionality, check whether shared logic should be extracted
- Duplicate logic across files is a design smell and should usually be refactored
- Do not solve problems with isolated local patches when the cleaner fix is a shared abstraction
- Do not be afraid to reshape existing code when it improves the long-term design

## Bun-first runtime (CLI)

- Prefer `Bun.spawn`, `Bun.which`, `Bun.connect` (Unix sockets), and `Bun.sleep` for deliberate delays on Bun-only code paths.
- Prefer `Bun.file` / `Bun.write`, or shared helpers such as [`apps/cli/src/infra/fs/atomic-write.ts`](apps/cli/src/infra/fs/atomic-write.ts) (`writeAtomicJson`), for whole-file JSON when you do not need append semantics or special permission flags.
- Prefer Node `fs` when you need append (`appendFile`), crash-safe atomic replace (temp file in the target directory + `rename`), `copyFile` with mtime checks, sync `mkdir` next to SQLite bootstrap, or tight `existsSync` / `unlink` sequences on mpv Unix socket paths.
- Prefer `setTimeout` when you need cancellable deadlines (for example mpv IPC per-command timeouts with `clearTimeout`).
- Prefer `node:crypto` synchronous hashing for small hot-path keys when async Web Crypto would add complexity without benefit.
- Do not change APIs for style alone; keep Node where behavior or cross-platform semantics are clearer.

## Read This First

- Start here for commands, routing, and repo-wide invariants
- Read [.docs/architecture.md](.docs/architecture.md) before changing loops, playback flow, scraping, caching, history, or data ownership
- Read [.docs/architecture-v2.md](.docs/architecture-v2.md) before changing target monorepo, daemon, web, desktop, package, or cache boundaries
- Read [.docs/runtime-boundary-map.md](.docs/runtime-boundary-map.md) before deciding whether work belongs in CLI app, app-shell, services, infra, providers, storage, core, or legacy
- Read [.docs/experience-overview.md](.docs/experience-overview.md) before changing user-facing scope, disclaimers, supported/unsupported behavior, or broad product messaging
- Read [.docs/product-prd.md](.docs/product-prd.md) before broad UX or product-shape changes
- Read [.docs/engineering-guide.md](.docs/engineering-guide.md) before broad refactors, service extraction, caching changes, or implementation-structure work
- Read [.docs/ux-architecture.md](.docs/ux-architecture.md) before changing shell flow, hotkeys, overlays, diagnostics, or setup UX
- Read [.docs/diagnostics-guide.md](.docs/diagnostics-guide.md) before changing debug logs, diagnostics panels, subtitle evidence, provider tracing, or playback/history troubleshooting
- Read [.docs/debugging-map.md](.docs/debugging-map.md) before broad reliability/debugging sweeps across playback, providers, storage, presence, or diagnostics
- Read [.docs/providers.md](.docs/providers.md) before adding or changing providers
- Read [.docs/playback-timing-and-aniskip.md](.docs/playback-timing-and-aniskip.md) before changing IntroDB/AniSkip fetch, MAL resolution, `PlaybackTimingFetchContext`, or auto-skip metadata wiring
- Read [.docs/provider-intake.md](.docs/provider-intake.md) before researching or hardening a provider, especially for new sites or major scraper changes
- Read [.docs/provider-examples.md](.docs/provider-examples.md) before implementing a new provider shape from scratch
- Read [.docs/design-system.md](.docs/design-system.md) before changing terminal UI styling or interaction patterns
- Read [.docs/ui-redesign-playbook.md](.docs/ui-redesign-playbook.md) when doing a major shell polish or redesign pass
- Read [.docs/testing-strategy.md](.docs/testing-strategy.md) before adding tests, changing test seams, or introducing new provider/runtime behaviors
- Read [.docs/repo-infrastructure.md](.docs/repo-infrastructure.md) before changing CI, Husky, lint-staged, issue templates, or PR templates
- Read [.docs/recommendations-and-discover.md](.docs/recommendations-and-discover.md) before changing `/discover`, recommendation services, or recommendation UI
- Read [.docs/presence-integrations.md](.docs/presence-integrations.md) before changing Discord Rich Presence or other social status integrations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KitsuneKode/kunai](https://github.com/KitsuneKode/kunai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
