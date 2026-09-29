---
trigger: always_on
description: The active codebase is entirely Python.
---

# Repository Instructions

The active codebase is entirely Python.
Before starting implementation work, update or read the relevant docs first so the intended public behavior, ownership rules, and limitations are explicit; then implement code and tests to match that documented contract.

Update `CHANGELOG.md` under **Unreleased** whenever a change adds or changes
user- or maintainer-visible behavior, public APIs, supported features,
examples, build or CI workflows, benchmark methodology, or documented
limitations. Keep entries concise and outcome-focused; do not add release
notes for internal cleanup that has no visible effect.

Write user documentation as a concise guide to the current product. Lead with
the task a user wants to complete, show the necessary command or example, and
state only the behavior, choices, and limitations needed to use it correctly.
Do not narrate implementation history, prior bugs, rejected designs, internal
mechanics, defensive checks, or why a newly added behavior differs from an old
one unless that context changes what the user must do. Integrate changes into
the existing workflow instead of appending a change report, and remove any
sentence whose only purpose is to justify the implementation or record the
development process.

Treat developer documentation as durable guides, not as per-change
implementation logs. Do not update developer pages merely because code changed,
and do not add incidental low-level details that are unnecessary for following
the documented architecture or maintainer workflow. Update them only when a
documented contract, ownership boundary, workflow, or limitation changes; keep
routine implementation findings in the review summary or a concise CHANGELOG
entry when appropriate.

Ignore:
- *.f90
- *.f95
- *.for
- *.c
- *.h
- *.json

Do not spend context window or analysis on those files unless explicitly requested.
Keep one path for one question. When two entry points answer the same
question -- one file and a project, a library route and its CLI wrapper, source
discovery and compile ordering, a source build and a contract replay -- they
must call the same owner and differ only in the inputs they pass, such as which
files or modules are in scope. Do not write a second loop, list, inventory,
regex, lexer, or conversion route that re-derives what an existing owner
decides, even as a fast path: a fast path may narrow what the owner reads, but
the owner's answer stays the only answer. Before adding a helper that
enumerates or classifies something -- a module's procedures, a file's program
units, a `use` nature, the intrinsic modules, Fortran source suffixes, a
submodule's identity -- find the existing owner and extend it. When two copies
are found, merge them into one owner instead of fixing only the copy that
failed, and prove the merge with a test that runs both entry points on one
input and compares their results.

When asked to change or move an API, import path, command, feature, or behavior, do not add or keep compatibility layers, aliases, shims, fallback paths, or legacy entrypoints unless explicitly requested. A requested change means the old behavior should be removed.
When updating tests, remove obsolete tests that only assert removed/old implementation behavior does not exist. Do not preserve rejection or absence checks for API/features that were intentionally removed unless explicitly requested.
Do not add tests whose purpose is only to prove that removed or nonexistent features are rejected. Test supported behavior and meaningful validation boundaries instead. For example, if `ArrayCategory` is removed, delete its tests; do not add a test asserting that `ArrayCategory` now fails.

Optimize the test suite for maximum confidence per test and minimum
maintenance burden, not for test count. Treat end-to-end tests as the primary
proof that a feature works: where practical, demonstrate a feature through the
real workflow (source, preprocessing, parsing, semantic IR, `.pyi` contract,
replay or build, generated wrapper, compile and link, import, runtime call) and
finish by checking a concrete, repeatable result such as runtime values, native
state, generated contract or source text, or the native build plan. One strong
end-to-end test that covers several cooperating features should replace
several lower-level tests that only repeat pieces of the same behavior.

Delete a test, rather than preserve it because it exists, when its only
purpose is to check implementation details, trivial getters, constructors,
dataclass fields, or plumbing; to repeat behavior a stronger end-to-end test
already proves; to assert an intermediate object only because it currently
exists; to test a tiny helper that is exercised thoroughly elsewhere; to repeat
one case at several stages; to lock internal architecture without protecting
user-visible behavior; or to add near-identical permutations that do not
represent distinct failure modes.

Keep a focused isolated test only when it is the cheapest or clearest way to
protect a boundary that end-to-end tests do not cover economically, and when
it has a clear answer to: **what realistic regression does this catch that
would otherwise be difficult, expensive, or ambiguous to detect?** Typical

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PyNumLab/prik](https://github.com/PyNumLab/prik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
