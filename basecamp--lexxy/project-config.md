---
trigger: always_on
description: Lexxy is a rich text editor built on Lexical, distributed as both a Ruby gem and an npm package.
---

# Lexxy

Lexxy is a rich text editor built on Lexical, distributed as both a Ruby gem and an npm package.

See [docs/development.md](docs/development.md) for local development setup, how to run tests, and release instructions.

## Worktrees

When working from a git worktree:

- Run all commands from that worktree, not from the main checkout.
- For system-level work, start `bin/dev` in the worktree before investigating or running `test/system/` after `src/` changes. Rails system tests use `app/assets/javascript/lexxy.js`, so the watcher must keep built assets in sync.
- Use a unique port per worktree, for example `PORT=3100 bin/dev` or `PORT=3200 bin/dev`, to avoid Rails server collisions.
- If you skip `bin/dev`, rebuild assets in the worktree before trusting system test results after any `src/` change.
- Playwright tests in `test/browser/` do not need `bin/dev`.

## Browser Test Organization

Playwright tests in `test/browser/tests/` are grouped into semantic folders:

| Folder         | What goes here                                                                          |
|----------------|-----------------------------------------------------------------------------------------|
| `attachments/` | Upload, delete, caption, gallery                                                        |
| `editor/`      | Core editor API: focus, isEmpty/isBlank, toString, setValue, CSS status classes          |
| `formatting/`  | Toolbar chrome, inline marks, highlights, block formats, lists, HR, escape, color       |
| `modes/`       | Plain-text mode, single-line mode                                                       |
| `paste/`       | Markdown conversion, URL/link paste, file/attachment paste, style canonicalization       |
| `prompts/`     | Prompt popover triggers, mention insertion/deletion, remote-filtering race conditions    |
| `tables/`      | Structure (create, add/remove rows/columns, delete), headers                             |

Single-file tests (`code_highlighting`, `events`) live at the root level.

When adding a new test, pick the folder whose description fits best. If none fits, consider whether a new folder is warranted or if the test extends an existing category.

## Performance Benchmarks

Use the browser benchmark harness for Lexxy performance work. It is intentionally scoped to the JS/editor side of the project, not Rails persistence or Action Text server behavior.

- Run browser benchmarks from the worktree with `yarn benchmark:browser`.
- Results are written to `tmp/browser-benchmarks.json`.
- For targeted iteration, use `yarn benchmark:browser --list-scenarios` and `yarn benchmark:browser --scenario <name> --warmup <n> --iterations <n>`.
- Compare two runs with `yarn benchmark:browser:compare <baseline.json> <current.json>`.
- CI uses `.github/workflows/benchmarks.yml` and compares PR results against the latest successful benchmark run on `main`.
- Treat CI benchmark failures as coarse regression alarms. The thresholds are intentionally loose enough to survive normal GitHub-hosted runner variance.

## Action Text Persistence

When changing how Lexxy formats or serializes content (new inline styles, node types, HTML export changes, sanitization rules), always verify the new format survives the full Action Text round-trip: editor → save → render → re-edit. The editor's HTML passes through DOMPurify (client), Loofah (server), and `highlightCode()` (rendered view) — any of these can strip markup that the editor preserved. Write a Capybara system test that edits a post in the dummy app (`test/dummy/`), saves it, and asserts the rendered show page and re-edited content are correct. Use `posts(:empty)` or `posts(:hello_world)` fixtures as starting points. See `test/system/code_highlighting_test.rb` and `test/system/color_highlighter_test.rb` for examples.

## Fixing Bugs

**Important:** Some bugs filed against Lexxy are actually bugs in the host application (e.g., BC5 modal behavior, server-side preview rendering, Turbo interactions). When a bug's root cause is in a 37signals app rather than in Lexxy itself, use the `/bugs-fix` skill from `basecamp/coworker` to fix it in the appropriate app repo (e.g., `~/Work/basecamp/bc3`). The workflow below applies only to bugs whose root cause is in Lexxy.

Follow this mandatory workflow. Every step must complete before moving to the next.

1. **Classify** — Determine if the bug is a **core editing bug** (JS behavior: typing, cursor, selection, formatting, paste, toolbar, nodes, extensions), a **system-level bug** (e.g., Action Text, uploads, Trix conversion, persistence, prompt/SGID resolution, Turbo), or a **host app bug** (e.g., BC5 modal/composer behavior, server-side preview rendering, Turbo frame interactions). Host app bugs should be fixed in the app repo using `/bugs-fix` from `basecamp/coworker`, not here.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basecamp/lexxy](https://github.com/basecamp/lexxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
