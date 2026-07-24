---
trigger: always_on
description: Use `build.ps1` as the entry point for local work. It bootstraps dependencies into `output/RequiredModules`, prepends the built module path to `PSModulePath`, and then delegates to InvokeBuild workflows defined in `build.yaml`.
---

# Copilot instructions for Sampler

## Build, test, and quality commands

Use `build.ps1` as the entry point for local work. It bootstraps dependencies into `output/RequiredModules`, prepends the built module path to `PSModulePath`, and then delegates to InvokeBuild workflows defined in `build.yaml`.

### Mandatory rule — never bypass `build.ps1`

- Always build the module with `./build.ps1 -Tasks build`. Never invoke `Build-Module` (or any other ModuleBuilder cmdlet) directly, and never copy files into `output/module/**` by hand. Bypassing the InvokeBuild pipeline produces an incomplete artifact (missing `Templates/`, `en-US/`, `scripts/`, `tasks/`), which silently breaks Plaster-driven commands such as `New-SampleModule` and any test that imports the built module.
- Always run tests with `./build.ps1 -Tasks test ...`. Do not call `Invoke-Pester` directly against `tests/**` from a fresh shell — `build.ps1` is what configures `PSModulePath`, ensures the module is freshly built, and applies the Pester configuration from `build.yaml`. Direct `Invoke-Pester` runs may pick up a stale or partial build and report misleading failures.
- For PowerShell module workflows, treat `./build.ps1 -Tasks test` as a validation step that expects the module build artifact to exist and be readable. If a task is operating on a built module artifact, fail fast when the built manifest is missing instead of recomputing module state from source.
- Always set up the environment through `build.ps1`. Do not manually prepend `output/RequiredModules` or `output/module` to `PSModulePath`. Instead, run `./build.ps1 -ResolveDependency -Tasks noop` (or any other `-Tasks <name>` invocation) at the start of a session — it bootstraps dependencies and configures `PSModulePath` for the current shell so subsequent `Import-Module Sampler -Force` calls resolve the freshly built artifact.
- After `./build.ps1 -Tasks build` completes in the current shell, `Import-Module Sampler -Force` is sufficient to load the built module for ad-hoc verification (for example, running `New-SampleModule` against a scratch path). Do not start a separate PowerShell session to test — the path setup performed by `build.ps1` only applies to the shell that ran it.
- For an agent (or any tool) that runs each shell command in its own fresh process with no persisted state, chain `./build.ps1 -Tasks build`, `Import-Module Sampler -Force`, and the verification/test logic together in one invocation (e.g. `./build.ps1 -Tasks build; Import-Module Sampler -Force; <verification code>`). Splitting them across separate invocations silently re-resolves `Import-Module Sampler -Force` against a stale, globally-installed Sampler version instead of `output/module/Sampler/<version>`, producing misleading errors (for example a `ValidateSet`/parameter error for a feature that actually exists in the current source) that look like real bugs but only reflect testing against the wrong module. Confirm `(Get-Module Sampler).ModuleBase` resolves under `output/module/Sampler/` before trusting any ad-hoc verification result.
- The same rule applies inside agents, skills, and CI helpers: every build/test/validation step must go through `./build.ps1`. If a workflow appears to require something `build.ps1` does not expose, extend `build.yaml` (or a `.build/tasks/*.build.ps1` task) instead of working around it.

```powershell
# Restore required modules into output/RequiredModules
./build.ps1 -ResolveDependency -Tasks noop

# Build the module into output/module/Sampler/<version>
./build.ps1 -Tasks build

# Run the default test workflow from build.yaml
./build.ps1 -Tasks test

# Run a single Pester file
./build.ps1 -Tasks test -PesterPath 'tests/Unit/Public/New-SampleModule.tests.ps1' -CodeCoverageThreshold 0

# Run all integration tests
./build.ps1 -Tasks test -PesterPath 'tests/Integration' -CodeCoverageThreshold 0

# Run the repo's HQRM/quality gate
./build.ps1 -Tasks hqrmtest
```

`build.yaml` is the source of truth for workflow aliases (`build`, `test`, `docs`, `pack`, `hqrmtest`, `publish`) and for Pester configuration such as default test paths and coverage thresholds.

### Running build/test commands without hanging the agent shell

Sampler builds and tests are slow (task discovery alone is ~60–120s; a focused integration test pass is 2+ minutes) and emit a lot of progress output. Two pitfalls to avoid:

- **Never wrap the `./build.ps1 ...` invocation in `| Select-Object -Last <N>` (or `| Select-Object -First <N>`, `| Out-String -Stream | Select-Object ...`, etc.) inline.** Those filters force PowerShell to buffer the *entire* output stream before emitting anything, and the wrapping pipeline call appears to hang from the agent's perspective even after the build has finished. Worse, if the build asks for input (it should not, but ResolveDependency prompts can sneak in), the prompt is buried in the buffer and the shell is effectively stuck.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gaelcolas/Sampler](https://github.com/gaelcolas/Sampler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
