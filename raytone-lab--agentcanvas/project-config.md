---
trigger: always_on
description: Before generating or changing any UI, read `DESIGN.md` (project root) — the visual + interaction spec (colors, typography, geometry, elevation, component contracts, motion). Use semantic tokens only; no raw hex or raw px motion durations. Component architecture lives in `docs/COMPONENT_SYSTEM.md`.
---

# AGENTS.md

## Design Source of Truth

Before generating or changing any UI, read `DESIGN.md` (project root) — the visual + interaction spec (colors, typography, geometry, elevation, component contracts, motion). Use semantic tokens only; no raw hex or raw px motion durations. Component architecture lives in `docs/COMPONENT_SYSTEM.md`.

## Project Mode

This project is AgentUX Scaffold Configurator: a default Agent frontend template plus focused UX presets on top of AgentUX SDK.

Optimize for the shortest path to a working, maintainable scaffold. Challenge goals or paths that drift toward a generic low-code builder, infinite canvas, heavy component assembly surface, or decorative UI prototype without exportable engineering structure.

## Core Product Rule

Do not build a Figma-style infinite canvas.

The product should feel configurable, but the implementation must be schema-driven and region-constrained so exported code remains clean. Users should start from a strong default Agent frontend, then adjust key UX presets.

The primary user journey is: users assemble and tune the Agent UI/UX in this app, click download/export, receive a runnable code package, and continue second-stage development of their own Agent frontend from that package. AgentCanvas is the scaffold configurator and package generator, not the long-term hosted runtime for the user's Agent product.

Do not treat export as a cosmetic manifest or preview-only summary. The export path must produce real project files that can be installed, run, edited, and extended outside AgentCanvas.

Allowed regions:

- `main`
- `composer`
- `right-panel`
- `bottom-dock`
- `overlay`

Free positioning is not a primary MVP interaction. Optional snap placement is acceptable only for composer, output, Git, and overlay/floating components with clear bounds.

Even without drag/drop as the primary interaction, keep each major UI area independent and copyable. The builder should be able to customize Chat, Composer, Output, Git, Activity UX, Provider/Harness controls, and Debug Dock separately.

Modules should communicate through typed config and AgentUX runtime/render-core view models, not hidden DOM coupling or broad global assumptions.

## MVP Product Shape

Default template: coding agent.

Core blocks:

- large chat frame,
- composer with send, upload, thinking budget, model switcher, tool toggle, optional mic,
- output/artifact frame,
- Git/project frame,
- Agent activity UX for thinking, writing, tool calls, and collapse behavior.

Left rail should be `UX Presets`, not `Component Library`.

Preset groups:

- UX Effects,
- Tool Calls,
- Blocks,
- Composer,
- Output,
- Git,
- Theme.

Do not make users manually drag `MessageViewport`, `Composer`, `ToolCard`, or `ArtifactPanel` into an empty canvas for MVP.

## Source Projects

The AgentUX SDK is vendored as prebuilt packages under `vendor/agent-ux/`. For
SDK behavior, read the shipped type declarations:

- `vendor/agent-ux/protocol/dist/index.d.ts` — canonical event and schema types
- `vendor/agent-ux/runtime/dist/index.d.ts` — run state machine
- `vendor/agent-ux/render-core/dist/index.d.ts` — view models
- `vendor/agent-ux/react/dist/index.d.ts` — React bindings

Upstream SDK source (not part of this repository):
`https://github.com/flamingtonForAI/agent-ux-sdk`.

Read only the files needed for the current task.

Some theme tokens, the activity spinner and the tool display spec were adapted
from an internal ArtifactOS UI that is not public. Treat the code in
`src/theme` and `src/components/agent-preview` as the authoritative version;
there is no external reference to consult.

## AgentUX Boundaries

UI components must consume AgentUX runtime/render-core state or view models. They must not consume provider raw streams directly.

Use existing canonical event names:

- `run.started`
- `run.finished`
- `run.error`
- `text.started`
- `text.delta`
- `text.finished`
- `reasoning.status`
- `reasoning.summary`
- `reasoning.finished`
- `tool.call.started`
- `tool.call.args.delta`
- `tool.call.awaiting_approval`
- `tool.call.running`
- `tool.call.progress`
- `tool.call.result`
- `tool.call.error`
- `tool.call.finished`
- `artifact.created`
- `artifact.delta`
- `artifact.finished`
- `capability.attached`
- `capability.suggested`
- `capability.detached`
- `heartbeat`

Do not introduce invented names such as `thinking.started`, `message.delta`, `artifact.updated`, or `run.completed`.

In product copy, `thinking` is acceptable. In protocol/code boundaries, prefer `reasoning`.

Never expose raw chain-of-thought in ordinary UI. Respect AgentUX render policy and visibility fields.

## MVP Export Target

Default export target is Vite + React + TypeScript.

Do not make Next.js the default in MVP. Next can be a later preset.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raytone-lab/agentcanvas](https://github.com/raytone-lab/agentcanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
