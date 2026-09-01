---
trigger: always_on
description: Read [`master-plans/00-master-plan.md`](master-plans/00-master-plan.md) first, before any other work. It explains how master plans are used and maintained. Then find every plan in [`master-plans/`](master-plans/) relevant to your task and read each one in full before starting.
---

# Agent Instructions

## Master plans

Read [`master-plans/00-master-plan.md`](master-plans/00-master-plan.md) first, before any other work. It explains how master plans are used and maintained. Then find every plan in [`master-plans/`](master-plans/) relevant to your task and read each one in full before starting.

Master plans are dictated by the user and describe where the product is going, in what order, and what counts as done. They outrank conclusions drawn from the existing code. Do not create, edit, rename, or delete a file in `master-plans/` unless the user explicitly asks for that change in the current task. When the code contradicts a master plan, report the contradiction instead of revising the plan.

All persistent plans must live in `master-plans/` and follow the master-plan rules above. Do not create design documents, implementation plans, slash-command plan artifacts, or planning directories anywhere else in the repository, including `docs/plans/`.

Discussion notes that support or contextualize master plans must live only in
`master-plans/notes/`. Do not create these notes in the `master-plans/` root,
in another documentation directory, or anywhere else in the repository.

## API specification

Do not create, edit, rename, or delete any `API.md` file without direct human
input in the current task that explicitly authorizes the specific API
specification change.

`packages/happy-agent/API.md` is the authoritative Happy Agent API contract. Client types,
runtime schemas, daemon behavior, tests, and documentation must conform to it exactly. Never
invent, implement, preserve, or release behavior that deviates from the specification; stop and
request a human-directed specification change when the desired behavior is not already described.

Happy Agent API compatibility begins with protocol version 22. From version 22 onward, every API
change must be backward-compatible and additive: never remove or rename an existing field, and
never add a required field. New fields must be optional so older clients and daemons can ignore
them safely. If a desired change cannot be made within those constraints, stop and request direct
human guidance rather than breaking compatibility.

Rig must support every Happy Agent API version from protocol version 22 onward and must be able to
work with any Happy Agent CLI version in that compatibility range. Do not couple Rig to only its
bundled or current CLI version; negotiate or tolerate protocol-version differences so users can
select and run any compatible CLI release.

## Modules

A module is a self-contained feature. It carries everything that feature needs to work: it extends the agent loop through its own hooks, owns its tools, starts and supervises its background processes, and holds its connections to third-party services. Adding a module to an agent is the whole installation — nothing elsewhere should have to be wired up, registered, or branched on for the feature to function.

A module may take only other modules as arguments. Not configuration objects, path strings, clients, callbacks, or loose handles. When a module needs something, it takes the module that owns that thing and asks it. This keeps the dependency graph a graph of features, and keeps a module's collaborators visible in its constructor rather than assembled by whoever happens to build it.

New modules must use Durable Functions for durable execution from the start. Take the Durable Functions module as a dependency and register durable work there instead of hand-rolling module-local recovery machinery.

Modules do not import from each other beyond the seam that joins them. A module may import another module's class and the public types that module exports from its `index.ts` — enough to declare it as a constructor dependency and to speak about the values it returns. Nothing else crosses the boundary: helper functions, `impl/` internals, stores, schemas, migrations, and prompt text belong to the module that owns them. A module that needs such behavior asks the owning module instance for it instead of importing the file, and when there is no method to ask, the method is added to the owning module rather than reached past.

The config module is what that rule leans on most. It is not merely parsed configuration: it resolves and owns the paths the product runs against, and it instantiates the providers. A module that needs a path or a provider depends on the config module and takes it from there, instead of deriving paths itself or constructing a provider of its own.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slopus/happy-agent](https://github.com/slopus/happy-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
