---
trigger: always_on
description: > This file is the project-memory index for AI agents working on the Cinderpaw
---

# Cinderpaw — Agent Working Notes

> This file is the project-memory index for AI agents working on the Cinderpaw
> repo. Topic files live in `docs/agents-memory/` and are referenced from
> here. Per the project-memory protocol: drift in these files is a real
> bug (the next agent will believe the wrong thing), so update them when
> the underlying fact changes.

## Agent rules (all models, all sessions)

- Never work on `main`. One branch per task.
- No refactoring outside the stated task. No drive-by "improvements".
- Prefer the smallest measured fix that satisfies acceptance. Do not add abstractions,
  redesign adjacent systems, or expand scope when a local change and focused test suffice.
- Do NOT modify unless the task names them explicitly:
  `useCallSession.ts`, `vad.ts`, the Rust audio pipeline, `mcp.json`.
- Before declaring done, run `./scripts/verify.sh` — CinderpawAgent tests/typecheck,
  React tests/typecheck, Rust workspace check/Tauri tests, and TUI tests/build
  must all be green.
- Keep diffs small. If a task needs more than 3 files, stop and ask.
- Do not invent library APIs. If unsure, stop and report instead of guessing.
- Conventional commits. One logical change per commit.

## How Cinderpaw works at a glance

Cinderpaw = Tauri (Rust) host + Leptos/React frontend + a Bun/TypeScript
sidecar (`CinderpawAgent/`) that the host spawns and talks to over
newline-delimited JSON on stdin/stdout. RSI (Faza 1) and Fractal Memory
Search (Faza 5) are the two big engine pieces. Both are correct and
unit-tested; what blocks live numbers in this dev env is documented
below.

## Sidecar rebuild workflow — the easy thing to forget

`cargo tauri dev` does **NOT** auto-rebuild the sidecar binary. The
build IS wired through `src-tauri/scripts/build-sidecar.mjs` (see
`tauri.conf.json` → `beforeDevCommand` / `beforeBuildCommand`), but only
when the script is actually executed. If you change TS in `CinderpawAgent/`
and just restart the app, you're running the old binary.

Quick rebuild, in the worktree root:

```bash
cd CinderpawAgent && bun run build
# then copy (or let the script do it):
cp dist/cinderpaw-agent.exe ../src-tauri/binaries/cinderpaw-agent-x86_64-pc-windows-msvc.exe
```

Verify the fix landed: scan the binary for the new string.

```powershell
$bytes = [IO.File]::ReadAllBytes('<binary>')
[Text.Encoding]::UTF8.GetString($bytes) -match 'your-new-string'
```

## Topic files

- **`project_fractal_bench_blockers.md`** — what's actually blocking the
  Fractal bench from producing live numbers on this dev box (GPU crash
  on embed, rebuild thrashing). Pipeline is correct.
- **`project_fractal_activity_pulses.md`** — the three `fractal_activity`
  event kinds (`grow` / `recall` / `seed`) and the regression guard
  for the per-iteration pulse. Read before touching the organism
  wiring.
- **`reference_windows_vulkan_build.md`** — the Windows Vulkan build
  recipe that finally worked (cl 14.44 + Ninja + short `CARGO_TARGET_DIR`).
  Re-use when next fighting llama.cpp × MSVC.
- **`project_local_models_gpu.md`** — the on-disk models, the bge-small
  Vulkan crash, the `CINDERPAW_EMBED_GPU_LAYERS=0` knob, and the
  `discover_active_model` "wrong chat model" footgun (now fixed).
- **`project_brsi_evolution.md`** — the BRSI (Bounded RSI) work: locked
  decisions (D1-D10), audit summary of the existing engine, refactor
  sequence (10 steps), landmines for any contract / dream-cycle work,
  and the opencode-vs-Opus division of labor. **Read before touching
  any file in `CinderpawAgent/src/rsi/` or `src-tauri/src/rsi/`.**
- **`project_brain_stack.md`** — Brain Stack (Faza 4.6) engine that picks
  the right model per task. Slice 1 done (CapabilityRegistry +
  `isConfigured`); Slice 2 plan (task classifier) + the four-
  responsibility split (Registry=Data / Router=Policy / Health=Observation
  / Cost=Optimisation). **Read before touching any file in
  `CinderpawAgent/src/brain/`.**
- **`project_memory_roadmap.md`** — rebalanced post-audit roadmap
  (Memory Foundation before Onboarding). Three sharpenings to the
  audit's implementation order + the writer contract design. Read
  before touching `CinderpawAgent/src/memory/`, `CinderpawAgent/src/db.ts`,
  `src-tauri/src/memory_*.rs`, or any `workspace_id` migration.
  **Sprint 1 (Memory Foundation + Memory Resume) + Sprint 2 (Terminal +
  Desktop Onboarding) shipped 2026-07-06** — see the Status table at
  the bottom of the file for the file-level landing points of every
  shipped item.
- **`project_chat_tui.md`** — `cinderpaw chat` is a Go/Bubble Tea TUI
  (`tui/`, launched by `crates/cinderpaw-cli/src/chat.rs`). Also flags the
  open follow-up: local-Ollama reasoning for models like MiniMax-M3
  arrives inline in `content` (no `think` tags), so the TUI cannot
  split it — needs a sidecar fix (`CinderpawAgent/src/sandbox/inference-providers.ts`
  local path) or a host-side `delta.reasoning_content` forward. **Read
  before touching chat reasoning rendering or the local Ollama path.**
- **`project_arc_agi3_campaign.md`** — ARC-AGI-3 public demo campaign: test
  plan (4 models × 4 harness stages on vast.ai RTX PRO 6000 WS 96GB), score
  targets anchored to NVIDIA AVO's verified 100% public-set result, budget

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bloom500/cinderpaw](https://github.com/bloom500/cinderpaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
