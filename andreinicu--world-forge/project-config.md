---
trigger: always_on
description: > Read this first. It applies to Kilo Code, Cline, and any other agentic IDE
---

# AGENTS.md — standing instructions for agentic tools

> Read this first. It applies to Kilo Code, Cline, and any other agentic IDE
> tool opening this workspace. It is deliberately short — it routes you to the right
> document for your session type instead of duplicating them.

---

## First: which kind of session is this?

### A. Running the pipeline (most sessions)

The user typed `/worldforge brainstorm`, `/worldforge start`, `/worldforge revise`,
`/worldforge resync-preset`, `/worldforge convert`, or a `resume`/`skip`/`status` variant.

- **Open `workflows/world-forge.md` and follow it.** It is the **router** and the
  source of truth for what runs when: it dispatches build runs to four stage files
  (`world-forge-discovery.md` → `world-forge-drafting.md` → `world-forge-validation.md`
  → `world-forge-construction.md`). Load the router plus the *active* stage file only —
  never all of them. Revise runs live in `workflows/world-forge-revise.md`,
  convert runs in `workflows/world-forge-convert.md`, and preset resync / audition /
  post-launch routing in `workflows/world-forge-postlaunch.md`.
- **Kilo Code Subagent Delegation:** If you are the top-level Kilo Code agent, do not run the pipeline commands inline. Instead, use your subagent delegation tool (e.g., the `task` tool) to route the command to the correct custom subagent:
  - `/worldforge brainstorm` or `/world-forge brainstorm` → Dispatch to `WorldForge-Brainstormer` (optional ideation upstream of Phase 0; writes informal `Brainstorm_Notes.md`, no World Seed)
  - `/worldforge start` or `/world-forge start` → Dispatch to `WorldForge-Interviewer`
  - `/worldforge revise` or `/world-forge revise` (with `--freeform`, `--target`, etc.) → Dispatch to `WorldForge-Reviser`
  - `/worldforge convert` or `/world-forge convert` → Dispatch to `WorldForge-Converter`
  - `/worldforge resume phase[N]` → Dispatch to the custom agent defined for Phase N (e.g., `WorldForge-Editor` for Phase 3)
- **If a dispatch or model call fails, never do the phase's work inline.** Retry the
  dispatch; if it stays unavailable, set the Pipeline State Ledger `status` to `BLOCKED`,
  halt, and surface it to the user (router → DISPATCH PROTOCOL rule 1). An inline-produced
  phase has no agent spec, no Context Manifest, and no valid sign-off — treat its output
  as if the phase never ran.
- **Pipeline files are READ-ONLY at runtime:** everything under `agent_roles/`,
  `templates/`, `workflows/`, plus `Notes_On_functionality.md` and
  `Notes_Quick_Reference.md`. You write only to the world project's `Drafts/`, `Export/`,
  `World_Seed.md`, and report files. If you find yourself about to edit a pipeline file
  mid-run, stop and surface it to the user — an agent went off-script.
- **Load only what each phase needs.** Every agent spec begins with a
  `📂 CONTEXT MANIFEST` listing exactly what to load. Honor it. In particular, never load
  `Samples/`, `wiki/`, `CLAUDE.md`, `CHANGELOG.md`, or `tutorial.md` during a run — they
  are human-facing or maintenance material and only burn context.
- **For SillyTavern runtime questions** (position values, lorebook flags, token budget,
  prompt assembly order): consult `Notes_Quick_Reference.md` first. Open the full
  `Notes_On_functionality.md` only when the quick reference does not settle the question
  or the agent spec explicitly requires a section of it.

### B. Editing the pipeline itself (maintenance)

The user asked you to change agent specs, templates, workflows, or documentation.

- **Read `CLAUDE.md` completely before any edit.** It holds the load-bearing
  architectural principles, the cross-file consistency table (most pipeline files have
  paired files that must change together), and the editing protocol.
- Never edit pipeline files while a `/worldforge` run is in progress — the runtime agent
  is reading them.

---

## Hard invariants (one line each — full rationale in `CLAUDE.md`)

1. **Three-tier lorebook architecture.** Tier 1 world (position 0), Tier 2 character
   (position 1), Tier 3 arc/sandbox (modular). Every piece of content belongs to exactly
   one tier.
2. **The `{{original}}` mandate.** Every card's `system_prompt` and
   `post_history_instructions` begins with `{{original}}` on its own line. Engine rules
   live in the preset; character specifics live in cards/lorebooks. Never mix the two.
3. **Audit/apply separation.** Editor, Voice/Arc/Intimacy Auditors, and Prompt Engineer
   are read-only on the files they audit. Only the Architects write drafts; only the
   Compiler writes Export/ JSON (narrow preset-only exceptions documented in CLAUDE.md #3).
4. **Position values come from the table**, not from memory — `Notes_Quick_Reference.md`
   has the enum (0–7); `Notes_On_functionality.md` is the authority behind it.
5. **Every lorebook entry carries a `Position Rationale:`** — `DEFAULT` or a one-sentence
   justification.
6. **ARC_STATE / SANDBOX_STATE entries use the two-subsection structure** — descriptive
   situation + binding Tonal Mandate with imperative directives.
7. **Export JSON is written as UTF-8, never through PowerShell.** After Phase 4, the
   read-only check `python tools/validate_export.py <Export dir>` verifies parse validity,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreiNicu/World-Forge](https://github.com/AndreiNicu/World-Forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
