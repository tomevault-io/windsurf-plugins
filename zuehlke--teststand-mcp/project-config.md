---
trigger: always_on
description: To reproduce an existing sequence file (a "rebuild"), prefer the **whole-sequence
---

# TestStandMCP — Behavior Rules for Claude

## Rebuilding a .seq 1:1 (whole-sequence clone) — the FAST, canonical path

To reproduce an existing sequence file (a "rebuild"), prefer the **whole-sequence
clone** over the per-step insert+configure dance. `duplicate_sequence` deep-clones a
whole sequence (steps, modules, locals, parameters, comment, all settings) — within a
file or **cross-file** via `target_file_path`. Recipe:

1. `create_sequence_file` (the new file)
2. `copy_typedefs` (all types) — so cloned sequences/globals resolve their types by GUID
3. `duplicate_sequence` source→target for **each** sequence, in source order, same name
4. `delete_sequence` the default `MainSequence`
5. `copy_file_globals` — file globals belong to no sequence, so the clone misses them
6. `copy_file_attributes` + `set_file_properties` (comment/version)
7. `save_sequence_file`, then **verify with `diff_sequence_files`** (the native FileDiffer)

**Verification semantics** (`diff_sequence_files` / `compare_sequence_files mode=native`
are the SAME diff — use `diff_sequence_files`):
- In the diff values, `{val}` = TYPE-DEFAULT (not explicitly set), `[val]` = EXPLICITLY
  set. An enum/member already at its type default must be **left unset** — setting it
  flips `{val}`→`[val]` and creates a spurious diff.
- A lone **`File Properties > Attributes`** delete (e.g. `NI.Analyzer.IgnoredMessages`)
  is IRREDUCIBLE: the engine API never loads those file attributes into memory (only
  FileDiffer's raw reader sees them), so no tool can read or reproduce them. Treat an
  Attributes-only diff as a functional 100% match (`identical=false` is expected).

See memory `teststand-whole-sequence-clone-rebuild-2026-07-08`.

## CRITICAL: How to build sequences from a flowchart/description

The `teststand-sequence-builder` workflow is **interactive** — it asks the user
per step whether to link a `SequenceCall` (target file + subsequence) or insert a
plain placeholder. That per-step question uses `AskUserQuestion`.

**`AskUserQuestion` is NOT available to spawned subagents** (it depends on the
main-conversation UI). Therefore:

- **NEVER** delegate sequence-building to the `teststand-sequence-builder` via the
  Agent/Task tool. If spawned as a subagent, every linking question fails silently
  and all steps degrade to Statement placeholders — exactly the failure to avoid.
- **ALWAYS run the builder workflow in the MAIN conversation thread.** When the
  user asks to "build a sequence from a flowchart" (or "use the Seq agent"), open
  `.claude/agents/teststand-sequence-builder.md`, read its workflow, and execute
  those steps yourself in the main thread — so the per-step `AskUserQuestion`
  linking prompts actually reach the user.

## Documenting sequence files (teststand-doc-generator)

The `teststand-doc-generator` agent turns a `.seq` file into a modern Word
documentation: title + short file description, real Word TOC, one section per
sequence (description, parameter table with By Value / By Reference, compact
flow-indented step listing with the Setup/Main/Cleanup groups preserved and
each step's original TestStand icon tinted monochrome in the document accent
color), and a rendered diagram of the call dependencies between the sequences.

- **Unlike the builder, this agent MAY (and should) be spawned as a subagent**
  via the Agent tool — it is non-interactive and strictly READ-ONLY toward
  TestStand.
- **Ask the document language FIRST — in the MAIN thread.** Before spawning
  the agent, ask the user via `AskUserQuestion` which language the
  documentation should be written in (offer at least Deutsch / English, with
  the language of the user's request as the recommended first option; "Other"
  free text covers further languages). The subagent cannot ask —
  `AskUserQuestion` does not work in subagents. Skip the question only when
  the user already stated the language in their request.
- Pass in the task prompt: the `.seq` path (required), the document language
  (from the question above; `de`/`en` built in, other languages via the
  script's `labels` override), optionally output `.docx` path and custom title.
- The heavy lifting is deterministic: `scripts/generate_teststand_doc.py`
  (data JSON → dependency diagram via headless Edge/Chrome → .docx via
  python-docx; run with the `py` launcher — plain `python` is not on PATH).
  The agent only collects data via the read-only MCP tools and runs the script.

## Presenting sequence files (teststand-presentation-generator)

The `teststand-presentation-generator` agent turns a `.seq` file into a modern,
interactive **HTML presentation** (dark glassmorphism single-page app): a header
with the sequence/source, Setup/Main/Cleanup phase cards, clickable subsequences
that open a detail overlay, and a "Code & Flowchart" compare view. The flowchart
nodes use the **original TestStand step icons in full color** (pulled from the
local install) and reconstruct loops (While/For/…) and branches (If/Select/Case)
as nested blocks. Output is ONE **self-contained `.html`** (icons embedded as
base64) — shareable as a single file. It is the visual counterpart to the
`teststand-doc-generator` (Word). Modelled on `.Demo_jcm/TSmcp_demo/index.html`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zuehlke/teststand-mcp](https://github.com/Zuehlke/teststand-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
