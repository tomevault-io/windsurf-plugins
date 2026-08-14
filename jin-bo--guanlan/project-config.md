---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

观澜 (GuānLán) is an implementation of the [Karpathy LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): an Agent incrementally builds and maintains a structured, cross-linked markdown knowledge wiki instead of doing fresh RAG retrieval on every query. The full design (in Chinese) is the authoritative spec — read [`docs/DESIGN.md`](docs/DESIGN.md) before any non-trivial change.

**Current status:** released through v0.1.21; the roadmap is fully implemented — P2 minimal closed loop, P3 health/graph family, P4 optional host layer (Web + MCP, through P4.17's Streamable HTTP transport, P4.18's move to MCP SDK v2 / protocol `2026-07-28`, P4.19's read-only Web panel for the *opposite* direction — the external MCP servers agentao already injects into this KB — and P4.20's in-browser rendering of ` ```flint ` chart-spec blocks into data charts), P5 retrieval + multi-format ingest, and every half-phase P2.1–P5.4. No roadmap spec remains unimplemented; per-phase flags, behavior, and red lines live in the `docs/P*.md` file for that phase, not here.

CLAUDE.md does **not** restate phase history or per-decision detail — authoritative sources:
- Per-version change detail → [`CHANGELOG.md`](CHANGELOG.md)
- Milestone table → [`docs/DESIGN.md`](docs/DESIGN.md) §7
- Single-phase design/decisions → `docs/P*.md` (one file per phase)

When adding features, match the phase boundaries in DESIGN §4.4 / §7 and preserve the Invariants below.

## Commands

```bash
uv run guanlan init /tmp/demo            # scaffold a knowledge base (deterministic, zero-LLM)
uv run guanlan -C /tmp/demo check        # deterministic validation (frontmatter / broken links / sources)
uv run guanlan -C /tmp/demo health       # P3: stub pages + index↔disk sync (advisory; --strict → exit 6)
uv run guanlan -C /tmp/demo lint         # P3: orphans / broken links / missing entities (advisory)
uv run guanlan -C /tmp/demo graph        # P3: write graph/graph.json + graph.html (--json-only skips html)
uv run guanlan -C /tmp/demo reindex      # P3.4: register disk pages missing from index.md (zero-LLM; --dry-run / --prune)
uv run guanlan -C /tmp/demo search 关键词 # P5.0: BM25 + CJK 2-gram whole-page recall, top-N (zero-LLM; --limit / --json)
uv run guanlan -C /tmp/demo remove <slug> # P3.9: retract a source → .trash/ + strip refs + fix index (zero-LLM; previews by default, --yes to write)
uv run guanlan -C /tmp/demo heal --dry-run   # P3.2: materialize high-frequency missing entities (LLM via the P2 write gate; --dry-run = zero-LLM worklist)
uv run guanlan -C /tmp/demo audit --dry-run  # P3.7: semantic audit of drifted sources (LLM via the P2 write gate; --dry-run = zero-LLM)
uv run guanlan -C /tmp/demo convert 报告.pdf  # P5.2: multi-format → raw/<slug>.md via pdf-to-markdown skill (zero-LLM host write; --dry-run / --overwrite / --ingest / --backend)
uv run guanlan -C /tmp/demo web --no-browser   # P4: optional local Web host (needs guanlan-wiki[web]; 127.0.0.1 only)
uv run guanlan -C /tmp/demo mcp          # P4.10/P4.17: optional read-only MCP server (stdio; --transport http for Streamable HTTP) (needs guanlan-wiki[mcp])
uv run guanlan install-skill             # copy the bundled skills (guanlan-wiki + the pdf-to-markdown / flint-chart-author aux pair) into ~/.agentao/skills/ (external-base mode)
uv run pytest                            # run all tests
uv run pytest tests/test_web.py          # P4 Web host tests (skipped if guanlan-wiki[web] absent)
uv run pytest tests/test_mcp.py          # P4.10 MCP host tests (skipped if guanlan-wiki[mcp] absent)
uv run pytest tests/test_web_mcpdiag.py  # P4.19 Web MCP diagnostics (the reverse direction: external servers injected *into* this KB)
uv run --extra web python scripts/smoke_p420.py  # P4.20 flint chart rendering — real-browser smoke (Playwright; not in pytest)
uv run pytest tests/test_convert.py      # P5.2 convert tests (mock skill backend; zero-LLM)
uv run pytest tests/test_init.py::test_init_is_idempotent_and_non_destructive  # single test
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jin-bo/guanlan](https://github.com/jin-bo/guanlan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
