---
trigger: always_on
description: `rangi` — a tiny, zero-dependency, fully synchronous syntax highlighter (fork of Speed Highlight JS). It takes code + a language and returns self-contained HTML (theme colors inlined as `style` attributes) or ANSI for the terminal.
---

# AGENTS.md

`rangi` — a tiny, zero-dependency, fully synchronous syntax highlighter (fork of Speed Highlight JS). It takes code + a language and returns self-contained HTML (theme colors inlined as `style` attributes) or ANSI for the terminal.

## Layout

- `src/highlight.ts` — the core: `eachToken()` (regex rule engine, callback based, internal), `tokenize()` (public wrapper returning `ShjTokenized[]`), `highlightText()` (tokens + line-number gutter), `codeToHtml()` (full block markup). It imports no grammar and no theme: both arrive per call, which is what makes `rangi/core` bundle nothing.
- `src/terminal.ts` — `codeToAnsi()` / `printHighlight()`, same tokens as 24-bit escapes.
- `src/core.ts` — the `./core` entry, re-exporting those functions as they are; their `languages` and `theme` options are required (`ShjCore*Options` in `src/types.ts`, the main option types with those two made required).
- `src/index.ts` — the `.` entry: the same functions, wrapped to fill in the bundled registry and `defaultThemes`. Custom languages are merged over the bundled ones, so they still win and still apply to sub-languages.
- `src/detect.ts` — `detectLanguage()`, scored regex heuristics. Nothing else in `src/` may import it: it is a public function the caller opts into, and a grammar reaching for it would put it in every bundle (`test/bundle.test.ts` fails if one does). A grammar picks its sub-language out of the code itself or leaves it plain.
- `src/languages/*.ts` — one file per grammar, default-exporting `ShjLanguageDefinition` (an array of `[match, type?, sub?]` rule tuples; a rule with a `sub` but no `type` leaves a hole, `[/…/g, , "js"]`). `src/common.ts` exports the shared rules (`num`, `str`, `strDouble`), imported and dropped into a grammar as is.
- `src/languages.ts` — static registry of every grammar; `ShjLanguage` is derived from it. It is the `./languages` entry too, so every grammar is also a named export under its exact registry key (the four that carry a `type` are assembled into `{ default, type }` there); a test asserts the exports and the registry stay in step.
- `src/themes/*.ts` — one file per theme (plain data: `bg`, `fg`, `tokens` per token type); `src/themes/index.ts` re-exports them and derives `ShjThemeName`.
- `src/defaults.ts` — the only two themes the main entry pulls in (`default` light + `dark`, inlined as `light-dark()`).
- `src/tokens.ts` — the token types: `TOKENS` (names, in order) plus one `const` per type holding its index. A grammar refers to a type by the constant, which the bundler inlines to a digit; `tokenName()` in `highlight.ts` maps it back, so nothing outside a grammar ever sees an index.
- `src/types.ts` — all public types.
- `test/` — vitest, asserts exact output strings.
- `test/bundle.test.ts` — bundles `codeToHtml` out of `.` and `./core` with rolldown (a dev dependency) and checks the result against `BUNDLES`: its min+gzip `size`, and the `modules` of `src/` it is allowed to reach, which is what catches an import that leaks a theme or the registry into an entry that should not carry it. The size has 2% of headroom upward and none downward, so a shrink fails too: take the win by recording the size the failure prints. Whenever you do, re-measure the approximate figures in the README (two of them, the feature list and the core entry section) and the one below — they are prose, so nothing else keeps them honest.
- `test/languages/` — one file per grammar, each an inline corpus handed to `testLanguage()` (`_harness.ts`) plus the differences from the judges it is allowed to have. `_judges.ts` holds the cross check against Prism (`refractor`) and Shiki, both dev dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pi0/rangi](https://github.com/pi0/rangi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
