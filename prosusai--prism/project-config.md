---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Prism** — knowledge layer for **Claude Code and Cursor**. Two things:
1. **Personal learning**: hooks observe tool usage, a two-phase extraction pipeline (fast model proposes, strong model validates — via `claude` or Cursor `agent` per backend) converts patterns into engrams (living, decaying knowledge). Engrams flow back into the IDE via context files (`.claude/prism.md`, `.cursor/rules/prism.mdc`) and MCP tools.
2. **Team skills**: high-confidence engrams promote to skills published to a Cloudflare Worker-backed registry that teams query.

### Hard constraints

- **Hooks never block the IDE** — `capture.sh` (Claude Code) and `capture_cursor.sh` (Cursor) must always exit 0; background spawns only.
- **Storage split** — observations + sessions live in SQLite (`~/.prism/prism.db`) via stdlib `sqlite3`. Engrams stay flat Markdown + YAML frontmatter; the engram index stays `index.json`. No external DB, no ORM.
- **AI calls via IDE CLIs only** — `claude --print` (Claude Code) or `agent -p` (Cursor). Never import the Anthropic SDK or cursor-sdk. No API keys for extraction. Routed through `lib/agent_runner.py` (`resolve_backend` + `run_agent`).
- **One extraction CLI per user** — Claude-only needs `claude login`; Cursor-only needs `agent login`. Not both unless you use both IDEs.
- **Custom YAML frontmatter parser** — never import PyYAML. Split on `---`, parse `key: value` lines.
- **`subprocess.run()` not `os.system()`** — always use `capture_output=True, text=True, timeout=N`.
- **MCP stdout is protocol-only** — any stray `print()` in lib code corrupts the JSON-RPC stream. All logging to stderr.
- **Never read `.env` files** — config comes from `os.environ` only. No dotenv parsing, no opening `.env` files.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Library / CLI | Python 3.12+ (stdlib only) | `argparse`, `json`, `pathlib`, `subprocess`, `hashlib`, `fcntl` |
| Hooks / installer | Bash (POSIX-compatible) | `capture.sh` / `capture_cursor.sh` → `capture.py`. Avoid Bash 4+ features (macOS ships 3.2) |
| AI calls | `claude` CLI or Cursor `agent` CLI | Claude: `haiku` / `sonnet`. Cursor: `cursor_models.fast` / `cursor_models.strong` (defaults: `composer-2.5[fast=false]`, `claude-4.6-sonnet-medium`). All via `lib/agent_runner.py`. |
| MCP server | Python stdio, JSON-RPC 2.0 | Protocol version `2025-03-26`. Tools only, no resources/prompts |
| Storage | SQLite (stdlib `sqlite3`) + flat files | `prism.db` = observations + sessions + `observations_fts` (FTS5); `index.json` = engram index; Markdown engrams |
| Registry API | Cloudflare Worker (TypeScript) | Wrangler 4.x, Node 22 LTS — for registry maintainers only, not end users |
| Registry backend | GitHub repo | Versioning, PRs, CI, and hosting for free. No database needed |

### IDE integration points

Prism supports **Claude Code and Cursor**; `prism init` configures both (unused IDE integration is inert until you use that IDE).

| Integration | Claude Code | Cursor |
|-------------|-------------|--------|
| Hook (observe) | `.claude/settings.local.json` → `PreToolUse`, runs `capture.sh pre` (sets `PRISM_SOURCE=claude_code`) | `.cursor/hooks.json` → `preToolUse`, runs `capture_cursor.sh pre` (sets `PRISM_SOURCE=cursor`) |
| MCP (query) | `~/.claude.json` → `projects[cwd].mcpServers.prism` | `~/.cursor/mcp.json` → `mcpServers.prism` |
| Skills | `.claude/skills/` symlinks → `~/.prism/skills/` | `.cursor/rules/` |
| Context push | `.claude/prism.md` | `.cursor/rules/prism.mdc` |

**Extraction backend** (`agent_backend: auto`): `--backend` flag → `PRISM_AGENT_BACKEND` → `config.agent_backend` → `PRISM_SOURCE` → unanimous pending observation source → `mixed_backend_preference` when mixed → `claude`. Hook auto-extract passes `--backend` from `PRISM_SOURCE` (`trigger.py`). Manual `prism extract` on mixed pending prefers `mixed_backend_preference` when both CLIs are installed.

Shared rules: hooks are `preToolUse`-only (one observation per tool call) and exit 0 always; hook scripts prepend `~/.local/bin` and `~/.cursor/bin` to PATH so background `prism extract` → `agent`/`claude` resolves from GUI-launched IDEs; MCP is a stdio JSON-RPC subprocess (flush stdout after every write); context files are written by `prism sync` and read as project instructions.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

- **Project ID** is SHA256[:12] of git remote URL (portable) or repo root path (fallback). Never hardcode it.
- **Index writes** (`index.json`) use `fcntl.flock` + atomic `os.rename` (write to `.tmp`, rename). A `.bak` is kept. Stale locks > 10 min are auto-broken.
- **Observations** are written to SQLite (`prism.db`) via `storage.insert_observation()`. The `observations_fts` (FTS5) virtual table is kept in sync by triggers. (Legacy per-project `observations.jsonl` was migrated to SQLite; `.migrated.*` leftovers may remain on disk.)
- **Engram IDs** are kebab-case slugs derived from trigger text, max 60 chars.
- **Frontmatter** is hand-parsed: split on `---` delimiters, parse `key: value` lines. No PyYAML.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ProsusAI/prism](https://github.com/ProsusAI/prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
