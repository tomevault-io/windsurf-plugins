---
trigger: always_on
description: Working rules for this repository. They apply to AI agents and human
---

# AdaptTable — agent and contributor guide

Working rules for this repository. They apply to AI agents and human
contributors alike; PRs are reviewed against them.

## Architecture: core is headless

`@adapttable/core` owns the engine — state, the filter model, operators, URL
serialization, chips, mutations, registries, and the state machines behind
every interaction. The framework bindings adapt them into hooks (`@adapttable/react`) or signals
(`@adapttable/angular`) and render structure. Adapters own the look. Switching the kit in
the live demo must change how the whole table looks, including the Filters
popover and drawer: an MUI table filters with MUI controls, a Mantine table
with Mantine controls.

Unify the **model**, never the **pixels**. The standing pattern is
**Chrome + slots**:

- The binding ships `*Chrome` components that own structure only — layout,
  the recursion over groups, keyboard wiring, localized labels, and the
  `data-adapttable-part` names — driven by the models and state machines in
  core.
- Every visible control (input, select, checkbox, button — anything the end
  user clicks) is a **required slot** the adapter fills with its own kit's
  component. Slots have no native fallback in core or the binding: a kit
  cannot silently render raw HTML. `adapter-unstyled` supplies native
  controls because native IS its kit; shadcn/tailwind build on unstyled.
- Do not add user-facing controls to core or the binding, and do not give a
  slot a default implementation. Invisible chrome (live regions, announcers,
  layout structure) is fine there.
- Feature parity means every kit has the feature **with its own components**.
  A shared look is not parity — copying one adapter's raw-HTML control into
  another adapter is the same defect as drawing it in core.
- If a kit's overlay or portal misbehaves inside the filter popover, fix it
  in that adapter (`disablePortal`, `getPopupContainer`, or that kit's native
  select) — never by weakening the slot contract.
- `data-adapttable-part` names and placement are part of the public contract:
  the same part lands on the same element in every adapter, and documented
  `classNames` keys are honored by every adapter that renders the part.
- A kit talks only to its binding: a React kit imports only
  `@adapttable/react/…`, a Vue or Angular kit only its own binding. Only a
  binding imports `@adapttable/core`, and it re-exports what its kits need.
  When a kit needs a core name its binding lacks, add the re-export to the
  binding — never import core from the kit. Core's deprecation notes point to
  core (`@adapttable/core/binding`), never to a framework package. ESLint
  enforces this on kit sources.

## Framework packages

- `packages/react/react` is the React binding; its eight supported kits live
  under `packages/react/adapter-*`.
- `packages/angular/angular` is the Angular binding, with signal controllers,
  templates, component renderers and structural Chrome.
- `@adapttable/angular-unstyled` (`packages/angular/adapter-angular-unstyled`)
  and `@adapttable/ng-zorro` (`packages/angular/adapter-ng-zorro`) provide
  native HTML and NG-ZORRO controls through that binding.
- `packages/angular/ai-angular` adds optional agent, assistant and speech
  bindings over neutral `@adapttable/ai`.
- `@adapttable/angular-unstyled`, `@adapttable/ng-zorro` and
  `@adapttable/ai-angular` are prepared for their first public `0.1.0` release.
  npm publication is a separate owner-controlled step; publishable manifests
  and implemented support do not imply registry availability.

## Product decisions — settled, do not reopen

- Core is headless; adapters own appearance. Chrome + slots (above) is the
  settled widget architecture — do not reopen core-drawn controls or per-kit
  copies of the layout.
- Junior-friendly, senior-targeted: easy defaults for everyone, full control
  for experts — never trade one for the other.
- One word per concept: `server` is the remote-tier word, `useQuerySource`
  the query-library hook. No synonyms, no parallel callbacks, no
  config-object API.
- The showcase is the only demo app. Competitor comparisons stay factual and
  dated.
- Everything is opt-in: omitting a prop renders nothing and costs nothing.
- The table never owns the data. Every write — edits, adds, deletes,
  reorders — is a callback to the host; the table asks, the host does.
- `packages/react/adapter-bootstrap` is private and unfinished. It is not
  published, and its `private` role in the kit registry (`scripts/kits.mjs`)
  keeps it out of the kit contracts — the parts-parity check, the
  conformance suite and feature parity. Its gaps are known: do not raise
  them as findings, add it to those contracts or bring it to parity unless a
  task names it.

## Process

- Branch off `main` for every change (`feat/…`, `fix/…`, `chore/…`); merge
  via PR. Never commit directly to `main`.
- One concern per commit. A commit that belongs to an issue names it with
  `Refs #N` in its body; the pull request body carries one `Closes #N` line per
  issue it completes, so the closing list is reviewed — and corrected if
  needed — before merge. A bare `(#N)` in a subject links but never closes. An
  issue closes when the PR that lists it merges: work that is only committed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orwa-mahmoud/adapttable](https://github.com/orwa-mahmoud/adapttable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
