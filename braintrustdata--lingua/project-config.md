---
trigger: always_on
description: This guide helps AI assistants understand and work with the Lingua codebase effectively.
---

# Lingua project guide for Claude

This guide helps AI assistants understand and work with the Lingua codebase effectively.

## Project overview

Lingua is a universal message format that compiles to provider-specific formats with zero runtime overhead. It's designed to allow seamless interoperability between different LLM providers without runtime penalties.

## Key principles

- **Universal compatibility**: Supports 100% of provider-specific quirks and capabilities
- **Zero runtime overhead**: Pure compile-time translation to native provider formats
- **Type safety**: Full TypeScript and Rust type generation with bidirectional validation
- **No network calls**: This is a message format library, not an API client
- **Explicit error handling**: All errors must be properly handled, never silently swallowed
- **No hidden marker fields**: Do not encode provider semantics via internal marker keys (for example in `provider_options`) to fake lossless roundtrips.
- **Ask when non-lossy mapping is unclear**: If the universal type cannot represent a provider feature non-lossily, stop and ask for clarification on the intended canonical representation before implementing a workaround.
- **No unapproved fallback logic**: Do not add ad-hoc fallback parsing/translation paths (for example `fallback_*` helpers) without checking with the programmer first.
- **Typed boundaries only**: At provider boundaries, parse into well-defined typed structs/enums. Do not add lenient raw-JSON parsing that guesses defaults for required fields (for example defaulting missing `role` to `user`, lowercasing unknown roles, or inventing empty `content`).
- **Do not handwrite provider-format structs**: Do not manually define Rust structs/enums that represent provider wire formats when generated or canonical provider types already exist. Fix generation or add typed adapters around canonical types instead.
- **Do not inspect `serde_json::Value` directly for provider semantics**: Do not branch on provider-format fields via ad-hoc `Value` map access. Deserialize into typed provider or typed compatibility structs first, then convert.
- **Lenient import paths are typed boundaries too**: Files like `processing/import.rs` are not exempt. For any `role`/`content`/`tool_call_id` compatibility handling, first deserialize into typed compatibility structs (with serde aliases as needed), then branch on typed enums/fields.
- **Pre-edit parser guardrail**: Before finalizing parser/converter changes in typed-boundary code, scan your diff for new `as_object()`, `.get(\"...\")`, `Value::Object`, or raw `Map<String, Value>` field-plucking used for semantics. If present, rewrite to typed deserialization or stop and ask.
- **Fix via types or explicit errors**: If fuzzing finds unsupported/ambiguous shapes, either model them explicitly in types/converters or return a clear error. Do not silently coerce invalid input into a "best effort" shape.
- **Typed-boundary CI gate**: CI enforces `make typed-boundary-check-branch BASE=origin/<base-branch>` on pull requests. Running `make typed-boundary-check` locally is recommended for faster feedback, but not required as a pre-commit hook.
- **Typed extras views over raw map access**: If provider extras must be read, deserialize extras into a typed view struct first; do not pluck fields ad-hoc with `map.get(...)`.

## Documentation style guide

**Always use sentence case for all headings, not title case**:
- ✅ `## Pipeline overview` 
- ❌ `## Pipeline Overview`

**Be concise and direct**:
- Focus on what, not why (unless specifically asked)
- Avoid unnecessary explanations or summaries
- Use bullet points and structured formats

## Project structure

```
src/
├── universal/             # Core Lingua message types
├── providers/             # Provider-specific API type definitions
├── translators/           # Bidirectional format conversion logic
├── capabilities/          # Provider capability detection
└── lib.rs                 # Main entry point and re-exports
```

## Working with providers

Each provider should have:
- **Separate request/response types**: Don't conflate them into single structs
- **Complete type coverage**: All fields from provider SDKs, even optional ones
- **Validation tests**: TypeScript compatibility tests in `tests/typescript/{provider}/`

## Type generation workflow

1. **Check for SDK updates** in provider test directories
2. **Extract TypeScript types** manually from provider SDKs
3. **Convert to Rust** following consistent patterns (see pipelines/ docs)
4. **Validate compatibility** through multi-layer testing
5. **Update translators** to use new types

## Common patterns

**Rust type derivations**:
```rust
#[derive(Debug, Clone, PartialEq, Serialize, Deserialize)]
#[serde(rename_all = "snake_case")] // when needed
```

**TypeScript exports** (for ts-rs):
```rust
#[derive(TS)]
#[ts(export, export_to = "bindings/typescript/")]
```

**Optional fields**: Always use `Option<T>` for optional provider fields

**Union types**: Convert TypeScript unions to Rust enums or separate structs

## Testing approach

**Type compatibility**: Verify Rust-generated TypeScript matches provider SDK types
**Round-trip testing**: Ensure lossless serialization/deserialization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [braintrustdata/lingua](https://github.com/braintrustdata/lingua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
