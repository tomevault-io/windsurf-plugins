---
trigger: always_on
description: Cursor 3.7 runtime guide: choose the right tool, canvases, Design Mode, /worktree, /best-of-n, Await, and parallel execution where safe.
---


# Cursor 3.7 Tools Mastery

Keep durable behavior in the core rule. Use this file for current Cursor **3.7** tool-selection patterns and the Agents Window surface.

## Cursor 3.7 workspace

In Cursor 3.7, agent work is centered in the **Agents Window**. Open it from the command palette: `Cmd+Shift+P -> Agents Window` on macOS (per the [Cursor 3 announcement](https://cursor.com/blog/cursor-3); 3.7 changelog at [cursor.com/changelog](https://cursor.com/changelog)). The classic three-pane editor is still available; switch between them at any time.

- **Multi-workspace / multi-repo:** the Agents Window can surface several workspaces at once. Name the repo or root path when handing off work, opening files, or running commands so edits land in the intended tree.
- **Per-session execution environment:** each agent tab picks its own environment — local working tree, isolated worktree (`/worktree`), cloud sandbox, or remote SSH. State the environment in the handoff contract; it is the unit of isolation.
- **Per-session model picker:** the model picker lives in the agent tab header (e.g. `composer-2.5`, `MiniMax-M3`, `auto`). Name the model in the handoff contract.
- **Integrated browser:** prefer the IDE's built-in browser for local UI verification when it is exposed in your session; it matches the "snapshot-first" flow in Browser Guidance below. Cursor 3.7 adds **Design Mode** (multi-select elements, voice input) — user selections there are ground truth for UI edits.
- **Canvases:** live React artifacts beside the chat for dashboards, reports, and structured analyses; load the `canvas` skill before `.canvas.tsx` work.

## Golden Rule

For significant actions:

```text
Check what exists
Choose the smallest correct tool
Act
Validate with the lightest useful verification
```

## Default Tool Map

**Cursor 3.7 / Composer-style agents (typical names in current desktop agents)** — the session usually exposes something close to:

| Step | Tool / command | Notes |
|------|----------------|-------|
| Read a path | `Read` | Prefer over shell `cat`/`sed` |
| Search by pattern | `Grep` | Ripgrep-backed; use for exact symbols and strings |
| Search by meaning | `SemanticSearch` | "Where does X happen?" not exact text |
| List paths | `Glob` | File discovery by pattern |
| Edit in place | `StrReplace` | Surgical edits; read the file first |
| Create or replace whole file | `Write` | Use when there is no stable old_string to patch |
| Remove file | `Delete` | |
| Terminal | `Shell` | Builds, tests, git, installs |
| Web | `WebSearch`, `WebFetch` | |
| Ask user | `AskQuestion` | |
| Track steps | `TodoWrite` | |
| Delegate branch work | `Task` | Subagent-style runs (types such as explore, debugger, verifier — see schema) |
| Wait for background work | `Await` | Wait for a background shell, subagent, or a specific output token (e.g. `Ready`, `Error`) |
| Isolated worktree | `/worktree` | Run a session in its own git worktree so concurrent edits do not collide |
| Parallel model compare | `/best-of-n` | Run the same task across N models in parallel worktrees, then compare |
| MCP servers | `call_mcp_tool` (+ resource helpers if present) | Read MCP tool descriptors before calling |
| Plan mode | `SwitchMode` | When the product exposes a planning mode |
| Images | `GenerateImage` | Only when the user asks for generated images |
| Notebooks | `EditNotebook` | `.ipynb` cell edits |
| Lint diagnostics | `ReadLints` | After substantive edits when available |
| Multimodal inputs | `Read` against attached image/video paths | M3 native multimodal — treat as first-class inputs |
| Interactive artifact | Canvas (`.canvas.tsx`) | Dashboards, reports, MCP tabular output — load `canvas` skill first |
| Context diagnostics | Context usage report canvas | 3.7 — token breakdown across prompt, tools, rules, skills |

**Browser:** often MCP tools (for example `cursor-ide-browser`) with names like `browser_snapshot`, `browser_navigate` — follow the server's schema. **Design Mode** (3.7) is user-driven; honor multi-select and voice instructions as visual ground truth.

**Parallel batching:** issue **multiple independent tool calls in one assistant turn** when the runtime allows it.

**Other Cursor surfaces** (CLI, older builds, API): names may differ (`ReadFile`, `ApplyPatch`, `Subagent`, etc.). The **exact tool list in the current session always wins** — same principle as `model-compatibility.mdc`.

## Pick The Right Surface (Cursor 3.7)

| Need | Best fit |
|------|----------|
| Trivial localized edit | IDE inline edit |
| Multi-file feature in one repo | Single agent session in the Agents Window |
| Risky exploration, parallel branches, anything that must not collide with the main tree | `/worktree` (isolated git worktree) |
| High-stakes decision (design, architecture, refactor) where you want a second opinion | `/best-of-n` (run across 2–4 models, then pick or merge) |
| Long-running shell, subagent, or CLI that takes seconds-to-minutes | `Await` on the background job, or a specific output token |
| Work that should keep running while you walk away | Cloud session, then hand back to local when environment-specific proof is needed |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m3-cursor-rules](https://github.com/madebyaris/advance-minimax-m3-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
