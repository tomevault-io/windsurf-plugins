---
trigger: always_on
description: > **Type:** Convention
---

# Conventions

> **Type:** Convention
> Locked naming and structural conventions that span the Story spec — reserved namespaces for Story-owned ids, the canonical id grammars, the `reg-*` macro / `*`-fn split, the chrome-installer pair shape, the `*-id` Var pattern, and the token-banning rules.

The framework's [`spec/Conventions.md`](../../../spec/Conventions.md) catalogues conventions that span the **whole** spec; this document catalogues the conventions that span only Story. The two are read together: Story honours every framework convention (single `:rf/*` root, `*`-suffix on programmatic fn partners, bang on process-level lifecycle, `:rf.assert/*` is framework-reserved per Story's library-owned `:story.*` carve-out), and adds the Story-specific rules below.

## Reserved namespaces (Story-owned)

Story is a [**library-owned prefix**](../../../spec/Conventions.md#library-owned-prefixes) under the framework's single-root convention. The carve-outs live outside `:rf.*` (`:story.*` and `:Workspace.*` for user-facing ids) and inside `:rf.story.*` (for Story-emitted framework events). Both sets are reserved by Story's spec; both follow the framework's "fixed-and-additive" discipline.

### The `:story.*` / `:Workspace.*` user-facing carve-outs

| Sub-namespace | Used for | Spec |
|---|---|---|
| `:story.<path>/<name>` | User-authored story ids (`:story.auth/login-form`) and variant ids (`:story.auth.login-form/empty`). The path segments mirror the source-tree feature hierarchy; the trailing `/<name>` names the story or variant within that path. | [001-Authoring.md](001-Authoring.md) §Story id grammar |
| `:Workspace.<path>/<name>` | User-authored workspace ids (`:Workspace.Auth/all-states`). The capitalised root segment is the convention — workspaces are top-level affordances, not story-graph leaves. | [001-Authoring.md](001-Authoring.md) §Workspace id grammar |
| `:Mode.<path>/<name>` | User-authored mode ids consumed by the toolbar (`:Mode.Theme/dark`, `:Mode.Viewport/mobile`). The capitalised root names the axis; the trailing `/<name>` names the value. | [010-Toolbar.md](010-Toolbar.md) §Mode id grammar |

User code MUST use one of these grammars for the corresponding registration kind. The capitalised root segments (`:Workspace.*`, `:Mode.*`) are visually distinct from the lower-case `:story.*` so a reader scanning a registration form can tell the kind from the id alone.

### The `:rf.story.*` framework carve-out

Story-emitted framework events and panel ids live under the framework's `:rf.*` root, in the `:rf.story.*` sub-namespace. The framework's [`spec/Conventions.md`](../../../spec/Conventions.md) reserves the sub-namespace; Story owns the closed member set.

| Sub-namespace | Closed member set | Spec |
|---|---|---|
| `:rf.story/*` | Story-emitted events (`:rf.story/save-current-as-variant`) and built-in decorator ids (`:rf.story/force-fx-stub`). | [005-SOTA-Features.md](005-SOTA-Features.md) |
| `:rf.story.panel/*` | Story-panel registration ids (`:rf.story.panel/schema-validation`, `:rf.story.panel/layout-debug`). The `panel` segment is the discriminator so a reader scanning a panel id can tell the registry kind. | [003-Render-Shell.md](003-Render-Shell.md) §Panel registration contract |
| `:rf.story/layout-debug.*` | Built-in layout-debug decorator ids (`:rf.story/layout-debug.measure`, `:rf.story/layout-debug.outline`, `:rf.story/layout-debug.pseudo`) — dotted names under the `:rf.story/*` root (`layout_debug.cljc`), not a sub-namespace. | [005-SOTA-Features.md](005-SOTA-Features.md) §Layout-debug overlay trio |
| `:rf.story.inline/*` | Anonymous inline-plan frame ids (`:rf.story.inline/plan-1`, `:rf.story.inline/plan-2`, …), minted per-run by `mint-inline-frame-id` for an inline (map-target) plan execution. Never a registered variant id, so an inline frame can neither collide with nor appear alongside a navigable variant. | [017-Testing-Story.md](017-Testing-Story.md) §Inline plan |

Third-party Story extensions MUST NOT register handlers, fx, subs, panels, or decorators under `:rf.story.*`. Library authors choose their own top-level prefix per [framework Conventions §Library-owned prefixes](../../../spec/Conventions.md#library-owned-prefixes).

## `reg-*` macro / `*`-fn split

Every registration kind in Story ships **two surfaces** under the same name:

- The `reg-<kind>` **macro** — author-facing. Captures call-site source-coords (file + line + ns) via `&form` meta + `*file*` + `(ns-name *ns*)`, threads them into the runtime helper via `re-frame.story.macros`. This is the form authors write in their stories ns.
- The `reg-<kind>*` **fn** — programmatic. Same body shape, no source-coord stamping (callers supply coords explicitly if needed). This is the form MCP write tools and hot-reload tooling call.

Mirror of [`re-frame.core`](../../../implementation/core/src/re_frame/core.cljc)'s `dispatch` / `dispatch*` precedent. The nine registration kinds ship both surfaces uniformly:

| Macro | `*`-fn partner | Body shape | Spec |
|---|---|---|---|
| `reg-story` | `reg-story*` | story body + optional `:variants` desugaring (Form B) | [001-Authoring.md](001-Authoring.md) |
| `reg-variant` | `reg-variant*` | variant body (`:doc`, `:component`, `:args`, `:decorators`, `:script`, …) | [001-Authoring.md](001-Authoring.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [day8/re-frame2](https://github.com/day8/re-frame2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
