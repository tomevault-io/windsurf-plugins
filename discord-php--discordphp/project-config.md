---
trigger: always_on
description: This file is the repo operating manual for AI agents. It describes how to work inside this repository without breaking its design. Specialist skills in `.agents/skills/` provide deeper playbooks for specific layers.
---

# DiscordPHP Agent Guide

This file is the repo operating manual for AI agents. It describes how to work inside this repository without breaking its design. Specialist skills in `.agents/skills/` provide deeper playbooks for specific layers.

If a change crosses layers, load multiple skills and use the playbooks in this file to keep boundaries clean.

## Start here

Before changing anything:

1. Identify the layer you are touching.
2. Read the base abstraction for that layer before copying a concrete class.
3. Read one representative concrete implementation from the same family.
4. Trace how that layer connects to adjacent layers.
5. Make the change in the narrowest layer that can own it.
6. Update companion surfaces that define the same contract.

In this repo, companion surfaces usually matter as much as the line you edit.

## Non-negotiable truths

1. **CLI-only runtime.** DiscordPHP is a long-running process built on ReactPHP. Do not design around web requests, controllers, middleware stacks, or per-request state.
2. **Async first.** Production I/O is Promise-based. Blocking helpers belong in tests only.
3. **Parts are canonical domain objects.** They model Discord resources and expose typed magic properties.
4. **Repositories are persistence and cache boundaries.** They are not generic service classes.
5. **Gateway handlers keep caches coherent.** They do more than relay notifications.
6. **Builders own outbound payload rules.** If a payload has meaningful shape or validation, it usually deserves a builder.
7. **Docblocks are runtime-adjacent documentation.** They are not optional decoration.
8. **Traits are preferred over deep inheritance or broad interface hierarchies for shared behavior.**
9. **Type maps are central dispatch points.** If a Discord payload is polymorphic, there is usually one place that decides the concrete subtype.
10. **Use current library idioms.** Prefer `Factory::part()` / `Factory::repository()`, prefer `$part->save($reason)` over repository `save($part)` in user-facing paths, and prefer builder `->create($repository)` helpers where they exist.

## Skill map

Each skill lives in `.agents/skills/<name>/SKILL.md` and is loaded automatically when relevant. When a task crosses layers, load multiple skills.

| If task touches... | Skill to load |
| --- | --- |
| `Discord.php`, startup, intents, cache, loop, gateway connection | `runtime-bootstrap-keeper` |
| `Parts/*`, domain modeling, mutators, typed nested data | `part-model-maintainer` |
| `Repository/*`, endpoint vars, cache, CRUD, fetch/save/delete | `repository-cache-keeper` |
| `WebSockets/Handlers.php`, `WebSockets/Event.php`, `WebSockets/Events/*` | `gateway-cache-sync-keeper` |
| `Builders/*`, `Builders/Components/*`, outbound payload rules | `builder-payload-smith` |
| subtype maps like `Channel::TYPES` or `Interaction::TYPES` | `type-map-keeper` |
| interactions, slash commands, resolved data, autocomplete, modals | `interaction-flow-keeper` |
| `DiscordCommandClient` or prefix-command behavior | `legacy-command-client-keeper` |
| tests, guides, docblocks, generated reference expectations | `async-test-and-doc-sync` |
| `Voice/*`, audio streaming, encryption, voice gateway protocol | `voice-subsystem-keeper` |
| `Helpers/*`, `Exceptions/*`, cache wrappers, `Endpoint::bind()`, `Collection` | `helpers-and-infra-keeper` |

## Architecture map

| Layer | Owns | Primary files | What to preserve |
| --- | --- | --- | --- |
| Runtime | process lifecycle, options, loop, gateway, HTTP, root repos | `src/Discord/Discord.php` | `__construct()` wires dependencies and connects; `run()` only starts loop |
| Factory | part/repository instantiation | `src/Discord/Factory/Factory.php` | callers should not construct repo families ad hoc |
| Parts | domain objects, mutators, typed nested data, high-level operations | `src/Discord/Parts/Part.php`, `src/Discord/Parts/PartTrait.php`, `src/Discord/Parts/**/*` | `$fillable`, mutators, PHPDoc, `save()` semantics, `created` lifecycle |
| Repositories | typed collections, cache, REST endpoints, CRUD | `src/Discord/Repository/AbstractRepository.php`, `src/Discord/Repository/**/*` | `$class`, `$endpoints`, `$vars`, cache writes, Promise-based API |
| Builders | outbound payload construction and validation | `src/Discord/Builders/**/*` | fluent setters, validation, `jsonSerialize()`, `fromPart()` symmetry |
| Gateway events | payload hydration, cache mutation, emitted return shapes | `src/Discord/WebSockets/Handlers.php`, `src/Discord/WebSockets/Event.php`, `src/Discord/WebSockets/Events/*` | typed part creation, related cache updates, event contract shape |
| Helpers | cross-cutting utilities: cache wrappers, BigInt math, multipart uploads, property mutator trait, domain exceptions | `src/Discord/Helpers/*`, `src/Discord/Exceptions/*` | no domain logic here; utilities only |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [discord-php/DiscordPHP](https://github.com/discord-php/DiscordPHP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
