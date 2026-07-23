---
trigger: always_on
description: <!-- The CUE project common guidance is imported below, managed
---

# CUE

<!-- The CUE project common guidance is imported below, managed
     by cueckoo. If the referenced file is missing on your
     machine, run "cueckoo version update" to write it (and
     pick up any newer cueckoo while you are at it). -->
@~/.cache/cueckoo/common-guidance.md

## Project-specific instructions

This is the CUE language repository. CUE (Configure, Unify, Execute) is a
general-purpose, strongly typed constraint-based language for data
templating, validation, code generation, and scripting.

Module: `cuelang.org/go`. Requires Go 1.25 or later.

### Common development commands

#### Running the "cue" command

```bash
# Build and run ./cmd/cue via a cached binary
go tool cue

# Or build and install it in $PATH
go install ./cmd/cue
```

#### Testing

```bash
# Run all tests. Any change to the repo should run all tests to ensure correctness.
go test ./...

# Run tests for a specific package
go test ./internal/core/adt

# Run a specific test function
go test -run TestEvalV3 ./internal/core/adt

# Run a specific sub-test
go test -run TestScript/eval_concrete ./cmd/cue/cmd

# Update golden test files (when output changes are expected)
CUE_UPDATE=1 go test ./...

# Run tests with race detector
go test -race ./...
```

CI enforces that `CUE_UPDATE=1 go test ./...` leaves no diffs. For every
commit touching `cue/testdata` or the evaluator (`internal/core/...`),
re-run the tests with `CUE_UPDATE=1` and include any resulting changes
(e.g. updated stats sections) in that same commit, so outputs stay up to
date on every commit.

#### Code quality

```bash
go vet ./...
go tool -modfile=internal/tools.mod staticcheck ./...
go fmt ./...
```

### Code architecture

#### Core language implementation
- `/cue/` - Core CUE language implementation
  - `ast/` - Abstract Syntax Tree
  - `parser/` - Language parser
  - `load/` - Package loading and imports
  - `format/` - Code formatting
- `/internal/core/` - Core evaluation engine
  - `adt/` - Core data structures and algorithms
  - `compile/` - Compilation logic
  - `dep/` - Dependency analysis
  - `export/` - Export functionality

#### Command-line tool
- `/cmd/cue/` - CLI implementation for all CUE commands (eval, export, import, fmt, vet, mod, etc.)

#### Standard library
- `/pkg/` - Built-in packages (crypto, encoding, math, net, path, strings, etc.)

#### Format support
- `/encoding/` - Encoders/decoders for JSON, YAML, TOML, Protobuf, OpenAPI, JSON Schema

#### Testing infrastructure
- **Test format**: Uses `.txtar` (text archive) files containing input files and expected outputs
- **Test organization**: Unit tests alongside code (`*_test.go`), integration tests in `testdata/` directories
- **Testscript framework**: Command-line integration tests in `/cmd/cue/cmd/testdata/script/`

### Working with tests
- Tests use the `.txtar` format which contains both input and expected output in a single file
- Use `TestX` functions in test files for debugging individual test cases
- The `CUE_UPDATE=1` environment variable updates golden files with actual output
- Prefer adding test cases to an existing txtar file that is appropriate for the type of reproducer

### Converting txtar tests to inline `@test(...)` format

When converting a txtar test from golden-file format to inline annotations:

1. **Section ordering**: The `-- out/errors.txt --` section must directly follow the
   last input section (e.g., `-- in.cue --`), before `-- out/compile --` and
   `-- out/eval/stats --`.

2. **Error annotations**: Always include position information using a `pos=[]`
   placeholder; `CUE_UPDATE=1` fills it in automatically. Positions are matched
   order-independently; commas between specs are optional. Example:
   ```
   x: bad @test(err, code=eval, contains="...", pos=[])
   ```

3. **Multiple sub-errors**: When an error has multiple sub-errors (e.g., from a
   failed disjunction), use `suberr=(...)` to match individual sub-errors instead
   of a single flat `contains=`:
   ```
   x: a | b @test(err, suberr=(code=eval, contains="..."), suberr=(code=eval, contains="..."))
   ```

3a. **`path=` flag**: Checks the dotted CUE path that the error self-reports via
    `cueerrors.Error.Path()`. This is distinct from `at=`: `at=` navigates to a
    sub-value before checking, while `path=` asserts what the error reports as its
    own location. `CUE_UPDATE=1` generates `path=` automatically with two suppression
    rules: (1) path= is omitted when the error's path equals the annotated field's path;
    (2) path= is omitted when the `at=` value is a path-boundary suffix of the error path
    (e.g. `at=w.y.z.b.E` suppresses `path=fieldNotAllowed.t3.w.y.z.b.E`). The flag is
    also supported inside `suberr=(...)` for discriminating sub-errors by location:
    ```
    // sub-errors carry distinct paths; path= is generated because they differ from "x":
    x: ({a: int & string} | {b: int & bool}) @test(err,
        suberr=(path=x.a, contains="string"),
        suberr=(path=x.b, contains="bool"))
    // at= covers the location; path= is suppressed on CUE_UPDATE=1 fill:
    outer: {a: {b: int & string}} @test(err, at=a.b, contains="conflicting values")
    ```

4. **Files with compile-time errors**: CUE source files that themselves produce

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cue-lang/cue](https://github.com/cue-lang/cue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
