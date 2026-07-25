---
trigger: always_on
description: OpenAPI 3 mock server built with Deno. Validates SDKs against OpenAPI specs with
---

# Steady

OpenAPI 3 mock server built with Deno. Validates SDKs against OpenAPI specs with
clear error attribution (SDK bug vs spec issue).

## Commands

**ALWAYS use the scripts in `scripts/` directory.

```bash
./scripts/bootstrap   # Install dependencies, setup environment
./scripts/test        # Run all tests
./scripts/lint        # Lint code
./scripts/format      # Format code
```

These scripts handle all the necessary flags and environment setup
automatically.

## Project Structure

```
steady/
├── cmd/steady.ts              # CLI entry point
├── src/                       # Main server
│   ├── server.ts              # HTTP server, request matching
│   ├── validator.ts           # Request/response validation
│   ├── errors.ts              # Error types
│   └── logging/               # Request logging utilities
├── packages/
│   ├── json-pointer/          # RFC 6901 implementation
│   │   ├── json-pointer.ts    # resolve(), set(), escape/unescape
│   │   ├── rfc6901-validator.ts # Syntax validation
│   │   └── resolver.ts        # Document reference resolver
│   ├── json-schema/           # JSON Schema 2020-12
│   │   ├── processor.ts       # Schema analysis
│   │   ├── runtime-validator.ts # Data validation
│   │   ├── schema-registry.ts # Document-centric schema resolution
│   │   └── ref-resolver.ts    # $ref resolution
│   └── openapi/               # OpenAPI 3.x parser
│       └── parser.ts          # YAML/JSON parsing
├── sdk-tests/                 # SDK test suites with their OpenAPI specs
│   └── cloudflare-python/     # Cloudflare spec at openapi-spec.yml
├── tests/edge-cases/          # Edge case tests
└── test-fixtures/
    └── openapi-directory/     # Git submodule: 1970 real-world specs (99.5% pass)
```

**Test specs**: Real-world specs live in `sdk-tests/` (e.g.,
`sdk-tests/cloudflare-python/openapi-spec.yml`). When looking for a spec file,
search the whole repo with glob, not just `test-fixtures/`.

**Submodule**: Run `git submodule update --init` to fetch test fixtures.

## Key Technical Details

**Stack**: Deno 2.x, TypeScript strict mode, no frameworks

**JSON Pointer (RFC 6901)**:

- Only `~0` (tilde) and `~1` (slash) escaping - NO percent encoding
- Percent-decoding happens at URI fragment layer (ref-resolver.ts:171)
- Array indices must be exact: "0", "1", "10" - reject "01", "1.5", "-1"

**JSON Schema**: 91.6% compliance (1151/1257 tests). Missing: `$dynamicRef`,
`$dynamicAnchor`. Full support for `unevaluatedProperties`/`unevaluatedItems`.

**$ref Resolution**: Handles URI fragment encoding. `#/$defs/User%20Name`resolves
to key`"User
Name"` (percent-decoded before JSON Pointer parsing).

## Code Rules

1. **Read before modify** - Never change code you haven't read
2. **No type hacks** - No `as`, no `!` assertions to silence errors
3. **No silent failures** - Never swallow errors or return fake success
4. **Test with red-green** - Write failing test first, then fix
5. **Fail loudly** - Invalid input = error, not silent pass
6. **No hacky solutions** - Use standard libraries (e.g., `@std/cli` for arg
   parsing), don't reinvent the wheel with brittle manual implementations
7. **No chained bash commands** - When running commands (not writing scripts),
   never use `|`, `&&`, or `||`. Run each command as a separate, distinct
   invocation
8. **No em-dashes** - Never use `—` (U+2014) anywhere in the codebase: comments,
   strings, messages, or docs. Em-dashes are non-ASCII and cause ByteString
   crashes when diagnostic messages flow into HTTP headers via `Headers.set()`.
   Use normal punctuation instead (periods, commas, semicolons). Two sentences
   are better than one long sentence joined by an em-dash.
9. **Raw at the edges, structured in the logic** - Parse untyped inputs
   (strings, JSON, HTTP bodies) once at the boundary into a domain type, and
   pass the domain type through all internal logic. Format back to raw form only
   at the outgoing boundary. Never thread a raw string through recursion and
   re-parse or string-concat it inside the logic. If a primitive you need for
   the domain type does not exist, add it to the owning package (e.g. pointer
   manipulation belongs in `@steady/json-pointer`, schema composition in
   `@steady/json-schema`). Never open-code a local helper that duplicates
   something the owning package should provide, and never introduce a second
   implementation of a primitive that already exists. Search the owning package
   first.

## Skills

### /design-review

Review a design/spec document by finding real-world patterns that stress it. Use
with: `/design-review docs/diagnostics-spec.md`

See `.claude/skills/design-review/SKILL.md` for full process.

### /steady-dev

Project context and working conventions. Read this before making changes.

See `.claude/skills/steady-dev/SKILL.md` for architecture, design philosophy,
and common task guides.

### /user-experiment

Simulate being a real SDK developer to find UX friction in Steady. Use with:
`/user-experiment sink-python`

See `.claude/skills/user-experiment/SKILL.md` for the full methodology.

## Investigation Standards

**INVESTIGATE BEFORE IMPLEMENTING**: Always research the correct behavior first.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dgellow/steady](https://github.com/dgellow/steady) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
