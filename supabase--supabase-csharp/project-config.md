---
trigger: always_on
description: The quality bar we are building **toward**. This is a target spec, benchmarked against best-in-class .NET SDKs (Azure SDK, Stripe.net) and the maintainer's Vonage .NET SDK, not a description of the code as it stands today. Where a rule and the existing code disagree, the rule is right and the code is debt: new and changed code meets the bar; untouched code is migrated deliberately, not churned.
---

# C# SDK Conventions

The quality bar we are building **toward**. This is a target spec, benchmarked against best-in-class .NET SDKs (Azure SDK, Stripe.net) and the maintainer's Vonage .NET SDK, not a description of the code as it stands today. Where a rule and the existing code disagree, the rule is right and the code is debt: new and changed code meets the bar; untouched code is migrated deliberately, not churned.

> Companion: [QUALITY_RUBRIC.md](QUALITY_RUBRIC.md) defines when a change is *done*. Each section notes the **From → To** so the direction is explicit.

---

## 0. Guiding principles

These decide any call the specific rules don't cover:

1. **Developer experience is the top priority.** A developer must be able to use the SDK correctly from IntelliSense, without reading its source — via clear types, predictable patterns, and examples in the docs.
2. **"Familiar" means the Supabase house style, expressed in idiomatic C#.** Familiar DX is *not* "throws like other .NET SDKs"; it's the transparent, `{ data, error }`-style contract Supabase developers know across every Supabase SDK, honored through C#'s type system (§9).
3. **Lean on the type system.** Make illegal states unrepresentable: immutability by default, results and optionality expressed as types rather than nulls, exceptions, or convention. C# is a functional language — use it.
4. **Typed contracts over stringly-typed anything.** Requests and responses are types, not dictionaries.
5. **Code reads like a story.** Orchestration says *what* it does (intent), not *how*; mechanics live below in named helpers (§5.5). This is a first-class readability requirement, not a nicety.
6. **The reference SDK's *behavior* is the spec; the *shape* is ours.** Port what supabase-js does, expressed idiomatically (§12).

## 1. Project & target frameworks

- **The monorepo cutover is done and the major (v8) has shipped.** Packages now target `netstandard2.1` as the reach baseline plus `net10.0`. New code may assume the `netstandard2.1` feature set (`IAsyncEnumerable<T>`, full nullable annotations, `Span<T>`).
- **Never assume net10-only APIs** in library code; guard with `#if` or use a `netstandard2.1` equivalent so the baseline target still compiles.

## 2. Solution & folder structure — vertical slices

**From:** flat per-package folders (`Gotrue/*.cs`) mixing clients, models, and options.
**To:** one folder per **operation** (Vonage-style vertical slices):

```
<Feature>/
  I<Feature>Client.cs        // public interface
  <Feature>Client.cs         // internal implementation
  <Operation>/
    <Operation>Request.cs    // immutable request, IRequest
    <Operation>Response.cs   // immutable response
```

- Clients are `internal` classes behind public interfaces (`I<Feature>Client`).
- Each operation is self-contained and is the unit the parity workflow and any future codegen produce.

## 3. Language & nullability

- Nullable reference types are `enable`d everywhere; a nullable warning is a bug.
- Model optionality precisely (`T?` vs non-nullable-and-initialized). Prefer a `Maybe<T>`/optional type over `null` where "absent" is a meaningful state (§9).
- File-scoped namespaces, one public type per file, filename = type name. **Spaces, indent 4** (per the repo-wide `.editorconfig`, which is the machine-enforced source of truth — §13); `dotnet format --verify-no-changes` is the gate. *(The repo-wide config has landed and the former tab-indented packages, e.g. gotrue, are migrated.)*

## 4. Naming

- Standard .NET naming: `PascalCase` types/members, `camelCase` private fields, `IPascalCase` interfaces. **From → To:** the `_camelCase` underscore-prefix convention is retired — new and changed code uses a plain `camelCase` field name (no `_`); existing underscore-prefixed fields are debt, migrated deliberately, not churned.
- **From → To:** existing async methods have no suffix. **New async public methods use the `Async` suffix** (`SignUpAsync`); the suffix-free surface is renamed at the breaking major. Don't mix styles within a new surface.
- Names come from .NET/Supabase vocabulary, not JS transliteration (a JS `onAuthStateChange` callback → a .NET `event`/`IObservable`/`IAsyncEnumerable`).

**Async & cancellation (library correctness):**
- **`ConfigureAwait(false)` on every await in library code.** We ship a library, not app code — never capture a caller's synchronization context. Non-negotiable on new/changed code.
- **Honor the `CancellationToken`, don't just accept it.** Thread it through the entire call path (transport, retries, streaming) and let it actually cancel in-flight I/O. A token parameter that's swallowed is worse than none — it lies to the caller (QUALITY_RUBRIC §7).

## 5. Models, requests & responses — immutable, typed, per-operation

**From:** mutable `{ get; set; }` classes with `[JsonProperty]`, one bag reused for send and receive.
**To:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supabase/supabase-csharp](https://github.com/supabase/supabase-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
