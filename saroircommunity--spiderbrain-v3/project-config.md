---
trigger: always_on
description: SpiderBrain v3 © 2026 Perform Digital Pvt Ltd
---

<!--
SpiderBrain v3 © 2026 Perform Digital Pvt Ltd

Licensed under BUSL-1.1. Free for personal, educational, research, and open-source use. Forking and internal modification for research are permitted. Redistribution, hosted usage, commercial deployment, resale, sublicensing, or distribution of derivative works require a separate commercial license from Perform Digital Pvt Ltd.

Contact: contact@perform.digital
-->

# AGENTS.md

> *Instructions for the agent walking into this web.*
>
> This file is the operating manual for any AI coding agent that touches the spiderbrain v3 codebase - whether you're **working on the brain** (contributing) or **wiring the brain into another project** (consuming). Read both sections. The cardinal rules apply to both.

---

## 0. First 60 seconds

You are reading this because you've been pointed at the spiderbrain v3 repository. Before you do anything:

1. **Read [`llms.txt`](llms.txt)** - the ~70-line skill summary. Tells you what this is.
2. **Read [`README.md`](README.md)** - the architectural overview with the Mermaid diagram.
3. **Read [`core/SKILL.md`](core/SKILL.md)** - the skill spec, BUILD / MAINTAIN / QUERY / CASCADE modes.
4. **Identify which job you have:**
   - *"Help me improve / fix / extend the spiderbrain code itself"* → §A
   - *"Help me use spiderbrain on a project (mine or someone else's)"* → §B

If you skip step 4 you will make wrong assumptions. The same vocabulary means different things in the two contexts.

---

## 1. Cardinal rules (read these even if you read nothing else)

These rules are non-negotiable. Violating any of them produces silent corruption, the worst class of bug for a memory system.

### 1.1 Derived data is **never hand-edited**
- `synganglion.json`, `spideyorder.md`, `spideymove.md`, per-cluster `webmap.md` - all generated.
- If you find yourself wanting to edit one of these, you actually want to edit the **source** it was generated from (a config file, an override file, a curated `changelog.md`, or a source file in the project being scanned), then re-run the generator.

### 1.2 Curated data is **never overwritten by the generator**
- `SPIDERBRAIN.md`, `spiderbrain.config.json`, `webscore-overrides.json`, `movemap.md`, per-cluster `changelog.md` / `rules.md` / `config.md` - all curated.
- The generators must **read** these and **fold them in**, never destroy them. If you're writing a generator, treat them as read-only inputs.

### 1.3 Every hook is **dumb, fast, and exit-0**
- Three hooks ship: `session-brief.mjs` (SessionStart, once per session), `prompt-brief.mjs` (UserPromptSubmit, per prompt, silent when nothing matches), `journal.mjs` (PostToolUse, per edit, append-only).
- Every hook swallows every error and exits 0 on every path. **Do not add validation, ordering, locks, or back-pressure** to any hook. A broken brain must never block a session, a prompt, or an edit.
- The journal is write-side. `prompt-brief` is read-side. `session-brief` is read-side. None of them mutate the graph - only `consolidate.mjs` does.

### 1.4 Consolidation is **single-writer**
- Only `core/scripts/consolidate.mjs` mutates the graph from journal input. It runs on deploy or by hand, never from a hook, never concurrently with itself.
- If you need a second writer, you're solving the wrong problem - fix the consolidator instead.

### 1.5 Build before write - for masters
- Editing a **master** (high `mass`, high `rhythm`, `isMaster: true`) without a `cascade.mjs` pass first is the highest-blast-radius mistake possible. Run cascade. Read the output. Then edit.
- Cascades hitting a master **hard-stop**. That's a feature. Don't paper over it.

### 1.6 The prey is sacred
- The `prey` field in `spiderbrain.config.json` is the project's stated purpose. Every `webscore` is judged against it.
- If the prey changes, **every score is suspect** until re-judged. Don't quietly edit prey without flagging it.

### 1.7 Honesty over hype
- This repo's claims (cost reduction, hallucination defence, etc.) are tied to receipts in `docs/`. If you add a claim, add the receipt. If you can't substantiate, mark it **estimated** or remove it.
- See `docs/benchmarks.md` §0 honesty contract for the discipline.

---

## §A. You are working on the spiderbrain codebase

### A.1 What you're looking at

```
spiderbrain v3/
  core/                  ← the brain itself (BUSL-1.1)
    SKILL.md             ← skill spec (entry point)
    README.md            ← 16-benefit catalogue
    reference/           ← architecture, neuroscience, upkeep, webscore rubric
    scripts/             ← build-brain, consolidate, cascade, query, molt + lib/
    concepts.md          ← 11 design pillars, honestly tagged (shipped/partial/v4/commercial)
  platforms/             ← adapters (Apache 2.0)
    claude/              ← the only shipped adapter (hooks + README)
                            OpenAI / Gemini / Cursor / Mistral / DeepSeek / Grok
                            are open challenges - see CHALLENGES.md
  docs/                  ← cost-reduction-analysis, benchmarks
  enterprise/, benchmarks/, licensing/  ← descriptive READMEs
  CHALLENGES.md          ← contributor challenges (adapters, benchmarks, screenshots)
  LICENSE.md, COMMERCIAL.md, BRANDING.md  ← binding terms (authored separately)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaroirCommunity/Spiderbrain-V3](https://github.com/SaroirCommunity/Spiderbrain-V3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
