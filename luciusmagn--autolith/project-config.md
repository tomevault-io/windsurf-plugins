---
trigger: always_on
description: Autolith is a small, live, self-modifying Common Lisp agent. Read
---

# Repository Guidelines

## Purpose and Sources of Truth

Autolith is a small, live, self-modifying Common Lisp agent. Read
`docs/autolith-minimal-technical-spec.org` before making architectural changes. The
specification defines product behavior and runtime boundaries; this file
defines repository, Common Lisp, testing, and commit policy.

Do not silently simplify the specification. Do not leave TODOs, FIXMEs,
stubs, placeholders, or knowingly partial implementations. If a requirement
is genuinely too broad or conflicts with another requirement, stop and ask.

The initial target is Linux x86-64 on SBCL with a terminal interface, one
primary agent, and no claim of hostile-code sandboxing.

## Upstream Reference Checkouts

Current upstream source is available in shallow, read-only reference
checkouts outside this Git worktree:

- OpenAI Codex: `/home/mag/common-lisp/frob-reference-sources/codex`
  at `5c19155cbd93bfa099016e7487259f61669823ff`
- Pi Agent Harness: `/home/mag/common-lisp/frob-reference-sources/pi`
  at `5416b1834a28b79800393b7bf792ab36065c49b2`
- OpenCode: `/home/mag/common-lisp/frob-reference-sources/opencode`
  at `9976269ab1accfc9f9dc98a4a688c516934de422`

Use these checkouts to study established agent behavior and implementation
details. They are references, not Autolith dependencies. Do not edit them or copy
their architecture wholesale. Record the inspected commit when a conclusion
depends on upstream behavior, and refresh a checkout before making claims
about current upstream code.

## Architectural Guardrails

- Keep the codebase small and prefer Common Lisp, ASDF, and UIOP for
  filesystem, process, networking, and build work.
- Do not generate ad hoc Python or shell files unless a non-Lisp dependency
  genuinely requires them.
- Keep the stable launcher, mutable active agent, disposable Lisp worker, and
  pristine recovery image as distinct components.
- All `lisp.*` operations run in a separate disposable SBCL worker. They do
  not share heap state with the active agent.
- `self.*` operations act on the active image. Normal `self.*` tools must not
  modify the stable launcher or pristine recovery artifacts.
- Treat process separation as an accidental-damage and reliability boundary,
  never as a security sandbox.
- Keep provider authentication and transport behind a replaceable interface.
  Do not launch or bundle the Codex CLI to implement subscription access.
- Keep credentials out of saved cores, conversation files, and Git.
- Source is authoritative for clean rebuilds. Saved cores preserve exact
  working live states, but never replace tracked source.
- Operate on Lisp forms for durable source edits. Do not use blind regular
  expression replacement of source code.
- Keep platform-specific behavior behind narrow adapters.

For a durable live mutation, preserve the specified order:

1. Journal the intended mutation.
2. Compile and install it in the active image.
3. Run relevant checks.
4. Persist the complete definition to an overlay file under the data root.
5. Mark the journal entry durable.

Autolith's runtime never patches its own tracked repository; overlays load at
startup after the tracked system, and repository changes remain with the
user and their development tools.

Conversation files and mutation journals are append-only sequences of
top-level readable forms. Bind `*read-eval*` to `nil` when reading persisted
conversation data, keep that data portable, and tolerate an incomplete final
form after a crash. Publish checkpoints and manifests atomically. Recovery
must remain possible without loading a damaged active core.

## Package Policy

Use one project package, `#:autolith`. Do not create scoped, subsystem, feature,
file-local, or test packages unless the user explicitly changes this policy.
Split the implementation into focused files while keeping those files in the
single project package. The runtime component boundaries in the specification
are not package boundaries.

- Define the package once and use `(in-package #:autolith)` in project source.
- `:use` only `#:cl`.
- Import individual third-party symbols with `:import-from`; do not wholesale
  `:use` third-party packages.
- Do not introduce packages merely to express internal architecture. Express
  those boundaries with files, functions, CLOS protocols, and clear naming.

## Code Organization

- Keep boot files limited to startup, shutdown, configuration, and component
  wiring. Put substantive behavior in focused implementation files.
- Split code by coherent responsibility. Do not create `misc`, `helpers`, or
  a growing multi-purpose `util` dumping ground.
- Keep utility files single-purpose.
- Preserve public entry points when splitting code, and move one coherent
  concern at a time.
- Group definitions by functionality, not alphabetically.
- Within a file, prefer this order where applicable: types and classes,
  generic functions, methods, public functions, private functions, and
  conditions.

Prefer simple, established, readable solutions. Keep business logic above
low-level mechanics. Prefer small, documented functions even when a helper is
used only once. Use CLOS when it provides a useful semantic protocol instead
of repeating type or state dispatch in `cond` trees.

## Common Lisp Style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luciusmagn/autolith](https://github.com/luciusmagn/autolith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
