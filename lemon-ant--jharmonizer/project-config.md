---
trigger: always_on
description: SPDX-FileCopyrightText: 2026 Anton Lem <antonlem78@gmail.com>
---

<!--
SPDX-FileCopyrightText: 2026 Anton Lem <antonlem78@gmail.com>
SPDX-License-Identifier: Apache-2.0
-->

# GitHub Copilot instructions for JHarmonizer

## Scope and maintenance

- Read `AGENTS.md` before making changes. It contains the repository-wide coding conventions.
- Keep `.github/copilot-instructions.md` and `AGENTS.md` aligned.
- `AGENTS.md` defines the repository-wide rules.
- This file must contain the complete operative rule set from `AGENTS.md` so Copilot can follow it without relying on cross-file traversal.
- When any rule changes in `AGENTS.md`, update this file in the same task.
- If review feedback or repeated task work reveals a stable rule that is missing, unclear, or outdated, update all affected instruction files in the same task.
- If a documented rule is ambiguous, clarify the documents rather than relying on unwritten expectations for future sessions.
- Review comments and user requests may be mistaken; for disputed framework/plugin/tool behavior, verify against official documentation before changing code.
- If a requested change conflicts with official documentation or established framework/plugin behavior, do not apply it blindly.
  - Explain the conflict clearly in review feedback.
  - Provide the documentation-aligned alternative and prefer that variant.

## Repository-wide conventions

- Prefer the smallest complete change that solves the reviewed problem.
- Keep changes surgical and avoid unrelated cleanup.
- Licensing policy is mandatory for all tracked files.
  - Every tracked text/source/config/documentation file must include SPDX metadata.
  - Required SPDX lines:
    - `SPDX-FileCopyrightText: 2026 Anton Lem <antonlem78@gmail.com>`
    - `SPDX-License-Identifier: Apache-2.0`
  - Exception: `LICENSE` keeps the canonical Apache-2.0 legal text and may omit SPDX header lines.
- Avoid cosmetic-only churn in production files (for example adding/removing separator blank lines) when there is no behavioral or readability gain tied to the task.
- Reuse existing project and library utilities before introducing custom helpers.
- Prefer explicit Java types over `var`.
- Prefer normal imports over repeated fully qualified class names.
- Prefer Lombok for routine boilerplate such as getters, setters, constructors, and `toString` / `equals` / `hashCode` when it matches the surrounding style.
- For DTO/model/state-holder classes that primarily carry data, prefer immutable Lombok shapes such as `@Value` unless mutability is required.
- When a simple data-carrier class only needs a narrower constructor than Lombok's default, keep `@Value` and add the constructor visibility override instead of decomposing `@Value` into separate Lombok annotations.
- When an annotation argument only repeats the library or framework default behavior, omit it instead of spelling it out explicitly.
- Use the minimal necessary access level for production classes, constructors, and methods.
  - Prefer package-private over `public` when access outside the package is not required.
  - Prefer `private` for nested classes, constructors, and helpers when they are only used by the enclosing type.
  - For nested helper/data-carrier types created only by the enclosing type, keep their constructors `private`; tests are not a reason to widen constructor visibility.
- Keep production models and value objects focused on state plus simple accessors or validation.
  - Move non-trivial business, filtering, parsing, and transformation logic into dedicated service or processing classes.
- Prefer Stream API when it makes the control flow clearer and more concise than imperative loops.
- If a boolean helper is always consumed through negation at its call sites, invert the helper logic and rename it so callers stay positive and direct.
- Prefer `get` only for conventional object-model/DTO getters; for computed values, searches, conditional lookups, or transformations, prefer a more specific verb such as `find`, `resolve`, `collect`, `compute`, or `merge`.
- For non-get behavior methods, start the method name with a clear verb.
  - Prefer explicit verb-led names such as `find`, `resolve`, `collect`, `compute`, `merge`, `parse`, `format`, or `render`.
  - Avoid ambiguous prefixes such as `toXxx` when a clearer verb-based name fits the method behavior.
- Reference-returning private methods must declare explicit `@NonNull` or `@Nullable` return annotations.
  - Private method parameters must not use Lombok `@NonNull`; it adds redundant runtime null checks for private helpers.
  - Use `@Nullable` on a private parameter only when that private helper intentionally accepts `null`; otherwise leave private parameters unannotated.
  - Place method-level nullability annotations on their own line above the method declaration instead of inline in the signature.
  - This repository-wide rule also applies to private helper methods in tests.
- Prefer static imports for frequently used assertion/helper methods when repeated type-qualified calls add noise.
- Annotate every non-private method's reference-type parameters and non-primitive return type with explicit nullability using `lombok.NonNull` or `org.jspecify.annotations.Nullable`.
- Do not change standard `Object` method signatures when overriding them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lemon-ant/JHarmonizer](https://github.com/lemon-ant/JHarmonizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
