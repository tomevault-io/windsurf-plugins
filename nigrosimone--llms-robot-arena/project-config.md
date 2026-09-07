---
trigger: always_on
description: **Rules version:** `0.2.2-draft`
---

# llms-robot-arena - rules and agent instructions

**Rules version:** `0.2.2-draft`  
**Engine:** `0.2.2-r1`

This is the llms-robot-arena project. Repository: [nigrosimone/llms-robot-arena](https://github.com/nigrosimone/llms-robot-arena). Keep project text, comments, documentation, and user-facing messages in English. The rules below apply when implementing, improving, or evaluating a bot. Project maintenance does not authorize using opponent internals to develop a bot.

## Opponents are black boxes

**Every other bot implementation is a black box. Do not inspect, copy, reverse engineer, or spy on it.**

- Read and edit only the bot file assigned to you. Do not open, search, diff, import for inspection, or print another file in `packages/bots`, including reference controllers.
- Do not bypass this restriction through other project folders, previous versions, Git history, generated bundles, source maps, archives, caches, development scripts, logs, or reports containing source code or internal strategy explanations.
- Do not inspect an opponent's serialized memory, module state, intermediate variables, or debugger traces. Do not modify the runtime to expose them.
- Do not ask another agent, model, tool, or person to inspect an opponent for you. These restrictions apply to every delegated task.
- The standard match runner may load opponent files internally to execute a match. You may pass their paths and public IDs as opaque inputs to that runner; this is not permission to display or analyze their contents.
- You may observe public match outcomes, rankings, replay events and trajectories, and the information available through the documented sensor interface. Infer behavior from these observations only.
- If opponent internals are accidentally exposed, stop examining them, disclose the exposure, and do not incorporate the information into your bot. Do not claim an uncontaminated evaluation.

## Scope and public information

This file is the single reference for game rules, TypeScript contracts, bot development, evaluation, local operation, and publication. Use the rules and constants below and evaluate results under matching rules, engine versions and runtime budgets. The public example in this document may be used as a starting template, without opening any other bot implementation.

Keep this file focused on instructions for agents. Omit camera controls, visual styling details and end-user interface walkthroughs.

For general project searches, explicitly exclude `packages/bots/**`, `dist/**`, `node_modules/**`, `artifacts/**`, `results/**`, archives, and the sibling implementation folders. Read your own assigned bot by its exact path. Do not browse other implementations to find a starting template.

## Implementing a bot

1. Use the requested name and create one file at `packages/bots/<bot-id>.js` or `.ts`. Never overwrite another bot. If no ID was supplied, choose a descriptive unused filename without opening existing bot files.
2. Export exactly one function: `tick(sensors, memory)`, returning `{ actions: { thrust, turn }, memory }`. Use no imports, dependencies, extra exports, network, filesystem, clock, randomness APIs, or dynamic code generation.
3. Treat sensors and incoming memory as immutable. Persistent state belongs only in returned JSON memory; each tick gets a fresh module scope. Initial memory is `null`. Return finite numeric actions and strict JSON memory within the documented 64 KiB limit.
4. Design using the public rules and sensor interface. Do not modify physics, energy, collision thresholds, runtime budgets, gates, fixtures, ranking, or an opponent to help your bot win. Report an engine issue separately.
5. Keep changes within your own bot, its entry in `bots.json`, and explicitly named evaluation artifacts/manifests. Ask for a separate maintenance task if platform integration requires unrelated changes.

## Validation and fair evaluation

From this directory, check your own file with:

```sh
node packages/runtime/gate-cli.js packages/bots/<bot-id>.js
```

For an authorized iterative comparison, create a manifest using your own source path and the opponent's opaque path, then run:

```sh
node packages/tournament/cli.js --bots match-bots.json --out results/<bot-id> --mode iterative --budget fuel
```

A manifest selects registered bots by ID, without repeating their metadata. For an unregistered controller, use an object with `id`, `model`, `provider`, and `file`; object paths are relative to the manifest file. See the Tournament CLI section below for the format. Use the standard mirrored-seed protocol, record source hashes and the engine/spec versions, and report wins, draws, losses, and violations honestly. Do not cherry-pick favorable seeds or combine results from different rules or budget modes.

The gate checks conformity, not tactical strength. A `fuel` result is a deterministic instruction-budget result, not proof of meeting the wall-clock budget. Use `--budget wall` when that evaluation is explicitly required.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nigrosimone/llms-robot-arena](https://github.com/nigrosimone/llms-robot-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
