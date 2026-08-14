---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code plugin (`logseq-brain`) that gives Claude persistent memory via a user-owned Logseq graph. There is **no build, no tests, no runtime code** — the plugin is entirely markdown skills (`skills/<name>/SKILL.md`) plus `.claude-plugin/plugin.json`. Claude itself is the runtime: skills instruct Claude to read/write markdown files in the user's `ClaudeBrain` graph using the standard Read/Write/Edit/Bash tools.

**Target: Logseq OG (the markdown version) only.** Logseq split in 2026 — OG moved to <https://github.com/logseq/og> and is in maintenance mode (security and Electron upgrades, no new features), while the DB/SQLite version continues at the original repo. There is no API or CLI for file graphs (`@logseq/cli` serves DB graphs only), so all leverage here is file layout, property discipline, and ripgrep. Do not propose DB-version features.

The plugin is distributed via the [skillsmith](https://github.com/jame581/skillsmith) marketplace, the Gemini extension URL, and (for Cowork) a locally-built `logseq-brain.plugin` zip. The `.plugin` archive is a **build artifact** — gitignored (`*.plugin`), not checked in. Edit files under `skills/` and `.claude-plugin/`, never inside an archive.

## Architecture

Five skills make up the save/load cycle against a Logseq graph:

- **brain-init** — First-time graph setup (creates `pages/Index.md`, `Meta.md`, `Decisions.md`, `logseq/config.edn`, `journals/.gitkeep`) and adds new project pages. New project pages ship with a digest scaffold, so a fresh page never lands in a `missing-digest` report.
- **brain-load** — Reads a project page back into the session. Since v0.10.0 the default is **digest mode**: one `Read(offset 0, limit 20)` of the page-top properties + `## Digest`, plus today's journal's mention(s) of the project (shrunk to fit), and then nothing else — everything further is fetched on demand through the escalation ladder, announced. A page with no digest falls back to the pre-v0.10.0 brief mode unchanged and is *offered* a digest, never given one uninvited. Also full mode, fuzzy project name matching, and cross-graph search ("what do we know about X").
- **brain-save** — Surgically appends session logs, decisions, plan updates to the relevant `Projects___<Name>.md` page via Edit. Also updates journals, Meta, Index. Detects cross-project decisions and decision conflicts (marks old as `status:: superseded`); seeds/updates task `status::`, suggests rotation of Session Logs past 64 KB/40 entries, refreshes the project's `Index.md` one-liner on every save, prompts on decision-shaped statements, and runs a mechanical post-write verify grep over the files it wrote. Since v0.10.0, step 9 of 13 **unconditionally refreshes the page's digest** — properties plus the `## Digest` section, with the Map bullet recomputed from the file rather than remembered.
- **brain-status** — Dashboard across all project pages, built from a **single ripgrep** over digest properties since v0.10.0; pages not yet backfilled fall back to section-targeted reads one page at a time. Flags stale projects; groups task pages by `status::`. Separate `brain-stats` analytics mode.
- **brain-doctor** — Graph-hygiene lint/repair (v0.8.0, extended in v0.9.0). Scans pages + journals for format violations that spawn phantom pages or broken macros (`{{ }}` mis-used for inline code, bare `#number`/hex tags, un-namespaced `[[Task]]` links, `[[file://]]` links, junk/description links), reports them, and — after a backup — repairs them. Includes the `jira-markup` residue check and the guided task-status backfill. v0.10.0 takes the rule catalog to **14** — adding `missing-digest`, `stale-digest`, `oversized-digest`, and (wave A) `stale-map`, all report-tier except the safe-only subset of `oversized-digest` — plus a guided whole-graph digest backfill ("backfill digests") that states its read cost before spending it. Maintenance tool, run on demand; not part of the per-session save/load cycle.

### Shared references (since v0.6.0)

Cross-skill logic lives under `skills/_shared/` — sibling to the skill folders, not inside any individual skill's `references/`. Each `SKILL.md` reads from `skills/_shared/<name>.md` on demand. This keeps `SKILL.md` orchestrators compact and avoids duplicating logic across skills.

Current shared references:

- `skills/_shared/path-resolution.md` — host-aware graph path resolution (Cowork vs. Claude Code/Copilot/Gemini)
- `skills/_shared/journey-log.md` — one-line activity-trail write logic, called by every brain skill
- `skills/_shared/staleness.md` — stale-project rules (used by `brain-load` and `brain-status`)
- `skills/_shared/section-locator.md` — grep-anchored section-targeted reads (used by `brain-load`, `brain-save`, `brain-status` to avoid full-page reads)
- `skills/_shared/logseq-format.md` — Logseq parse-time normalization behaviors + read-before-edit survival rules + compose-time content-generation invariants (used by brain-save, journey-log, brain-doctor; defers detection/remediation to hygiene-rules.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jame581/LogseqBrain](https://github.com/jame581/LogseqBrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
