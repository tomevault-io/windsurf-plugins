---
trigger: always_on
description: Moo is a terminal music player built with TypeScript, Bun, React 19, and OpenTUI.
---

## Project

Moo is a terminal music player built with TypeScript, Bun, React 19, and OpenTUI.

## Tech Stack

Bun runtime, React 19 + OpenTUI (TUI rendering), Drizzle ORM + SQLite, XState store + mutative (immutable updates with mutable syntax), RxJS, MPV (audio playback via IPC socket), MPRIS (media keys), ts-pattern, Remeda, typescript-result, Zod.

## Commands

- `bun dev` — run with watch
- `bun prod` — production run
- `bun compile` — build binary
- `bun types` — typecheck
- `bun lint` / `bun lint:fix` — oxlint
- `bun format` — prettier
- `bun db:generate` — regenerate Drizzle SQL migrations

## External Runtime Dependencies

- **MPV** — audio playback via IPC socket
- **FFmpeg** — tag writing
- **Nerd Font** terminal

## Architecture

Ports & Adapters (Hexagonal). **The one rule:** Core never imports infrastructure. Dependencies point inward.

| Layer           | Purpose                                                  | Can import from                                                       |
| --------------- | -------------------------------------------------------- | --------------------------------------------------------------------- |
| `core/`         | Pure state + reducers, command defs, playlist DSL schema | `core/`, `shared/`                                                    |
| `ports/`        | Interfaces (Player, AppDatabase, AppFileSystem)          | `ports/`, `shared/`                                                   |
| `adapters/`     | Implementations (MPV, SQLite, filesystem)                | `ports/`, `shared/`                                                   |
| `application/`  | Use cases, orchestration                                 | `core/`, `ports/`, `application/`, `shared/`                          |
| `ui/`           | React components + hooks                                 | `core/` (types), `ports/` (types), `app/` (context), `ui/`, `shared/` |
| `app/`          | Composition root, wiring                                 | everything                                                            |
| `shared/`       | Pure utilities, config, types                            | `shared/`, `ports/` (types), `core/` (types), `app/` (context)        |
| `test-helpers/` | Test mocks, fixtures, helpers                            | everything                                                            |

Enforced via oxlint `no-restricted-imports` overrides in `.oxlintrc.json`.

## Key Paths

- Entry: `source/app/index.ts` → `source/app/start.tsx` → `source/app/App.tsx`
- Context: `source/app/context.tsx`
- State: `source/core/state/state.ts`, types in `source/core/state/types.ts`
- Ports: `source/ports/player.ts`, `source/ports/database.ts`, `source/ports/filesystem.ts`
- DB: `source/adapters/sqlite/database.ts`, `source/adapters/sqlite/schema.ts`
- Player: `source/adapters/mpv/mpv.ts`
- Playback: `source/application/playback/playback.ts`
- Smart playlists: `source/core/playlists/` (DSL), `source/application/playlists/` (manager)
- Config: `~/.config/moo/`, Data: `~/.local/share/moo/`, Cache: `~/.cache/moo/`

## Tech Rules

- Never use abbreviations or single-letter variable names
- Always put a linebreak before the final return
- Early returns should have a line-break after them
- Noun-first naming, fixed order: **`<Entity><Part><Variant>`**. Use only the minimum words needed.
  Examples: `ImagePreview`, `ImagePreviewLarge`, `NodeSocketInput`, `AudioBandBass`
- Functions start with a verb: `createNode`, `getConnection`, `connectSocket`
- Main exports at the top of the file, implementation details below — readers see the API first (write a comment after them to prevent the linter from sorting them down)
- Keep functions short and focused — extract complex operators into named functions
- Add a brief `/** ... */` doc comment to non-trivial functions (5+ lines or non-obvious purpose)

**TypeScript**

- `const` only — no `let`. Use declarative expressions; `ts-pattern` for conditional assignments
- `type` over `interface` — never `any`
- Prefer `readonly` on types
- Discriminated unions over type assertions — make illegal states unrepresentable
- Use `satisfies` for type-safe object literals without widening
- Explicit return types on exported functions (except React components)
- Assert types, throw on failure — no coalescing fallbacks for type narrowing
- `undefined` over `null` for unset values
- `??` over `||`
- `function` keyword for components and longer functions, arrow functions for callbacks and inline helpers
- Named exports over default exports
- Reference existing types — never duplicate type literals
- No `else` — use early returns, ternaries, or `ts-pattern`
- Prefer `ts-pattern` `match()` over `switch` and `if-else` chains — especially in JSX
- Exhaustive checks with `never` in match/if chains
- Prefer Remeda (`pipe`, `entries`, `filter`, `map`, `fromEntries`) for multi-step collection transforms — cleaner than nested `Object.fromEntries(Object.entries(...)...)`
- Prefer declarative collection operations over imperative loops
- Prefer `pipe` + `O.maybe` (`source/lib/option.ts`) over manual null-check chains — chain optional lookups declaratively instead of nesting `if (x) { if (x.y) { ... } }`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vdawg-git/moo](https://github.com/vdawg-git/moo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
