---
trigger: always_on
description: Use Django and djust as part of the implementation. Prefer their supported APIs
---

# Application engineering conventions

Use Django and djust as part of the implementation. Prefer their supported APIs
and declarative options before writing application plumbing. This guide applies
to applications; framework contributor build/review rules remain in `CLAUDE.md`.

## Before writing code

1. Read the affected flow and identify the installed djust version. A sibling
   checkout or online main branch may differ from the deployed release.
2. Check the relevant [AI reference](README.md), installed API, and existing
   application pattern. An unfamiliar API is not a missing capability.
3. Reuse native behavior before reusing an application wrapper that duplicates
   it. Write custom code for product behavior or a reproduced framework gap.
4. Prefer readable declarations and the smallest correct implementation. Do not
   optimize for line count, fewer files, or fewer tests at the cost of clarity,
   security, accessibility, or the requested behavior.

## Ownership and native patterns

| Concern | Convention | Reference |
| --- | --- | --- |
| Lifecycle and state | Initialize state in `mount()`. Keep internal objects in private attributes; expose only rendering data before calling `super().get_context_data()`. Refresh authorized queries when their inputs change. A `_refresh()` helper is an application pattern, not another lifecycle to implement. | [Lifecycle](lifecycle.md), [JIT](jit.md) |
| Ordinary editors | Use `FormMixin`, `form_class`, native field validation and submit hooks. Use `as_live()` or `as_live_field()` with declarative widget options. Do not maintain a second draft/error/renderer system. | [Forms](forms.md) |
| Input rules | Prefer explicit-field `ModelForm`s for single-model editors; use `Form` for composite inputs and actions. Derive lengths/choices from model metadata. Share validators, not copied schemas. Coercion alone is not validation. | [Forms](forms.md), [Security](security.md) |
| Template data | Pass authorized models/querysets to native rendering. Templates own related access, labels, dates, URLs and loops. Do not flatten models into presentation dictionaries merely for transport. Declare sensitive-field exclusions and verify HTTP/socket output. | [JIT](jit.md), [Templates](templates.md) |
| Custom events | Use `@event_handler()` for client-callable handlers, compatible signatures/defaults, `value` for input/change events, and native debounce/throttle where needed. Validate untrusted parameters and authorize the selected object at mutation time. | [Events](events.md), [Security](security.md) |
| Navigation | Use `dj-navigate` between live pages and `dj-patch` for state within a view. Preserve real URLs and URL-backed filters/pagination. Auth/admin/external boundaries can use ordinary navigation. | [Navigation guide](../website/guides/navigation.md) |
| Form markup and fallback | Keep semantic forms and CSRF. Native JSON HTTP fallback invokes the event flow when WebSockets are unavailable; it is not JavaScript-disabled HTML POST support. Do not add duplicate mutation routes unless the product explicitly requires them. | [Forms](forms.md) |
| Loading and background work | Use native loading/disabled directives and `start_async()` or `@background` when appropriate. Show save/error feedback beside the action. Do not block event handlers with long work or add redundant async mixins already inherited by LiveView. | [Loading/background](loading-states.md) |
| Shared UI | Reuse templates, existing components, and the application's theme. Keep stable element identities in changing lists; preserve focus, keyboard behavior and accessible feedback. Use client hooks only for browser behavior that needs them. | [Templates](templates.md), [Components](components.md) |
| Auth, uploads and realtime | Inspect native auth, upload, presence, streaming and push APIs before adding parallel systems. Keep binary uploads out of serializable form state. Scope presence/push and recheck current access on refresh. | [Security](security.md), [Capability lookup](#capability-lookup) |

## Application policy stays in the application

A simple authorized `form.save()` or ORM operation may live in a view. Services
are optional: use them for shared workflows, multi-record transactions, locking
or rules used by multiple entry points. Do not create pass-through services.
Django apps and workflow-based view packages are organization choices, not extra
framework requirements.

Define each input rule once, but validate at every relevant trust boundary.
A service accepting raw input must remain safe for commands/background callers.
Native form validation does not replace fresh authorization, current-state checks
or database constraints. Model `save()` does not automatically call `full_clean()`.
Publish mutation notifications after commit when consumers must observe committed
records. Keep genuine transformations such as image sanitization and sanitized
Markdown; do not confuse them with redundant model serialization.

## Small examples to follow

- [Lifecycle example](lifecycle.md): private queryset preparation and public assigns.
- [FormMixin examples](forms.md): native form validation, rendering and save hooks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djust-org/djust](https://github.com/djust-org/djust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
