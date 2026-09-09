---
trigger: always_on
description: Agent/developer operating rules for the **start-technologies monorepo root**. This repo is the monorepo for all Start9 products. `CLAUDE.md` is a one-line `@AGENTS.md` import — do not edit it.
---

# AGENTS.md

Agent/developer operating rules for the **start-technologies monorepo root**. This repo is the monorepo for all Start9 products. `CLAUDE.md` is a one-line `@AGENTS.md` import — do not edit it.

See [ARCHITECTURE.md](ARCHITECTURE.md) for the layout and [CONTRIBUTING.md](CONTRIBUTING.md) for the build/test/format workflow.

**Keep these docs current.** Every scope carries `AGENTS.md` / `ARCHITECTURE.md` / `README.md` (and `CLAUDE.md`, a one-line `@AGENTS.md` import). When a change alters structure, conventions, the build/test/release flow, or product context, update the matching doc(s) in the **same change** — never defer. These docs are **hierarchical**: each scope's docs cover only what is specific to it and must not repeat anything already stated at a higher scope (e.g. commit/PR conventions live only in this root `CONTRIBUTING.md`).

**Anything an agent must follow belongs _in_ `AGENTS.md`, not behind a link from it.** The `AGENTS.md` standard is plain Markdown with **no import syntax** — unlike `CLAUDE.md`, whose `@file` imports expand into context at launch. So a pointer in `AGENTS.md` ("see X for the release process") is just a suggestion that the agent spend a tool call, and agents routinely don't: that is exactly how start-sdk 2.0.4 and 2.0.5 shipped untagged. Inline the rule; link only to _reference_ material an agent can safely skip (`ARCHITECTURE.md`, a product book). `CLAUDE.md` stays a one-line `@AGENTS.md` import — Claude Code does not read `AGENTS.md` natively.

**So `CONTRIBUTING.md` is being folded into `AGENTS.md` — in every scope _but_ the root.** A sub-scope `CONTRIBUTING.md` earns nothing (GitHub gives it no special treatment) and costs an agent a hop, so its contents belong in that scope's `AGENTS.md`. When you next do substantial work in a scope that hasn't migrated, fold its `CONTRIBUTING.md` into its `AGENTS.md`, delete it, and repoint every inbound link in the same change — don't leave dangling cross-references, and don't migrate scopes you aren't otherwise touching. **The root [`CONTRIBUTING.md`](CONTRIBUTING.md) stays**: GitHub surfaces it (the contributing prompt on new issues/PRs, the community profile), so it remains the human-facing front door — the social layer, and a pointer _to_ `AGENTS.md` for the mechanics. That pointer runs one way; a human will follow a link, an agent won't.

**A product's user docs and changelog ship with the code.** Any change that alters user-visible behavior **must** update that product's user-facing documentation book (its `docs/` directory — e.g. `projects/start-os/docs/`, `projects/start-tunnel/docs/`, `projects/start-sdk/docs/`) in the **same change**, and **must** add a `CHANGELOG.md` entry for that product (a version bump always pairs with its changelog). Don't land code and defer its docs or changelog to a follow-up. The conventions for authoring **any** of those books — mdBook versions, admonitions, tabs, `SUMMARY.md`, the shared `theme/` — live in [`projects/start-docs/AGENTS.md`](projects/start-docs/AGENTS.md) and its `CONTRIBUTING.md`. That project is a sibling, not an ancestor, so nothing loads it for you: read it before editing book pages anywhere in the repo.

**The changelog's top heading is the prospective _next_ version, and git tags decide released-vs-unreleased.** **Before deciding where a changelog entry goes, freshly pull tags from origin _first_** — run `git fetch --tags origin` (or query origin live: `git ls-remote --tags origin '<product>/v*'`, `gh release list`), every time; never trust stale local tags, and never infer release state from the changelog file, a `## [x.y.z]` heading, or a manifest constant. Those origin tags (`<product>/v<version>`) are the only source of truth for what has shipped. Keep the top `CHANGELOG.md` heading set to the **actual prospective next version** (e.g. `## [1.1.1]`) matching the product manifest — not a bare `## [Unreleased]` — because the number itself signals the tier of change accumulated (patch/minor/major). When that top version has **no matching origin tag it is unreleased: add your entry _under_ it** (in the right `### Added`/`### Changed`/`### Fixed`/`### Security` subsection), and raise both the heading and the manifest a tier only if your change warrants it (a fix leaves an accumulating `1.1.1` alone; a breaking change bumps it to `2.0.0`). **If your change fixes or refines a feature that was _added in that same still-unreleased version_, edit that feature's existing entry** (only where its wording needs it) rather than adding a separate `### Fixed` line — to the user the feature simply ships correct, so there is no fix to a thing they never received. **Only cut a _new_ heading — and bump the manifest — once the current top heading is a cut origin tag** (that line is fully released). The release tooling turns that prospective heading into the shipped one when it cuts the tag.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Start9Labs/start-technologies](https://github.com/Start9Labs/start-technologies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
