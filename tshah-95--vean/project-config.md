---
trigger: always_on
description: > **vean studio** — Video Editor, Agent Native · vean.studio
---

# vean — the agent-native video editing core

> **vean studio** — Video Editor, Agent Native · vean.studio

vean is the **typed document, edit algebra, and diagnostics layer** for video
editing on top of MLT. Think of it as a *language server for video*: a
frame-exact, rationally-timed model of a timeline that an agent (or a UI, or a
human) mutates through a closed set of operations, each of which reports its
consequences **before a single frame renders**.

vean is **not only an app.** Its core is the headless editing engine the app
sits on: files in, files out, no network, no secrets. Product surfaces (the
Commander CLI, LSP, MCP, and the future local Tauri Mac app) share local
coordination state in a gitignored `.vean/vean.db` SQLite database. The render is
delegated to `melt` (MLT/FFmpeg) as a separate process; the Mac app may bundle
pinned renderer sidecars, while the source/CLI/Homebrew artifact treats them as
system dependencies. vean owns the part nobody else does: a *typed, validated,
diagnosable, agent-authorable* representation of an edit.

This file is the **resolver** — it doesn't hold the knowledge, it routes to it.
It is the single canonical brain; `CLAUDE.md` is a one-line `@AGENTS.md` shim so
Claude and Codex read the same bytes.

## What it is / isn't (read first)

- **IS:** a typed IR for an MLT timeline (multi-track, audio, keyframes,
  filters, transitions); a deterministic serializer + parser (IR ⇄ `.mlt` XML);
  a pure edit algebra (`op(state) → {state', consequences, inverse}`); a
  diagnostics engine (the LSP); a render/inspect driver that shells out to
  `melt` and `ffmpeg`; a local product-state substrate for projects, setup
  choices, jobs, and future UI/agent coordination.
- **ISN'T:** a renderer (MLT/FFmpeg do that), a motion-graphics engine (Remotion
  does that), or a UI implementation inside the core (the local Mac app does
  that through the action runtime).

## The seam: depend / mine / build

The whole strategy in one table. We **fork the engine** (depend on MLT, which we
already drive), **mine Shotcut's source as the spec** (its edit algebra and dial
schemas are the answer key — we lift the semantics, drop the Qt), and **build**
the typed/diagnosable layer nobody else has.

| | What | Why |
|---|---|---|
| **Depend on** (reuse as-is) | MLT + `melt` + FFmpeg; Remotion's renderer + `@remotion/player` | Solved problems — engine semantics, codecs, the React-in-Chrome render. Never reimplement. Driven at arm's length (see Hard boundaries #1). |
| **Mine** (source as spec, don't run) | Shotcut `src/commands/` (edit algebra + undo mechanics); `src/qml/filters/*/meta.qml` (dial schemas); the nested-tractor-dissolve + blanks-as-gaps XML shapes | Debugged knowledge, but Qt-coupled. Lift the semantics, drop the framework. |
| **Build** (own, greenfield) | the typed IR; serialize + parse; the keyframe model; the dials schema (from `melt -query` + overrides); the edit algebra as pure ops; the diagnostics LSP; the action registry; the Commander CLI; the agent bridge + skills; the local Mac app | This is the project. The LSP especially is net-new — no NLE exposes consequences. |

## The layer model

Four layers. The agent-vs-UI ownership split falls out of them cleanly.

1. **The core (this repo's heart, headless, no UI).** The typed video *document*
   and everything that reasons over it: IR, serialize+parse, keyframe model,
   dials schema, edit algebra, diagnostics, the `melt`/ffmpeg driver. The video
   equivalent of a language server + the engine driver. ~Agent-buildable,
   because it's well-specified (the spec is Shotcut's source).
2. **The action runtime.** A typed registry defines product behaviors once, then
   projects them to Commander CLI, MCP tools, narrow LSP code actions, and Tauri
   invoke commands. This is where project/media ergonomics, policy, permissions,
   output envelopes, and job semantics stay consistent across surfaces.
3. **The agent bridge.** Two coordinated surfaces over the same core:
   **`vean-lsp`** for ambient feedback (push diagnostics, code actions,
   definitions/references/hover over the timeline document) and **MCP/CLI tools**
   for domain actions (`apply-op`, `preview-op`, `undo`, `render`, `still`).
   `diagnose` exists for CI/debug/manual inspection, but it is **not** the normal
   agent loop. Agents should see adverse effects the same way they do in code:
   edit/change happens, diagnostics are pushed without a separate "remember to
   call diagnose" step.
4. **The local product layer.** The local Mac app is the primary GUI: project
   list, media catalog, timeline drawn from the IR, render/still preview, and
   agent-orchestration UI (sessions, diffs, git worktrees for exploration). The
   website is only for download/docs, not a web editor.

## Local state contract

The timeline core stays file-in/file-out. Product coordination state lives in
`.vean/vean.db`, a repo-local SQLite database ignored by git. It is owned by
`src/state/`, modeled with Drizzle, and migrated by committed SQL under
`drizzle/`.

- **Store in `.vean/vean.db`:** project metadata, setup choices, preferences,
  render/agent jobs, job leases, media probe caches, future UI session metadata.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tshah-95/vean](https://github.com/Tshah-95/vean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
