---
trigger: always_on
description: This repository is product-neutral UI infrastructure. Components may depend on
---

# GPUI Box contributor guidance

## Boundaries

This repository is product-neutral UI infrastructure. Components may depend on
GPUI, tokens, theme, assets, and semantic testability. They must not depend on
application hosts, databases, credentials, RPC transports, or product models.

Components:

1. read caller-owned data;
2. emit caller-owned actions;
3. hold only visual transient state such as hover, focus, open, selection, and
   animation.

## Framework infrastructure

Do not hide a missing GPUI primitive behind a component-specific workaround.
When a requirement is product-neutral, will be reused by more than one
component, or must coordinate rendering, layout, clipping, hit testing, input,
or platform behavior, first verify whether the GPUI Box framework
already provides it. If it does not, implement the smallest complete primitive
at the framework boundary. GPUI Box is the sole development authority; implement
framework and platform changes directly here and never restore a Zed Cargo Git
dependency or continuous source synchronization.

Keep product and component policy in this repository. Node routing, port
meaning, semantic ids, and caller-owned events belong to GPUI Box Kit; generic
subtree transforms, pointer capture, renderer behavior, and platform event
delivery belong to the framework. Do not add a partial framework API that works for one
primitive or platform while leaving its layout, clipping, accessibility bounds,
or hit testing inconsistent.

A framework infrastructure change must:

1. be product-neutral and documented at the primitive boundary;
2. carry focused GPUI tests, including the affected platform-independent input
   or rendering invariants;
3. preserve the frozen historical import receipt without treating it as an
   update lane;
4. keep the root and `tools/headless-visual` workspaces on the same local GPUI
   Box package authority, without Git sources or `[patch]` overrides;
5. update `PROVENANCE.md`, `THIRD_PARTY_NOTICES`, and compatibility documentation;
6. pass `cargo run -p xtask -- dependencies check` and the relevant Linux,
   macOS, and Windows validation.

Local geometry remains appropriate when it occurs once and does not create a
second implementation of a renderer or input primitive. Record any deliberately
deferred framework gap in coverage documentation rather than presenting a local
approximation as complete support.

## Token authority

`crates/gpui-kit-tokens/tokens/*.json` is the source of truth, and every theme carries the same key
set. Repeated semantic color,
spacing, radius, typography, motion, and effect values belong there. Local
geometry that occurs once may stay next to the component.

After token changes:

```bash
cargo run -p xtask -- tokens generate
cargo run -p xtask -- tokens check
```

Do not hand-edit `docs/token-reference.md`.

## Truthful UI

- Loading, Empty, Unavailable, Error, and Ready are distinct states.
- A refresh failure keeps the last verified value visible.
- A disabled control must not install its action handler.
- Host refusals are displayed as refusals, not converted to empty data.
- Fixtures and product-backed data must be explicitly distinguishable.

## Testability

Every user-visible action and assertion target needs a stable semantic id.
IDs derive from business identity, never list position. Bounds are measured
during prepaint. Never put credentials or unredacted user-generated content in
semantic snapshots.

Tests assert behavior and generated artifacts, never source text.

## Provenance

Source ports and translations must update `PROVENANCE.md` and
`THIRD_PARTY_NOTICES`. Preserve upstream copyright notices and exact revisions.
Do not add product or provider trademarks to the generic asset crate.

## The generated API index

`docs/api-index.json` carries every component, the exact signature of every
public method, what each reports, and the scenes that review it. It is
generated from the source, so it is the answer when it and any prose disagree:

```bash
cargo run -p xtask -- api generate
cargo run -p xtask -- api check     # runs inside `gate`
```

A new component appears there by existing. Adding one and leaving the index
stale fails the gate, which is the point: a reader who is told a signature that
does not compile was failed by this file, not by the compiler.

## A component is reviewed by an exhibit

A rendering in `crates/gpui-kit/src/scenes/` declares what it is for, and the
declaration is checked:

- `Shows::Subjects` names the components the rendering is *about*. It lives in
  the file for those components' family, and it is where a reader is sent to
  review their states.
- `Shows::Composition` is an arrangement built the way a product would build
  one, kept because components interact in ways none of them shows alone. It
  is nobody's coverage.

Three things fail `api check`:

1. a public component with no exhibit anywhere;
2. an exhibit whose own source never builds a component it claims to review —
   a component drawn inside another component has been recognised, not
   reviewed, and the picture that would fail when its states change belongs to
   whatever mounted it;
3. a declaration naming something the rendering cannot reach at all.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fran0220/gpui-box](https://github.com/fran0220/gpui-box) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
