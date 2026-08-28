---
trigger: always_on
description: Project-specific operating manual for any AI agent (Claude Code, Codex, Cursor) working in this repo. Read this first when a conversation opens. This file is sufficient for **repo mechanics + invariants** - directory map, dispatch wiring, test harness, the rules you must not break - so a fresh mid-size model can add/modify a subcommand or fix a bug respecting every invariant without spelunking the source. For the exact **behavioural contract** of a subcommand (precise flag semantics, output shap
---

# AGENTS.md - csift operating manual

Project-specific operating manual for any AI agent (Claude Code, Codex, Cursor) working in this repo. Read this first when a conversation opens. This file is sufficient for **repo mechanics + invariants** - directory map, dispatch wiring, test harness, the rules you must not break - so a fresh mid-size model can add/modify a subcommand or fix a bug respecting every invariant without spelunking the source. For the exact **behavioural contract** of a subcommand (precise flag semantics, output shape, edge cases), open the matching `SPEC.md` section 6 subsection; SPEC is the authority on *what* each command does, this file on *how* to work in the repo.

> **About this file.** `AGENTS.md` is the canonical, vendor-neutral filename. `CLAUDE.md` is a **symlink to `AGENTS.md`** - edit `AGENTS.md` only; the symlink follows. **Companion doc:** [`SPEC.md`](./SPEC.md) (~175KB / ~1240 lines) is the product/behaviour spec - open it whenever you change search/verbatim/recover/etc. behaviour. Concrete pointers: SPEC **section 4** = record semantics, **section 5** = the full label taxonomy (`search -t` role.class.sub), **section 6** = per-subcommand specifications (flags, output shape, edge cases), **section 7** = the performance contract, **section 8.2** = the JSON envelope v2; `## 6 Subcommand specifications` opens with the per-version CHANGE LEDGERS, ordered newest-first (the newest entry is authoritative; historical entries describe their own versions' surfaces). This file = how to work in the repo; SPEC.md = what to build. **The FIVE-DOCUMENT CONTRACT** (user-ruled, v0.5.1): `SKILL.md` = the LLM manual (density + weak-attention retrievability) · `--help` = the human (CLI-proficient) manual — INFORMATION-PARITY with SKILL, never thinner, but plain prose with stronger structure/layout (help IS surface: changing it bumps the version triple) · `README.md` = promotion · `SPEC.md` = design intent · this file = maintenance. When code and a doc disagree, **code wins** - re-verify, then fix the doc.

---

## 1. What csift is

**csift - "ripgrep for Claude Code session transcripts".** A fast Rust CLI that **lists** and **regex-searches** Claude Code session `.jsonl` files under `~/.claude/projects/`, plus recovers files, reconstructs compaction-clipped turns, maps subagents, and extracts pasted images.

- **Primary consumer is an LLM** - a Claude Code agent searching/recovering its own or a peer session. Output is clean, token-efficient, regex-driven. Default output is human/LLM-readable (session/turn/label/timestamp headers); `--format json` is the machine format (there is no bare `--json` flag). `search --raw` / `show --raw` emit VERBATIM jsonl lines (the escape hatch for unrendered fields).
- **Explicitly NO BM25 / embeddings / semantic search.** Pure regex only. Lexical tokenisation across scripts (CJK / multi-byte) is intractable for scoring; regex is the strength and the whole point. Never add semantic search.
- **Eleven subcommands** (`Command` enum, `cli.rs`): `list`, `search`, `show`, `stats`, `agents`, `whoami`, `files`, `recover`, `plan`, `verbatim`, `image` (+ ONE hidden `Turns` tombstone variant, v0.6.4 — never runs, always bails with the rename pointer to `verbatim`; keep it bare + hidden). `show` = record FETCH (by `--line`/`--uuid`/`--turn`, one transcript, rendered full or `--raw` verbatim bytes — search no longer fetches; `--turn N|A..B` fetches EVERY record of the named turn(s) in the SAME `·tN` numbering `search`'s `<tok>·tN` headers print (v0.7.0: `<tok>` = a STABLE first-8 id-prefix token, an `@` target as-is; the `sN` legend/ordinals are GONE), so `show --turn -3..` is the tail-peek/monitoring path — "read a session's recent turns from the live transcript"); `verbatim` = the compaction-fidelity specialist (reconstruct the verbatim user/assistant turns a compaction summary clipped — the command FORMERLY named `turns`, a zero-BC rename: `csift turns …` is now an unrecognized subcommand; the module file stays `src/turns.rs`, handler `turns::run_verbatim`, args `cli::VerbatimArgs`, variant `Command::Verbatim`); `stats` = one-scan per-session aggregates (tokens by model / tools / turns / span / compactions / a whole-file line-type census). (There is NO `pending` subcommand — the elicitation sidecar it once read is merged TRANSPARENTLY into every record-reading surface; see §3.10.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wdhwg001/csift](https://github.com/wdhwg001/csift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
