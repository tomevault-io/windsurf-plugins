---
trigger: always_on
description: Mandatory Nyra workflow — tests, examples, webDocs, test-all.sh (always apply)
---

**Not sure which folder to edit?** See [`docs/contributor-map.md`](../docs/contributor-map.md).
# Nyra change guidelines (mandatory)

Apply on **every** task that touches the compiler, stdlib, CLI, runtime, tests, or user-visible behavior — without waiting for the user to ask.

## Cross-impact warning (mandatory)

**Any code change can break something elsewhere.** A fix in one module, pipeline stage, or test suite is not done until you verify it does not create regressions in other areas.

Before finishing:

- Trace **who else calls or depends on** what you changed (compiler stages, expand passes, stdlib, CLI, snapshots, compiletest, Nyra integration tests).
- Run **more than the test that failed** — at minimum the suites that touch the same subsystem; for compiler/stdlib work prefer `make test-all` or `cargo test --workspace` plus affected Nyra tests.
- If you updated snapshots or relaxed/f tightened comptime or typecheck behavior, re-run **fail suites** (`tests/suite/fail/`) and related gates — a local green test is not enough if another gate still fails.
- Do **not** ship a narrow workaround that fixes one file while silently breaking borrowck, typecheck, codegen IR, or unrelated examples.

**Goal:** never fix a problem in one place while introducing a new problem somewhere else.

## Checklist (complete before finishing)

1. **Tests** — Run relevant suites; for language/compiler work run `make test-all` or at minimum `cargo test --workspace` plus affected Nyra tests.
   - **Special note:** Nyra is by default a zero-types language, like GoLang and JavaScript; this means no types, no structs, no enums, etc, unless the programmer wants to add them for better organization. 
   - **You MUST ensure all language/compiler tests are run both in zero-types mode and with explicit types.** Always cover both ways: programs should run with zero types and also with explicit types, with no issues arising for the user in either style. If a programmer chooses to use one or the other, both must work perfectly and consistently.
2. **Examples** — Add or update an example under `examples/` demonstrating new or changed behavior, showing both zero-types and explicit types usages where applicable.
3. **No regressions** — Confirm the change does not break unrelated features (see **Cross-impact warning** above: run broad enough tests and check dependent code paths).
4. **webDocs** — Update `webDocs/` (and `webDocs/nyra-skill.md` if syntax/stdlib/CLI changed); run `node webDocs/scripts/build-nyra-skill.mjs` and `node webDocs/scripts/build-search-index.mjs` when HTML or skill content changed.
5. **Makefile** — If you add a new test gate, wire it into the root `Makefile` (`make test-all` dependencies).
6. **Project Structure** — Keep project files small, well-organized, and professionally divided. As the project grows, this structure should make future updates, development, and enhancements easy for developers, and prevent problems with large, unwieldy files.
7. Every feature must ensure that the language supports the following six types: types, strings, numbers, arrays, objects, and booleans. Types are always optional; developers are never required to specify them. This requirement must not impact language speed or memory usage: maintaining extremely high performance and very low memory usage is a primary design goal of the language.
8. Since you are using LLVM, you must design stdlib code in a way that makes it easy for LLVM to eliminate dead code.
   - Divide files into micro-modules: Do not put every function into one huge file. The stdlib should be made up of very small functions. At link time, the linker (especially with Thin LTO) will completely remove any function that is not called.
   - Avoid arbitrary dynamic dispatch: Prefer static dispatch (generics that generate specialized code at compile time), since LLVM excels at optimizing and inlining these, eliminating any runtime overhead.

8. By default, the language should support zero types, like GoLang and JavaScript—no types at all, no structs, no enums, etc. However, if the programmer wants to add types for organization, that's excellent and they should be able to do so. But fundamentally, the language is built with zero types by default.

9. And most importantly, type inference happens behind the scenes: the type is inferred very intelligently by the language to make the developer experience (DX) extremely smooth. If the language fails to recognize the type of something, it stops the program with an error and explains to the developer that it could not determine a certain type, and that they must specify the type manually. This is the only case where the developer is required to add the type themselves, and such cases should be very rare and exceptional.
10. **Version bump (selective, not every change)** — Do **not** bump `Cargo.toml` / `CHANGELOG.md` on every commit. Bump the language version **only** when at least one applies:
   - **Bug fix** — a real code problem was found and fixed (user-visible or correctness/runtime/linker/typecheck).
   - **Major update** — a significant, user-facing feature, stdlib addition, ABI/toolchain change, or breaking behavior worth calling out in release notes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyra-lang/nyra](https://github.com/nyra-lang/nyra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
