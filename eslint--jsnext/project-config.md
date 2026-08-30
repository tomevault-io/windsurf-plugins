---
trigger: always_on
description: An npm workspace holding a fast, ESLint-compatible toolchain for the latest
---

# jskit

An npm workspace holding a fast, ESLint-compatible toolchain for the latest
JavaScript, TypeScript, and JSX syntax. TypeScript source, bundled with
`esbuild`, tested with `vitest`.

| Package                  | Name                    | What it does                                                                                                                                                                                                                                                                                                                                           |
| ------------------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `packages/jskit`         | `@eslint/jskit`         | The toolkit: parser, scope analyzer, and control flow analyzer, in one package with one entry point.                                                                                                                                                                                                                                                   |
| `packages/jskit-native`  | `@eslint/jskit-native`  | The four buffer producers — `parse()`, `analyze()`, `createGraph()`, `inferTypes()` — plus `validate()`, reimplemented in Rust behind Node-API bindings, writing byte-identical buffers and reporting identical diagnostics. `@eslint/jskit`'s Node entry uses it when it is built and falls back to TypeScript when it is not. See the section below. |
| `packages/jskit-inspect` | `@eslint/jskit-inspect` | Web app (Astro + React) that runs all four in the browser: code in a left-hand editor, AST/scope/flow/type trees in tabs on the right, with the flow tab offering a Mermaid diagram of one chosen execution unit. Its `start`/`build`/`lint:types` scripts build `@eslint/jskit` first.                                                                |

**The four analyses are directories, not packages.** `parse`, `scope`,
`flow`, and `types` split the source, the tests, the documentation, the
scripts, and the benchmarks alike:

```
packages/jskit/
  src/index.ts        the public surface: export * from all four
  src/parse/          tokenizer, parser, validator, ESTree decoder
  src/scope/          the scope walk and the binary scope format
  src/flow/           the control flow walk and the binary flow format
  src/types/          the type walk and the binary type format
  tests/{parse,scope,flow,types}/ integration tests, *.test.ts
  docs/{parse,scope,flow,types}/  api.md, architecture.md, requirements.md
  scripts/{parse,scope}/          the differential conformance runs
  benchmarks/{parse,scope}/       the performance comparisons
```

Everything ships from `src/index.ts` as one bundle, `dist/jskit.js`. Within
`src/`, `scope/` imports `../parse/index.js`, and `flow/` and `types/` import
both — always through the sub-index, never a module inside another
directory. The package is
`sideEffects: false`, so importing one analysis still leaves the others behind;
`tests/scope/tree-shaking.test.ts` proves it against the built bundle.

**The buffer formats describe their headers with the same field names**, so
the scope one is prefixed `SCOPE_H_*`/`SCOPE_HEADER_WORDS`, the flow one
`FLOW_H_*`/`FLOW_HEADER_WORDS`, and the type one
`TYPES_H_*`/`TYPES_HEADER_WORDS`. They are the only names the four surfaces
would otherwise collide on, and `export *` would silently drop a collision
rather than report it — so if you add a constant to one format, check the
others for the name first. `npm run lint:types` catches what slips through.

`flow`'s `createGraph()` reads the two binary buffers directly and returns
a binary control flow graph; `toGraphTree()` is its JSON debugging view and
`FlowBufferReader` its point-query reader. It stores byte offsets into both
input buffers, which is why it accepts scope buffers only from `analyze()`,
never `analyzeTree()`, and why `scope/scope-buffer.ts`'s layout constants are
exported. The format is specified in
[`packages/jskit/docs/flow/architecture.md`](./packages/jskit/docs/flow/architecture.md),
along with the four places it deliberately trades precision for simplicity.
It has no differential conformance suite — there is no reference
implementation to diff against — so its integration tests in
`packages/jskit/tests/flow/` are the contract.

`types`' `inferTypes()` reads the parse and scope buffers and returns a
binary record of what the program states about its types — classification
without a type checker. The `Types` class answers `isNullish()`,
`isTypeOf()`, `isAwaitable()`, and the rest, keyed by a node or `NodeRef`;
`toTypeTree()` is its JSON debugging view and `TypesBufferReader` its
word-level reader. Like `flow`, it accepts scope buffers only from
`analyze()`, and like `flow` it replaces no existing implementation — its
integration tests in `packages/jskit/tests/types/` are the contract, the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eslint/jsnext](https://github.com/eslint/jsnext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
