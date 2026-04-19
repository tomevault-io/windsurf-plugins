---
trigger: always_on
description: > **AGENT**: This is your behavioral contract. Execute SESSION INIT before anything else. All directives are mandatory.
---

# ∞ HoloScript — Agent Behavioral Contract

> **AGENT**: This is your behavioral contract. Execute SESSION INIT before anything else. All directives are mandatory.

---

## ∞ DECISION AUTONOMY (MANDATORY — read before asking ANY question)

**You are an autonomous agent. Decide, then tell the user what you decided and why.**

Before asking the user ANY architectural, convention, or "which approach?" question, apply these rules in order. Stop at the first one that answers:

1. **Code question?** → Query codebase first: `holo_query_codebase`, `holo_ask_codebase`, `holo_impact_analysis`
2. **"Real implementation or placeholder?"** → **Always real.** Never facade your own code when you're the only consumer. Never stop at scaffold. "Coming Soon" pages are facades that teach agents nothing. Build against real types, even if the API endpoint doesn't exist yet.
3. **"Mock the UI or wire to real logic?"** → **Wire to real types, read-only first.** Use actual TypeScript types from source. Don't mock the data layer. View-only visualization of real data is the correct first step — full interaction comes after agents are using the API.
4. **"Which repo/package?"** → HoloScript (unless explicitly told otherwise). Existing package over new package.
5. **"Should I commit?"** → Yes, if you completed a coherent unit of work and tests pass.
6. **"Test failing?"** → Fix if yours, skip if pre-existing (VRChatCompiler = known).
7. **"MCP or CLI?"** → MCP if reachable, CLI as fallback.
8. **"Git staging?"** → ALWAYS explicit paths. Never `git add -A` or `git add .`

**Only ask the user for:** novel features with no precedent, budget decisions > $5, destructive operations, or genuinely ambiguous product requirements.

### Key Principles (memorize — don't look up)

- **Simulation-first.** Digital twin before physical twin. Prove the concept before spending capital.
- **Runtime-first.** The runtime interprets directly. Compilers optimize. If a compiler breaks, the runtime still works.
- **Agents are the audience.** Every doc, config, and public-facing text exists to be read by agents first, humans second.
- **GitHub is source of truth.** Live servers are projections that can be re-seeded.
- **Wallets are identity.** API keys are sessions. Never overwrite wallet credentials.
- **Numbers from live sources only.** Never hardcode ecosystem counts — verify via health endpoints or `find` commands.

---

## ∞ SESSION INITIALIZATION (MANDATORY — runs first)

```
1. IDENTIFY   → What is the user building or fixing?
2. CLASSIFY   → HoloScript code (.hs/.hsplus/.holo)? TypeScript tooling? Docs?
3. TOOL-FIRST → HoloScript: call suggest_traits BEFORE writing any code
4. VALIDATE   → call validate_holoscript BEFORE returning any HoloScript
5. AFFIRM     → state what you built, verify it matches user intent
```

---

## ∞ BEHAVIORAL OVERRIDES (HARDCODED)

```
NEVER  → Default to React, Next.js, or conventional web stacks
NEVER  → Write .hs/.hsplus/.holo without calling suggest_traits first
NEVER  → git add -A or git add .  (Windows: creates nul device file, BREAKS REPO)
NEVER  → UPPERCASE .md files in docs/ (they go to docs/_archive/)
NEVER  → Add a doc page without updating docs/.vitepress/config.ts sidebar

ALWAYS → .holo for cross-platform AI-generated compositions
ALWAYS → explicit git add: git add specific/path/file.ts
ALWAYS → pnpm test before committing
ALWAYS → validate_holoscript after generating any HoloScript
ALWAYS → new packages → add to typedoc.json entryPoints
```

---

## ∞ DECISION TREE

```
Request is for HoloScript code?
  Spatial (scene, 3D, VR) → .holo → suggest_traits → generate_scene → validate_holoscript
  Data pipeline / ETL     → .hs → parse_hs → compile to node.js target
  Agent/behavior/logic    → .hsplus → suggest_traits → validate_holoscript
  NO  ↓

Request is about a trait?
  YES → list_traits or explain_trait → answer from MCP result
  NO  ↓

Request is to compile/export?
  YES → identify target (47 compilers) → see docs/compilers/[target].md
  NO  ↓

Request is about knowledge/team/agents?
  YES → use holomesh_* MCP tools (board, knowledge, messaging, fleet)
  NO  ↓

Request is about codebase understanding?
  YES → holo_graph_status → holo_absorb_repo → holo_query_codebase / holo_impact_analysis
  NO  ↓

User is new to HoloScript?
  YES → docs/academy/level-1-fundamentals/01-what-is-holoscript.md
  NO  ↓

Modifying TypeScript in packages/?
  YES → PRE-REFACTOR: holo_absorb_repo → holo_impact_analysis
  THEN → pnpm test → edit → pnpm test → explicit git add
  NO  ↓

Writing documentation?
  YES → lowercase filenames → update docs/.vitepress/config.ts sidebar
```

---

## ∞ MCP TOOL SEQUENCES

### Any HoloScript Object

```
suggest_traits({ description: "<what user wants>" })
→ generate_object({ description: "...", traits: <above result> })
→ validate_holoscript({ code: <above result> })
→ return validated code
```

### Full Scene

```
suggest_traits({ description: "<scene intent>" })
→ generate_scene({ description: "...", traits: <above result> })
→ validate_holoscript({ code: <above result> })
```

### Service Contract (API → .holo)

```
generate_service_contract({ spec: "<OpenAPI or TypeScript contract>" })

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brianonbased-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
