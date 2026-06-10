---
trigger: always_on
description: Operational guidance for agents working on the `.phr` rewrite rules.
---

# CLAUDE.md

Operational guidance for agents working on the `.phr` rewrite rules.
`README.md` explains *what* the pipeline does and how it is staged
(`1xx`–`7xx`), the shape of a `.phr` file, and the role of each phase — read
it first. This file covers only *where things live, how to add and debug a
rule, how to test, and what to avoid*.

## Where to look

<!-- markdownlint-disable MD013 -->
| Concern                              | File                                                                  |
| ------------------------------------ | --------------------------------------------------------------------- |
| Stream rules (one per file)          | `src/main/resources/org/eolang/hone/rules/streams/Nxx/NNN-name.phr`   |
| Demo / sanity rules                  | `src/main/resources/org/eolang/hone/rules/{none.yml,33-to-42.yml}`    |
| Pinned tool versions                 | `src/main/resources/org/eolang/hone/default-{phino,jeo}-version.txt`  |
| Pipeline orchestration               | `src/main/resources/org/eolang/hone/scaffolding/entry.sh`             |
| Phino invocation loop                | `src/main/resources/org/eolang/hone/scaffolding/rewrite.sh`           |
| Rule discovery and pattern selection | `src/main/java/org/eolang/hone/Rules.java`                            |
| Mojo with all user-facing knobs      | `src/main/java/org/eolang/hone/OptimizeMojo.java`                     |
| End-to-end test fixtures             | `src/test/resources/org/eolang/hone/optimize/streams/*.yml`           |
| Single-rule unit tests               | `src/test/phino/*.yml`                                                |
<!-- markdownlint-enable MD013 -->

## How to add a new rule

1. **Pick the stage.** Decide which of the seven phases (`1xx`–`7xx`,
   described in README) the rule belongs to — or that you are introducing a
   new phase between two — and pick the smallest unused `NNN-` prefix that
   places it correctly. Execution order is alphabetical by filename
   (`Collections.sort` in `Rules.discover()`), so the prefix *is* the
   schedule. Check `streams/` for the nearest neighbours before choosing.
2. **Write the `.phr` file** under
   `src/main/resources/org/eolang/hone/rules/streams/Nxx/` (the `Nxx/`
   subdirectory whose digit matches the rule's hundreds prefix). Start from
   the closest existing rule and keep its header block (`SPDX` +
   `# yamllint disable rule:line-length`).
3. **Run small-steps locally** on a representative `.phi` to see what your
   rule produces:

   ```bash
   mvn -Dhone.small-steps=true -Dhone.rules='streams/*' \
       org.eolang:hone-maven-plugin:build \
       org.eolang:hone-maven-plugin:optimize
   ```

   Inspect `target/hone/phi-optimized/...phi.NN` files to verify the delta.
4. **Add a single-rule unit test** under `src/test/phino/` (a `.yml` pack
   named after the rule) with a minimal input and the expected output.
5. **Add or update an end-to-end YAML** in
   `src/test/resources/org/eolang/hone/optimize/`. These specify the Java
   source to compile, the expected `log` lines, and the expected opcode
   counts (`before` = `javac` bytecode, `after` = optimized bytecode; a count
   of `0` asserts absence). Updating the `after` counts is normal when a rule
   changes how a pipeline is lowered.
6. **Bump the phino version** in `default-phino-version.txt` only if the rule
   depends on syntax or behaviour that ships in a newer phino.

## How to debug a rule

Run a single rule by hand without Maven:

```bash
phino rewrite --max-cycles 1 --max-depth 500 --sweet \
  --rule src/main/resources/org/eolang/hone/rules/streams/4xx/401-fuse.phr \
  /tmp/Foo.phi
```

The rewritten expression is printed on stdout. In small-steps mode
(`<smallSteps>true</smallSteps>` or `-Dhone.small-steps=true`) `rewrite.sh`
calls phino once per rule and saves `Foo.phi.01`, `Foo.phi.02`, … in
`target/hone/phi-optimized/`; `diff Foo.phi.07 Foo.phi.08` then shows exactly
what rule `nnn` did — the canonical way to bisect a misbehaving rule.

## How to extend beyond `streams/`

Two independent mechanisms:

- **Built-in subfolder.** Add a sibling directory next to `streams/` (for
  example `arithmetic/`), drop `.phr` files in with the same conventions, and
  select them with `<rules>arithmetic/*</rules>`. Anything under
  `org/eolang/hone/rules/` is discovered automatically via ClassGraph. The
  selection grammar supports wildcards and negation:
  `streams/4xx/*,!streams/4xx/411-*`.
- **External (extra) rules.** Without rebuilding the plugin, point `<extra>`
  at a directory of `.phr`/`.yml` files. `OptimizeMojo` copies them into a
  `hone-extra/` directory and `entry.sh` appends them *after* the built-in
  selection. Useful for project-local experiments.

## Running the test suite

`pom.xml` sets `<excludedGroups>deep</excludedGroups>` by default, which skips
the end-to-end optimize fixtures (`optimize/streams/*.yml`) and the
single-rule packs (`src/test/phino/*.yml`) — so `mvn test` alone exercises
only about half the suite and silently hides `.phr` regressions. Always run

```bash
mvn -Pdeep test
```

when verifying a rule change; the `deep` profile clears `excludedGroups` so
every `@Tag("deep")` test runs against the real `phino` binary on the host.

## Tools to keep on hand


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [objectionary/hone-maven-plugin](https://github.com/objectionary/hone-maven-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
