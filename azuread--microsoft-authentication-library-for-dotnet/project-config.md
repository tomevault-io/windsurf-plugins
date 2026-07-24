---
trigger: always_on
description: These rules apply to Copilot code review. Read all rules before commenting.
---

# Code Review Rules

These rules apply to Copilot code review. Read all rules before commenting.

## Review scope

- Only comment on lines added or modified in the PR diff
- Do not comment on pre-existing code unless the PR directly introduces the issue
- Do not comment on style, formatting, or indentation
- Focus exclusively on: bugs, security issues, logic errors, API contract violations
- If unsure whether something is a bug, do not comment
- Prefer no comment over a speculative comment
- Do not re-post a comment already made on an earlier commit in the same PR

## Repo-specific patterns — do NOT flag these

These patterns are correct in this repo. Do not suggest changes:

- `[RunOn]` inherits from `TestMethodAttribute`. Do not flag as missing `[TestMethod]`
- `Client.AppConfig.X` resolves via parent namespace `Microsoft.Identity`. Do not flag as unresolved namespace
- `Assert.IsTrue(bool?)` is a valid MSTest overload. Do not flag nullable bool as a type mismatch
- `Assert.DoesNotContain(substring, value)` — MSTest v4 signature is substring first, value second
- `ConfigureAwait(false)` is intentional in library code. Do not suggest removal

## ConcurrentDictionary.GetOrAdd — always use factory delegate

`GetOrAdd(key, value)` eagerly evaluates the value arg. Flag any call where the second argument is not a delegate/lambda/method group:

- Bad: `pool.GetOrAdd(key, new ExpensiveObject());`
- Good: `pool.GetOrAdd(key, _ => new ExpensiveObject());`

## C# coding standards

- Use `is null` / `is not null` instead of `== null` / `!= null`
- No reflection in product code (`/src`). Acceptable in tests
- Static fields: `s_camelCase` (e.g., `s_knownHosts`)
- Ordinal string comparisons for protocol values, identifiers, cache keys
- Validate inputs at method boundaries (fail fast with specific exception types)
- Do not include secrets/tokens/PII in exception messages or logs
- Use `nameof` instead of string literals for member names

## Testing standards

- MSTest SDK v4 with NSubstitute for mocking
- Use `// Arrange`, `// Act`, `// Assert` comments
- Prefer deterministic tests (no timing flakiness)

## Public API changes

- Update `PublicAPI.Unshipped.txt` for any public API additions/removals
- XML doc comments required on all public APIs
- Maintain backward compatibility

## MSAL-specific rules

- Use certificate-based auth over client secrets when possible
- Use async APIs consistently
- Keep dependencies minimal and well-justified

---

<!-- Everything below this line is for Copilot Chat and Copilot Agent only. -->
<!-- Copilot code review reads only the first 4,000 characters of this file. -->

Carefully review all markdown documents in the ../.clinerules folder. Those are your custom instructions.

---

# GitHub Copilot Agent Skills (Repository Skills)

This repository defines **Copilot Agent Skills** under `.github/skills/`.

## How skills work
- A **skill** is defined by a folder: `.github/skills/<skill-folder>/`
- Each skill must contain a file named **`SKILL.md`**
- `SKILL.md` must start with YAML frontmatter that includes at least:
  - `name`
  - `description`

> Note: Copilot does **not** read `copilot-instructions.md` files inside subfolders.
> Only `.github/copilot-instructions.md` is treated as the repo-wide instructions file.
> Skill content must be inside `.github/skills/**/SKILL.md`.

## How to use skills in Copilot Chat
- Prefer invoking a relevant skill explicitly when available:
  - `@<skill-name> ...your question...`
- If unsure which skill applies, ask:
  - “What skills are available in this repo?”
  - “Which skill should I use for this task?”

## Expectations when using skills
- Follow the skill’s guidance and patterns exactly (APIs, naming, examples).
- If code is requested, provide complete, runnable code with required imports.
- If multiple approaches exist, explain the tradeoffs and recommend one.

---

# MSAL.NET Agent Guidance

## Warning-clean API changes
- The repo builds with `TreatWarningsAsErrors=true`. When adding `[Obsolete]`, `EditorBrowsable`, or other public API annotations, build both the product project and affected test project(s).
- If tests intentionally exercise a newly obsolete API, add a narrow warning suppression around that assertion/test instead of suppressing broadly.

## Downstream compatibility checks
- Before obsoleting, hiding, or changing request-builder authority APIs, telemetry parameters, or query-parameter/cache-key behavior, check known downstream consumers.
- Treat soft-obsolete changes as downstream-breaking when consumers build with warnings-as-errors. Adding `[Obsolete]` with `error: false`, `[EditorBrowsable]`, or analyzer-facing warnings can still break `Microsoft.Identity.Web` package-bump PRs.
- `Microsoft.Identity.Web` is commonly available as a sibling checkout at `D:\source\microsoft-identity-web`; search it for production usages before deciding whether a change is safe.
- Use targeted searches for the exact public API/member names, for example:
  - `rg "WithB2CAuthority|AffectedApiName" D:\source\microsoft-identity-web\src`
  - `rg "Microsoft.Identity.Client" D:\source\microsoft-identity-web\Directory.Packages.props D:\source\microsoft-identity-web\src`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzureAD/microsoft-authentication-library-for-dotnet](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
