---
trigger: always_on
description: A Claude Code **skill plus an MCP server** that analyze a Telegram channel and
---

# tg-scraper / slop-writer

A Claude Code **skill plus an MCP server** that analyze a Telegram channel and
can queue a post to it. The eleven MCP tools are the whole agent-facing
surface; the skill under `skills/slop-writer/` says which tool answers which
question and what the numbers mean. Published to PyPI as `slop-writer`; two
install channels (`slop-writer install`, `npx skills@latest add …`) serve that
same skill directory, and one version covers package and skill.

Rationale for a decision lives in `docs/adr/`; vocabulary in `CONTEXT.md`;
structure (what calls what, where a symbol is) in codegraph.

## Commands

- `uv run pytest` — the suite, from the repo root. `uv` installs the project
  editable, so it exercises the working tree.
- `uv run tools/check_schema_doc.py` — dev-only drift guard; run after editing
  `SCHEMA` or `references/schema.md`. CI runs it as its own step.
- `uv run --with-editable . tools/tg_*.py` — the dev CLIs against the tree.
- `slop-writer` (the shipped console script): `install` wires this project's
  Claude Code config, `uninstall` removes exactly what it wrote, `serve --mcp`
  runs the server over stdio, `init` writes Telegram credentials and logs in.
  **`init` needs a TTY for the SMS code — the user runs it, not the agent.**

## Library rules (`src/slop_writer/`)

- **The domain raises, the entrypoint reports.** Nothing under
  `src/slop_writer/` prints an error or exits: it raises `SlopWriterError`
  (`UsageError` for exit 2) and `cli.py` or `server.py` decides how to report.
  That is what lets the server call the same function and build JSON instead.
- **`errors.ERROR_CODES` is a closed vocabulary**, enforced at construction.
  Reuse a code; adding one is a deliberate edit with a raise site.
- **A message or a hint names an argument or an operation, never a surface** —
  the same string reaches a human at stderr and a model at a tool result, and
  only one of them has a flag or a script. `tests/test_errors.py` scans every
  literal statically over the modules `server.py` can reach. When the surface
  genuinely differs, the caller supplies the noun (`prepare_schedule(
  body_source=…)`) or the boundary swaps the hint (`server._SETUP_HINT`).
- **The domain uses a client, the entrypoint owns it.** Scrape and scan paths
  come in twins: `X_with_client(client, …)` does the work, `X(…, session_file)`
  wraps it in one `channel_session`. A **new command adds both forms.**
- **`resolve_peer` runs before any `open_db`** — `open_db` lives inside the
  `channel_session` body (for the group scan, inside `scan_group_with_client`
  after `resolve_group_target`). A typo then fails on the handle instead of
  leaving a stray empty `.tg-analytic/<typo>.db`, and zero posts always means
  an empty window.
- **Renderers return a string.** `render.py` takes plain dicts and returns
  Markdown — no Telethon or SQLite types, and no `print`: on a stdio server
  stdout is the JSON-RPC transport. The domain returns the result shape
  (`ScrapeResult`, `GroupScanResult`, …); the entrypoint calls `summarize_*`.
- `db.py`, `errors.py` and `query.py` stay **stdlib-only** — a query stays
  answerable without a Telegram client.
- `publish.py` is the **write surface** (adr/0003); nothing on a read path
  imports it, which is why `scheduled.py` is separate.
- **`MESSAGE_TOO_LONG` comes from the network, by design** — the cap depends on
  the account, so `_too_long` translates Telethon's error rather than measuring
  the body.
- **A refusal is not an absence.** `CANNOT_RESOLVE` means the handle names
  nothing, `NOT_A_MEMBER` means Telegram confirmed the peer and refused this
  account — one is fixed by correcting the handle, the other by joining. Every
  new Telethon call that can refuse classifies the two.
- `cli.py` is **argparse**, so an installed server carries no CLI framework;
  typer stays a `tools/` script dependency.
- `.tg-analytic/` is runtime state at the project root (gitignored): `.env`,
  `session.session`, one `<channel>.db` per channel, `media/`. **Entrypoints**
  anchor it on the cwd (`--project` overrides); the library never reads the cwd.

## MCP server (`server.py`)

- **The `publish_` prefix is the read/write split**, not a naming style: Claude
  Code matches permission rules by tool *name*. Roster and `permission_rules()`
  live in one module because a renamed tool without its rule is silently
  ungated; `tests/test_server.py` compares the halves.
- **Text only, never `structuredContent`** — Claude Code drops the content
  blocks when structure is present, so structure travels *inside* text. Register
  through the local `_tool` wrapper (`structured_output=False`);
  `assert_text_only` refuses to start a server whose tools carry an
  `outputSchema`.
- **Errors are `{code, message, hint}` JSON in the text block** with
  `isError: true`. `_guarded` wraps every tool, names `FLOOD_WAIT` (with
  `seconds`), and labels anything unanticipated `INTERNAL`.
- **`isError` is the *call* refusing, never one request inside it** (adr/0007).
  `run_query` takes a list, and a statement SQLite rejects comes back as that
  question's section — bad SQL beside three good answers must not discard them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lancetnik/slop-writer](https://github.com/Lancetnik/slop-writer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
