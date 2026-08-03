---
trigger: always_on
description: These conventions apply to all AI coding assistants (Copilot, Claude Code, etc.) working in this repository.
---

# Repository Guidelines for AI Coding Agents

These conventions apply to all AI coding assistants (Copilot, Claude Code, etc.) working in this repository.

## Code style

- **Line width: 120 columns.** Applies to C# code, doc comments, and regular comments. Reflow accordingly.
  This is enforced informally; see `guidelines = 120` in [.editorconfig](.editorconfig).
  Two exceptions:
  - A method whose return type is a long tuple keeps the tuple on the same line as the method name, even if
    that line exceeds 120 columns.
  - Long string literals may exceed 120 columns rather than being broken up.
- Indentation: 4 spaces (no tabs) for C# files.
- Trailing newline at end of file.

## Comments and documentation

- Prefer `#1234` over full GitHub URLs when referencing issues or PRs in this repo
  (`icerpc/icerpc-csharp`). Use full URLs only when linking to other repositories.
- XML doc comments use `<summary>`, `<remarks>`, `<param>`, etc. — not Markdown.

## Branch conventions

- `main` is the active development branch. **API-breaking changes are allowed and expected on `main`.**
  Do not flag, warn about, or suggest `[Obsolete]` shims for source-/binary-breaking changes targeting `main`.
- For PRs targeting release branches, treat API-breaking changes as warnings worth flagging.

## Tests

- Test projects live under `tests/`. Use NUnit (`[Test]`, `Assert.That(..., Is.X)`).
- Test method names use `Snake_case_descriptions` (e.g. `Scoped_service_is_disposed_before_response_payload_is_read`).

## Build / test commands

- Build the whole solution: `dotnet build IceRpc.slnx`
- Run all tests: `dotnet test`
- Run a single test project: `dotnet test tests/<Project>/<Project>.csproj --filter "FullyQualifiedName~<TestClass>"`

## Dismissed audit patterns

This section captures the reasoning behind `ai-audit` findings that have been closed as "not planned". Before opening a
new audit finding (or reporting an existing concern), check whether it falls under one of these patterns — the
rationale has already been considered and rejected. Cite the canonical issue when relevant.

For dismissals newer than this section, also consult:
`gh issue list --repo icerpc/icerpc-csharp --label ai-audit --state closed --search "reason:not-planned"`.

### 1. Trusted IDL inputs (`.slice` / `.proto` / `.ice`)

`.slice`, `.proto`, and `.ice` files are authored as part of the build, alongside `.cs` files — not untrusted runtime
data. Code-injection framing does not apply: anyone who can author IDL in a project can already author C#. Malformed
values (cs::type, cs::attribute, cs::identifier, csharp_namespace, deprecated message, attribute payloads, deeply
nested types, slicec symbol output) surface as C# compiler errors on the generated source, which is an acceptable
failure mode.
(See #4444, #4449, #4459, #4467, #4470, #4485, #4495, #4496, #4497, #4502, #4503.)

### 2. Trusted toolchain downloads

Binaries fetched during source build come from trusted upstreams (NuGet.org, the official protoc GitHub repo, the
icerpc GitHub org). We rely on HTTPS + the ecosystem trust model, consistent with the rest of the .NET/NuGet world.
Findings asking for integrity hashes on these specific sources are not actionable. (#4450, #4471.)

### 3. Generator identifier-mapping collisions

The Slice and Protobuf code generators map IDL identifiers to C# identifiers via PascalCase / underscore stripping.
These mappings can collapse distinct source identifiers to the same C# name. Real-world collisions are rare; when they
happen the supported fix is `cs::identifier` (Slice) or a future `csharp_identifier`-style option (Protobuf). Don't
propose validator changes that close the heuristic at the cost of breaking schemas that use it intentionally.
(#4446, #4467.)

### 4. Output filename collisions

Two `.slice` / `.proto` files sharing a basename collide in a shared `OutputDir`. The supported workaround is per-input
`OutputDir` metadata (which triggers separate generator runs). The current behavior fails at build time, not silently —
that's the contract we're keeping. (#4445, #4452, #4473.)

### 5. Contract-violation findings

Defensive code for paths that can only fire when a documented contract is already violated
(e.g. `IAsyncDisposable.DisposeAsync` throwing, callers racing `IDuplexConnection.WriteAsync` and `ShutdownWriteAsync`,
slicec emitting structurally impossible symbol output, `[Service]` declared multiply on partials when the attribute
already disallows it). We prefer to surface real-world contract violations as their own bugs, not paper over them at
every call site. (See #4414, #4422, #4458, #4480, #4502, #4503.)

### 6. Architectural mitigation already in place

Findings that flag an unbounded value when a higher-level limit already caps it: `SliceDecoder` collection-allocation
budget,`MaxControlFrameBodySize`, the consistent 4-byte varuint62 size placeholder used at every encode site. Adding a
local guard on top is cosmetic or inconsistent; revisits should target the systemic limit, not the call site.
(#4464, #4490, #4499, #4524.)

### 7. Forward-compatibility ignores of unknown values


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icerpc/icerpc-csharp](https://github.com/icerpc/icerpc-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
