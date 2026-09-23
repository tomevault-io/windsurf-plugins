---
trigger: always_on
description: This project uses **Mano** for planning. Mano is a structured thinking tool: almost every command produces planning artifacts rather than code. The two exceptions are deliberate — `mano dev` and `mano build` implement what the planning already decided.
---

<!-- MANO:BEGIN -->
# AGENTS.md

This project uses **Mano** for planning. Mano is a structured thinking tool: almost every command produces planning artifacts rather than code. The two exceptions are deliberate — `mano dev` and `mano build` implement what the planning already decided.

## For coding agents

### Running Mano commands

Mano commands are repo-local workflow instructions, not installed OpenCode skills.

If the user types a Mano command in chat, do **not** try to load an external skill named `mano`.

Instead, execute the corresponding Mano planning flow by reading the local files in this repository:

- `_mano/skills/[command].md` — the skill file for that command; its front-matter names the `_mano/rules/` files it requires
- any referenced templates or current `_mano_output/` artifacts
- `_mano/workflow.md` — only for the bare `mano`, `mano help`, `mano status`, and `mano continue` commands; skills name their own rule files and never need the whole workflow

Examples:

- `mano import` → read `_mano/skills/import.md` and follow that flow (PRD/document → backlog)
- `mano start` → read `_mano/skills/start.md` and follow that flow
- `mano spec` → read `_mano/skills/spec.md` and follow that flow
- `mano rules` → read `_mano/skills/rules.md` and follow that flow
- `mano stories` → read `_mano/skills/stories.md` and follow that flow
- `mano review` → read `_mano/skills/review.md` and follow that flow
- `mano dev` → implement the next pending story; read `_mano/skills/dev.md` plus `_mano/rules/implement.md` and follow the complete contract in those files
- `mano build` → build the active phase straight from its brief, with no story files; read `_mano/skills/build.md` plus `_mano/rules/implement.md` and follow the complete contract in those files
- `mano continue` → read `_mano/workflow.md`; it runs the implementation action the projection's `IMPLEMENTATION_ENTRY:` names (`mano build`, or `mano dev` for one story), or one unambiguous planning action — it never prints a card telling the human to type the command they just typed
- `mano mode [auto|manual]` → read `_mano/skills/mode.md`; show or set whether finished actions chain automatically
- `mano track [name]` → read `_mano/skills/track.md`; show, set, or clear the optional local experiment/work track

Note: `mano dev` and `mano build` are the two Mano commands that produce code. Every other command above is planning only. Their contracts live in `_mano/skills/dev.md` and `_mano/skills/build.md`, both of which require the shared `_mano/rules/implement.md`. A phase uses one of the two, never both: `mano dev` implements stories from `stories/README.md`, `mano build` works the Scope rows of `PHASE_DIR/progress.md`.

**Run mode.** Every `state.js` projection prints `MODE: manual|auto`. In `manual` (the default) each command hands back when it finishes. In `auto`, after the human has approved a phase scope, each finished action runs the next one automatically through to `mano build` (or `mano dev yolo` when the phase already has a stories index) — but it pauses for **any** question (a `❓ Decide:`, a clarifying question, an ambiguous next action, hook findings, a gate or blocker) and **never runs `mano review` or scopes a new phase**. Auto mode changes who types the next command; it never changes what a skill may write or which decisions are the human's. The narrative contract is `_mano/workflow.md` → **Run Mode: manual and auto**; the mid-chain execution rules are `_mano/rules/auto.md`, loaded only when the projection reports `MODE: auto`.

**Continuing the chain means invoking the next action in the same turn — never announcing it.** Ending a turn on "Continuing — running `mano ui` next" stops the chain while claiming to continue it. Mid-chain, omit the `Next:` block (nobody is typing a command); it returns only in the closing block, on the action that actually ends the chain. Every hand-back names its pause condition; a chain that stops without naming one is a bug.

**`mano build` runs to completion, in both modes.** It is `mano dev yolo` with no opt-in word: one invocation goes from wherever the ledger stands to the phase's terminal line. Creating the ledger, passing pre-flight, and closing a pass are things that happen inside that run — never things the run is finished having done. A build that reports its new ledger and hands back has written a file and built nothing.

If a platform skill named `mano` is not available, that is not an error. Continue by using the local `_mano/` files.

**Write for humans.** Every Mano skill follows `_mano/rules/artifact.md` → **Plain-language contract**. Apply it to chat and artifact prose. Assume a teammate has no prior context.

The skill name uses a **hyphen, never a colon**: `mano import` → `mano-import` (read `_mano/skills/import.md`), not `mano:import`. The colon form is plugin-namespace syntax and matches no Mano skill. If a `mano <action>` seems unavailable, try the hyphenated `mano-<action>` before concluding it doesn't exist.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FormidableLabs/react-native-ama](https://github.com/FormidableLabs/react-native-ama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
