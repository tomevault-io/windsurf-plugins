---
trigger: always_on
description: Write Javadoc so that no warnings are produced. Do not rely on suppression flags like `-Xdoclint:none`.
---

# Raoh Project Instructions

## Javadoc

Write Javadoc so that no warnings are produced. Do not rely on suppression flags like `-Xdoclint:none`.

Specifically:

- Always include `@param`, `@return`, and `@throws` tags when the corresponding element exists.
- Do not use heading tags such as `<h3>` — Java 25's javadoc enforces strict heading level rules. Use `<p><strong>...</strong></p>` instead.
- All public methods should have a Javadoc comment to avoid `no comment` warnings.

## Examples

Example code under `examples/` must also have Javadoc and inline comments written in English.
Examples serve as documentation — treat them with the same care as the library itself.

## Git Workflow

- The main integration branch is **`develop`**. All feature branches must be based on `develop` and PRs must target `develop`.
- **`main`** is the release branch only. Never open a PR targeting `main` for feature work.
- Branch naming: `feature/<short-description>` (e.g., `feature/bool-constraints`).
- **ALWAYS** pass `--base develop` when creating a PR — never omit it, as `gh pr create` defaults to `main`:

  ```sh
  gh pr create --base develop --title "..." --body "..."
  ```

- If a PR was accidentally opened against `main`, fix it immediately with `gh pr edit <number> --base develop`.

## Error Code Implementation Checklist

When adding a new error code and its associated decoder constraint, verify all of the following:

- **Meta values must never be null**: `Map.of()` and `List.copyOf()` reject null silently at runtime with `NullPointerException`. If the decoded value may be null (e.g., `nullable(...)` combinator), use `new ArrayList<>(collection)` instead of `List.copyOf()` for meta payloads.
- **Fallback message must be self-contained**: `Issue.message()` is the pre-resolved message stored at decode time. Users who never call `Issues.resolve()` rely on this string alone. Always include the key variable values (e.g., the missing elements, the duplicate values) in the fallback message — do not assume `resolve()` will be called.
- **`messages.properties` templates must reference the same meta keys**: The placeholders in `raoh.<code>=...` must match the keys actually put into the meta map by the decoder. Mismatches silently produce literal `{key}` output.
- **`MessageResolver.DEFAULT` and `messages.properties` must both be updated**: The `ErrorCodesDefaultCoverageTest` reflection test enforces this automatically — a new `ErrorCodes` constant without coverage in both will fail the build.

## Effect Audit

`raoh-effect-audit-maven-plugin` (built first in the reactor, not published) checks every
external member that `raoh`, `raoh-json` and `raoh-jooq` use, right after compilation (#151).
Members are written as the class file refers to them, return type included
(`java.util.List#get(int):java.lang.Object`). When it fails, the message lists entries ready to
paste.

- **Member missing from `effect-audit/catalog.txt`**: file it under the effect its API contract
  gives it, not what its implementation happens to do. `CLOSED` means it runs no code a caller of
  Raoh can supply: not through its receiver, not through an argument whose type a caller can
  implement (a `CharSequence`, a `Collection`, an `Object`'s `hashCode`), not through a class it
  initializes. Anything that may is `DELEGATED`, observing or converting alike; the approval's
  reason is where observing (allowed) and adopting a caller's behaviour (not allowed) are told
  apart. The audit also rejects `CLOSED` / `EXPLICIT` for an overridable method called virtually,
  and requires a note (`  -- why`) on a `CLOSED` / `EXPLICIT` member that takes an argument of a
  type a caller can implement, saying why none of its code runs. Map and Set iteration order is
  not part of any contract Raoh relies on. JDK classes are read from the `--release` API
  (`ct.sym`), so the result does not depend on the JDK running Maven.
- **Use without an approval in `effect-audit/<module>.txt`**: file the `caller -> member` line under
  the existing `[reason]` it fits. An `AMBIENT` use names its caller exactly, descriptor included,
  so one method's approval never covers an overload. An approval covers the caller's use of the
  member as a whole, not a place in its code: the key cannot tell two calls in one method apart, so
  a use is listed once, and its reason states what holds of every call the caller makes to the
  member (a copy "of a collection the caller passed, or of one Raoh built"). The audit checks the
  use mechanically; whether every call still satisfies the reason is the reviewer's to check. Do
  not reshape code to fit a reason; write the reason that covers the use. A call that copies what
  the caller passed is filed under the reason for copies, and a call on such a copy (a
  `List.copyOf` or `Set.copyOf` result may be the JDK collection it was given) or on a collection
  Raoh built under what Raoh created. Add a new reason only when none fits, and never approve a
  delegation that adopts the input's behaviour as the meaning of a conversion; fix the code
  instead (#152 is the model).
- **`AMBIENT` reached from a decoder package**: cannot be approved, whether the decoder reads it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raoh-project/raoh-java](https://github.com/raoh-project/raoh-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
