---
trigger: always_on
description: Orientation for contributors and AI agents working on athanor. Read this before making changes; the invariants below are not negotiable without an explicit user decision.
---

# AGENTS.md

Orientation for contributors and AI agents working on athanor. Read this before making changes; the invariants below are not negotiable without an explicit user decision.

## What athanor is

Athanor — personal LLM alchemy on Apple Silicon. It scans the Hugging Face cache, registers MLX and `llama.cpp` (GGUF) models in `~/.athanor/models.json`, assigns each one a **stable port**, supervises detached child processes, and exposes models to [pi-agent](https://github.com/badlogic/pi-mono) as one custom provider per model. Surfaces are an Ink TUI (`athanor`) and a hand-rolled CLI (`athanor <cmd>`).

Not a library. Not a daemon. No network listeners except the runtime children themselves (and an optional local control API, off by default).

## Invariants

These are load-bearing. If a change seems to need to break one, stop and ask.

1. **Stable port per model.** Ports are allocated on first discovery from `config.portRange` and persisted on the registry entry forever. A model's port never changes behind the user's back; pi-agent provider URLs must be stable across restarts.
2. **Atomic registry writes.** `~/.athanor/models.json` is always written via temp-file + rename in `src/registry/index.ts`. Never partial-write it, never keep it open across awaits.
3. **Preserve non-athanor pi entries.** `src/sync/pi.ts` rewrites only providers whose name starts with `athanor-`. Everything else in `~/.pi/agent/models.json` (OpenAI, Anthropic, Ollama, OpenRouter, user customs) must round-trip untouched. Same for `~/.pi/agent/settings.json` — only `defaultProvider` / `defaultModel` are touched, and only when a model is started as the active default.
4. **Pi sync shape follows `config.router.enabled`.** Default (router off): each exposed athanor model becomes its own pi provider `athanor-<runtime>-<slug>` with one `baseUrl` per model. Router on: up to two aggregator providers — `athanor-mlx` and `athanor-llama` — both pointing at the router `baseUrl`, each listing only models of its runtime and carrying runtime-specific compat flags (MLX sets `supportsDeveloperRole: false`; llama-server doesn't). A provider with zero exposed members is suppressed. Never emit both shapes. The CLI verbs are `expose` / `hide`; the underlying registry field is `publish: boolean` (storage name, kept stable for backward-compat of on-disk `models.json`).
5. **Runtime model id matches launch argument literally.** `mlx_lm.server` compares the request's `model` field to whatever was passed as `--model` and falls back to a HuggingFace network lookup on mismatch. The pi model `id` we emit must equal the adapter's `--model` (or `--alias` for `llama-server`). See `src/sync/pi.ts` and `src/adapters/*.ts`.
6. **Pi context metadata reflects effective served context.** `src/sync/pi.ts` must advertise the model's effective runtime context window from merged runtime config (`mergedConfigFor(entry)`), not only explicit per-model preset fields and not the model's theoretical maximum. pi should see what athanor will actually serve.
7. **MLX capability detection and flavor routing are separate axes.** Two fields live on an MLX entry:
   - `mlxCapabilities: ("vlm")[]` — a detected *fact* about the model (does config.json advertise a vision tower?). Refreshed by `ingestDiscovered` and `pull` via `detectMlxCapabilities()` in `src/discovery/scanner.ts`. Safe to overwrite on re-scan.
   - `mlxFlavor: "lm" | "vlm"` — user *intent* about which server binary to launch. `"vlm"` routes to `mlx_vlm.server`; `"lm"` (or absent) routes to `mlx_lm.server`. Only set by `athanor flavor <slug> lm|vlm` (`cmdFlavor` in `src/cli/commands.ts`). Discovery and ingest must never touch it.

   Detection is advisory because many VLM-tagged repos run fine as text-only under `mlx_lm.server` with no torch/torchvision installed, and that's usually the preferred path. `cmdShow` surfaces the capability with a hint that points at `athanor flavor`. Do not add VLM detection anywhere other than `detectMlxCapabilities`; keep it a single source of truth.
8. **Supervisor default policy is `single-active`.** Starting model B stops model A unless the user opts into `multi-active-lru` (or `manual`) in `config.json`. Policies live in `src/supervisor/policies.ts`.
9. **Presets are additive, scans are non-destructive.** `athanor scan` refreshes `path`, `sizeBytes`, and `mlxCapabilities`. `preset`, `publish`, `piAlias`, `tags`, `port`, `slug`, and `mlxFlavor` must survive re-scans. Built-in recipes are explicit presets; `balanced` is not shorthand for clearing a preset, and `preset clear` is the separate remove action.
10. **All mutations go through helpers.** Use `setPresetFields` / `unsetPresetFields` / `recipeToPreset` from `src/presets/edit.ts` and `updateModel` from `src/registry/index.ts`. Do not hand-edit registry objects in commands or UI components.

## Layout

```
src/
  adapters/     # mlx_lm + mlx_vlm + llama-server command builders, health probes
  cli/          # dispatcher (index.ts), commands.ts, doctor, formatting
  config/       # config load + defaults
  control/      # optional HTTP control API (opt-in)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MylesBorins/athanor](https://github.com/MylesBorins/athanor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
