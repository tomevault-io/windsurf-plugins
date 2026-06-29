---
trigger: always_on
description: This file is the durable playbook for Claude across sessions.
---

# jatatui — Claude playbook

This file is the durable playbook for Claude across sessions.

## What this repo is

**jatatui** is a Java port of [ratatui](https://github.com/ratatui/ratatui), the community continuation of [tui-rs](https://github.com/fdehau/tui-rs). It targets ratatui v0.30.0 (the workspace-split release).

This repo is the successor to `tui-scala` (a Java port of the much older 2022-era tui-rs codebase). `tui-scala` will be archived once jatatui is feature-complete.

## Build

- Build tool: bleep `1.0.0-M9` (pinned in `bleep.yaml`'s `$version`). The bleep launcher reads that and uses the right version automatically — no `BLEEP_VERSION` env var needed.
- JVM target for Java code: `--release 21`. The `crossterm` JNI binding stays at `--release 17` (consumer-facing JNI surface).
- Test framework: **JUnit 5** (`org.junit.jupiter:junit-jupiter:5.x`).
- Compile: `bleep compile <project>`.
- Test: `bleep test jatatui-tests`.
- Local snapshot publish: `bleep publish local-ivy` (version derived from `git describe` via dynver — e.g. `0.30.0+14-shaabcdef`).
- Sonatype release: tag `vX.Y.Z` (also doubles as the dynver base for snapshot publishing), push — CI runs `bleep publish sonatype --version X.Y.Z --assert-release`.

## Submodules and what they track

| Submodule | Upstream | Bump policy |
|---|---|---|
| `submodules/ratatui` | `github.com/ratatui/ratatui` | Pinned at `ratatui-v0.30.0` for the initial port. Bump per release thereafter. |
| `submodules/crossterm` | `github.com/crossterm-rs/crossterm` | Pinned at `0.29`. Bump when ratatui-crossterm bumps. |
| `submodules/kasuari` | `github.com/ratatui/kasuari` | Pinned at `v0.4.12`. Bump when ratatui-core bumps `kasuari` workspace dep. |

Submodules are reference material — Rust source we read while porting. They are not built.

## Bleep projects

| Project | Upstream crate | Source | Tests |
|---|---|---|---|
| `crossterm` | n/a (existing JNI binding to crossterm-rs) | `crossterm/src/java/tui/crossterm/` | (smoke-tested via jatatui-crossterm tests) |
| `jatatui-core` | `ratatui-core` | `jatatui-core/src/java/jatatui/core/` | in `jatatui-tests/.../core/` |
| `jatatui-widgets` | `ratatui-widgets` | `jatatui-widgets/src/java/jatatui/widgets/` | in `jatatui-tests/.../widgets/` |
| `jatatui-crossterm` | `ratatui-crossterm` | `jatatui-crossterm/src/java/jatatui/crossterm/` | in `jatatui-tests/.../crossterm/` |
| `jatatui-tests` | n/a | `jatatui-tests/src/java/jatatui/tests/` | self, isTestProject |
| `jatatui-demo` | `examples/apps/*` | `jatatui-demo/src/java/jatatui/examples/` | smoke-tested manually |
| `scripts` | n/a (Scala 2.13 build infra) | `scripts/src/scala/tui/scripts/` | n/a |

Java package roots:
- `tui.crossterm.*` — JNI binding (kept; this is the JNI contract)
- `jatatui.core.*`
- `jatatui.widgets.*`
- `jatatui.crossterm.*` — high-level Backend over `tui.crossterm.*`
- `jatatui.examples.*`

## Publishing coordinates

- groupId: `com.olvind.jatatui`
- artifacts: `crossterm`, `jatatui-core`, `jatatui-widgets`, `jatatui-crossterm`

## Per-source-file porting workflow

For each `.rs` source file in `submodules/ratatui/ratatui-core/src/` or `ratatui-widgets/src/`:

1. **Read the upstream Rust file end-to-end.**
2. **Translate to a Java class** in the corresponding `jatatui-{core,widgets}/src/java/jatatui/{core,widgets}/...` location.
3. **Translate the inline `#[cfg(test)] mod tests`** at the bottom of the file to a JUnit 5 `*Test.java` in `jatatui-tests/src/java/jatatui/tests/{core,widgets}/`. One test class per source file. Method names mirror upstream `fn xxx_yyy()` → Java `void xxx_yyy()` annotated with `@Test`.
4. **Skip Rust doctests** (`/// ``` ` examples in comments). They're Rust-specific executable doc snippets — no equivalent runner in Java. If a doctest demonstrates behavior not covered by an inline test, copy it as a real `@Test`. Doc text (without the executable assertion) becomes Javadoc on the Java method.
5. **Run `bleep compile jatatui-core jatatui-widgets jatatui-crossterm`** — green is a hard requirement.
6. **Run `bleep test jatatui-tests`** — green is a hard requirement.
7. **Commit.** Subject: `Port <crate>/<file>.rs`. Body lists every Rust `#[test]` ported with its name, plus N/A reasons for any that were skipped.

**Hard rule:** every commit that adds Java source under `jatatui-{core,widgets,crossterm}/` must also add corresponding tests under `jatatui-tests/`. No "tests in a follow-up commit". No "deferred". If a file has no inline tests upstream, the commit message says so explicitly.

## Translation conventions

### Hard rules — never violate

- **Nullables are `Optional`, always.** No method returns `null`. No field is `null`. Anywhere upstream uses `Option<T>` → `Optional<T>` in Java. For `Map.get(k)` style lookups: prefer `Optional.ofNullable(map.get(k))` so `null` never escapes the call site.
- **Tuples get dedicated record types.** No `Map.Entry`, no `Object[]`, no generic `Pair<A, B>`. If upstream returns `(Symbol, Row)`, declare `record SymbolAndRow(Symbol symbol, Row row)` with a domain-meaningful name. Tuples without a domain meaning still get a name — invent one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oyvindberg/jatatui](https://github.com/oyvindberg/jatatui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
