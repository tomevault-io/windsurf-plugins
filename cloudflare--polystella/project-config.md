---
trigger: always_on
description: PolyStella — an Astro integration that translates content into
---

# AGENTS.md

PolyStella — an Astro integration that translates content into
additional locales at build time using AI, caches translations in
Cloudflare R2, and injects locale-prefixed routes.

This file is the entry point for coding agents working **on the
PolyStella package itself**. Three companion docs:

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — system design, invariants,
  glossary, per-subsystem reference. The "why" answers.
- [`skills/polystella-contributor/SKILL.md`](./skills/polystella-contributor/SKILL.md)
  — step-by-step recipes for common contributor tasks (add an
  adapter, add a CLI subcommand, debug a translation, etc.).
- [`skills/polystella-consumer/SKILL.md`](./skills/polystella-consumer/SKILL.md)
  — for agents working in a **downstream Astro project** that
  depends on this package.

All cross-references use stable slug anchors (`#cache-key`), not
section numbers. Inserting new sections never breaks links.

---

## Commands

| Command                  | What it does                                                                                                                                                                                                 |
| :----------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pnpm test`              | Run vitest (1104 tests / 56 files / ~1.2s at time of writing).                                                                                                                                               |
| `pnpm test:watch`        | Vitest in watch mode.                                                                                                                                                                                        |
| `pnpm build`             | Compile `src/` → `dist/` via `tsc -p tsconfig.build.json` (mirrored layout, `.js` + `.d.ts` + sourcemaps + declaration maps). Produces the standalone `polystella` CLI at `dist/cli.js` and library entries. |
| `pnpm exec tsc --noEmit` | Typecheck against the root `tsconfig.json` (which includes tests). The build config (`tsconfig.build.json`) sets `noEmit: false` and narrows `include` to `src/**`.                                          |
| `pnpm changeset`         | Add a Changesets entry for package-affecting work. Use `pnpm changeset add --empty` only for changes that intentionally do not need a package release.                                                       |

No lint step yet.

> Test counts age. The authoritative count is `pnpm test`'s output;
> the number here is a snapshot pinned by [`tests/docs.test.ts`](./tests/docs.test.ts).

---

## Where do I make changes?

Task → entry-point file(s) → key contract → deep-dive link.

| Task                                                   | Entry point                                                                                      | Contract                                                              | See                                                                                                                                                 |
| :----------------------------------------------------- | :----------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| Add a file-format adapter                              | `src/parsing/adapters/<name>.ts`; register in `src/parsing/registry.ts`                          | `FileTypeAdapter` in `src/parsing/adapter.ts`                         | [#adapter-contract](./ARCHITECTURE.md#adapter-contract); recipe in [contributor SKILL](./skills/polystella-contributor/SKILL.md#add-adapter)        |
| Add a CLI subcommand                                   | Handler in `src/cli/<name>.ts`; register in `src/cli.ts` (`parseSubcommand` + switch)            | Argv parser + `run<Name>(args, deps)`                                 | Recipe in [contributor SKILL](./skills/polystella-contributor/SKILL.md#add-cli-subcommand)                                                          |
| Add a translation provider                             | New branch in `createTranslator` (`src/translation/provider.ts`)                                 | `Translator` interface; permanent vs retriable error classification   | [#translator-contract](./ARCHITECTURE.md#translator-contract); recipe in [contributor SKILL](./skills/polystella-contributor/SKILL.md#add-provider) |
| Change cache key formula                               | `src/storage/hash.ts`                                                                            | **Invariant 1** — cache-wide invalidation                             | [#cache-key](./ARCHITECTURE.md#cache-key)                                                                                                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/polystella](https://github.com/cloudflare/polystella) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
