---
trigger: always_on
description: - Use Makefile settings or commands for building
---

## Building
- Use Makefile settings or commands for building
- All commands located in `cmd/`

## Testing
- All `pkg/runtime` functions should aim for 100% test coverage
- Run `go test -cover ./pkg/runtime/...` to check coverage

## Configuration
- Consult `examples/` for sample configurations for different use cases
- Running without config uses default settings:
  ```go run ./cmd/oapi-codegen <spec-path>```

### Key config options
- `output.use-single-file: true` - Generate all code in one file (default) vs multiple files
- `output.directory` - Output directory (when `use-single-file: false`, package name is appended as subdirectory)
- `generate.client: true` - Generate HTTP client code
- `generate.models: false` - Skip model generation (when models are in separate package)
- `generate.validation.skip: true` - Skip Validate() method generation
- `generate.validation.response: true` - Generate Validate() for response types (useful for contract testing)
- `generate.always-prefix-enum-values: true` - Prefix enum constants with type name (default)
- `generate.default-int-type: int64` - Use int64 instead of int for integer types
- `generate.handler.output.overwrite: true` - Force regeneration of scaffold files (service.go, middleware.go)
- `skip-prune: true` - Keep unused types (normally pruned)
- `error-mapping` - Map response types to implement error interface (key: type name, value: json path to message)
- `filter.include/exclude` - Filter paths, tags, operation-ids, extensions

### Handler/Server generation config
- `generate.handler.kind` - Router framework: `chi`, `echo`, `fiber`, `gin`, `std-http` (required)
- `generate.handler.name` - Service interface name (default: "Service")
- `generate.handler.models-package-alias` - Prefix for model types when models are in separate package
- `generate.handler.validation.request/response` - Enable request/response validation in handlers
- `generate.handler.output.directory/package` - Output for scaffold files (service.go, middleware.go)
- `generate.handler.middleware: {}` - Enable middleware.go generation
- `generate.handler.server` - Enable server/main.go generation with `directory`, `port`, `timeout`, `handler-package`

## Verifying changes
- After making changes to the code generator, ensure to run `make generate` which regenerates the code for all OpenAPI specs in `examples/`
- Verify that the generated code matches the expected output in `examples/`
- Run `make test` to ensure all unit tests pass
- Consider running integration tests in Connexions to verify end-to-end functionality:
  ```make test-integration```. That could take 5 minutes.

## Investigating integration test failures

### Running specific specs
- Re-run with `SPEC=<path-to-spec.yml> make test-integration` to focus on a specific spec
- Never run all integration tests at once - always use SPEC= to limit scope
- Example: `make test-integration SPEC=3.0/github.com/ghes-3.5.1.1.4.yml`
- **Never run oapi-codegen in the project root directory** - always use `/tmp` or run via `make test-integration`

### Common failure types and resolutions

1. **libopenapi limitation - JSON Pointer refs to paths**
   - Error: `component '#/paths/~1api~1v1~1...' does not exist in the specification`
   - Cause: Spec uses `$ref` pointing to path elements which libopenapi can't resolve
   - Resolution: Skip spec by prefixing filename with `-` (e.g., `mv spec.yml -spec.yml`)

2. **Missing schema/component reference**
   - Error: `component '#/components/schemas/SomeName' does not exist`
   - Cause: Spec references a schema that doesn't exist (broken spec)
   - Resolution: Remove the spec from testdata

3. **x- prefixed schema names**
   - Error: `undefined: XAny` or similar
   - Cause: Spec has schemas named with `x-` prefix (e.g., `x-any`) which libopenapi treats as extensions
   - Resolution: Skip spec with `-` prefix

4. **External file references**
   - Error: `unable to open the rolodex file` or `../some-file.yaml does not exist`
   - Cause: Spec references external files that aren't present
   - Resolution: Remove the spec from testdata

5. **Build failures (undefined types, syntax errors)**
   - Check the generated code at the debug path shown in error output
   - Look for patterns in the generated code that indicate the issue
   - Fix the generator code, then re-run the specific spec

### Workflow
1. Get the failing spec path from test output
2. Run: `go run ./cmd/oapi-codegen <spec-path>` to see generation errors
3. If generation succeeds, check the debug path for the generated code
4. For libopenapi limitations, skip with `-` prefix
5. For broken specs, remove from testdata
6. For generator bugs, fix and re-test

### Creating reproducible test cases
When debugging complex issues, create a minimal reproducible example:
1. Create a new directory in `examples/` (e.g., `examples/issue-123/`)
2. Add a simplified spec that reproduces the issue
3. Add a `cfg.yaml` with the relevant configuration
4. Create a `gen_test.go` to verify the generated code compiles:
   ```go
   package example_test

   import (
       "testing"
       _ "github.com/doordash-oss/oapi-codegen-dd/v3/examples/issue-123"
   )

   func TestCompiles(t *testing.T) {}
   ```
5. Run `go generate ./examples/issue-123/...` to generate code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doordash-oss/oapi-codegen-dd](https://github.com/doordash-oss/oapi-codegen-dd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
