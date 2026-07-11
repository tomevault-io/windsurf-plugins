---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is (read first)

This repo **is** the installable `claude-context` Claude Code plugin, and it is **implemented and shipped**. The plugin itself is nested under `plugins/claude-context/`: its manifest (`plugins/claude-context/.claude-plugin/plugin.json`), `plugins/claude-context/SKILL.md`, `plugins/claude-context/scripts/` (collector + report engine + `lib/` helpers), and `plugins/claude-context/commands/setup.md`. `package.json`, `LICENSE`, `README.md`, the marketplace manifest (`.claude-plugin/marketplace.json`), and the `tests/*.test.mjs` suite stay at the repo root. Run the suite with `npm test` (`node --test tests/*.test.mjs`).

The design and plan under `docs/superpowers/` are **historical design records**, not a to-do list. Where they and the code disagree, **the code wins** — it has evolved past the plan's literal listings (most notably the snapshot is now session-keyed; see Conventions).

- **Design spec** — `docs/superpowers/specs/2026-07-07-claude-context-plugin-design.md`. The *what* and *why* (numbered §1–§10).
- **Implementation plan** — `docs/superpowers/plans/2026-07-07-claude-context.md`. The original 12-task TDD build.

`README.md` describes the shipped plugin.

## The one non-negotiable contract: instrument, not advisor

The plugin's entire purpose is that **agents and subagents can query their own resource state** (context %, 5h/7d usage, cost, compactions) and act on the *user's* rules — e.g. checkpoint before Claude Code's silent auto-compaction drops CLAUDE.md guidance, or pause before a rate limit. For that to work, the plugin must **only report facts**:

- **No** recommendations, evaluative/level/traffic-light labels, policy ("should compact", 70/85/90 thresholds), or derived/predictive metrics. Raw measurements and provenance only.
- The plugin supplies facts; the **user's instructions** (CLAUDE.md, skills, subagent prompts) supply any policy. A tool with baked-in thresholds would fight the ones the user wrote.
- This is enforced by a denylist test over **`report.mjs`'s rendered output** (plan Task 9) — **not** by grepping `plugins/claude-context/SKILL.md`, which legitimately uses words like "makes no recommendations" to *describe* the contract. Don't reintroduce a doc-vocabulary grep; it false-positives on the contract's own wording.

## Architecture

Two cooperating pieces plus the skill that ties them together:

- **Collector `plugins/claude-context/scripts/collect.mjs`** — an *opt-in* statusline script. Claude Code pipes stdin JSON to it (~300ms cadence); it writes a private per-session snapshot (`snapshot-<session-id>.json`, atomic, `0600`, throttled) keyed off stdin's `session_id`, and prints one compact line. Exposes `buildSnapshot`, `compactLine`, and `runCli()`.
- **Report engine `plugins/claude-context/scripts/report.mjs`** — what the skill runs. Resolves to exactly three states, keyed on whether a collector snapshot has ever been written: **collector** (a fresh snapshot exists → full numbers, `source: collector`), **collector idle** (a snapshot exists but is stale → context stays accurate — from the transcript against the snapshot's real window size, `source: transcript`, or from the stale snapshot's own `context` field, `source: collector` — while 5h/7d/cost go `null`, flagged `unavailable_stale`), and **not set up** (no snapshot has ever been written → zero transcript I/O, everything `null` including `compactions`, `source: unavailable`, `rate_limits_status: unavailable_collector`, `setup_required: true`). Emits a human block **and** a machine-readable JSON block.
- **Shared `plugins/claude-context/scripts/lib/*`** — dependency-free single-responsibility helpers (`config-dir`, `token-math`, `time`, `transcript`, `snapshot`). There are four I/O entry points — `collect.mjs`, `report.mjs`, `coexist.mjs` (the invisible-tap wrapper a co-existed statusline runs through), and `wiring.mjs` (the tested `settings.json`/sidecar surgery setup and teardown call); `lib/*` stays pure.

**The core constraint that explains the whole design:** Claude Code pushes `context_window`, `rate_limits`, and `cost` **only to the statusline process via stdin**. A skill/command invoked by an agent gets **no stdin** — it can read only the transcript JSONL, which has per-message token `usage` but **no `rate_limits`**. Hence: the collector exists to capture stdin into a snapshot the report can later read, and the transcript fallback (used only in the collector-idle state) is context-only.

**Provenance is first-class.** Every reported number carries `source`, `rate_limits_status` (`available` / `unavailable_plan` / `unavailable_provider` / `unavailable_stale` / `unavailable_collector`), `setup_required` (always present; `true` only in the not-set-up state), and `session_selection` (`session_id` / `unverified-snapshot`), so a consumer's rule never fires on a stale, never-set-up, or wrong-session value. A stale-but-present snapshot is a distinct state from "collector never installed."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ovid/claude-context](https://github.com/Ovid/claude-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
