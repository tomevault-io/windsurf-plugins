---
trigger: always_on
description: Please keep commentary to a minimum. Keep in mind the user is likely interfacing with you via terminal. You have many tools. Before making any changes, be sure to have received authorization from the user to act. Your objective is the continuous improvement of the enclosed AI-powered terminal coding CLI. Instead of relying on unit tests, use the `coder` CLI non-interactively to facilitate testing.
---

# @TJ/coder Terminal Coding TUI

Please keep commentary to a minimum. Keep in mind the user is likely interfacing with you via terminal. You have many tools. Before making any changes, be sure to have received authorization from the user to act. Your objective is the continuous improvement of the enclosed AI-powered terminal coding CLI. Instead of relying on unit tests, use the `coder` CLI non-interactively to facilitate testing.

Your model, qwen3-coder, is running on limited hardware in the cloud. Please consider this in your responses to me as they will be temporally expensive for me. Your consideration is greatly appreciated.


Our top objective with this codebase is to finalize it for open source release as a new design and AI-oriented terminal coding assistant/buddy running on TJ Coder / AI Labs platform technology. The ai.tjcoder.com and / or coder.tjcoder.com model coding services are offered as a paid, proprietary provider integration into the TUI as a means of promoting the TJ Coder set of platform technologies, Sentinel Radar, Sentinel Protect and now the TJ Coder CLI, an open source coding terminal assistant for and from the future. But now.

This means:

-- Completing the tasks/articles integrations (done in the appropriate feature branch for this)
-- Integrate the custom model provider through ai.tjcoder.com which builds upon specific open source ollama-compatible models, fine tuned for coding, cybersecurity and administration, as the proprietary '@tjcoder/ergo` (or some derivation of such).
-- Release as open source (when ready) to github via tjcoder-labs
-- Promote on hacker news, reddit, and locally.
-- Release Sentinel set of products to the Google Play Store.


Please always directly invoke your tools. 
Please keep attribution to the developer, TJ Coder(tj@tjcoder.com) via github.com/tjcoder-labs
Please keep your commentary to under 3 paragraphs


## Active Development Tasks

The following items are in flight. They are also tracked in the in-app
task tracker (`manage_items` → task) so they show up in the TUI's
/tasks pane, and persisted to `.ergo-cli-go/tasks.json` in the
workspace. Re-read this list at the start of every session — another
session on the same branch may have closed, retitled, or added tasks
since you last loaded the file.

The following tasks should be a reflection of those managed by the agent via the /task and /tasks commands, however because the feature is still unstable, you should always use the following task list as the source of truth, so be sure to add, update and mark as completed here in addition to using your tools. 

- **T1 — Activity panel underline bug.** Tool-error activity entries
  in the right-hand `ACTIVITY` panel render with an underline.
  Investigate the tview color tags in `internal/tui/app.go`
  `appendActivity` and the `EventError` branch (around line 1822);
  remove the offending `u` attribute from the tag string. Verify
  with the headless CLI before declaring done.
- **T2 — Fullscreen toggle does not work.** Pressing the global
  fullscreen shortcut (`Ctrl+F` / `F11`) toggles `a.fullscreen` and
  emits the activity log entry but the right column does not
  actually disappear. The bug is in `rebuildLayout` /
  `toggleFullscreen` in `internal/tui/app.go` (around lines 1464
  and 2118). Confirm whether the layout is being swapped on the
  right `Pages` and whether the cached `a.right` reference is
  being cleared and rebuilt on the next `showPanel` call.
- **T3 — Asynchronous background command execution.** Add a way for
  long-running `run_command` invocations to operate in the
  background and stream their output back to the agent as it
  becomes available, rather than blocking the whole conversation
  turn. Decide whether this is a new `run_command_async` tool, a
  flag on the existing `run_command`, or a session-level option;
  the right answer is the smallest change that doesn't break
  existing callers.
- **T4 — Tool-call budget reached: model response is mangled.**
  When the runner hits `r.MaxSteps` and falls into the
  "checkpoint response" branch in `internal/tooling/runner.go`
  (around lines 256–285), the next assistant message frequently
  contains raw `‹tool_call›…‹/tool_call›` or `<tool_call>…</tool_call>`
  blocks that were never scrubbed by `ExtractFallbackToolCall`
  (that helper only runs in the main `Run` loop, not the
  budget-reached fallback). Run the same scrubbing on the
  checkpoint response before appending to history or returning
  to the caller; also strip the wrappers from `msg.Content` so
  the user does not see the leaked markup in the transcript.
- **T5 — /tasks pane crash/hang.** Opening the interactive
  `/tasks` panel (via `showPanel("tasks")` in
  `internal/tui/app.go`) reportedly crashes or hangs the TUI.
  Suspected areas: `refreshTasksList` (around line 1557) which
  builds the `tview.List`, the `SetDoneFunc` closure that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tjcoder-labs/cli](https://github.com/tjcoder-labs/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
