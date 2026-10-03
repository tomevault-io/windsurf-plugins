---
trigger: always_on
description: Use the plugin `comment-hygiene@roboz0r` to check for common mistakes in agent source comments.
---

# AGENTS.md

## Agent Source Comments

Use the plugin `comment-hygiene@roboz0r` to check for common mistakes in agent source comments.
If it is not setup in your environment, ask your USER for permission to set it up.

For `///` XML documentation comments, read the `fsharp-xml-docs` skill
(`.claude/skills/fsharp-xml-docs/SKILL.md`) before writing, reviewing or fixing them. It covers the
tag set and ships an audit script (`scripts/audit.fsx`) for FS3390 and convention checks.

## F# semantics — use `fslangmcp`, not grep

The repo ships an `fslangmcp` MCP server (`.mcp.json`, FsLangMCP over FSAC +
FSharp.Compiler.Service). It loads `Ranvier.slnx`, so it sees exactly the projects the
solution references.

Requires the `fslangmcp`, `fsautocomplete` and `fantomas` global tools; `fslangmcp
--bootstrap-tools` installs the pinned set.

If your environment doesn't have them, ask your USER for permission to set it up.

**Answer semantic questions about F# code with it. Reach for grep only for prose, JSON, `.mts`
tooling and other non-F# files.** Textual search in large source bases over-matches badly:
short binding names `create` can recur in thousands of members, and `find` resolves the real symbol instead.

- `find` — definitions and cross-project use sites, each tagged `definition`/`reference` with a
  coverage block. **Run `check` first.** When the workspace does not type-check, `find` returns
  `outcome="not_found"` *with* `coverage.complete: true` — a confidently wrong negative, not the
  indeterminate answer the coverage contract implies. Verified: `find "VirtualFileSystem"` matches
  on a clean tree and reports not_found on the same tree with unrelated compile errors. Never
  conclude "no usages" from a `find` taken while `check` says `errors`.
- `check` — fresh whole-workspace type-check verdict (`clean`/`errors`) with structured
  diagnostics, no build artifacts. Roughly 8s. An incremental `dotnet build Ranvier.slnx` is
  about as fast, so prefer `check` for the structured diagnostics, not for speed.
- `fcs_refactor_impact` — run this *before* changing any public signature. Returns blast radius,
  whether the symbol is public API (i.e. a breaking change), covering tests, and a verify list.
- `fcs_tests_for_symbol` — which tests cover a symbol, resolved to the enclosing Expecto
  `testCase` name. Only finds direct call sites; much of the suite exercises the library
  indirectly through the wire, so an empty result means "not called by name", not "untested".
- `fcs_public_api` — stable-ordered public surface, for diffing API before/after a change.
- `fcs_nuget_types` / `fcs_nuget_members` / `fcs_referenced_symbols` — inspect referenced
  assemblies without unpacking packages.

Pass `projectPath` explicitly on `fcs_*` calls when several agents run at once; caches are keyed
per resolved `.fsproj`. `fcs_dead_code` on some projects is dominated by generated-file
internals — treat its output as candidates to filter, not a work list.

## F# experiments — use `sagefs`; the build CLI stays the gate

The repo ships a `sagefs` MCP server (`.mcp.json`, SageFs live F# REPL with a project loaded)
and its playbook as the `sagefs` skill (`.claude/skills/sagefs/SKILL.md`). Read the skill before
any F# change, then apply the limits below: the skill treats the REPL as the whole inner loop, and
for this layout it is not.

Requires the `sagefs` global tool (`dotnet tool install --global SageFs`) and the .NET 10 SDK.
`sagefs mcp` starts the daemon if one is not running.

If your environment doesn't have it, ask your USER for permission to set it up.

### When to use it

- **Type and overload questions.** `check_fsharp_code` returns compiler diagnostics for a snippet
  in milliseconds. A CE, an overload set or an inferred type is answered by evaluating it and
  printing the result, not by a scratch project and a rebuild.
- **Shape questions.** Build a value (a pipeline, a parse result, a record) and walk it, or reflect
  over a type to read its members, rather than guessing an API.
- **Pure logic and pure tests.** Redefine a function in the session until a case passes, then
  write it to the file. Tests that touch no process, file system path or console run here fine.

### When to use the build CLI instead

- **Anything that starts a process.** Code compiled into the project that uses `use` inside
  `task { }` fails in a session with `MissingMethodException: TaskBuilderBase.Using`: the session
  loads the SDK's `FSharp.Core`, and the project was compiled against the NuGet one
  ([SageFs#141](https://github.com/WillEhrendreich/SageFs/issues/141)). Partas.Build's process
  runner is written that way, so a `run` step fails in the REPL and passes under `dotnet`.
- **Tests that locate files.** `AppContext.BaseDirectory` is SageFs's host directory in a session,
  and `Assembly.Location` is a shadow copy
  ([SageFs#142](https://github.com/WillEhrendreich/SageFs/issues/142)). A helper that walks up to
  the repository root must read an environment variable first, or fail in the REPL.
- **Release-only compiler errors.** The REPL compiles like Debug. An `inline` CE member that
  applies a function-typed alias compiles in Debug and fails in Release with `FS1118`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shayanhabibi/Ranvier](https://github.com/shayanhabibi/Ranvier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
