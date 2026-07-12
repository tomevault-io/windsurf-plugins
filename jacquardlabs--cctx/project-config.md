---
trigger: always_on
description: Open-source Python CLI that diagnoses individual Claude Code sessions: when they went wrong, why they went wrong, what they cost, and what to add to your `CLAUDE.md` so it doesn't happen again. Reads the JSONL session logs Claude Code writes to `~/.claude/projects/` and produces actionable autopsy reports.
---

# cctx

Open-source Python CLI that diagnoses individual Claude Code sessions: when they went wrong, why they went wrong, what they cost, and what to add to your `CLAUDE.md` so it doesn't happen again. Reads the JSONL session logs Claude Code writes to `~/.claude/projects/` and produces actionable autopsy reports.

The complete product pitch, example outputs, growth staircase, and positioning vs. adjacent tools are in [`cctx-project-brief.md`](cctx-project-brief.md). Read it once.

## Tech stack

- **Python 3.10+**
- **click** — CLI argument parsing and subcommand routing
- **rich-click** — re-skins click's `--help` through rich (drop-in: `import rich_click as click`). Pure shininess win on the `--help` surface, no behavioral cost
- **rich** — terminal output: tables, banners, severity badges, diff blocks
- **textual** — the TUI for `cctx trace`
- **anthropic** — token counting only, via `anthropic.messages.count_tokens` in the tokenizer module. **Not imported anywhere else.**
- **pandas** — optional, only inside the cross-session aggregator if/when row-level work justifies it. Stdlib-first.
- **Jinja2** — HTML report templates for `cctx autopsy --html`

Explicitly not used: web frameworks, databases, ORMs, async runtimes, cloud SDKs. cctx is a local CLI.

**Multi-provider.** `parsers/otel.py` (shipped v1.12.0) parses OTLP `gen_ai.*` spans, so `cctx autopsy <trace>` also diagnoses OpenAI Agents SDK and LangGraph sessions exported via OTEL. `cli._detect_source()` sniffs the first lines of a trace file and routes to the Claude Code or OTEL parser. Both parsers return the same `SessionTrace`, so everything downstream is provider-agnostic. See `docs/quickstart-otel.md`.

## Architecture

```
Session log (Claude Code JSONL on disk)
  ↓
Parser           ← dependency-free; takes a path, returns SessionTrace
  ↓
Tokenizer        ← only place that imports anthropic; offline-mode safe for CI
  ↓
Diagnostician    ← per-turn investigation: inflection detection + pattern
                   classifiers (retry loop, scope creep, stale context,
                   tool thrash, dead end, fan-out waste, cache hygiene,
                   compaction, exploration thrash, unused context;
                   project-specific runs cross-session). Produces a Diagnosis.
  ↓
Recommender      ← turns Findings into Patches: copy-pasteable CLAUDE.md /
                   rule / skill diffs, evidence-backed when cross-session.
  ↓
Renderers        ← rich (terminal), Jinja2 (HTML report), textual (trace
                   TUI overlay).
  ↓
Exporters        ← jsonl, csv.
```

## Project layout

```
cctx/
├── cli.py              # click + rich-click; routes all 7 subcommands
│                       # (ls, autopsy, harvest, watch, trace, export, init).
├── parsers/
│   ├── claude_code.py  # SHIPPED. Parse ~/.claude JSONL logs.
│   └── otel.py         # SHIPPED (v1.12.0). Parse OTLP gen_ai.* spans —
│                       # OpenAI Agents SDK, LangGraph. Same SessionTrace out.
├── tokenizer.py        # SHIPPED. anthropic.count_tokens wrapper; CCTX_OFFLINE heuristic.
├── pricing.py          # SHIPPED. get_pricing() -> ModelPricing (2D input/output +
│                       # cache mults) via longest-prefix match; Anthropic + OpenAI.
│                       # PRICING_LAST_VERIFIED date + freshness tripwire test. Single
│                       # source of truth for cost; price_per_tok() kept as a shim.
├── models.py           # SHIPPED. Turn, ToolUse, ToolResult, Usage, Attachment,
│                       # RawToolResultFile, SessionTrace, Finding, Patch, Diagnosis,
│                       # SubagentAttribution, KIND_LABEL, MANAGED_HEADINGS,
│                       # AggregateReport, CrossProjectDigest.
├── diagnostician/
│   ├── __init__.py     # public: run(trace) -> Diagnosis. Wires the classifiers,
│   │                   # runs the 9 per-turn ones recursively inside each subagent
│   │                   # (priced per-model), + per-subagent cost attribution.
│   ├── inflection.py   # detect the turn where the session diverged
│   └── patterns/
│   │   ├── retry_loop.py
│   │   ├── scope_creep.py
│   │   ├── stale_context.py
│   │   ├── tool_thrash.py
│   │   ├── dead_end.py
│   │   ├── fan_out.py            # subagent overlap + retry waste
│   │   ├── cache_hygiene.py      # KV-cache hit rate + cause
│   │   ├── compaction.py         # compaction events + re-fetch waste
│   │   ├── exploration_thrash.py # read-heavy circling without progress
│   │   ├── unused_context.py     # MCP servers loaded but never called
│   │   └── project_specific.py   # cross-session only -> PROJECT_PATTERN
├── aggregate.py        # SHIPPED. Cross-session pipeline ORCHESTRATOR (--since mode):
│                       # globs sessions, runs parse -> tokenize -> diagnostician.run
│                       # per session. Lives at top level — it orchestrates, not analyses.
├── recommender/
│   ├── claude_md.py    # Finding -> Patch (CLAUDE.md diff proposals)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacquardlabs/cctx](https://github.com/jacquardlabs/cctx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
