---
trigger: always_on
description: Before diving in, read [*DESIGN.md*](./DESIGN.en.md) to understand the overall
---

Contributing
============

Before diving in, read [*DESIGN.md*](./DESIGN.en.md) to understand the overall
design, architecture, and roadmap.

This repository uses [mise] as the single entry point for development tools and
commands.

[mise]: https://mise.jdx.dev/


Setup
-----

Install the configured tools:

~~~~ sh
mise install
~~~~

List available project commands:

~~~~ sh
mise tasks
~~~~


Running checks
--------------

Run fast checks suitable for pre-commit hooks:

~~~~ sh
mise run check
~~~~

> [!TIP]
> To register that same task as this clone's Git pre-commit hook:
>
> ~~~~ sh
> mise generate git-pre-commit --task=check --write
> ~~~~

Run the full local verification gate:

~~~~ sh
mise run ci
~~~~

Useful individual commands:

~~~~ sh
mise run fmt
mise run fmt-check
mise run clippy
mise run ci
mise run doc
mise run test
mise run typecheck
~~~~


Conventions
-----------

Avoid relying on globally installed Rust tools or ad hoc command variants when
working on this repository. Add or update a mise task when the project needs a
new repeated development command.

Every new source file (Rust, TypeScript, JavaScript) must open with the GPLv3
notice block:

~~~~ rust
// Gukhanmun: <one-line description of what this file does>.
// Copyright (C) <year>  Hong Minhee
//
// This program is free software: you can redistribute it and/or modify
// it under the terms of the GNU General Public License as published by
// the Free Software Foundation, either version 3 of the License, or
// (at your option) any later version.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License
// along with this program.  If not, see <https://www.gnu.org/licenses/>.
~~~~

Use `//` line comments for both Rust and TypeScript/JavaScript files.  Place
the block before any other content (imports, `#![...]` attributes, module
docs, etc.).

Document every new public Rust API with rustdoc comments. Public documentation
should explain the API's role, important invariants, and where it fits in the
pipeline so that `mise run doc` stays useful as an API review gate.


Writing docs
------------

After editing any Markdown file, format it with `hongdown -w`:

~~~~ sh
hongdown -w path/to/file.md
~~~~

Follow these prose conventions:

 -  Use sentence case for headings and subheadings, not title case.
 -  Avoid em dashes (—); use a comma, semicolon, or rewrite the sentence.
 -  No spaces around slashes (write “input/output”, not “input / output”).
 -  Use italics for file paths and file names (*CONTRIBUTING.md*,
    *src/main.rs*), and for document, section, and book titles (*DESIGN.md*,
    *Rendering modes*).
 -  Wrap dictionary names in italics for English prose
    (e.g., *Standard Korean Dictionary*, *Open Korean Dictionary*) and in
    double angle brackets (《…》) for Korean prose
    (e.g., 《標準國語大辭典》, 《우리말샘》).
 -  Wrap inline code in backticks (`mise run check`, `--flag`).
 -  Use the official spelling of proper nouns exactly.  If unsure, verify on
    the official website (e.g., Node.js, not NodeJS or Node).
 -  Write “hangul” and “hanja” in lowercase; they are common nouns, not proper
    nouns.  Ethnic and language names are proper nouns and take an initial
    capital: Korean, Sino-Korean, Chinese.
 -  When linking between pages of the Rspress documentation site under *docs/*,
    use a relative path ending in the *.md* or *.mdx* extension, optionally
    followed by an anchor (for example,
    `../cli/dictionary.md#building-a-custom-dictionary`).  The explicit
    extension keeps the anchor intact through `hongdown`.


Writing the changelog
---------------------

When adding a feature, changing existing behaviour, or fixing a bug, add an
entry to *CHANGES.md* under the current development version. Documentation-only
changes, including edits under *docs/*, prose in Markdown files, and code
comments, do not need an entry.

Organize entries by the public surface whose users will encounter the change.
Use a level-3 heading named after the Cargo crate or JavaScript package, such as
`### gukhanmun-core` or `### @gukhanmun/wasm`. Repeat a change under every
affected surface, including the crate where the behaviour originates and each
umbrella crate, CLI, binding, or shared type package that exposes it. This
duplication is intentional: a reader should be able to follow only the sections
for the components they use. Changes to internal generators belong under the
public crate whose behaviour or data they alter, not under the generator
itself.

Place an entry directly under the version heading only when it applies to the
workspace as a whole and cannot usefully be attributed to a particular crate or
package. Release-wide packaging and repository infrastructure changes are
typical examples.

Keep each bullet focused on one user-visible change. Split distinct behaviours
into separate bullets when that helps readers find the part relevant to them.
State what changed and how users access or configure it; include implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dahlia/gukhanmun](https://github.com/dahlia/gukhanmun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
