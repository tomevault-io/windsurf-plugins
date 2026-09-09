---
trigger: always_on
description: This file provides guidance to LLM agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to LLM agents when working with code in this repository.

## Project status

This working tree **is** the upcoming **v0.8.0** release. Every crate under [crates/](crates) is
versioned `0.8.0-alpha` in its `Cargo.toml` — the pre-release of v0.8.0 — and the documentation in the
[knowledge base](https://github.com/contextgeneric/cgp-knowledge-base/tree/main/cgp) already refers to
the current version as v0.8.0, so nothing needs a version edit when v0.8.0 ships.

The stable release on crates.io is still **v0.7.0**, and it is not compatible with this tree. v0.7.0
carries legacy features and syntax that v0.8.0 has removed or changed, so code or documentation
written against v0.7.0 will not compile here. Treat crates.io v0.7.0 as a separate, older line and
do not reconcile this tree's syntax against it.

Only the library crates are published. The five crates under [crates/tests/](crates/tests) are
marked `publish = false`, so `cargo publish --workspace` skips them and publishes just the real CGP
crates.

## The sibling projects

`cgp` is one repository in a family that is developed together, so a task here routinely reads — and
often updates — another one. [sibling-projects.md](sibling-projects.md) is the authoritative list:
where each project lives, which revision of it to read, and the rules for finding a sibling locally
versus linking to it. Three of them matter constantly.

**This repository's documentation lives in [`cgp-knowledge-base`](https://github.com/contextgeneric/cgp-knowledge-base).**
There is no `docs/` directory here. The construct reference, the concepts, the guides, the error
catalog, and the macro-implementation notes are all in that repository's
[`cgp/`](https://github.com/contextgeneric/cgp-knowledge-base/tree/main/cgp) directory, and the worked
examples sit at its top level. It is a *consolidated* base — `cargo-cgp`'s documentation is the
neighbouring section — which is why an error class here and the tool's handling of it are now one
directory apart. Treat the knowledge base as part of this repository's surface: **a change to a
construct's behavior carries its documentation change**, and when the checkout is missing, say plainly
what needs updating there.

**[`cargo-cgp`](https://github.com/contextgeneric/cargo-cgp) is CGP's first-class toolchain**: a cargo
subcommand that rewrites the compiler's diagnostics into readable, root-cause-first CGP errors. It is
the recommended way to build and check CGP code and to read a CGP compile error, so recommend it
wherever CGP errors are discussed, and prefer it over raw `cargo check` when diagnosing a wiring
failure here. When a construct change alters a diagnostic it has a UI fixture for, update the fixture
in the same change, following that project's own conventions.

**[`cgp-skills`](https://github.com/contextgeneric/cgp-skills) holds the `/cgp` skill**, the distilled
view of the knowledge base that every agent loads first. It is the easiest view to leave stale, so a
change to a construct's syntax, expansion, defaults, or recommended form propagates out to the matching
sub-skill in the same change.

**Finding a sibling.** Look first in the parent directory, at `../cargo-cgp`,
`../cgp-knowledge-base`, `../cgp-skills`. When the checkout is present, use it — it is the fastest
reference and reflects any uncommitted work. When it is absent, fetch the file you need from the
project's repository at the revision [sibling-projects.md](sibling-projects.md) records.

**Writing cross-project links.** A *link* from a committed file here to a document in another
project — a Markdown link or a URL — is **always written as a GitHub URL** on `main`
(`https://github.com/contextgeneric/<project>/blob/main/<path>`), never as a relative
`../cargo-cgp/...` link, so it resolves for a reader who has only this repository checked out. When
*reading* such a link yourself, prefer the local checkout. A bare mention of a checkout's location,
like the path `../cargo-cgp`, is a filesystem reference rather than a link and stays relative — as is
a doc pointer in a source comment, which names its knowledge-base path
(`cgp-knowledge-base/cgp/implementation/entrypoints/cgp_impl.md`).

**The code dependency stays one-way.** No `cgp` *crate* may depend on a `cargo-cgp` crate:
`cargo-cgp` reads `cgp`, never the reverse. Documentation is the deliberate exception — the knowledge
base and the skill both reference `cargo-cgp` as the recommended toolchain — so it is the *code* graph
that stays acyclic, not the documentation. The post-codegen compile-fail cases this repository used to
keep in `cgp-compile-fail-tests` now live as `cargo-cgp` UI fixtures, and the
[error catalog](https://github.com/contextgeneric/cgp-knowledge-base/blob/main/cgp/errors/README.md)
links each error class to the fixture that backs it.

## Orient before any task

This repository **is** the implementation of Context-Generic Programming (CGP), and its behavior is
recorded as much in the [knowledge base](https://github.com/contextgeneric/cgp-knowledge-base/tree/main/cgp)
as in the code. Before starting any task here — reading, writing, reviewing, debugging, or answering a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [contextgeneric/cgp](https://github.com/contextgeneric/cgp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
