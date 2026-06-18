---
trigger: always_on
description: This file is the loaded guidance for agents working in this repository. Treat
---

# Kandelo Agent Guidance

This file is the loaded guidance for agents working in this repository. Treat
it as a contract router: it names the platform contracts that must be preserved
and points to the focused guidance and reference docs that carry the full
detail.

Kandelo is a POSIX-compatible multi-process kernel for WebAssembly. The project
is built around contracts between the platform and processes, between hosts and
the kernel, between packages and the build system, and between the platform and
users. When in doubt, identify which contract a change touches before editing.

If work touches a contract below, read the linked agent guide before making
substantive edits. The short rules here are the invariants agents must keep in
working memory; the linked files hold the operational detail.

## Platform Values Contract

Kandelo's north star is complete POSIX conformance for the system surface it
exposes. Known gaps and browser-imposed limits must be documented as gaps or
boundaries, not hidden behind package- or demo-specific behavior. Missing or
incomplete POSIX behavior is a platform gap to close, not permission to weaken
the model or add software-specific behavior.

Correct behavior must come from accurate internal system state. Process tables,
file descriptors, address spaces, signals, VFS metadata, devices, networking,
packages, and host adapters should reflect the real state of the system. Do not
shape terminal output, preset behavior, UI state, wrappers, or package scripts
to create the appearance of correctness when the underlying platform is wrong
or incomplete.

Prefer truthful failure over convenient illusion. A missing feature, failed
build, unsupported syscall, unavailable host capability, ABI mismatch, or stale
artifact should be visible as the real boundary it is, not disguised as success
through special-case behavior.

User software should build and run through the normal platform path: SDK, libc,
package resolver, VFS image, syscalls, host runtime, and kernel. A fix that
only makes one program, demo, package script, or button work is suspect unless
the special case is explicitly the product behavior being implemented.

When user software fails to build or run, first ask what the failure reveals
about Kandelo. Is a POSIX API missing? Is a syscall semantically wrong? Is
libc or the SDK misrepresenting the target? Is package resolution incomplete?
Is the VFS image wrong? Is Node/browser behavior diverging? Treat porting
failures as platform feedback before treating them as package quirks.

Workarounds are acceptable only at explicit compatibility boundaries: upstream
defects, browser sandbox limits, host platform constraints, unavailable
hardware capabilities, or intentionally unsupported behavior. A workaround must
document the boundary it belongs to and must not hide a platform defect.

Demos are consumers of Kandelo, not alternate implementations of Kandelo. Demo
code must not bypass, simulate, or paper over runtime, VFS, package, libc,
syscall, kernel, or host defects.

## Agent Work Loop

For nontrivial work:

1. Identify the contract touched.
2. Read the local implementation, the focused agent guide, and the relevant
   reference docs.
3. Trace root causes through the normal platform path.
4. Preserve Node.js/browser parity unless a documented platform boundary says
   otherwise.
5. Update the authoritative docs for any changed behavior.
6. Run validation that supports the exact claim you will make.
7. Report what changed, what was run, and what was not run.

Do not stop at code reasoning for browser demo bugs, syscall semantics, ABI
changes, package artifacts, or performance claims.

## Contract Map

| Contract | Agent guide | Reference docs |
|---|---|---|
| Validation and completion claims | `docs/agent-guidance/validation.md` | `docs/repository-organization.md` |
| Debugging, POSIX, process, VFS, devices | `docs/agent-guidance/debugging-and-posix.md` | `docs/architecture.md`, `docs/posix-status.md` |
| ABI versioning and snapshot policy | `docs/agent-guidance/abi.md` | `docs/abi-versioning.md`, `docs/fork-instrumentation.md` |
| Host runtime and Node/browser parity | `docs/agent-guidance/host-runtime.md` | `docs/architecture.md`, `docs/browser-support.md` |
| Package schema, builds, resolver, cache | `docs/agent-guidance/packages-and-builds.md` | `docs/package-management.md`, `docs/binary-releases.md`, `docs/porting-guide.md`, `docs/sdk-guide.md` |
| Browser demos, VFS images, sharing, users | `docs/agent-guidance/browser-and-user.md` | `docs/browser-support.md` |
| Performance claims and benchmarks | `docs/agent-guidance/performance.md` | `docs/profiling.md` |
| Dev shell, docs, PR/final reports | `docs/agent-guidance/build-docs-and-prs.md` | `docs/repository-organization.md`, `README.md` |

## Validation Contract

Validation is evidence for a specific claim. Do not say "tests pass", "the
branch is complete", "the browser works", "ABI is fine", or "performance
improved" unless the evidence for that exact claim has been run and reported.

Do not use a narrow check to support a broad claim. A passing unit test does
not prove POSIX behavior. A passing Node/Vitest path does not prove browser
behavior. A passing browser demo does not prove ABI compatibility. A

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Automattic/kandelo](https://github.com/Automattic/kandelo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
