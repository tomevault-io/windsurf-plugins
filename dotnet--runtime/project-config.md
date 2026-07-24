---
trigger: always_on
description: When reviewing pull requests, use the `code-review` skill unless the user has stated they will review the changes themselves.
---

When reviewing pull requests, use the `code-review` skill unless the user has stated they will review the changes themselves.

**Any code you commit MUST compile, and new and existing tests related to the change MUST pass.**

You MUST make your best effort to ensure any code changes satisfy those criteria before committing. If for any reason you were unable to build or test code changes, you MUST report that. You MUST NOT claim success unless all builds and tests pass as described above.

If you make code changes, do not complete without checking the relevant code builds and relevant tests still pass after the last edits you make. Do not simply assume that your changes fix test failures you see, actually build and run those tests again to confirm.

When running under CCA and before completing, use the `code-review` skill to review your code changes. Any issues flagged as errors or warnings should be addressed before the task is considered complete.

When NOT running under CCA, skip the `code-review` skill if the user has stated they will review the changes themselves.

Before making changes to a directory, search for `README.md` files in that directory and its parent directories up to the repository root. Read any you find — they contain conventions, patterns, and architectural context relevant to your work.

If the changes are intended to improve performance, or if they could negatively impact performance, use the `performance-benchmark` skill to validate the impact before completing.

When writing or reviewing SIMD / hardware-intrinsics code (anything using `Vector128`/`Vector256`/`Vector512`, `Vector<T>`, or the platform intrinsics in `System.Runtime.Intrinsics.*`), use the `vectorization` skill.

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
- When adding new test files, examine the directory structure of sibling tests first. Some test directories use flat files (e.g., `GCEvents.cs` alongside `GCEvents.csproj`) while others use per-test subdirectories. Match the existing convention.
- When working with tests, look for `README.md` files along the directory hierarchy (starting from the test's directory and walking up). These contain build, run, and authoring guidance specific to that test area.
- When adding new unit tests, avoid adding a regression comment citing a GitHub issue or PR number unless explicitly asked to include such information.
- When writing tests, prefer using `[Theory]` with multiple data sources (like `[InlineData]` or `[MemberData]`) over multiple duplicative `[Fact]` methods. Fewer test methods that validate more inputs are better than many similar test methods.
- If you add new code files, ensure they are listed in the csproj file (if other files in that folder are listed there) so they build.
- When running tests, if possible use filters and check test run counts, or look at test logs, to ensure they actually ran.
- Do not finish work with any tests commented out or disabled that were not previously commented out or disabled.
- When writing tests, do not emit "Act", "Arrange" or "Assert" comments.
- For markdown (`.md`) files, ensure there is no trailing whitespace at the end of any line.
- When adding XML documentation to APIs, follow the guidelines at [`docs.prompt.md`](/.github/prompts/docs.prompt.md).

When NOT running under CCA, guidance for creating commits and pushing changes:

- Never squash and force push unless explicitly instructed. Always push incremental commits on top of previous PR changes.
- Never push to an active PR without being explicitly asked, even in autopilot/yolo mode. Always wait for explicit instruction to push.
- Never chain commit and push in the same command. Always commit first, report what was committed, then wait for an explicit push instruction. This creates a mandatory decision point.
- Prefer creating a new commit rather than amending an existing one. Exceptions: (1) explicitly asked to amend, or (2) the existing commit is obviously broken with something minor (e.g., typo or comment fix) and hasn't been pushed yet.
- **Before posting to GitHub (PRs, issues, comments):** Include the AI-generated content disclosure (see below).

## AI-Generated Content Disclosure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/runtime](https://github.com/dotnet/runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
