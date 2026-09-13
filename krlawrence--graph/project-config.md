---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md

Guidance for AI agents working in this repository.

## What this repository is

This repository is the source for the book **"Practical Gremlin: An Apache TinkerPop
Tutorial"** by Kelvin R. Lawrence and Stephen P. Mallette. It is a book, not a
software project. The primary deliverable is prose. Code exists here in service of
the prose - as examples in the manuscript, as runnable samples under `sample-code/`,
and as the build/validation tooling under `bin/`.

The current work is the **second edition**, which lives on the `main` branch and
targets the Apache TinkerPop release named by the `:tpvercheck:` attribute in
`book/Practical-Gremlin.adoc` (see "Technical accuracy" below). The first edition is
archived on the `v1` and `first-edition` branches and must not be edited.

Treat the book as a *living book*: it is published continuously from `main`, so every
commit should leave the manuscript in a publishable state.

## Repository layout

| Path | Contents |
| --- | --- |
| `book/` | The AsciiDoc manuscript. This is where nearly all editing happens. |
| `book/Practical-Gremlin.adoc` | Master document: attributes, title, and `include::` list. |
| `book/Section-*.adoc` | One file per chapter, included by the master document. |
| `bin/` | Build and validation scripts. |
| `bin/llms/` | Ruby scripts that produce the Markdown mirror and `llms.txt`. |
| `sample-code/` | Runnable samples: `dotnet`, `golang`, `groovy`, `java`, `javascript`, `python`, `ruby`. |
| `sample-data/` | The `air-routes` data set and other sample graphs, in several formats. |
| `make-route-graph/` | Tooling that generates the `air-routes` data set. |
| `demos/` | Standalone vis.js visualizations of the graph. |
| `images/` | Cover art and figures. |
| `target/` | Build output. Git-ignored, never commit it. |

## Building the book

The build depends on Asciidoctor (Ruby), and optionally on Pandoc, Calibre, and
Node.js for the additional formats. Run everything from the **repository root**.

```shell
# Full build: validation, then HTML, DocBook, EPUB, MOBI, PDF, Markdown + llms.txt
./bin/make-book.sh

# Just the agent-friendly Markdown mirror and llms.txt index
./bin/make-llms.sh
```

Single formats can be produced directly from the `book/` directory:

```shell
asciidoctor Practical-Gremlin.adoc          # HTML
asciidoctor-pdf Practical-Gremlin.adoc      # PDF
```

`bin/make-book.sh` deletes and recreates `target/`, so do not put anything of value
there. If Node.js is not installed, the Markdown/`llms.txt` step is skipped
gracefully and the rest of the build still succeeds.

CI (`.github/workflows/build-asciidoc.yml`) rebuilds and republishes to GitHub Pages
on every push to `main`, so a broken build is immediately a broken published book.

## Validation - run this before every commit

```shell
./bin/check.sh                 # cross-references + code block formatting
./bin/check.sh --check-urls    # additionally verifies every URL in the manuscript
```

`check.sh` runs two checks, both of which gate CI:

- **`check-refs.sh`** - every `<<anchor>>` cross-reference must have a matching
  `[[anchor]]` definition somewhere in `book/*.adoc`.
- **`check-formatting.sh`** - every `[source,...]` line must be immediately followed
  by a `----` delimiter, and every code block must be closed.

The URL check is slow and network-dependent; run it when adding or changing links.

If you touched the Markdown target, also run `./bin/validate-llms.sh`, which serves
the built site locally and validates it against the
[Agent Friendly Documentation Spec](https://agentdocsspec.com/) with a pinned version
of `afdocs`.

## Technical accuracy: TinkerPop is the source of truth

**Nothing in this book may contradict the official Apache TinkerPop documentation.**
When adding or changing anything about Gremlin semantics, step behavior, defaults,
deprecations, or availability, verify it against the reference documentation first.

The book is bound to one TinkerPop release, recorded in the `:tpvercheck:` attribute
in `book/Practical-Gremlin.adoc`. **That attribute is the single source of truth for
the version.** This file deliberately does not restate the number, because a second
copy would silently go stale the moment the book moves to a new release. Read it at
the start of any work that touches version-sensitive material:

```shell
grep '^:tpvercheck:' book/Practical-Gremlin.adoc
```

Then verify against the documentation for *that* release, substituting the value for
`<version>`: `https://tinkerpop.apache.org/docs/<version>/llms.txt`

So when `:tpvercheck:` reads `3.8.2`, the agent-friendly index to consult is
https://tinkerpop.apache.org/docs/3.8.2/llms.txt - that is an illustration of the
substitution, not a pinned value. Always resolve it from the attribute.

Further rules:

- In manuscript prose, write `{tpvercheck}` rather than hard-coding the version
  number, so the text tracks the attribute. Hard-code a version only when stating a
  historical fact that will not move, such as "TinkerPop 3.8.0 was released November
  2025" or a note about when a step first appeared.
- Do not invent steps, options, predicates, or output formats. If a claim cannot be
  verified against TinkerPop's documentation or by actually running the query, do not
  make it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krlawrence/graph](https://github.com/krlawrence/graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
