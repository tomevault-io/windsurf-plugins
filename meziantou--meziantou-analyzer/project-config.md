---
trigger: always_on
description: **Any code you commit SHOULD compile, and new and existing tests related to the change SHOULD pass.**
---

**Any code you commit SHOULD compile, and new and existing tests related to the change SHOULD pass.**

You MUST make your best effort to ensure your changes satisfy those criteria before committing. If for any reason you were unable to build or test the changes, you MUST report that. You MUST NOT claim success unless all builds and tests pass as described above.

Do not complete without checking the relevant code builds and relevant tests still pass after the last edits you make. Do not simply assume that your changes fix test failures you see, actually build and run those tests again to confirm.
Also, always run `dotnet run --project src/DocumentationGenerator` to update the markdown documentation after modifying analyzer code or documentation comments. Note that this command returns a non-zero exit code if any markdown files were changed.
After running the command, review the changes made to the markdown files and ensure they are accurate and appropriate. If you make any changes to the markdown files, you MUST re-run the command to verify that no further changes are necessary.

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
- When running tests, if possible use filters and check test run counts, or look at test logs, to ensure they actually ran.
- Do not finish work with any tests commented out or disabled that were not previously commented out or disabled.
- Do not update `global.json` file
- When writing tests, do not emit "Act", "Arrange" or "Assert" comments.
- There should be no trailing whitespace in any lines.
- Add a blank line before XML documentation comments (`///`) when they follow other code (methods, properties, fields, etc.).

## Documenting equivalent or similar rules

When a rule in this analyzer is equivalent to or similar to a rule in another analyzer (e.g., Roslyn IDE rules, CA rules, SonarQube rules), document the relationship in [`docs/comparison-with-other-analyzers.md`](/docs/comparison-with-other-analyzers.md):

- **Equivalent rules**: Add an entry to the "Equivalent rules" table (two columns: external rule | MA rule). Do not add a note to the individual rule's documentation file.
- **Similar rules**: Add an entry to the "Similar rules" table (three columns: external rule | MA rule | explanation of differences).

Do NOT add equivalence/similarity notes directly to individual rule documentation files (e.g., `docs/Rules/MA0158.md`).

## Maintaining Meziantou.Analyzer.Annotations

When you change any file under `src/Meziantou.Analyzer.Annotations`, you MUST:

- Update [`src/Meziantou.Analyzer.Annotations/README.md`](/src/Meziantou.Analyzer.Annotations/README.md) if the package behavior, exposed attributes, or usage guidance changed.
- Bump `<Version>` in [`src/Meziantou.Analyzer.Annotations/Meziantou.Analyzer.Annotations.csproj`](/src/Meziantou.Analyzer.Annotations/Meziantou.Analyzer.Annotations.csproj).

## Implementing Roslyn analyzers

- When creating a new rule, create a new constant in `src/Meziantou.Analyzer/RuleIdentifiers.cs` using the name of the new rule. The value must be unique and incremented from the last rule.
- When updating an existing rule, update the corresponding documentation file under `docs/Rules/` to reflect the change.
- The analyzers must be under `src/Meziantou.Analyzer/Rules/`
- The code fixers must be under `src/Meziantou.Analyzer.CodeFixers/Rules`
- The tests must be under `tests/Meziantou.Analyzer.Test/Rules`

The analyzer must use `IOperation` or `ISymbol` to analyze the content. Only fallback to `SyntaxNode` when the other ways are not supported.

Code snippets in tests must use raw string literals (`"""`) and must be minimized to only include the necessary code to reproduce the issue. Avoid including unnecessary code that does not contribute to the test case.
When reporting a diagnostic, the snippet must use the `[|code|]` syntax or `{|id:code|}` syntax. Do not explicitly indicates lines or columns.

### Code fixer best practice: validate before registering

In `RegisterCodeFixesAsync`, validate **all** conditions that could prevent the fix from being applied **before** calling `context.RegisterCodeFix`. Do not register a code fix whose action would return the document unchanged.

**Wrong** — registers the fix without validating whether it can be applied:
```csharp
public override async Task RegisterCodeFixesAsync(CodeFixContext context)
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meziantou/Meziantou.Analyzer](https://github.com/meziantou/Meziantou.Analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
