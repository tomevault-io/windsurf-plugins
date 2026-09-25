---
trigger: always_on
description: Autolith is a small, live, self-modifying Common Lisp agent. This file contains
---

# Repository Guidelines

## Purpose and Sources of Truth

Autolith is a small, live, self-modifying Common Lisp agent. This file contains
its enduring architectural and repository policy. `docs/guide.org` documents
user-visible behavior, `docs/architecture.org` maps runtime and source
boundaries, and tracked source plus behavioral tests are executable truth. Do
not create another omnibus product specification.

Core principles:

- Treat the live Lisp image as the primary runtime while keeping source
  sufficient for a clean rebuild.
- Prefer Common Lisp and focused CLOS protocols over generated scripts,
  parallel type dispatch, duplicated data structures, or overlapping public
  interfaces.
- Keep self-modification explicit, auditable, reconstructible, and confined to
  a small operation set.
- Preserve durable conversations, user state, working generations, and a
  pristine recovery path.
- Use process boundaries for reliability and accidental-damage containment,
  never as a hostile-code security claim.
- Keep platform-specific behavior behind narrow adapters.
- Treat migrations and compatibility readers as temporary release machinery.
  Give them an explicit removal boundary instead of accumulating them forever.

Do not leave TODOs, FIXMEs, stubs, placeholders, or knowingly partial
implementations. If a requirement is genuinely too broad or conflicts with
another requirement, stop and ask.

Supported source-development targets are Linux x86-64, macOS arm64, and
Windows x86-64 on SBCL with a terminal interface, one primary agent, and no
claim of hostile-code sandboxing. Nix builds support Linux x86-64 and macOS
musl, macOS x86-64 and arm64, FreeBSD x86-64, NetBSD x86-64, OpenBSD x86-64,
    and Windows x86-64.

Host differences live behind the platform protocol in `src/core/platform.lisp`
with one adapter per host family; `#+win32` appears only in those adapters, in
`autolith.asd` feature expressions, and in the standalone scripts that run
before the system loads. Windows supports supervised detached sessions and
exact-heap restart checkpoints, while withholding fork-based image saves and
filesystem sockets. Windows commands use the native AppContainer
helper with explicit workspace, private temporary, and read-only tool scopes. Checks
that depend on POSIX facilities go through the test fixture protocol in
`tests/test-support.lisp` and are recorded as skipped where the host lacks
them. Sources and tests alike change the environment through
`platform-setenv` and `platform-unsetenv`, resolve links through
`platform-truename`, and remove trees through `platform-delete-directory-tree`,
never through `sb-posix`, `truename`, or `uiop:delete-directory-tree`
directly: Windows keeps two environments, leaves links unresolved in
`truename`, and refuses to delete read-only files. Names carrying pathname
metacharacters or backslashes belong only behind the `:wildcard-file-names`
fixture.

## Upstream References

When a change depends on established behavior in another agent, inspect a
current upstream checkout outside this Git worktree. Keep reference checkouts
read-only, record the repository URL and exact inspected commit in the review
notes, and refresh the checkout before making claims about current behavior.
References are research inputs, not Autolith dependencies; do not edit them or
copy their architecture wholesale.

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
4. Publish the complete reconstructible state as an immutable private image
   commit, clean-process probe its replay script, and retain it in private Git
   history.
5. Atomically select the private commit and mark the journal entry durable.

Autolith's durable live mutations never patch its own tracked repository;
selected private replay scripts load after the tracked system. The ordinary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lambda-symbolics/autolith](https://github.com/lambda-symbolics/autolith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
