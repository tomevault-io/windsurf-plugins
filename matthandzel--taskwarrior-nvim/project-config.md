---
trigger: always_on
description: validates downstream output (mmdc for Mermaid, task export for
---

# taskwarrior.nvim

Neovim plugin + Python CLI for editing Taskwarrior tasks as markdown.

## Structure
- `bin/taskmd` — Python CLI tool (parser, adapter, render, diff, apply)
- `lua/taskwarrior/init.lua` — thin orchestrator (~273 lines)
- `lua/taskwarrior/{buffer,apply,capture,delegate,review,saved_views,projects,
  completion,commands,help,validate,taskmd,config,views,diff_preview,feedback,
  health,statusline,cmp}.lua` — focused modules (split landed in v1.2.0)
- `lua/telescope/_extensions/task.lua` — Telescope extension (deliberately
  named `task` even after the v1.3.0 rename, because the extension name is
  the user-facing `:Telescope <name>` slug, not the module path)
- `plugin/taskwarrior.lua` — runtime entrypoint (registers `:Task` lazily)
- `doc/taskwarrior.txt` — vim help reference
- `tests/test_taskmd*.py` — Python tests (pytest, 358 tests)
- `tests/lua/spec/*_spec.lua` — Lua tests (plenary busted, 121 assertions)

## Development
- Python tests: `uv run --with pytest python -m pytest tests/ -q --ignore=tests/e2e`
- Lua unit tests: `./tests/lua/bootstrap.sh`
- **Lua e2e tests: `./tests/e2e/run.sh`** — spawns a temp TASKDATA,
  seeds fixtures, drives each feature against a real `task` CLI, and
  validates downstream output (mmdc for Mermaid, task export for
  mutations, window state for floats). **This is where "verified"
  lives**; unit tests catch syntax errors, e2e catches behaviour.
- Integration tests use a temp TASKDATA dir — they don't touch real tasks
- The CLI must have zero external dependencies (stdlib only)
- `lua/taskwarrior/init.lua` should remain a thin orchestrator (≤300 lines);
  business logic belongs in submodules

## Demo Assets
- Render demos: `demo/render-all.sh` (validates tapes + renders + size-checks)
- Validate tapes only: `demo/validate-tapes.sh`
- Never run `vhs` directly — the render wrapper enforces env isolation to prevent leaking real task data
- Pre-commit hook (`.githooks/pre-commit`) blocks commits with unsafe tapes or oversized assets

## Conventions
- Python: type hints, argparse, no external deps
- Lua: follow NvChad/lazy.nvim patterns
- All Taskwarrior commands must include `rc.bulk=0 rc.confirmation=off` to avoid interactive prompts

## Verification — "works" means end-to-end, not "module loads"

Before reporting a feature done, it must be verified against a real
Taskwarrior DB (use the `tw_env` fixture pattern from
`tests/test_taskmd.py`: temp TASKDATA + isolated `.taskrc`). Unit
tests that only assert "module loads", "command registers", or
"helper returns a list" do NOT verify the feature. They catch syntax
errors, nothing else.

**Hard rule: every user-facing flow gets a smoke test.**

A "user-facing flow" is any code path triggered by: a `:Task*` command
callback, a `vim.ui.select` / `vim.ui.input` callback, a buffer-local
keymap (e.g. `g?`, `<CR>`), a global keymap (e.g. `<leader>tF`), or any
popup/picker/buffer the user can hit. For each one, add an entry to
`tests/lua/spec/smoke_user_flows_spec.lua` that:

  1. Stubs `vim.ui.select` / `vim.ui.input` so the flow doesn't block.
  2. Invokes the entry point headlessly (e.g. `tutor.start()`,
     `feedback.last_error()`, the keymap callback).
  3. Drains pending `vim.schedule` callbacks via `vim.wait(50, …)` so
     errors thrown inside scheduled closures fire before the assertion.
  4. Asserts no real Lua / vim API error surfaced — match on
     `Error executing`, `stack traceback:`, `attempt to`, vim error
     codes (`E\d+:`), or specific error fragments like
     `'replacement string'`. **Do not** treat intentional ERROR-level
     `vim.notify()` calls as failures — those are user messages, not
     bugs.

The smoke test bar is "feature does not crash on the happy path of
every selection / argument value". It is intentionally weaker than the
feature-correctness bar (which requires asserting outputs). It exists
to catch the class of bug that ships when unit tests verify primitives
in isolation but no test ever drives the actual user journey.

Concrete example: the v1.4.1 verify-buffer bug
(`'replacement string' item contains newlines` at `init.lua:497`)
shipped because every existing tutor test exercised _begin_session,
_cleanup, the argv prefix, and orphan recovery — but none invoked
`:TaskTutor` and selected "Show me the exact `task` commands first".
The smoke spec added in that commit reproduces the original failure
when the fix is reverted (verified) and is what you must add for any
new user-facing flow before claiming "shipped".

Bar per feature category:

- **Commands that mutate a task** (`:TaskAppend`, `:TaskModifyField`, …):
  seed a task, invoke the command headlessly, `task export` the UUID,
  assert the field changed. Stubbing `vim.ui.input/select` is fine for
  driving the flow.
- **Commands that read and render** (`:TaskGraph`, `:TaskReport`,
  `:TaskInbox`, dashboard, query blocks): actually run a downstream
  validator on the output. For Mermaid, pipe through `mmdc` (it is
  installed). For markdown export, round-trip through `taskmd apply`.
  For reports, assert the buffer contents match the expected filter.
- **Commands with side effects on Neovim state** (`:TaskFloat`, `gf`
  float, embedded query blocks): assert the resulting buffer or window

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MattHandzel/taskwarrior.nvim](https://github.com/MattHandzel/taskwarrior.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
