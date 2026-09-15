---
trigger: always_on
description: This file contains the mandatory rules for working on **LucidSSH for Windows**. Read before every task. On conflict: the spec (`private/TZ.md`) and the implemented UI in `src/renderer/components/` take priority over this file; this file takes priority over general habits.
---

# CLAUDE.md — operating rules for Claude Code

This file contains the mandatory rules for working on **LucidSSH for Windows**. Read before every task. On conflict: the spec (`private/TZ.md`) and the implemented UI in `src/renderer/components/` take priority over this file; this file takes priority over general habits.

> **Public/private repository.** Part of the documentation lives in `private/` — not published, contains unimplemented requirements, roadmap, and internal decision context. `docs/` is the public showcase (README, implemented ✅ requirements, security architecture, data schemas) — published in **English**, since the repository audience isn't limited to Russian speakers; `private/` stays in **Russian**, it's the working source for the developer and Claude Code, not meant for outside readers. For internal work (including resolving open questions, planning features beyond 1.0) always use the full versions in `private/`, not the public ones in `docs/`. Split details — `docs/agent/domain.md` or ask the developer.

---

## 0. What this project is

A desktop SSH client for Windows built on Electron + TypeScript. Target audience: beginners, without getting in the way of experienced users. Fully local: **no account, no cloud, no telemetry**.

Four features set the product apart (must not be simplified without explicit agreement):
- **Dangerous command guard** — intercepts `rm -rf` and similar before they reach the server.
- **Error detector** — explains stderr/SSH errors offline, without an LLM (localized; Russian by default).
- **"Where am I" breadcrumb** — current path above the terminal, clickable.
- **Command catalog** — sidebar with explanations (localized, Russian by default).

---

## 1. Sources of truth (in priority order)

1. **`private/TZ.md`** — requirements, IDs, acceptance criteria (full version, including unimplemented requirements and open questions). If a rule here conflicts with the spec, the spec wins. The public showcase (✅ requirements only) is `docs/TZ.md`, not a source of truth for internal work.
2. **Implemented code in `src/renderer/components/`** — **source of truth for the UI**. `doc/screenshots/` has been removed (moved into git history); an archived design mockup lives at `private/backup/2026-07-05_v1.1/*.dc.html` and is useful as a reference for intent, but not as ground truth. Before laying out or changing any screen, open the corresponding component and check how it's already implemented: placement, states, colors, text. Don't invent a layout that exists neither in the code nor in `Design_Brief.md`. If a screen's behavior is unclear and undocumented — **stop and ask**, don't guess.
3. **`private/Design_Brief.md`** — design intent, tokens, behavior.
4. **`package.json`** — the single source of truth for the version number.
5. This file — operating rules.

Unsure about a requirement — ask, don't guess. I (the developer) prefer to understand the rationale before a rule is set; propose trade-offs, not directives.

### Project document map (`docs/` — public, `private/` — not published)

Where to look for what before working on a module. For internal work, Claude Code always uses the `private/` version when one exists — it has the full context (unimplemented requirements, roadmap, open questions); the public version in `docs/` is only a trimmed, English showcase for GitHub.

| Document | Path | When it must be opened |
|---|---|---|
| `TZ.md` | `private/TZ.md` (full, Russian), `docs/TZ.md` (public, ✅ only, English) | Any task: requirements, IDs, acceptance criteria. Highest priority. Work from `private/TZ.md`. |
| `src/renderer/components/` | — | Any UI work — source of truth for screens (screenshots/ removed, see §1). |
| `Design_Brief.md` | `private/Design_Brief.md` (Russian) | Layout, tokens, component behavior. |
| `Data_Structures.md` | `private/Data_Structures.md` (full, Russian), `docs/Data_Structures.md` (public, English) | **`hosts/`, `history/`, `errors/`** — SQLite schemas, content-database formats (`errors.core.json`, `commands.core.json`, translations in `locales/`), LLM extension points. Don't write a table schema or JSON parser without checking first. |
| `Security_Guide.md` | `private/Security_Guide.md` (full, Russian), `docs/Security_Guide.md` (public, English) | Any code touching section 4 (keytar, keys, IPC, fingerprint, masking). Rationale for security requirements. |
| `Development_Roadmap.md` | `private/Development_Roadmap.md` (Russian) | Doubt about "is this in 1.0 or later" — check against scope (§3). |
| `Release_and_Update_Strategy.md` | `private/Release_and_Update_Strategy.md` (full, Russian), `docs/Release_and_Update_Strategy.md` (public, English) | Work on the auto-update module (`updates/`), version bumps, channel/signing strategy. Check before any release-pipeline change — use the `private/` version. |
| `Local_LLM_Spec.md` | `private/Local_LLM_Spec.md` (Russian) | **Reference only, for a future version.** Do not implement in 1.0 code. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xykyma/LucidSSH](https://github.com/Xykyma/LucidSSH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
