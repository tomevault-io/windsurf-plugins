---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup
make install          # pip install requirements + dev deps + editable install

# Testing
make test             # pytest tests/ -v (all tests)
make test-unit        # Unit tests only (tests/unit, -m unit)
make test-integration # Integration tests only (tests/integration, -m integration)
make coverage         # pytest --cov with HTML + term-missing reports
pytest tests/unit/test_database.py -v                              # Single file
pytest tests/unit/test_database.py::TestDatabase::test_save -v     # Single test

# Code quality
make format           # black + isort
make lint             # flake8 (max-line-length=100) + mypy
make clean            # Remove build artifacts and __pycache__
```

## Architecture Overview

### Data Model

**ConversationTree** (`ctk/core/models.py`) is the central data structure. All conversations are trees; linear chats are single-path trees, branching conversations (e.g., ChatGPT regenerations) preserve all paths. Key methods: `get_all_paths()`, `get_longest_path()`, `add_message()`.

**Tree primitives** (since 2.13.0): every "fork / branch / detach / promote / snapshot" UI op decomposes into one of six primitives. Five live on the tree itself: `delete_subtree(n)`, `prune_to(n)`, `copy()`, `copy_subtree(n)`, `graft(n, other)`. The sixth is DB-level `delete_conversation(id)`. Tests in `tests/unit/test_tree_primitives.py`. Add a new tree operation by composing these, not by reaching into `message_map` directly. Reasoning helpers `descendants_of(n)` and `ancestors_of(n)` are public for the same reason.

**Database** is a two-layer design:
- `ctk/core/db_models.py`: SQLAlchemy ORM models (`ConversationModel`, `MessageModel`, `TagModel`, `EmbeddingModel`, `SimilarityModel`).
- `ctk/core/database.py`: `ConversationDB`, the high-level wrapper (save, load, search, list). FTS5 full-text search with LIKE fallback. The "db_path" is a directory; the SQLite file lives at `<dir>/conversations.db` and an associated `media/` directory holds image attachments.
- `ctk/core/db_operations.py`: maintenance ops (merge, diff, intersect, filter, split, dedupe).
- `ctk/core/pagination.py`: cursor-based keyset pagination via `encode_cursor()` / `decode_cursor()`.

### CLI surface (since 2.12.0)

`ctk` with no subcommand opens the Textual TUI. The full top-level command list is intentionally small:

| Command | Purpose |
|---|---|
| `ctk` (no args) | Open the TUI on the configured DB |
| `ctk tui` | Same; alias for muscle memory |
| `ctk import` | Bulk import conversation exports |
| `ctk export` | Bulk export to file |
| `ctk query` | Filter/search with formatted output (table/json/csv) |
| `ctk sql` | Read-only SQL on the DB |
| `ctk db` | Maintenance: init, info, vacuum, backup, merge, diff, intersect, filter, split, dedupe, validate |
| `ctk net` | Build embeddings + similarity graph (analytical queries are MCP tools, see TUI) |
| `ctk auto-tag` | Bulk LLM-driven tagging |
| `ctk llm` | Provider config: providers, models, test |
| `ctk config` | Edit `~/.ctk/config.json` |

The previous per-conversation, per-library, per-view, chat REPL, and ad-hoc network analysis subcommands all moved into the TUI as bindings, slash commands, or MCP tool calls.

### Textual TUI (`ctk/tui/`)

`CTKApp` composes a tabbed sidebar and a chat main pane.

**Sidebar tabs**: All / Starred / Pinned / Recent / Archived. Search overlay at `/`. Implemented in `ctk/tui/sidebar.py`.

**Sidebar pagination** (since 2.14.0): cursor-based keyset pagination via `ConversationList.DEFAULT_PAGE_SIZE` (200 rows). The header shows `conversations · N loaded · more (Ctrl+L)` when more pages remain. `load_more()` appends the next page in place; switching mode/search resets the cursor and reloads. Backed by `db.list_conversations(cursor=…, page_size=…)` and `db.search_conversations(cursor=…, page_size=…)`, both of which return `PaginatedResult(items, next_cursor, has_more)` when `cursor` is not None.

**Main pane** (`ctk/tui/main_pane.py`): scrollable message bubbles, multi-line chat input at the bottom. Bubbles are focusable (`Tab` / `Shift+Tab` between them). Branch indicators with `[` / `]` to switch siblings.

**Bindings**:
- `q` quit; `Ctrl+R` refresh; `Ctrl+N` new conversation; `Ctrl+H` help modal
- `Ctrl+F` fork at focused message (truncate); `Ctrl+B` branch (preserve full tree)
- `Ctrl+D` delete subtree at focus; `Ctrl+E` extract subtree at focus; `Ctrl+P` promote focused path
- `Ctrl+L` load more conversations into the sidebar (when `more` indicator is showing)
- `Ctrl+S` toggle star; `Ctrl+G` system prompt modal; `Ctrl+O` attach file modal
- `[` / `]` previous/next sibling at focused message

**Slash commands** (`ctk/tui/slash.py`): typed in the chat input. Routed to dispatcher before the LLM. `/help` lists them all. Includes `/mcp`, `/model`, `/system`, `/title`, `/star`, `/pin`, `/archive`, `/tag`, `/untag`, `/export`, `/attach`, `/fork`, `/branch`, `/clone`, `/snapshot`, `/delete`, `/delete-subtree`, `/extract`, `/detach`, `/promote`, `/graft`, `/clear`, `/sql`, `/quit`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [queelius/ctk](https://github.com/queelius/ctk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
