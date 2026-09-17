---
trigger: always_on
description: AgentStart is a Rust daemon with a Chrome MV3 workbench and a native SwiftUI iOS companion for running
---

# AGENTS.md

AgentStart is a Rust daemon with a Chrome MV3 workbench and a native SwiftUI iOS companion for running
coding agents across many git worktrees on local, WSL, and SSH hosts.

This file is the contract for every agent writing code here: structure, naming, cohesion, code
quality. For browser visual work, [`docs/style-guide.md`](./docs/style-guide.md) is canonical. For
mobile visual work, use [`apps/mobile/DESIGN.md`](./apps/mobile/DESIGN.md). For the daemon's runtime
ownership and transport boundaries, [`apps/daemon/ARCHITECTURE.md`](./apps/daemon/ARCHITECTURE.md).

**The organizing principle.** Tailwind won because the style lives next to the markup: one place to look, one place to change. Apply that to all code. A feature's directory, filenames, and module boundaries exist so an agent can find the code from the feature name alone and change it without touching the rest of the tree. Optimize for *"where does this live?"* being answerable in one guess.

---

## Read first

These are hard rules regardless of who enforces them. Some fail CI; the rest are convention plus
review. The table says which is which — "no automated check" does not mean "not a rule." No
exceptions inside a feature task.

| Never | Enforced by |
| --- | --- |
| Write or retain any test, smoke check, E2E harness, or validation-only script. | Section 9 |
| Add an `eslint-disable`/`oxlint-disable max-lines`, a max-lines baseline, or a `#[allow(…)]` that silences clippy | Convention only |
| Add a hand-written project-owned `.d.ts`; generated platform bindings are the only exception | Convention only |
| Add a variable to the `@theme inline` block in `packages/client/src/assets/main.css` | Convention only |
| Use a native `<button>`/`<input>`/`<textarea>`/`<select>` in client feature TSX; use `bg-black/N`-style alpha washes; import `ui/*-styles.ts` from feature code | Convention only |
| Use `interface`, `enum`, `namespace`, or `any` | oxlint + `erasableSyntaxOnly` |
| Write `unsafe`, `.unwrap()`, or `panic!` in daemon code | `unsafe_code = "forbid"` + Section 4a |
| Ship a user-visible string that isn't wrapped in `t()` / `translate()` | Convention only |
| Hardcode `e.metaKey`, a path separator, or a platform font | Section 7 |
| Import daemon or extension-host implementations from `packages/client/src` | Section 1 |
| Reach a client or host across the process boundary through anything but a protobuf capability declared in `packages/protocol/proto/` | Section 1 |
| Name a file or folder `helpers`, `utils`, `common`, `misc`, or `shared-stuff`; add an `index.ts` re-export barrel | Section 2 |
| Rename or move a file without updating the paths written as *strings* — build scripts, CI jobs, allowlists, `Why:` comments | Convention only |
| Follow an absolute path from a subagent result into the main repo instead of this worktree | Section 12 |

---

## 1. Structure: a feature is a folder

```
apps/
  computer-use-macos/
                standalone signed SwiftPM helper for macOS accessibility and screen capture
  daemon/       Rust runtime, CLI, Native Messaging host, host adapters, and authoritative state
  extension/    WXT-managed Chrome MV3 host: background, side panel, DevTools, browser bootstrap
  macos/        SwiftPM menu bar host that supervises the daemon binary on macOS
  mobile/       native SwiftUI app and widgets
packages/
  client/       source-only browser workbench UI consumed through declared exports
  protocol/     protobuf source of truth plus generated Swift, Rust, and TypeScript bindings
  cli/          npm/bunx installer shim for compiled daemon releases
skills/         agent skill packages shipped to end users, one folder per skill
scripts/        workspace-level tooling whose inputs cross package boundaries
```

**Scripts live at the level they serve.** `scripts/` holds only tooling whose inputs cross app
boundaries, and the root `package.json` owns their npm scripts. Everything
scoped to one app lives in that app's own `apps/<app>/scripts/`, invoked from
that app's `package.json`. A script that reaches outside its app is in the wrong
folder; `config/` is for configuration, never executables. A script also has to earn its file:
if a one-line `package.json` task can express it, it does not exist — and every surviving script
opens with a `Why:` line naming the reason it can't be one. An unexplained script is a deletion
candidate. Durable scripts exist only for necessary product workflows such as starting, building,
packaging, publishing, deploying, or generating application artifacts. A script written only to
verify a task is temporary: run it, remove it before finishing, and never add it to a package script
or CI workflow.

`skills/<name>/SKILL.md` is product content, not app source. It sits at the repository root because
it is shipped to users' agent installs rather than built into one client, and it is the source of
truth for that skill.

**Import direction is one-way.** `packages/client` may import pure contracts and models, but never
`apps/extension`, Node, or Chrome globals — and `apps/daemon` is a separate Rust binary it cannot
import at all, only call over the protocol. `apps/extension` imports only declared

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xinyao27/agentstart](https://github.com/xinyao27/agentstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
