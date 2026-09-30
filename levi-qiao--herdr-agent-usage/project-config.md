---
trigger: always_on
description: Notes for agents working on `herdr-agent-usage`. Read this before touching
---

# Agent guide

Notes for agents working on `herdr-agent-usage`. Read this before touching
anything that talks to Herdr.

## Working method

1. Establish the exact requested scope and inspect the current diff before
   editing. Treat unrelated worktree changes as user-owned.
2. Separate observed facts, inferences, and unknowns. When evidence is missing,
   name the cheapest useful verification instead of guessing.
3. Prefer the smallest surgical change that creates a checkable behavior. Add
   abstractions only when two real callers or adapters need the same seam.
4. Give every implementation step a verification condition and run the
   repository gates before calling it complete.
5. For multi-goal work, keep the decision record and dispatch prompts under
   the ignored `.agents/` directory. Public documentation must describe shipped
   behavior, not private execution state.

Before changing dependencies, inspect `Cargo.toml`, `Cargo.lock`, and
`rust-toolchain.toml`. Use the pinned Rust toolchain and repository-local Cargo
artifacts; do not install project tooling globally.

## The rule that matters most: reading or writing a pane is not free

Read pane output only with `--source visible` or `detection`; `recent` and
`recent-unwrapped` rebuild scrollback and visibly repaint the agent TUI.
Metadata writes also carry repaint risk, so avoid no-op writes. Extract the topic
from the event pane while visible; if extraction fails, preserve its existing topic.

For scrolling reports or changes to pane-read behavior, read
[pane repaint diagnosis](docs/pane-repaint-diagnosis.md) before probing.
Scroll offsets and before/after content hashes cannot detect the transient repaint;
use the documented human observation rather than polling live panes.

Concretely, this means:

1. **Never read every pane of a provider.** An event names one pane; read only
   that one. Fanning out across panes multiplies the repaints by the number of
   panes the user has open for that agent.
2. **Publish once per invocation.** Two `publish` passes in a row means each
   pane can take two metadata writes for one user action.
3. **Keep `metadata_matches` honest** (`src/herdr.rs`). It is the only thing
   stopping a no-op refresh from repainting every pane. If you add a token,
   add it to `METADATA_TOKEN_NAMES` too, or the comparison silently stops
   covering it and every refresh becomes a write.
4. **Preserve, don't clear.** When a topic read fails or finds nothing, keep
   the previously published topic. Clearing it churns the token and triggers
   a write on the next refresh, which triggers a repaint.

## Event paths, and what each is allowed to do

| Entry point | Fired by | Allowed to read panes? |
|---|---|---|
| `startup` | Herdr's `[[startup]]` hook | No |
| `refresh` | manual action, `startup` | No |
| `event` | `pane.agent_detected`, `pane.agent_status_changed` | Only the pane named in `HERDR_PLUGIN_EVENT_JSON`, and never a Pi, omp, Muse, or Cursor pane — their transcripts carry the evidence |
| `focus` | `pane.focused`, `workspace.focused`, `tab.focused` | No |
| `watch` | detached from a working status event | No (agent metadata only) |

`startup` exists because Herdr drops plugin-owned Agent views when the server
exits, and startup hooks run again after a restart or a live handoff. It
restores plugin-owned views, forces one quota refresh, and restores the watcher.
It is also the second home of the one repair `configure --apply` can only make
once: when omp is selected and Herdr says its integration is still missing,
and omp's own agent directory exists, startup installs it. A machine that
installed omp after this plugin would otherwise keep detecting omp panes
without a session forever, and a pane in that state has nothing to publish and
says so (`unattributed_session_reason`) instead of rendering an empty row.
Plugin enable alone does not run startup; the configure action runs it after
repair. Server-owned event/refresh paths also record the current Herdr binary
and socket so an older watcher can adopt the new connection.

`pane.agent_status_changed` fires **twice per turn** (idle→working on submit,
working→idle on completion). Anything `event` does, the user pays for twice
every time they press Enter. Budget accordingly.

The working event starts one global `watch` pulse. It calls `herdr agent list`
once per configured interval for every supported harness, including Pi, OMP,
OpenCode, Muse, and Cursor. Event-spawned watchers defer their first poll. They resolve local
billing targets, refresh active/settling targets, and publish to siblings with
the same target without reading terminal output. A finishing target stays in
the pass until the 60-second debounce has elapsed. The interval defaults to
60 seconds and is bounded to 30 seconds–1 hour. While a pane is working or
has an unseen completion, the watcher also checks the metadata-only Herdr
snapshot once per second. Herdr 0.9 can miss TUI focus hooks; the snapshot
reconciles those changes without reading pane output or writing unchanged
metadata. The watcher stays alive for unseen completions until they are seen.
Local stop/connection checks interrupt sleeps. Uninstall writes a stop marker.

## omp's quota does not come from a provider endpoint


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levi-qiao/herdr-agent-usage](https://github.com/levi-qiao/herdr-agent-usage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
