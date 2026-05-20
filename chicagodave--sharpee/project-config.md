---
trigger: always_on
description: Sharpee is a parser-based Interactive Fiction authoring tool built in TypeScript.
---

# Project Instructions for Claude

## Overview

Sharpee is a parser-based Interactive Fiction authoring tool built in TypeScript.

## MAJOR DIRECTIONS

- **Never auto-retry failed builds or tests.** If a build or test fails, report the error and WAIT. Do not attempt to fix and rebuild without explicit user instruction. Do not loop on build-fail-fix-rebuild cycles.
- Never delete files without confirmation. Not even "to get a build working" or "to get the other tests working".
- We currently don't care about backward compatibility.
- **Platform changes require discussion first.** Any changes to `packages/` (engine, stdlib, world-model, parser-en-us, etc.) must be discussed with the user before implementation. Story-level changes (`stories/`) can proceed autonomously.

## Per-Package Instructions

Each package owns its own conventions. Read the relevant file when working in that package:

| Package                   | Owns                                                                     | See                                  |
| ------------------------- | ------------------------------------------------------------------------ | ------------------------------------ |
| `packages/sharpee`        | npm package; auto-generated API reference (`docs/genai-api/`)            | `packages/sharpee/CLAUDE.md`         |
| `packages/world-model`    | Traits, behaviors, item portability, root barrel discipline              | `packages/world-model/CLAUDE.md`     |
| `packages/stdlib`         | Standard actions, capability dispatch (ADR-090), migration audits, action testing | `packages/stdlib/CLAUDE.md`  |
| `packages/parser-en-us`   | Grammar patterns (ADR-087), story grammar extension                      | `packages/parser-en-us/CLAUDE.md`    |
| `packages/lang-en-us`     | User-facing text, message ID mappings, formatter chain (ADR-158)         | `packages/lang-en-us/CLAUDE.md`      |

## Logic Location

Be deliberate about where logic belongs:

| Layer               | Responsibility                                | Examples                            |
| ------------------- | --------------------------------------------- | ----------------------------------- |
| **engine**          | Turn cycle, command execution, event dispatch | SchedulerService, NPC turn phase    |
| **world-model**     | Traits, behaviors, entity state               | LightSourceBehavior, ContainerTrait |
| **stdlib**          | Standard actions, common patterns             | Opening action, guard behavior      |
| **parser-{locale}** | Grammar patterns, command parsing             | `core-grammar.ts`, verb patterns    |
| **lang-{locale}**   | All user-facing text                          | Error messages, descriptions        |
| **story**           | Game-specific content and overrides           | Custom NPCs, puzzle logic           |
| **client**          | UI rendering, input handling                  | React components, terminal I/O      |

Ask: "Where does this belong?" before implementing new features.

## Always Trust the Architecture

When implementing new features, **extend existing patterns rather than inventing workarounds**.

**Example:** When the troll's axe needed to block taking with a custom "white-hot" message, the wrong approach was to propose hacks like:
- Ad-hoc properties (`(axe as any).cannotTake = true`)
- Moving items to "limbo" locations
- Pre-action event hooks (new platform concept)
- Story-specific action overrides

The right approach: Sharpee already has **Capability Dispatch (ADR-090)** where traits register behaviors for actions. Standard actions just need to check `findTraitWithCapability()` before running their logic. Story creates a clean `TrollAxeTrait` with a behavior that blocks taking while the guardian is alive.

**Checklist before proposing platform changes:**
1. Does an existing trait/behavior pattern handle this?
2. Can the capability dispatch system be extended?
3. Can event handlers react to the action?
4. Is there an ADR that addresses this pattern?

If all four are "no," then discuss a platform change. Usually the architecture already supports what you need.

> Full Capability Dispatch (ADR-090) pattern reference — decision tree, verb tables, behavior shape, story-action and event-handler patterns — lives in `packages/stdlib/CLAUDE.md`.

## Core Concepts Reference

Read `/docs/reference/core-concepts.md` at the start of each session for:

- Entity system and creation
- Trait system and usage
- Four-phase action pattern (validate/execute/report/blocked)
- ActionContext and sharedData (NOT context pollution!)
- Behaviors vs Actions (behaviors own mutations, actions coordinate)
- Event system and handlers
- Reporting is done after a turn completes by a customized report service

## Testing Commands

- **DO NOT** use `2>&1` with pnpm commands — they don't work together properly.
- Preferred format: `pnpm --filter '@sharpee/stdlib' test <test-name>`.

### Build Script

**IMPORTANT**: Use `./build.sh` instead of manual `pnpm build` commands to avoid issues.

```bash
# Run without arguments to see help
./build.sh

# Common workflows
./build.sh -s dungeo                          # Build platform + story

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChicagoDave/sharpee](https://github.com/ChicagoDave/sharpee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
