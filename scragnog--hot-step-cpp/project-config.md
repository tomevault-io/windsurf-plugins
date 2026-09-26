---
trigger: always_on
description: Orientation map for agents, and the **single source of truth** for project rules. Claude Code reads it through `CLAUDE.md` (a one-line `@AGENTS.md` import); Codex reads it directly. Edit this file only. Keep it short and navigational — point at the deep docs, don't duplicate them.
---

# AGENTS.md — HOT-Step CPP

Orientation map for agents, and the **single source of truth** for project rules. Claude Code reads it through `CLAUDE.md` (a one-line `@AGENTS.md` import); Codex reads it directly. Edit this file only. Keep it short and navigational — point at the deep docs, don't duplicate them.

## Shared skills

Project skills live in `.claude/skills/`. On this Windows checkout,
`.agents/skills` is a directory junction to that folder, so Codex discovers the
same files Claude Code uses. Edit or add skills in `.claude/skills/`; do not
create a separate copy for another agent. Each skill needs a `SKILL.md` with
`name` and `description` frontmatter for discovery. Keep mandatory project rules
in this file, since skill selection depends on the task.

The junction and skill folders are local and gitignored. A new checkout needs
the skills and discovery link installed separately. Refresh the client's skill
list after changes, or restart it if the changes do not appear.

## What this is

A desktop app for **local AI music generation** — a heavily-extended superset of [acestep.cpp](https://github.com/ServeurpersoCom/acestep.cpp) (a C++/GGML port of ACE-Step 1.5). Caption + lyrics in → stereo 48 kHz audio out, fully local. Ships as portable releases (Windows CUDA/Vulkan/CPU, Linux, macOS Metal). GitHub: `scragnog/HOT-Step-CPP`.

## Architecture (3 tiers)

| Tier | Stack | Location | Role |
|------|-------|----------|------|
| **Engine** | C++17 / CUDA / GGML | [engine/](engine/) | Inference binaries: `ace-lm`, `ace-synth`, `ace-server`, `ace-understand`, `neural-codec`, `mp3-codec`, `quantize`. Pipeline: LM → DiT → VAE. Also hosts the **MiniMax-Music3 backend** ([engine/src/minimax/](engine/src/minimax/), `/mm3/*` endpoints) |
| **Server** | Node / TypeScript / Express / better-sqlite3 | [server/src/](server/src/) | Orchestrates the engine, manages songs/jobs/SQLite, serves UI. Per-feature [routes/](server/src/routes/) + [services/](server/src/services/) |
| **UI** | React 19 / Vite / Zustand / Tailwind | [ui/src/](ui/src/) | Browser frontend. Component folder per "studio" |

```
LAUNCH.bat → Node server (Express :3001)
  ├── serves React frontend (prebuilt ui/dist/)
  ├── /api/* → SQLite
  └── spawns child: ace-server.exe (C++ engine) on :8085
```

| Service | Port |
|---------|------|
| Node server | 3001 (prod) |
| Vite dev server | 3000 (dev, HMR) |
| ace-server (C++ engine) | 8085 (default, `config.ts`) |

## Environment

- **Windows 11 + PowerShell.** This repo's primary dev environment is Windows. Your harness (Claude Code, Codex) may also give you a Bash (POSIX) tool — each takes its own syntax. In PowerShell use `;` not `&&`.
- **Node 18–22 LTS only.** Node 24+ breaks dependencies (`engines` field enforces `<24`).
- **Call `.bat`/`.cmd` by absolute path.** Some agent shells run with `NoDefaultCurrentDirectoryInExePath=1`, so `cmd.exe` will not resolve `build.cmd` from the working directory — you get `'build.cmd' is not recognized` even though it is right there. Worse, `cmd.exe /c "script.bat"` can return **exit 0 having run nothing**, so never take a batch exit code as proof it ran: check the output for the script's own first line.

## Build & run rules (IMPORTANT — learned the hard way)

- **C++ engine changes → `dev-rebuild.bat`, NEVER `engine/build.cmd` directly.** The Node server auto-respawns ace-server on crash; killing it without clean shutdown causes an infinite respawn + file-lock loop. `dev-rebuild.bat` handles clean shutdown + rebuild — it does **not** relaunch; start the app again yourself with `dev.bat`/`LAUNCH.bat`.
  - Recompile **immediately** after editing any `engine/src/` or `engine/tools/` file — don't wait to be asked.
- **NEVER `cmake --build . --clean-first`** unless the GGML/CUDA layer itself changed — CUDA kernel recompilation is **20+ min**. For stale `.obj` issues, delete only `engine/build/acestep-core.dir/` and `engine/build/Release/acestep-core.lib`.
- **Don't `npm run build` during dev.** Only build before user testing. Type-check with:
  - `server/` → `npx tsc --noEmit`
  - `ui/` → **`npx tsc --noEmit -p tsconfig.app.json`** (or `npx tsc -b`). A bare `npx tsc --noEmit` in `ui/` **silently checks nothing and exits 0** — `ui/tsconfig.json` is `{"files": [], "references": [...]}`, so the root project has no inputs. It is not a passing check, it is no check.
- **`dev.bat`** = dev mode (Vite :3000 HMR + Node :3001, tsx watch auto-restart). **`LAUNCH.bat`** = prod. Use `dev.bat` for development.

## Git rules

- **All work on `master`. No feature branches, ever.**
- **Never `git add -A`** (re-adds gitignored dirs: `.agents/`, `checkpoints/`, `node_modules/`, etc.). **Never `git add -f`** on gitignored paths. Stage explicit paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scragnog/HOT-Step-CPP](https://github.com/scragnog/HOT-Step-CPP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
