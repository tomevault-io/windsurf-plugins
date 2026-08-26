---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Repo-wide conventions live here; each package's implementation detail lives in its own `packages/*/AGENTS.md` (see the convention below).
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Repo-wide conventions live here; each package's implementation detail lives in its own `packages/*/AGENTS.md` (see the convention below).

## Project overview

**Blue** is the interactive terminal UI for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (`dsh`). It is a renderer over the harness's [Cordis](https://github.com/deepseek-ai/cordis) plugin architecture, built on `@earendil-works/pi-tui`, and shipped as seven `@dsh-blue/blue-*` packages (all at version `0.1.0-rc.6` — the preview line, the number the website's tagline promises; blue-api first published with rc.3, the blue-cli launcher shell joins with its first release). The packages were extracted from the `deepseek-harness` monorepo; this standalone repository builds and tests against the published npm releases of the harness and vendored Cordis. Blue is **not** part of a default `dsh` installation — it is added to a `dsh` profile as an out-of-tree plugin bundle; the `@dsh-blue/blue-cli` shell (S37) wraps that into a single-command install carrying its own pinned host.

- Language: TypeScript (ESM only, `"type": "module"` everywhere).
- Runtime: Node `^22.19.0 || >=24.0.0`; package manager pnpm 11 (pinned, `pnpm@11.7.0`).
- Repository type: pnpm workspace + TypeScript project references; website CI via GitHub Actions (`.github/workflows/website-pages.yml`), no formatter (only oxlint).

## External documentation

Consult these while developing instead of guessing API shapes:

- pi-tui component/rendering model (the L0 renderer behind `packages/core`): <https://pi.dev/docs/latest/tui>
- DeepSeek Harness service and API reference (the host services Blue plugins consume): <https://deepseek-harness.github.io/deepseek-harness/reference/>
- In-repo design docs, indexed (living vs archived): [docs/README.md](docs/README.md)

## Repository layout

```
packages/
  core/         @dsh-blue/blue-core        — the tree's ONLY pi-tui adapter
  transcript/   @dsh-blue/blue-transcript  — session events → transcript rendering
  interaction/  @dsh-blue/blue-interaction — input editor, slash commands, dialogs, /yolo mode cycle
  app/          @dsh-blue/blue-app         — CLI startup + Agent driver
  bundle/blue/  @dsh-blue/blue             — installable bundle (cordis.patch.yml)
  cli/          @dsh-blue/blue-cli         — the `blue` launcher shell (global bin, S37)
script/install-dev.sh  — one-shot local dev install into a dsh profile
website/               — VitePress documentation site (@dsh-blue/website): zh source at the top
                         level, en mirror under website/en/; deployed to GitHub Pages (ADR D32)
.github/workflows/website-pages.yml — website Pages build-check-deploy CI
docs/                  — design docs, indexed by docs/README.md (living docs at the top level,
                         completed phase designs and surveys under docs/history/)
```

Each package has the same shape: `src/` (source), `tests/` (vitest specs), `lib/` (build output, git-ignored here but the runtime entry), its own `tsconfig.json` extending `tsconfig.base.json`, `README.md` + `README.zh.md` (bilingual user-facing docs — keep both in sync), and `AGENTS.md` (implementation detail for agents).

## Package quick reference

| Package | Import name | Role | Owns (key surfaces) | Detail |
|---|---|---|---|---|
| api | `@dsh-blue/blue-api` | stable renderer-independent public contracts and manifest validation | `BlueView` · readonly session/request lifecycle · `BlueResult` · capabilities | [AGENTS.md](packages/api/AGENTS.md) |
| core | `@dsh-blue/blue-core` | the tree's ONLY pi-tui adapter; terminal lifecycle, L1 services, component factory, themes, shared chrome | `blueScreen` · `blueKeymap` · `blueTerminalInfo` · `blueComponents` · 4 theme subpath plugins · `./chrome` | [AGENTS.md](packages/core/AGENTS.md) |
| transcript | `@dsh-blue/blue-transcript` | session events → transcript items and rendering | fold · tool cards/read groups · `blueStatus` + footer shell + 5 status plugins · dock panes (activity/todo/btw/agents) · `blueIntents` + `./intent-diff`/`./intent-terminal` · window/step folding · `./banner` | [AGENTS.md](packages/transcript/AGENTS.md) |
| interaction | `@dsh-blue/blue-interaction` | input editor, slash commands, dialogs | commands + alias registry · dialog panels (D30 editor-slot replacement) · completions (slash/`@`/`#`) · skills pipeline (`#` mark → `/name` gesture + `/skills` panel, S29) · model/session-info/export/tools/preset/permission/plan-review families · shared editor seams · `./attachments`/`./paste-image`/`./pane-queue`/`./editor-plus` | [AGENTS.md](packages/interaction/AGENTS.md) |
| app | `@dsh-blue/blue-app` | CLI startup + Agent driver | startup (`[task]`, `--resume`) · session switch queue · `modelRef` (D38) · preset mount (D37) | [AGENTS.md](packages/app/AGENTS.md) |
| bundle/blue | `@dsh-blue/blue` | installable unit | `cordis.patch.yml` (three segments, 21 Blue rows + thin-host roster/disables) · `bundle.spec.ts` drift guard | [AGENTS.md](packages/bundle/blue/AGENTS.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsh-blue/blue](https://github.com/dsh-blue/blue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
