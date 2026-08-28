---
trigger: always_on
description: *By MC Dean · Requires Figma MCP server tools (`get_design_context`, `use_figma`, `search_design_system`)*
---

# Agent Ready — for Gemini CLI

*By MC Dean · Requires Figma MCP server tools (`get_design_context`, `use_figma`, `search_design_system`)*

> This is the Gemini CLI adaptation of Agent Ready. The canonical version
> lives in `SKILL.md` (Claude Code format). If you edit this file, please
> also update `SKILL.md` and the other agent adapter files to keep them
> in sync.

Assess a Figma file's readiness for agent interaction and compensate
for gaps before generating code or modifying designs. Applying these
instructions improves the quality of everything you produce from
Figma context.

## Installation (for end users)

Gemini CLI auto-loads `GEMINI.md` files as persistent instructions.
There are two places you can put this file:

- **Per project:** save this file as `GEMINI.md` at the root of the
  project you're working in. Gemini CLI will load it whenever you
  run `gemini` inside that folder.
- **Globally:** save this file as `~/.gemini/GEMINI.md`. Gemini CLI
  will load it in every project. Use this if you work across many
  Figma-connected codebases.

No command needs to be run. Once the file is in place, Gemini CLI
reads it automatically on startup.

## When to apply these instructions

Unlike Claude Code skills, Gemini CLI does not auto-trigger instructions
based on a description field. This file is always loaded when you're
in the project. You should apply the checks below whenever any of the
following is true:

- The user asks you to read or work with a Figma file
- The user asks you to generate code from a Figma design or component
- The user asks you to create or modify a component in a Figma file
- The user complains that code generated from a Figma file feels
  generic, inaccurate, or poorly named
- The user asks "how agent-ready is this file?" or any variant

If none of the above applies (e.g. you're working on a pure backend
task with no Figma involvement), ignore these instructions and proceed
normally.

## Prerequisites

This workflow requires a **Figma MCP server** to be connected, providing
these tools: `get_design_context`, `use_figma`, `search_design_system`.

If the Figma MCP server is not connected, tell the user: "I need the
Figma MCP server to read your file directly. You can set it up at
https://github.com/nichochar/open-figma-mcp — or paste your Figma
file URL and I'll explain what to check manually."

## Why this matters

Figma files are communication artefacts. They were designed for human
readers. When you read them through MCP, you encounter unnamed layers,
empty descriptions, hard-coded values, and missing states. Each gap
forces you to guess, and guesses compound into poor output. These
instructions teach you to identify those gaps and compensate for them.

## Instructions

### Step 1: Read the file context

1. Use `get_design_context` to pull the frame or component the user pointed you to.
2. Note the layer tree structure, component names, property definitions, styles, and variables.

### Step 2: Run the readiness assessment

Evaluate the file against these 13 checks, in order of impact on your output quality.

#### Critical impact (address these first — they change your output most)

1. **Description coverage.** Check every component and component set for a `description` field. If empty, you must infer purpose from the component name, its variants, and its visual context. State your inference explicitly in code comments so a human can verify.

2. **Layer naming.** Scan for default Figma names: Frame, Group, Rectangle, Ellipse, Line, Vector, Polygon, Star, Boolean, Slice, Image, Text followed by a number. For any you find, infer a meaningful name from context before using it in code. Prefer semantic names: `hero-heading-group` not `Frame 247`.

3. **Component properties.** Check component sets for `componentPropertyDefinitions`. If a component has variants but no boolean, text, or instance swap properties, treat the variant names as your property source. Parse the variant string (e.g. "State=Hover, Size=MD") to extract structured props for your code.

4. **Code Connect.** Check whether components have Code Connect mappings that bridge Figma props to code props. If Code Connect is present, always use its mappings — they are the authoritative translation between design and code. If absent, you must infer the mapping yourself: "Type=Primary" likely maps to `variant="primary"`, "State=Hover" is likely a CSS pseudo-class not a prop. State your inferences in comments so a human can verify.

#### High impact (significantly improves your comprehension)

5. **Auto-layout.** Check frames for `layoutMode`. If a frame has multiple children but no auto-layout, infer layout intent from child positions: are they stacked vertically, arranged horizontally, or in a grid? Use this inference for your flexbox/grid decisions rather than absolute positioning.

6. **Token usage.** Check whether fills, strokes, and text styles reference Figma styles or variables. If they do, use the style/variable name in your code (e.g. `var(--brand-primary)`). If they use raw hex values, check `search_design_system` for a matching style before falling back to the raw value.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Owl-Listener/agent-ready](https://github.com/Owl-Listener/agent-ready) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
