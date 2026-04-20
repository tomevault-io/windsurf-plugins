---
trigger: always_on
description: A deterministic, schema-driven PDF generation tool for business documents (Purchase Orders, Invoices, Requests for Quotation, etc.). It is invocable by **any AI agent** that can execute shell commands — Claude, Cursor, Gemini, Codex, or any other tool. No LLM is involved in rendering. Same input always produces the same PDF.
---

# doc-generator

A deterministic, schema-driven PDF generation tool for business documents (Purchase Orders, Invoices, Requests for Quotation, etc.). It is invocable by **any AI agent** that can execute shell commands — Claude, Cursor, Gemini, Codex, or any other tool. No LLM is involved in rendering. Same input always produces the same PDF.

---

## How to Run Locally

```bash
# Install Python dependencies
uv sync

# macOS system dependency — install once via Homebrew:
# brew install pango
# Ubuntu/Debian system dependency:
# sudo apt-get install libpango-1.0-0 libharfbuzz0b libpangoft2-1.0-0
# WeasyPrint requires Pango/GObject. On macOS the dylibs are in /opt/homebrew/lib/,
# which is not on the default dyld search path. Prefix every uv run with:
# DYLD_LIBRARY_PATH=/opt/homebrew/lib

# Generate a Purchase Order from a JSON payload
DYLD_LIBRARY_PATH=/opt/homebrew/lib uv run python scripts/generate.py \
  --doc_type purchase_order --payload tests/fixtures/sample_po.json

# Same, but open the PDF immediately after generation
DYLD_LIBRARY_PATH=/opt/homebrew/lib uv run python scripts/generate.py \
  --doc_type purchase_order --payload tests/fixtures/sample_po.json --preview

# Generate an Invoice
DYLD_LIBRARY_PATH=/opt/homebrew/lib uv run python scripts/generate.py \
  --doc_type invoice --payload tests/fixtures/sample_invoice.json --preview

# Generate a Request for Quotation
DYLD_LIBRARY_PATH=/opt/homebrew/lib uv run python scripts/generate.py \
  --doc_type request_for_quotation --payload tests/fixtures/sample_rfq.json --preview

# Test validation error output (non-zero exit code, structured error to stdout)
DYLD_LIBRARY_PATH=/opt/homebrew/lib uv run python scripts/generate.py \
  --doc_type purchase_order --payload tests/fixtures/invalid_po.json
```

---

## CLI Contract (Platform-Agnostic Interface)

Any agent invoking this tool must use the following interface. This is the **complete** contract — there are no interactive prompts, no assumed environment variables, and no implicit state.

```text
uv run python scripts/generate.py --doc_type <type> --payload <path> [--preview] [--save_payload]
```

| Argument | Required | Description |
|---|---|---|
| `--doc_type` | Yes | Document type slug. Must match a registered type (e.g. `purchase_order`, `invoice`, `request_for_quotation`). |
| `--payload` | Yes | Absolute or relative path to a JSON file containing the document data. **File path only** — not inline JSON. This avoids shell escaping issues and lets agents write a temp file before invoking. |
| `--preview` | No | If provided, opens the generated PDF using the OS default viewer after generation. Gracefully no-ops in headless environments (no display, CI). |
| `--output_name` | No | Custom filename stem. If provided, output is `<doc_type>_<name>.pdf`. Defaults to date + sequential counter auto-naming. |
| `--output_dir` | No | Directory to save the generated PDF. Defaults to `<project_root>/output/`. Pass `$(pwd)` to save in the caller's working directory. |
| `--save_payload` | No | If provided, saves the validated payload (with all computed fields) as a `.json` file alongside the PDF, using the same filename stem. |

**On success:** Writes the PDF to the target directory (default `<project_root>/output/`) and prints the **absolute** output path to stdout. Exit code `0`. Agents must use this path directly — never prepend the working directory. If `--save_payload` was passed, a `.json` file with the same stem is also written.

**On validation error:** Prints a structured, human-readable error to stdout describing which fields failed and why. Exit code `1`. No PDF is written.

**On unknown doc_type:** Prints a list of registered doc types to stdout. Exit code `1`.

Agents should capture stdout and check the exit code to determine success or failure.

---

## Folder Structure

```text
doc-generator/
│
├── CLAUDE.md                    ← You are here. Entry point for all AI agents.
├── SKILL.md                     ← Claude-specific skill instructions (orchestration layer: trigger conditions, invocation, error relay — delegates data collection detail to references/<doc_type>.md) — uses ~/.agents/skills/doc-generator as the canonical install path
│
├── .claude/
│   └── settings.json            ← Pre-approved permissions: Write(/tmp/) + Bash CLI invocation (no prompts for team)
│
├── .github/
│   └── workflows/
│       └── sync-skill.yml       ← Auto-opens PR to vercel-labs/agent-skills when SKILL.md changes on master
│
├── pyproject.toml               ← uv project manifest with dependencies (weasyprint, jinja2, pydantic)
├── uv.lock                      ← Locked dependency versions (auto-managed by uv)
│
├── scripts/
│   └── generate.py              ← Thin CLI entrypoint: argparse + generation pipeline (~95 lines)
│
├── builders/                    ← Context builder package — one module per doc type
│   ├── __init__.py              ← DocTypeConfig dataclass + REGISTRY (single registration point)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julioccorderoc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
