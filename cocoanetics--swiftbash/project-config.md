---
trigger: always_on
description: A pure-Swift, sandboxed bash interpreter for embedding in Mac / iOS
---

# Agent instructions

A pure-Swift, sandboxed bash interpreter for embedding in Mac / iOS
apps. No `Process`, no `fork`, no `exec` — every command is a Swift
type, every byte streams through `AsyncStream<Data>`, every host
axis (FS / network / processes / identity) is virtualised.

Read [README.md](README.md) for the user pitch, then
[Docs/InterpreterArchitecture.md](Docs/InterpreterArchitecture.md)
for how the pieces fit together.

## Commands

- `swift build` — compile.
- `swift test --no-parallel` — full suite. **Always run before
  committing.** Currently ~1780 tests; failures from your change
  are the first signal something's off.
- `swift test --no-parallel --filter <name>` — single suite.
- `swift run swift-bash exec [--sandbox PATH] /dev/stdin <<<'…'`
  — fastest way to verify behaviour end-to-end during development.

## Where things live

```
Sources/BashSyntax/          Parser + tokenizer + AST. No interpreter deps.
Sources/BashInterpreter/
  API/                       Shell, Environment, Command, FileSystem, BinCatalog
  Builtins/                  cd, echo, export, declare, source, …
  Execution/                 Shell+Run, Shell+Pipeline, Shell+ControlFlow, …
  FileSystems/               Real, InMemory, SandboxedOverlay, VirtualBin
  Network/                   curl plumbing + URL allow-list
  Arithmetic/                ((…)) lexer + Pratt parser + evaluator
Sources/BashCommandKit/
  API/                       ParsableBashCommand bridge
  Commands/                  ls, cat, grep, sed, awk, find, jq, curl, …
Sources/swift-bash/          CLI (`exec`, `parse`)
Tests/                       One suite per source module
Docs/                        Architecture, sandboxing, networking, etc.
```

One declaration per file. `Type+Concern.swift` extensions for
splitting big types — `Shell` lives in `API/Shell.swift` plus ~15
`Shell+*.swift` extension files.

## Conventions to follow

- **Reach shell state via `Shell.current`.** Don't add `shell:`
  parameters to functions. `Shell.current` is a `@TaskLocal`,
  propagated automatically across pipelines, subshells, and child
  Tasks. Adding a parameter to thread "the current shell" is
  almost always wrong.
- **New shell-scoped option? Add it to `Shell.copy()`.** That's
  the single source of truth for what propagates into subshells
  and pipeline stages. Forgetting to add a field there means it
  silently doesn't inherit, which is a real footgun.
- **New command? Decide built-in vs kit.**
  - Pure shell semantics (manipulates env, control flow, registry):
    `BashInterpreter/Builtins/`, conform to `Command`.
  - System-style tool (operates on files, text, network):
    `BashCommandKit/Commands/`, conform to `ParsableBashCommand`,
    register in `Shell+StandardCommands.swift`.
- **File-shadowed commands need a `BinCatalog` entry.** Update
  `Sources/BashInterpreter/API/BinCatalog.swift` so the command
  appears under `/bin/<name>` or `/usr/bin/<name>` and `which`/
  `type` resolve correctly.
- **Long-running loops must call `try Task.checkCancellation()`
  each iteration.** `kill PID` against background jobs depends on
  cooperative cancellation. Loops that don't check are uninterruptible.
- **Non-local exits use thrown errors, not return-value flags.**
  Existing patterns: `ShellExit`, `ReturnSignal`,
  `LoopControlSignal`. Don't invent `enum Result { case ok, exit,
  returned }`-style returns.
- **File system writes go through `Shell.current.fileSystem`.**
  Never `FileManager.default.createFile(...)` from a command.
- **All paths to the FS are absolute.** Convert via
  `Shell.current.resolvePath(_:)` first. Relative paths reaching
  the FS layer are a bug.
- **Identity-revealing commands defer to `Shell.current.hostInfo`.**
  Never call `getuid()`, `gethostname()`, `uname()`, etc. directly
  — that leaks the real machine even in sandbox mode.

## Test discipline

- **Add a test in the same commit as the behaviour change.** The
  full suite is fast (~5s); leaving regressions invisible until
  later is more expensive than writing the test now.
- **Suite-per-source-file convention.** A new `FooCommand.swift`
  ships with `FooCommandTests.swift` next to the related ones.
- **Use `CapturingShell` for stdio assertions.** It wires fresh
  in-memory `OutputSink`s to a `Shell` and exposes `cap.stdout` /
  `cap.stderr` strings.
- **Assert full output rather than `.contains(...)` where possible.**
  Substring assertions hide whitespace bugs and ordering issues
  that exact-string assertions catch.
- **Sandbox tests use `SandboxedOverlayFileSystem`** with a real
  temp dir as root and `mountPoint: "/batch"`. Clean up the temp
  dir in `defer`.
- **Never re-test the parser from interpreter tests.** Parser
  behaviour belongs in `BashSyntax` tests; interpreter tests
  exercise execution against pre-parsed shapes.

## Things to avoid

- **Don't call `Process` / `fork` / `exec` / `posix_spawn`.** The
  interpreter is in-process by design. If you need parallelism,
  use `Task` / `TaskGroup` and route I/O through `OutputSink` /
  `InputSource`.
- **Don't read `ProcessInfo.processInfo.environment` from
  commands.** That's the host process's env — bypasses
  `Shell.current.environment` entirely. The CLI does this *once*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cocoanetics/SwiftBash](https://github.com/Cocoanetics/SwiftBash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
