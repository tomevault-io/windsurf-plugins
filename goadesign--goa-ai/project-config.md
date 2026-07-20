---
trigger: always_on
description: transforms. Service code should call the generated codec instead of compiling
---

You are an agentic systems engineer. Optimize for elegance, strong contracts, conceptual correctness, and less code. Prefer deleting bad abstractions to preserving them. Do not add fallbacks, coercions, or defensive code that hides bugs.

# Repository Guidelines

## Core Operating Rules

- Plan before acting: for <=2 files, state a brief plan then implement; for >=3 files, write a step-by-step plan first.
- Read before editing. Search instead of guessing.
- Fix root causes, not local workarounds.
- Prefer the simplest design that satisfies the contract. Reduce surface area, delete dead code, and avoid new concepts unless they clearly pay for themselves.
- Be concise in progress updates and summaries.
- Keep one canonical implementation and one source of truth per concept. Delete unused code and commented-out code.
- Validate only at boundaries: HTTP/gRPC handlers, event consumers, DB results, third-party APIs, `ctx.Value()`, type assertions, and required map lookups. Inside the codebase, trust Goa and construction-time invariants.
- Fail fast on invariant violations. Do not add nil/empty guards, fallback behavior, back-compat fishing logic, or "should not happen" branches for values guaranteed by contracts.
- Do not perform best-effort coercions in runtime/codegen. If a payload, result, or type assertion does not match the contract, return a precise error instead of silently remapping it.
- Configuration belongs in constructors, not environment-variable reads in core logic.
- Keep docs in sync with behavior:
  - User-facing `goa-ai` DSL, runtime, or codegen changes must update `content/en/docs/2-goa-ai/` and translated pages when applicable.
  - Update `README.md` and `DESIGN.md` when behavior changes.

## Language And Code Rules

### Go style

- Go 1.24+ with `go fmt ./...`.
- Group imports with stdlib separate from external.
- Use `lower_snake_case.go`; split large files proactively and prefer <=1000 lines.
- Use short lowercase package names. Exported identifiers and exported struct fields need GoDoc.
- Prefer `any` over `interface{}`.
- Always check errors. Wrap with `%w`, use `errors.Is/As`, and never ignore errors or write `_ = call()`.
- Keep signatures on one line when they fit within about 100 columns.
- Use `len(x) == 0` for slices/maps; do not check nil before `len`.
- Use multi-line blocks. Short literals may stay inline; long literals should use one field per line with trailing commas.
- Prefer exact string comparison; use `strings.EqualFold` only when the external contract is case-insensitive.
- Use modern Go helpers such as `min`, `max`, and `clear` when they simplify the code.

### Structure and comments

- Order declarations as: types, consts, vars, public funcs, public methods, private funcs, private methods.
- Within each category, keep main logic first and helpers last.
- Prefer named helpers or methods over anonymous functions, especially for concurrency.
- Split complex logic into smaller helpers with explicit contracts.
- Every exported type, function, method, and field needs GoDoc.
- Every non-trivial file needs a header comment explaining purpose, invariants, and adjacent-layer contract.
- Non-trivial helpers, especially generator helpers that build `*codegen.File` or resolve ownership/type information, need short contract comments.

### Goa, codegen, templates, and tests

- Never edit `gen/`; regenerate.
- Put validation in the Goa design, not in service code. Avoid `Any`. Service code trusts validated payloads.
- Required arrays must be non-empty. If empty is valid, make the field optional. OneOf/union values must set exactly one variant.
- Do not rely on nil vs empty slices to encode presence.
- Every `Field` must include an inline description string. Prefer `SharedType` for shared types, use DSL `Description(...)`/field descriptions instead of comment-only docs, and add `Example(...)` plus validations where appropriate.
- Prefer codegen-time specialization over runtime interpretation. If the DSL or generator already knows the branch, loop domain, identifier set, metadata, or wiring shape, emit the final code/data directly instead of generating generic runtime logic to rediscover it.
- Apply partial evaluation aggressively: if a branch, collection, or structure is known at generation time, emit only the applicable code. Use template `if`, `range`, and helper composition to specialize the output; do not emit runtime loops or runtime conditionals over static inputs.
- Generated code should expose canonical precomputed artifacts for static facts, such as typed lookups, metadata, routing tables, and configuration. Runtime code should consume those artifacts, not reconstruct them from broader specs on every call.
- Keep runtime branching for truly dynamic inputs only, such as user/model input, network results, database state, and registry-discovered catalogs.
- When runtime dispatch is truly required, prefer a shared runtime library plus generated configuration over duplicating near-identical generated algorithms.
- Generator edits must be section-driven and guard-first: match the target section early and `continue`; avoid redundant `s.Source == ""`-style guards.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goadesign/goa-ai](https://github.com/goadesign/goa-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
