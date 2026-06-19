---
trigger: always_on
description: Identifies a repo's scopes (frontend, backend, jobs, data, integrations, feature slices, …), persists them as a durable catalog, then renders `.drawio` diagrams per scope — Claude picks the best diagram type for each scope from a menu of eight flowchart kinds (Process Flowchart / Map, Swimlane, Data Flow Diagram, Workflow Diagram, System Flowchart, Document Flowchart, EPC, Influence Diagram). Every rendered diagram ships with paired Technical / Plain-English page tabs so the reader picks their a
---


# Daemonstrate

Identifies the scopes of the **target repo**, persists that catalog so it survives between runs, and renders one or more light-theme `.drawio` diagrams. For each scope, Claude picks the diagram type that best fits what the scope *is* (a stepwise process? a handoff between actors? a data flow? a decision?) from a menu of eight well-established flowchart kinds.

**Every rendered diagram ships with paired Technical + Plain-English page tabs** inside the same `.drawio` file. A reader clicks a bottom-tab to switch views — the Technical page uses stack-native vocabulary (HTTP verbs, JWT, file paths, library names); the Plain-English page translates to user-facing language ("app talks to server", "saves the note", "checks your login"), hides tech badges and file-path subtitles, and is optimized for anyone unfamiliar with the codebase. Translation runs once per label and caches to the catalog. Readers choose the audience at view time, not at generation time.

**Interactive teaching mode** activates when `/daemonstrate` is invoked from chat with an existing catalog. The user picks one area from a numbered list, then (recursively) drills into any component as deeply as the source structure supports — file → functions → branches → library calls — and Claude produces a focused single-topic diagram for each level. Ideal for explaining one specific piece of the system (e.g., "how does auth middleware actually work?") to a non-technical audience.

All outputs share one visual language — dashed pastel group containers, orthogonal routed edges, dark text on white, drop shadows — so a reader can move between per-scope diagrams without re-orienting. By default they live under `docs/` in the target repo.

## Path resolution

Resolve these paths once at the start of every run:

- **`SKILL_DIR`** — the directory containing this `SKILL.md`. You already know it because you just loaded this file. All bundled scripts (`drawio_builder.py`, `install-hooks.sh`) live at `$SKILL_DIR/scripts/`. **Never hardcode an absolute path** — this skill must work on any machine.
- **`TARGET_REPO`** — the repo being diagrammed. Default: the current working directory's git root (`git rev-parse --show-toplevel`). Override: if the caller explicitly gave you a different repo path, use that.
- **`OUT_DIR`** — where diagrams and the scopes catalog are written. Default: `$TARGET_REPO/docs/`. Override: if the caller explicitly gave you a different output directory (e.g., during an evaluation run), use that instead — and in that case *do not* also write into `$TARGET_REPO/docs/`.

Before invoking the builder or installer, confirm the resolved paths — mention them in your first user-facing status message, **or** echo them into the run's report file if the context is non-interactive (hook, eval, script). Either way, make the paths visible so a misrouted run is caught early.

**All working artifacts respect `$OUT_DIR`** — not just the diagrams and the scopes file, but the intermediate `graph-spec.json`, any report/log, any scratch files. Never drop working files into `$TARGET_REPO` even if OUT_DIR differs from the default.

## Interactive vs. non-interactive runs

Three invocation patterns — two interactive, one non-interactive. The catalog's presence decides which interactive flow applies.

**Interactive — catalog exists (teaching mode).** User invokes `/daemonstrate` from chat on a repo with `.daemonstrate-scopes.json` already in `$OUT_DIR`. Drop into **Phase 0.5** — area selection → optional recursive drill-down → render one focused diagram for the selection. This is the default for "explain how X works" requests.

**Interactive — no catalog (first run).** User invokes `/daemonstrate` on a repo that's never been mapped. Run full Phase 0 discovery, then fall through to whole-repo generation (portfolio + hybrid-detailed, paired Technical/Plain-English pages). Offer the teaching flow as a follow-up: "Want to drill into a specific area now that everything's mapped?"

**Non-interactive** — triggered by the post-commit hook, a CI job, or an eval harness. No prompts possible. Always run whole-repo generation; never enter Phase 0.5. Apply these defaults:
  - Diagram-type picks: commit Claude's judgment without confirmation.
  - Page structure: **hybrid**.
  - Audience pages: always emit **both** Technical and Plain-English tabs.
  - Hook install: **skip** (assume the caller controls hooks).
  - Hand-edited diagram detected: save proposed output as `architecture-*.proposed.drawio` alongside the original, do **not** overwrite. Log the conflict.
  - All status messages go to `$OUT_DIR/daemonstrate-run.log` instead of chat.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyofalcon/daemonstrate](https://github.com/wyofalcon/daemonstrate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
