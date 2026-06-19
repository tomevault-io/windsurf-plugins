---
trigger: always_on
description: >-
---


# spec-debate — debate a spec with a second model, with veto

You orchestrate a debate between yourself (Claude) and OpenAI Codex to make a **spec** measurably
better. You are the **editor with veto power**: never apply a point blindly — verify each against the
actual spec and keep only what genuinely improves it. The vetting is the whole point; a critic that's
always obeyed is just a second author.

**The iterated artifact is always a spec** (requirements / design / plan / PRD — any domain). There are
three ways in, all converging on iterating one spec:
- **TASK** → you draft a solution spec.
- **CODE** → you draft a *change-spec*; the code is reference material, **not edited during the
  debate** (it is applied afterwards, as a separate step, if you have access).
- **EXISTING SPEC** → you take it as the artifact.

Each round, **both models independently propose improvements**; you **merge with veto** and apply the
result to the spec. **Goal: a better spec, not a bigger one** — close real gaps, cover core scenarios,
resolve contradictions, fix real risks, remove ambiguity; accept added complexity only when a real
gap/risk/UX need justifies it. One invocation = one round; state persists in
`.<filename>.debate-state.json` beside the spec so rounds don't relitigate settled points.

---

## Step 0 — Preconditions
1. `command -v codex >/dev/null 2>&1 && echo FOUND || echo MISSING`. If MISSING, stop: "Codex CLI not
   found. Install: `npm install -g @openai/codex`, then re-run." Then confirm auth: `codex login status`;
   if not logged in, stop and tell the user to run `codex login`. Don't substitute another tool — the
   debate needs an *independent* second model.
2. Parse reasoning effort (`--high|--medium|--low|--xhigh`, `effort=…`, or an unambiguous "maximum
   reasoning depth" → `xhigh`). Default `high`; `xhigh` is much slower, only on explicit request.
3. Parse a round directive (a count like "run 3 rounds", or "until no significant findings remain");
   default one round. Parse a `thorough` request (cross-critique, Step 3c).

## Step 1 — Pick the mode, then resolve the working spec
First a **surface scope scan** — structure, size, number of files/components, whether the design is
non-trivial. This is a *shallow look, not deep reading*: deep study **of the referenced material**
happens inside the chosen mode (Step 3b), so you never pay for it twice. Then choose:
- **prompt-only** — a bounded advisory question whose output is advice/a comparison the user applies
  directly, with nothing worth iterating and the scan showing it closes in one pass.
- **iterable spec (the main flow)** — breadth or complexity (several files/components or substantial
  material; a design with several coupled decisions), or the user wants iteration / a written spec.

State the chosen mode in one line. If a prompt-only consult turns out under-scoped mid-pass, finish
that pass, then offer to escalate to an iterable-spec debate (don't abandon it half-done).

**Prompt-only path (compact):** one Codex pass on the question — a free-form prompt (role line + the
question + relevant conversation context: user statements verbatim, your summaries marked as yours),
`<workdir>` = current dir. Vet the answer with the Step 4 lens. Report inline: Codex's position, your
vetted take, the prompt-file path, and a one-line "sent to Codex" note (subject; refs/snippets;
anything privacy-mode withheld). No state, no rounds. For any follow-up round or edit, materialize the
subject into a spec file and seed `.<filename>.debate-state.json` as round 1 (each settled conclusion becomes a
finding). Then continue below.

**Iterable spec — seed the artifact:**
- Use the explicit path if given; else the spec you drafted/took earlier in *this* conversation; else
  ask — don't guess across the filesystem. If the spec lives only in the conversation, write it to a
  markdown file first (the debate needs a file to edit and to hold `.<filename>.debate-state.json`) —
  a descriptive `<topic>-spec.md` beside the related material or in the working dir; state the path.
- **TASK** → draft a solution spec. **CODE** → draft a change-spec. **EXISTING SPEC** → take the file.
- **Minimal spec shape (quality gate)** — so the debate doesn't converge on something under-specified.
  Every spec should conceptually carry: goal · non-goals · assumptions · constraints · acceptance
  criteria · open questions · what material was studied and what was deliberately skipped (if any). A
  **code change-spec** adds: changes by file/component · behavior preserved vs changed · risks/migrations
  · acceptance checks/tests. Tiny tasks may compress this, but the fields are conceptually present. If
  **material facts** are missing and would shape the spec, **ask the user before debating** — don't debate
  a fabricated spec. You ensure this shape when *you* author (TASK/CODE). For a **given** spec it is NOT
  a precondition — missing fields become debate findings, and you don't rewrite the input before
  discussing it; but if the given input is essentially empty (a stub, not a real spec), treat it as a
  TASK and draft.
- **Name the spec type and altitude** — it governs every later judgment — and read the spec artifact in
  full (distinct from deep-studying the referenced material, which is paced per Step 3b):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OlegTestov/spec-debate](https://github.com/OlegTestov/spec-debate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
