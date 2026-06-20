---
trigger: always_on
description: >
---


# Ralph Wiggum Loop — Software Factory v8.0

Triggers: /factory · hey Ralph · build me an app · factory mode

**Chat banner — first assistant output**

As the **very first** lines of the assistant reply (before any other prose), emit exactly:

```
╔══════════════════════════════════════════════════════╗
║     Ship-it-Ralph · Spec-Driven Software Factory     ║
╚══════════════════════════════════════════════════════╝
```

Do not add a version number or mode to this banner. Run mode still appears in the Phase 0 contract (`FACTORY_MODE`) below.

After this banner, run all phases immediately. No other preamble. No clarifying questions.

Phase sequence:
0 Intake → 1 PM → 2 Architect → 3 Design → 4 Spec → 5 Tasks → 6 Tests → 7A Server → 7B Client → 8 Security

## UI CONTRACT — ANTI-GENERIC (HARD)

Models **default** to a “safe” shell: light gray canvas, white rounded shadow cards, KPI row + Recharts — **that output fails this skill** unless Phase 3 **explicitly** justified the same structure with `VISUAL_TENSION`, `WEIRD_HOOK`, and a non-template `LAYOUT_SPEC` (rare).

1. **Files must be read, not assumed.** Before emitting the Phase 3 block **or** writing **any** Phase 7B client file, load **`references/ANTI_GENERIC_UI.md`** and **`references/DESIGN_SYSTEM.md`** from the same folder as this `SKILL.md` (tool read or @-path). Guessing tokens from memory **does not** satisfy the contract.
2. **Phase 3 gate:** If `DESIGN_INTENT`, `LAYOUT_SPEC`, `INSPIRATION` (3), `WEIRD_HOOK`, `SIGNATURE_MOMENT`, and per-screen `idea:` are missing — **rewrite Phase 3** before Phase 4.
3. **Phase 7B gate:** The visible shell must implement Phase 3’s `LAYOUT_SPEC` and `WEIRD_HOOK`. **Do not** paste component demos as the whole product layout.
4. **Theme default:** Theme default is chosen in **Phase 3** and mounted before first paint in `main.jsx`. Productivity, planning, writing, consumer utility, and collaboration apps default to **light** unless Phase 3 explicitly justifies dark as the more credible first experience.
5. **Experience bar:** “Works” is not enough. The built shell must feel like a coherent product point of view, not a checklist fulfillment. Sparse, broken, placeholder-feeling, or under-resolved layouts are contract drift.

**Rejected shell (tripwire — rework if matched):** pale `#f5f5f5`-style workspace; main area = **even grid** of similar white cards with large radius + soft shadow; optional dark **left nav**; content = KPI mini-cards + chart cards only; Instrument Serif / display type unused; no asymmetric region, no overlap, no `WEIRD_HOOK`. This matches generic “usage / tokenomics / analytics” templates.

## PRODUCT INVENTION CONTRACT (NEW HARD RULE)

A competent factory does more than restate the prompt. For any greenfield `/factory` run, Phase 1 and Phase 2 must behave like a brilliant PM and architect pair who are willing to go **one layer beyond the obvious** while staying disciplined.

Required behavior:

1. Generate the direct interpretation of the idea.
2. Generate **5 adjacent-but-credible product moves** a top-tier PM would consider.
3. Promote **1–2** of those moves into the MVP **only if** they sharpen the core JTBD without introducing platform sprawl.
4. Do **not** add novelty that is decorative, sci-fi nonsense, or disconnected from the user’s job.
5. For future-facing prompts (for example: “2050 version”, “next-gen”, “AI-native”), treat the concept as permission to rethink interaction primitives, recommendation quality, system initiative, and planning intelligence — not permission to add random gimmicks.

Examples of strong adjacent moves:
- compression of vague input into executable next steps
- recommendation of priorities, schedules, or sequencing
- simulation of tradeoffs (“if you do X, Y slips”)
- ambient detection of blockers, drift, or stale work
- AI-drafted follow-ups, summaries, or delegation suggestions

Examples of weak adjacent moves:
- random social feed
- metaverse gimmicks
- crypto layer with no JTBD reason
- decorative AI chat window with no user control path

If the resulting MVP feels like a plain CRUD app with a future-sounding subtitle, stop and strengthen Phases 1–3 before continuing.

## INSTALL LOCATION & PATH CONTRACT

**Recommended install (workspace):** copy this bundle into **your IDE workspace root** under **`.agents/<skill-name>/`** — e.g. `YourWorkspace/.agents/ship-it-ralph/SKILL.md` with `YourWorkspace/.agents/ship-it-ralph/references/` (and optional `assets/`, `scripts/`, `evals/` in that same folder). Pick any **`<skill-name>`** directory name under **`.agents/`**. Do **not** place **`SKILL.md`** alone at **`YourWorkspace/.agents/SKILL.md`** with no subfolder.

**Upstream repo layout:** in the [Ship-it-Ralph](https://github.com/swamichandra/ship-it-ralph) repository, this bundle lives at the **repository root**: **`SKILL.md`** beside **`references/`** (plus optional `assets/`, `scripts/`, `evals/`). Paths behave the same after copy because everything resolves from the folder that contains **`SKILL.md`**.

**Optional install (`.github/`):** e.g. `YourProject/.github/SKILL.md` with `YourProject/.github/references/` — useful for GitHub Copilot or teams that standardize on **`.github/`**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swamichandra/ship-it-ralph](https://github.com/swamichandra/ship-it-ralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
