---
trigger: always_on
description: |
---


# claude-design-skill

> Status: **Step 1–7 shipped · v1.0.0 (2026-05-12)**
> All body sections are authored; the design-knowledge catalog,
> animation engine, and prebuilt showcases ship alongside. Step 5
> promotes `team-brand-spec` from placeholder to operational default
> (evidence-anchored to an 11-service style sweep) and adds a
> Figma → spec extractor for adopters who manage brand in Figma.
> Security and routing rules are load-bearing and govern every
> external call this skill can make. See `PROJECT-PLAN.md §7` for
> the per-step decisions log and license-clean evidence.
>
> **For AI agents in a fresh session**: read `HANDOFF.md` at the repo root
> first. It contains the working-style briefing and anti-pattern list
> that prevents the most common rework loops.

## What this skill is for

Hi-fi prototypes, slide decks, animations, and Figma-MCP-driven precision
design work, executed by an LLM agent through Claude Code. The skill
provides:

- **Security baselines** that govern every external fetch (allowlist,
  WebSearch policy, Codex CLI policy, codename redaction).
- **Sanitizers** for external assets (SVG XXE/script removal, PNG chunk
  scan, AI-generated PNG strip-and-import).
- **Figma MCP workflow** with five sub-guides covering selection-aware
  edits, layer renames, component grouping, brand-spec import, and the
  meta hub.
- **Codex CLI bridge** for GPT-5.5 reasoning + gpt-image-2 image
  generation, with a hard-fail import gate that strips C2PA
  metadata and re-scans the result.

Body sections — design philosophies, scene templates, slide rules,
animation rules, prototype scaffolding — were authored from scratch
across Step 2 and Step 3 (2026-05-09 → 2026-05-10). They live below
under their own `##` headings (Junior Designer workflow, Anti-AI-slop
checklist, App prototype rules, Slide deck conventions, Tweaks
live-tuning system, Critique guide) and route into
`references/design-styles.md`, `references/scene-templates.md`, and
the animation references for the deeper catalogs. No upstream prose
was inherited.

## Core Principle #0 · Fact verification before assumptions

Any factual claim about the existence, release status, version number,
or specifications of a specific product, technology, event, or person —
**the first action MUST be `WebSearch`**. Asserting from training-corpus
memory is forbidden.

### Trigger conditions (any one)

- The user mentions a specific product name you are not familiar with
  or unsure about.
- Anything involving release timelines, version numbers, or specs from
  2025 onward.
- You catch yourself thinking "I think it's…", "should not have launched
  yet", "probably around…", "might not exist".
- The user requests design materials for a specific product or company.

### Hard procedure (execute before clarifying questions)

1. **Confidentiality gate (must answer before any external call)**: is
   the term a publicly released external brand or product (e.g., DJI
   Pocket 4, Apple Vision Pro, Stripe), or could it be an internal
   codename, unreleased team product, or partner under NDA? If the
   second — **do not search and do not send to Codex**. Ask the user
   instead. The same gate applies to `WebSearch`, `Bash(curl …)`,
   `Bash(codex exec …)`, and any image-generation prompt. Pattern
   table lives in `references/security-config.md §1.5`;
   `scripts/codex-image-import.py` re-runs the check at file-move time
   and exits 4 on any match.
2. `WebSearch` for the product name + a recent time keyword
   ("2026 latest", "launch date", "release", "specs") — only for public
   external brands cleared in step 1.
3. Read 1–3 authoritative results to confirm: existence, release
   status, latest version, key specs.
4. Write the facts into the project's `product-facts.md` (per-project
   scratch file, gitignored). Don't rely on memory.
5. Can't find anything or results are ambiguous → ask the user, don't
   assume.

### Forbidden phrasing

- ❌ "I remember X hasn't launched yet"
- ❌ "X is currently version N" (an unsearched claim)
- ❌ "X probably doesn't exist"
- ❌ "As I recall, X's specs are…"
- ✅ "Let me `WebSearch` X's latest status"
- ✅ "Authoritative sources I found say X is…"

This principle outranks "ask clarifying questions" — asking questions
presupposes the facts are right. If facts are wrong, every question is
skewed.

## Core Principle #1 · Security-first asset handling

Every external asset must pass through a sanitizer before it lands in
this repository. The sanitizer is whitelist-based and recorded by SHA-256
in PROVENANCE so future audits can detect tampering.

| Asset type | Gate | Implementation |
|---|---|---|
| External SVG (logos, illustrations) | XXE-guarded XML parse + tag/attr whitelist + CSS sub-sanitize + visibility comment | `scripts/svg-sanitize.py` + `references/svg-sanitize.md` |
| External PNG / JPG | Chunk/segment scan: trailing data, non-whitelist chunks, oversized text metadata | `scripts/scan_assets.py` |
| Codex / gpt-image-2 generated PNG | C2PA `caBX` strip + recursive discovery + scan re-run + hard-fail | `scripts/codex-image-import.py` + `references/codex-design-workflow.md` |
| External fetch (curl, wget, etc.) | Domain allowlist + per-call user approval + WebSearch policy | `references/security-config.md` + `examples/dot-claude-settings.json` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xmhha/claude-design-skill](https://github.com/0xmhha/claude-design-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
