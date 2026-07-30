---
trigger: always_on
description: Instructions specific to rootcause documentation
---


# Documentation Style Guide for Rootcause

This document establishes consistent standards for documentation across the rootcause library.

## Core Philosophy

Documentation length tracks the amount of non-obvious information, not the item's visibility. `Vec::push` in std is two lines plus an example; the `Vec` type-level doc is an essay. Follow the same shape here: rich narrative at the crate/module/type level, lean contracts at the method level.

The guiding principles:

1. **Every sentence must beat the signature.** A doc sentence earns its place by telling the reader something they cannot see from the item's name and types. "Converts the error into a [`Report`]" on `fn into_report(self) -> Report<E>` fails this test. If nothing non-obvious remains, a single crisp summary line is a complete docstring.
2. **Document the contract, not the paraphrase.** Method prose is for what the signature can't say: allocation and cloning semantics (Arc refcount bump vs deep copy), whether hooks run, `#[track_caller]` location capture, effect on children and attachments, formatting interplay, panics.
3. **Push shared explanation up; keep siblings thin.** A method family (e.g. the `attach*` methods, the [`OptionExt`] methods, the `into_*` conversions) gets one thorough trait- or module-level explanation with when-to-use-which guidance. Each sibling then gets a one-liner plus a link. Never stamp a doc template across siblings: visible repetition trains readers to skip docs.
4. **Every example must assert something.** See [Example Standards](#example-standards).
5. **Right layer, no repetition.** Crate docs teach the problem and the mental model; module docs explain the subsystem's role and lifecycle; item docs state the contract. Don't restate a higher layer in a lower one.
6. **State behavior positively.** Say what the code does. Drop clauses that justify what it doesn't do ("...rather than exposing it via the source chain"). Don't add `compile_fail` doctests to assert negative type contracts; positive type contracts belong in compile-time unit tests (e.g. an `assert_send_sync` helper). In conceptual docs such as the [`markers`] module, a compile-fail example is acceptable only where the failure itself is the lesson being taught.
7. **Direct sentences, no filler.** Write "Allocates a new root node containing the context.", not "This allows you to...", "You can use this to...", or similar scaffolding.

## Documentation Depth by Visibility

### Public Items (`pub`)

- **Required** for all public items.
- A summary line stating the contract; further prose only where the contract has non-obvious parts (see principle 2).
- An example that asserts concrete behavior (see [Example Standards](#example-standards)).
- Panics, errors, and safety sections when applicable.
- Cross-references to the family-level explanation and closely related items.
- Target audience: library users.

### Internal Items (`pub(crate)`, private, or private modules)

- **Optional**: add documentation when it meaningfully helps readers understand the implementation.
- Brief and concise: what it does and why it exists.
- No examples required.
- Target audience: library developers and contributors.

## Structure Patterns

### Crate- and Module-Level Documentation (`//!`)

1. **Hook line** (1-2 sentences): what this module/crate does
2. **Overview**: the mental model and how this subsystem fits into the whole
3. **Core concepts** (if complex): key ideas, decision guidance (when to use which variant), lifecycle (e.g. when hooks run and what they see)
4. **Usage examples**: realistic multi-step scenarios (this is where the rich examples live, not on individual methods)
5. **Cross-references**: link to related modules/types

### Item-Level Documentation (`///`)

1. **Summary line**: one sentence stating the contract
2. **Detailed explanation** (only if the contract has non-obvious parts)
3. **Example**: a concrete behavioral assertion
4. **Errors/Panics/Safety** (if applicable)
5. **See also**: link to the family-level explanation where one exists

## Language Conventions

### Terminology Consistency

- **"Report"** (capitalized) when referring to the type
- **"report"** (lowercase) when referring to an instance
- **"context"** for the root node's data
- **"attachment"** for additional data added to nodes
- **"attachment data"** for the actual data stored in attachments
- **"handler"** for types that process contexts/attachments
- **"hook"** for customization points in the reporting process

### Code References

- **Use intra-doc links for types**: [`Report`], [`Error`] (not plain `Report` or `Error`)
- **Use intra-doc links for methods**: [`Report::new`], [`into_dyn_any`] or [`into_dyn_any()`]
- **Use intra-doc links for modules**: [`crate::handlers`]
- **Use full paths for external crates**: [`std::error::Error`]
- **Especially important for internal references**: Always use [`ReportRef`], [`ReportMut`], [`Cloneable`], etc. rather than plain backticks
- **Exception for well-known standard library types**: Don't use intra-doc links for `String`, `Vec`, or other ubiquitous standard library types.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rootcause-rs/rootcause](https://github.com/rootcause-rs/rootcause) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
