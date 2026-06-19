---
trigger: always_on
description: Generate a self-contained HTML presentation with Magic Move transitions. Use when the user wants to create a slideshow, presentation, or slide deck from a topic or outline, or invokes magic-slide preview to start the preview server.
---


# Magic Slide

Generate polished HTML presentations with smooth Magic Move transitions — elements that appear on multiple slides animate fluidly between their positions. Treat Magic Move as a story-planning constraint, not an afterthought: arrange the outline so meaningful headings, card titles, numbers, images, or diagram nodes can persist across adjacent slides.

## Skill Command Arguments

The user invoked this skill with: `$ARGUMENTS`

## Visible TODO Requirement

Whenever this skill is invoked, create a visible TODO/plan before the first
meaningful action and keep it updated until the turn stops.

- Use the host's planning tool when available: Codex `update_plan`, Claude Code
  `TodoWrite`, or the environment's equivalent visible checklist. This is
  required for every `$magic-slide` run, including preview-only commands,
  intake/question-only turns, new deck generation, follow-up deck edits, and
  `visual-issues.json` repair passes.
- Keep the list short and concrete: 3-7 items that match the current mode.
  Examples: "Resolve deck path", "Start preview server", "Read QA notes",
  "Repair slide 02 source", "Merge and inject runtime", "Mark QA notes pending
  confirmation", "Leave QA Overview open for review".
- Update item statuses as work advances. Do not wait until the end to mark
  everything done; the user should be able to see what is happening while the
  skill is running.
- If a required checkpoint stops the turn, such as requirements intake, web
  search choice, outline confirmation, or user QA confirmation, leave the TODO
  showing the waiting/checkpoint item.
- If no visible TODO tool exists, write a compact `TODO` checklist in the chat
  before proceeding and update it manually in later progress messages.

When invoked as `/magic-slide preview [topic]` in Claude Code or
`$magic-slide preview [topic]` in Codex, run the preview fast path:

1. Create the visible TODO list first, with preview-specific steps such as
   resolving the deck path, starting `serve.py`, and reporting the URL.
2. Treat the argument after `preview` as the topic/deck directory. If omitted,
   use `.`. Preview the `index.html` inside that directory. If the user passes
   an explicit `.html` file, preview that file directly.
3. Locate this skill directory if needed:
   `SKILL_DIR=$(find ~ -type d -name "magic-slide-skill" 2>/dev/null | head -1)`
4. Resolve a Python 3 interpreter using the Script Runtime Requirements below.
5. Start the preview server with the existing script:
   `$PYTHON_BIN "$SKILL_DIR/scripts/serve.py" "$DECK_PATH"`
6. Keep the server process running and give the user the displayed URL.
7. Do not ask deck-generation questions, create an outline, merge slides, or
   inject runtime unless the user explicitly asks for those tasks too.

For any other `magic-slide` invocation arguments, treat the arguments as the
user's presentation request and follow the normal generation workflow.

## Non-Preview Intake Gate

Before running commands, searching, creating folders, writing an outline, or
generating slides for any non-preview deck request, complete Step 1 in
`references/workflows/step-01-requirements.md`.

Hard gate:
- The visible TODO requirement still runs before this gate. If the turn stops
  for intake, leave the TODO showing that requirements are being gathered and
  that generation is waiting on the user's answers.
- If topic/audience-lens, aesthetic style, presentation language, or image
  policy is missing, inferred, or only implied, ask the Step 1 requirements
  question and stop.
- Do not treat the user's chat language as the presentation language.
- Do not treat a URL, company/product name, or "介绍一下 X" as a complete topic
  unless the audience/lens has also been explicitly supplied or confirmed.
- If the structured question tool is unavailable, use the plain-text fallback
  template from Step 1 as the whole response for that turn, then wait.

## CRITICAL: Script Runtime Requirements

Magic Slide's bundled scripts require a Python 3 runtime. Core merge, inject,
repair-note, and preview scripts use the Python standard library plus a modern
browser; they do not require external Python packages. PipeLLM scripts require
`PIPELLM_API_KEY` in the environment only when web search or image generation
is used. Playwright is only required for agent-run screenshot QA, including
`scripts/check-magic-text-wrap.py`.

PipeLLM web search is optional, user-approved, and returns sanitized untrusted
evidence records. PipeLLM image generation is optional and user-approved. The
scripts require `--allow-external` so external requests cannot happen without
an explicit workflow gate; Step 2 owns the detailed prompt-injection boundary.

Before the first script execution in a turn, resolve a Python 3 interpreter:

```bash
PYTHON_BIN=$(command -v python3 || command -v python || true)
if [ -z "$PYTHON_BIN" ]; then
  echo "Magic Slide requires Python 3 to run its bundled scripts." >&2
  echo "Install Python 3, then rerun the Magic Slide command." >&2
  exit 1
fi

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniel-style/magic-slide](https://github.com/daniel-style/magic-slide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
