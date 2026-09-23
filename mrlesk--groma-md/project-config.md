---
trigger: always_on
description: <!-- BACKLOG.MD GUIDELINES START -->
---

<!-- BACKLOG.MD GUIDELINES START -->
<!-- backlog.md-instructions-version: 1.48.0 -->
<CRITICAL_INSTRUCTION>

## Backlog.md Workflow

This project uses Backlog.md for task and project management.

Run `backlog instructions overview` before code work or Backlog task administration. Skip it for questions,
read-only audits, and standalone documentation changes that do not involve Backlog records.

Use the overview to decide whether to search, read, create, or update Backlog tasks.

Before task lifecycle actions, read the matching detailed guide:

- `backlog instructions task-creation` before creating or splitting tasks
- `backlog instructions task-execution` before planning, changing status or assignee, adding a plan or implementation
  notes, or implementing task work
- `backlog instructions task-finalization` before checking acceptance criteria, writing final summaries, or moving tasks
  to terminal statuses

Use `backlog <command> --help` before running unfamiliar commands. Help shows options, fields, and examples.

Do not edit Backlog task, draft, document, decision, or milestone markdown files directly. Use the `backlog` CLI so
metadata, relationships, and history stay consistent.

</CRITICAL_INSTRUCTION>
<!-- BACKLOG.MD GUIDELINES END -->

## IMPORTANT: Point of view

Write for a junior developer who knows the language but is new to the project and has no access to this conversation.
They should be able to find where a change belongs, follow the flow from entry point to result, and identify who owns
each responsibility. Describe the final system and its reasons, not the approaches tried.

## OKF and C4 are design foundations

When proposing or changing architecture concepts, relationships, flows, or stored knowledge, reason explicitly about
both OKF 0.2 and C4 before choosing the model.

- OKF defines how knowledge remains readable, linked, and portable. Prefer standard metadata, ordinary Markdown, and
  Markdown links. Keep Groma-specific metadata under `groma`; do not duplicate information already expressed by standard
  fields or the document body.
- C4 defines architecture levels and boundaries. Decide whether a concept is an actor, system, container, component,
  relationship, or supporting knowledge about the architecture. A new OKF concept does not automatically become a C4
  element, containment level, or box on the map.
- Groma's application profile defines the additional meaning and constraints needed by its supported behavior.
  Distinguish those rules from requirements imposed by OKF or C4.

For a relevant proposal, briefly explain:

1. Where the concept belongs in OKF and C4.
2. What an ordinary Markdown or OKF reader can understand without Groma.
3. What Groma must interpret, and which existing concept owns that meaning.

Read [the architecture Markdown contract](docs/component-markdown.md) and the relevant product documentation.
Consult the authoritative [OKF specification](https://github.com/GoogleCloudPlatform/open-knowledge-format/blob/main/SPEC.md)
or [C4 model](https://c4model.com) when the decision depends on rules those documents do not establish. Do not invent a
standard requirement or add optional metadata merely because the standard supports it.

## Test decisions across projects and languages

When making product, design, architecture, or implementation decisions, ask:

> If Groma ran against millions of projects across hundreds of programming languages, would this still be the right decision?

Use this question to identify assumptions tied to the current project, technology, workflow, or example. Prefer choices
whose reasoning remains sound across different contexts. Explain any dependence on the current context and why the
requested result requires it.

This is a test of the decision, not permission to expand the task. Implement and verify the smallest approved result for
the current supported example. Do not add infrastructure, abstractions, or capabilities solely for that
future scale. See the [manifesto principle](MANIFESTO.md#principles-that-hold-across-projects-and-languages).

## IMPORTANT: Experimental prototype

Groma is an early experimental prototype used only by its developers. It has no external users and no released data,
storage, CLI, or API contracts that must remain compatible.

Do not preserve previous versions. Do not add backward compatibility, migrations, legacy formats, compatibility adapters,
or deprecation paths unless the current user explicitly requests one. When the product direction changes, replace the old
behavior directly and delete obsolete code, documentation, tests, and prototype data.

It is acceptable to wipe and recreate all Groma-owned prototype state required by the current task instead of migrating
it. This permission applies only to explicitly scoped Groma artifacts and never to unrelated developer files or systems.


## Backlog task scope

As a project-specific override to the overview's general task-creation guidance, create Backlog tasks only when the
requested work includes code changes. Do not create a task for standalone documentation changes or Backlog record
administration, such as relabeling, status corrections, or metadata maintenance. Documentation required to deliver an
in-scope code change may remain part of that code task.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MrLesk/Groma.md](https://github.com/MrLesk/Groma.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
