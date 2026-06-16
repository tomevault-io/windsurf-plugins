---
trigger: always_on
description: generates a versioned tarball that the backend's release workflow
---

# Notes for Claude

A short orientation file for an LLM working in this repo. Skim
before making changes; keep edits to existing code consistent
with what's described here.

**Before writing code, read [README.md → "Code structure
policies"](README.md#code-structure-policies).** Those rules
(500-600 line file cap, component decomposition, folder layout,
TypeScript / DOM / localization / comment policies) are the
authoritative coding standard for this repo, set and maintained
by the human maintainers; everything in this CLAUDE.md sits on
top of them. When a rule in README.md and a rule here disagree,
README.md wins; flag the conflict in the PR so this file can be
brought back into line.

## What this project is

The **frontend** for the ESPHome Device Builder dashboard, a Lit
and TypeScript SPA that ships **prebuilt and bundled** inside the
backend wheel ([esphome/device-builder](https://github.com/esphome/device-builder)).
End users never install this directly. A release of the frontend
generates a versioned tarball that the backend's release workflow
picks up; that's the only deployment path.

## Frontend-backend deployment is **lockstep, not a wire contract**

This is the load-bearing fact that shapes most of the rules below.
The frontend in this repo and the backend that runs it always
ship together; a given backend version pins a specific frontend
version via the wheel's bundled assets. There is no installation
in the wild that runs frontend N with backend N±1.

Practical consequences:

- **Don't write backwards-compatibility shims for the backend.**
  No `instanceof APIError && err.errorCode === "unknown_command"`
  fallbacks for "an older backend that doesn't know this WS
  command yet". The backend ALWAYS knows every command this
  frontend issues, because it's the backend that bundled this
  frontend. A failure on a known command is a real bug, not a
  version drift.
- **Don't probe for feature support before using a feature.** If
  the backend just landed `remote_build/list_hosts`, the frontend
  PR that consumes it lands at the same time. There's no
  "feature flag" or "is this command available" check. Either
  the frontend uses it or it doesn't ship yet.
- **Backend WS-command renames / shape changes are coordinated
  PRs.** Always link the companion backend PR in the frontend
  PR's description. CI doesn't enforce the link but a reviewer
  will catch a frontend PR that consumes a command nobody added
  on the backend side.
- **Old translation keys**: when removing a `_localize("foo")`
  call site, also delete the key from `en.json` (the only
  committed locale; the rest live in Lokalise). No legacy keys
  retained "in case some downstream uses them": there is no
  downstream.

A real failure path (WS dropped, server bug, validation rejection)
still warrants a `try/catch` with revert + toast for
security-sensitive controls. The rule is "don't write code that
exists only to handle a version skew that can't happen", not
"never catch errors".

## What this means at PR-review time

If a reviewer leaves a comment suggesting a defensive branch for
"older backend without command X" or "fallback for older client",
push back: that case can't happen in this deployment shape.
Linking this CLAUDE.md inline (e.g. "see CLAUDE.md §
Frontend-backend deployment is lockstep") is the canonical reply.

## Code style

See [README.md → "Code structure
policies"](README.md#code-structure-policies) first; the file
size, component-decomposition, folder layout, and DOM / a11y
rules there are authoritative. The bullets below are practical
expansions, not substitutes. The high-leverage ones to keep in
working memory while editing:

- **File size cap: 500-600 lines.** Split before crossing it; no
  exemptions for "it's just one big component." If a render
  block exceeds ~100 lines, that's the signal to extract a
  sub-component.
- **One `@customElement` per file.** File name matches element
  name (`esphome-foo-bar.ts` → `<esphome-foo-bar>`). When a
  feature grows beyond ~3 files, give it its own subfolder
  (`src/components/settings-dialog/` is the pattern).
- **No `document.querySelector`, no direct DOM mutation.** Go
  through shadow DOM via `@query` / refs, and use reactive
  properties to re-render. No business logic in `render()`;
  extract it to private methods or computed values.
- **TypeScript strict** throughout. No `any` in new code; use
  `unknown` and narrow when truly necessary. Existing `as never`
  casts are legacy and shouldn't be cargoed.
- **Lit components** use `@customElement("esphome-foo-bar")` and
  decorators (`@state`, `@property`, `@query`, `@consume`,
  `@provide`). Mirror the existing patterns rather than
  introducing new ones.
- **Context for cross-component state.** When two unrelated
  components both need a value (theme, locale, the labels
  catalog, the API instance, ...), provide it via Lit context
  from `app-shell` and consume it where it's needed. Avoid prop
  drilling and avoid a global singleton; the context-based
  pattern is what lets `app-shell` own the WS lifecycle and
  every consumer pick up reconnect events for free.
- **Styles** live in `src/styles/shared.ts` (`espHomeStyles`)
  for cross-component utilities; component-local rules go in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esphome/device-builder-frontend](https://github.com/esphome/device-builder-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
