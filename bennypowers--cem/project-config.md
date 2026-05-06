---
trigger: always_on
description: When running cem commands against the example projects, you don't need to `cd`, you can `cem ${cmd} -p examples/${example}`.
---

## cem binary usage

When running cem commands against the example projects, you don't need to `cd`, you can `cem ${cmd} -p examples/${example}`.

example:

```shell
$ make
$ dist/cem -p examples/kitchen-sink validate
```

## Go

Getter methods should be named `Foo()`, not `GetFoo()`.

When adding debug logs in go code, always use `bennypowers.dev/cem/internal/logging`, to avoid polluting stdio.

Run `go vet` to surface gopls suggestions. Common examples:
- replace `interface{}` with `any`
- replace `if/else` with `min`
- replace `m[k]=v` loop with `maps.Copy` [mapsloop]
- Loop can be simplified using slices.Contains [slicescontains]

## Debugging

When debugging Go code, always use the logger `logger.Debug`, etc. Don't use `fmt.Printf`, which pollutes stdio, breaking the LSP and MCP commands.

To print debug logs to the console, pass `-vvv`

## Testing

Practice TDD. When writing tests, always use the fixture/golden patterns we've established:

- **Fixtures**: Input test data - the content directories/files your code operates on (e.g., `serve/testdata/transforms/http-typescript/simple-greeting.ts`). Use `NewFixtureFS()` helper from `internal/platform/testutil` instead of `os.ReadFile()` to load fixtures into in-memory file systems.
- **Goldens**: Expected output files to compare against (e.g., `serve/middleware/routes/testdata/chrome-rendering/expected-basic.html`). Tests should support `--update` flag to regenerate golden files when intentional changes occur.
- Always use Makefile targets for running tests or builds, since they export the necessary env vars.
- For LSP tests, ALWAYS use `testutil.RunLSPFixtures()` - NEVER use direct `os.ReadFile()` calls.
- If frontend visual regression tests fail consistently, check if it's merely 
  an issue of font rendering.

### LSP Testing Patterns

- **Fixture Structure**: Each test scenario is a subdirectory containing:
  - `input.html` or `input.ts` (required)
  - `manifest.json` (optional)
  - `expected.json` or `expected-*.json` (optional, for assertions)
  - `package.json` pointing `customElements` block to `manifest.json`

- **RunLSPFixtures Pattern**:
  ```go
  testutil.RunLSPFixtures(t, "testdata/my-test-suite", func(t *testing.T, fixture *testutil.LSPFixture) {
      // Setup context
      ctx := testhelpers.NewMockServerContext()

      // Use fixture.InputContent, fixture.Manifest, etc.
      // Load expected data with fixture.GetExpected("key", &expected)
  })
  ```

- **Multiple Expected Files**: Use `expected-variant.json`, `expected-size.json` pattern for testing multiple positions/cases in one fixture

- **Regression Test Isolation**: Keep regression test fixtures in separate directories (e.g., `testdata-regression/`) to avoid interference with standard test discovery

### Testing Tiers

See `docs/TESTING.md` for the full testing convention. In brief:
- **Tier 1 (pure functions):** exhaustive table-driven tests, in-package. A pure function is one where all effects depend on its formal parameters.
- **Tier 2 (logic with effects):** fixture-based tests with real dependencies (e.g., QueryManager). No percentage target.
- **Tier 3 (thin wiring):** covered by integration/e2e tests. No separate unit tests required.

New LSP methods or MCP tools must include at least Tier 2 tests before merging.

## Per-package guidelines

- Docs: When working on the docs site, refer to `docs/CLAUDE.md` for docs-specific guidelines.
- Generate and Validate: When touching code related to the custom-elements manifest schema, double-check with the canonical TypeScript schema.d.ts at https://github.com/webcomponents/custom-elements-manifest/. This is to avoid hallucinations. Note that different schema versions may have different types, and we'll have to deal with that if they do.

### LSP

- **Tree-sitter concurrency**: Never share `QueryMatcher` between goroutines. Cache queries, create fresh cursors per operation.
- **Nil safety**: Check for nil `DocumentManager` before use; test with nil parameters.
- **Protocol compliance**: Match LSP return types exactly; declare all capabilities in `initialize`.
- **Testing**: Use `package lsp_test` for public API tests. Use `testhelpers.NewMockServerContext()` for test context. Fixtures in `testdata/` dirs, regression fixtures in `testdata-regression/`.
- When adding LSP methods, implement diagnostics + code actions together and update server capabilities.

### MCP

- **Design principle**: Data + context + LLM decision-making. Provide rich manifest data and contextual templates; let the LLM decide. No hardcoded suggestions.
- **Respect element authors**: Use their documented constraints and intent. Follow documented accessibility patterns, don't add generic ARIA advice.
- **Template naming**: Use dashes (e.g., `element-attributes.md`), not underscores. Reference subresources (`cem://element/{tagName}/attributes`), not tool names.

## Git

- Always run `make lint` and address issues before committing
- Always check go LSP diagnostics and address them before committing
- When commit messages mention AI agents, always use `Assisted-By`, never use `Co-Authored-By`.

## Frontend

When writing CSS, use modern native syntax including nesting, :has, layers, and light-dark()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bennypowers/cem](https://github.com/bennypowers/cem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
