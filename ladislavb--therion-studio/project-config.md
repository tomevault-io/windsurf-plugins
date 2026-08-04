---
trigger: always_on
description: These instructions apply to the whole repository.
---

# Therion Studio Agent Instructions

## Scope

These instructions apply to the whole repository.

## Project State

- This repository is currently specification-first. The primary source of truth is [SPECIFICATION.md](SPECIFICATION.md).
- Treat the specification as implementation-grade requirements for a Qt reimplementation of Therion Studio.
- Treat [ARCHITECTURE.md](ARCHITECTURE.md) as the current architecture direction and boundary contract. It does not override product requirements in the specification, but implementation choices should align with it unless the divergence is explicit and documented.
- Treat [docs/THERION_COMPATIBILITY.md](docs/THERION_COMPATIBILITY.md) as the current Therion language compatibility guardrail for parser, validator, project-index, source-range, namespace, reference-resolution, completion, highlighting, and map/source synchronization work.
- Do not invent product behavior that conflicts with the specification. If a requested change requires behavior not covered by the specification, update the specification as part of the same change or clearly flag the gap.
- If implementation, tests, architecture, and the specification diverge, prefer bringing them back into alignment explicitly rather than silently choosing one source of truth.
- Treat [plans/REVIEW_CODEX.md](plans/REVIEW_CODEX.md) as the current architecture review record. It is not a product specification, but actionable findings from it should be reflected in `AGENTS.md`, `WORKLOG.md`, focused tests, or the specification before being treated as resolved.

## Specification Editing Rules

- Preserve the document's SRS style and section structure unless the change explicitly requires restructuring.
- Use normative language consistently:
  - "shall" for mandatory behavior
  - "should" for strong recommendations
  - "may" for optional behavior
- Keep requirements testable. New functional requirements should be specific enough to verify.
- Keep implementation notes and guidance separate from normative requirements.
- Preserve explicit MVP and Post-MVP distinctions where relevant.
- Prefer additive edits over broad rewrites. Do not weaken or broaden requirements casually.
- When adding a new behavior, update the most relevant acceptance criteria section if verification expectations change.

## Implementation Expectations

- Default technology assumptions should match the specification: Qt 6, cross-platform desktop, shared core logic across macOS, Windows, and Linux.
- Follow established best practices and relevant platform standards for modern Qt and C++ desktop application development unless the specification explicitly requires a different approach.
- Treat [WORKLOG.md](WORKLOG.md) as the active short-form implementation roadmap and backlog. Keep stable architectural principles in [ARCHITECTURE.md](ARCHITECTURE.md), keep detailed working plans in [plans/](plans/), and keep `WORKLOG.md` focused on current/open work.
- Preserve separation of concerns:
  - domain model, parsing, serialization, and editing rules stay outside UI classes
  - widgets, views, and scene items should not own file I/O or document parsing
  - services own external interactions such as file access, process execution, and metadata loading
- Favor straightforward, testable designs over speculative abstractions.
- Do not introduce new uses of deprecated functions or APIs. Prefer supported, current equivalents and update existing deprecated usages when touching related code.
- Keep files and translation units focused and reasonably sized. There is no hard line limit, but once a file starts mixing multiple responsibilities or becomes difficult to scan in one pass, split it by responsibility. Try to avoid files with >2000 lines if possible.
- Prefer splitting large features into focused types or files such as model, parser, serializer, widget, controller, scene item, or service components rather than accumulating unrelated behavior in one class.
- Avoid generic catch-all modules or class names such as Helpers, Utils, Misc, Support, or Manager unless the code is genuinely cohesive and no clearer responsibility word exists.
- Do not introduce dependencies outside Qt and the standard library without explicit justification.

## Architecture Guardrails

- Keep the intended dependency direction explicit:
  - presentation (`src/app/**` widgets, dialogs, views, scene items) may depend on application/core contracts and injected infrastructure interfaces
  - application workflow services should avoid QtWidgets and should be testable without launching the full GUI
  - core/domain code shall not depend on QtWidgets, QGraphicsView, dialogs, windows, or UI event classes
  - infrastructure/platform adapters may use Qt/platform APIs, but should expose narrow interfaces upward
- Do not add new business rules, persistence rules, parser/serializer logic, process orchestration, command-catalog loading, settings access, or platform-path decisions directly into widgets, dialogs, QGraphicsItems, or scene items.
- Do not add new direct `DocumentFile` calls from widgets. Route document loading, saving, and encoding decisions through focused document workflow/IO services.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ladislavb/therion-studio](https://github.com/ladislavb/therion-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
