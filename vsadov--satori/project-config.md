---
trigger: always_on
description: **Any code you commit MUST compile, and new and existing tests related to the change MUST pass.**
---


**Any code you commit MUST compile, and new and existing tests related to the change MUST pass.**

You MUST make your best effort to ensure any code changes satisfy those criteria before committing. If for any reason you were unable to build or test code changes, you MUST report that. You MUST NOT claim success unless all builds and tests pass as described above.

If you make code changes, do not complete without checking the relevant code builds and relevant tests still pass after the last edits you make. Do not simply assume that your changes fix test failures you see, actually build and run those tests again to confirm.

Before completing, use the `code-review` skill to review your code changes. Any issues flagged as errors or warnings should be addressed before completing.

If the changes are intended to improve performance, or if they could negatively impact performance, use the `performance-benchmark` skill to validate the impact before completing.

You MUST follow all code-formatting and naming conventions defined in [`.editorconfig`](/.editorconfig).

In addition to the rules enforced by `.editorconfig`, you SHOULD:

- Prefer file-scoped namespace declarations and single-line using directives.
- Ensure that the final return statement of a method is on its own line.
- Use pattern matching and switch expressions wherever possible.
- Use `nameof` instead of string literals when referring to member names.
- Always use `is null` or `is not null` instead of `== null` or `!= null`.
- Trust the C# null annotations and don't add null checks when the type system says a value cannot be null.
- Prefer `?.` if applicable (e.g. `scope?.Dispose()`).
- Use `ObjectDisposedException.ThrowIf` where applicable.
- When adding new unit tests, strongly prefer to add them to existing test code files rather than creating new code files.
- When adding new unit tests, avoid adding a regression comment citing a GitHub issue or PR number unless explicitly asked to include such information.
- If you add new code files, ensure they are listed in the csproj file (if other files in that folder are listed there) so they build.
- When running tests, if possible use filters and check test run counts, or look at test logs, to ensure they actually ran.
- Do not finish work with any tests commented out or disabled that were not previously commented out or disabled.
- When writing tests, do not emit "Act", "Arrange" or "Assert" comments.
- For markdown (`.md`) files, ensure there is no trailing whitespace at the end of any line.
- When adding XML documentation to APIs, follow the guidelines at [`docs.prompt.md`](/.github/prompts/docs.prompt.md).

---

# Building & Testing in dotnet/runtime

## ⚠️ MANDATORY: Run Baseline Build First

**You MUST complete a baseline build BEFORE making any code changes.** Skipping this causes "missing testhost" and "shared framework" errors that waste time.

### Step 1: Identify Your Component

Based on file paths you will modify:

| Files Changed | Component |
|---------------|-----------|
| `src/coreclr/` | CoreCLR |
| `src/mono/` | Mono |
| `src/libraries/` (no Browser/WASM or WASI targets) | Libraries |
| `src/libraries/` with Browser/WASM or WASI targets in the affected `.csproj` | WASM/WASI Libraries |
| `src/native/corehost/`, `src/installer/` | Host |
| `src/tools` | Tools |
| `src/native/managed` | Tools |
| `src/tasks` | Build Tasks |
| `src/tests` | Runtime Tests |

**WASM/WASI Library Detection:** A change under `src/libraries/` is WASM/WASI-relevant if the library's `.csproj` has explicit Browser/WASM or WASI targets (`TargetFrameworks`, `TARGET_BROWSER`, `TARGET_WASI` constants, or `Condition` attributes referencing `browser`/`wasi`), **and** the changed file is not excluded from those targets via `Condition` on `<ItemGroup>` or `<Compile>`.

### Step 2: Run the Baseline Build (from repo root)

**First, checkout the `main` branch** to establish a known-good baseline, then run the appropriate build command:

| Component | Command |
|-----------|---------|
| **CoreCLR** | `./build.sh clr+libs+host` |
| **Mono** | `./build.sh mono+libs` |
| **Libraries** | `./build.sh clr+libs -rc release` |
| **WASM Libraries** | `./build.sh mono+libs -os browser` |
| **Host** | `./build.sh clr+libs+host -rc release -lc release` |
| **Tools** | `./build.sh clr+libs -rc release` |
| **Build Tasks** | `./build.sh clr+libs -rc release` |
| **Runtime Tests** | `./build.sh clr+libs -lc release -rc checked` |

For System.Private.CoreLib changes, use `-rc checked` instead of `-rc release` for asserts.

⏱️ **This build can take up to 40 minutes.** Do not cancel unless no output for 5+ minutes.

### Step 3: Configure Environment

```bash
export PATH="$(pwd)/.dotnet:$PATH"
dotnet --version  # Should match sdk.version in global.json
```

**Only proceed with changes after the baseline build succeeds.** If it fails, report the failure and stop. After the baseline build, switch back to your working branch before making changes.

---

## Component-Specific Workflows

After completing the baseline build above (the baseline build MUST be completed before running tests), use the appropriate workflow for your changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VSadov/Satori](https://github.com/VSadov/Satori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
