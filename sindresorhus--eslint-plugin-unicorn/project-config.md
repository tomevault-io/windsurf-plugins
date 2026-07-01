---
trigger: always_on
description: Keep rules simple. Target common patterns, skip rare edge cases rather than overcomplicating the rule.
---

# Agents

## Philosophy

Keep rules simple. Target common patterns, skip rare edge cases rather than overcomplicating the rule.

## Rule anatomy

Rules export a default config object with `create` and `meta`. The `create` function uses `context.on(NodeType, listener)` to register visitors (this is the Unicorn-specific API, not standard ESLint). See the [ESLint custom rules guide](https://eslint.org/docs/latest/extend/custom-rules) for the underlying API.

Key differences from standard ESLint:

- Use `context.on('NodeType', listener)` and `context.onExit('NodeType', listener)` instead of returning a visitor object.
- Listeners return or yield problem objects (`{node, messageId, fix, suggest, data}`) directly. The adapter calls `context.report()` for you.
- Fix functions receive `(fixer, {abort})`. Call `abort()` to bail out of an unfixable case.

```js
const MESSAGE_ID = 'rule-name';

const messages = {
	[MESSAGE_ID]: 'Error message with {{placeholder}}.',
};

/** @param {import('eslint').Rule.RuleContext} context */
const create = context => {
	context.on('CallExpression', node => {
		return {
			node,
			messageId: MESSAGE_ID,
			data: {placeholder: 'value'},
			fix: fixer => fixer.replaceText(node, 'replacement'),
		};
	});
};

/** @type {import('eslint').Rule.RuleModule} */
const config = {
	create,
	meta: {
		type: 'suggestion',
		docs: {
			description: 'Enforce …',
			recommended: true, // 'unopinionated' (safest, in both presets), true (in recommended only), or false (opt-in)
		},
		fixable: 'code', // or omit; add hasSuggestions: true for suggestions
		schema: [],
		defaultOptions: [{option: 'default'}], // merged automatically
		messages,
	},
};
export default config;
```

Options are accessed via `context.options[0]`. Use `meta.defaultOptions` for defaults (no manual merging).

### `recommended` config level

`meta.docs.recommended` picks the preset that enables the rule. `'unopinionated'` does NOT mean "too opinionated" — it means the opposite:

- **`'unopinionated'`** — Uncontroversial; in both `unopinionated` and `recommended` (the former is a subset). Safest bucket and the default for new rules.
- **`true`** — A more opinionated call, still on by default. In `recommended` only.
- **`false`** — Off by default, only in `all`. For niche or opt-in rules.

| `recommended` | `unopinionated` | `recommended` config | `all` |
|---|---|---|---|
| `'unopinionated'` | on | on | on |
| `true` | off | on | on |
| `false` (or omitted) | off | off | on |

So a rule too opinionated or niche for broad use is `false`, never `'unopinionated'`. If unsure which level fits, share your recommendation and ask.

Name boolean options in the positive `check*` form (for example, `checkProperties`), never the negated `ignore*`/`skip*` form, so option naming stays consistent across rules. This does not apply to array/pattern options like `ignore` (a list of patterns to ignore), which follow ESLint's own conventions.

### Helper naming

Name helpers after what they return or do:

- `is*`/`has*`/`should*`/`can*`/`needs*` must return booleans. Prefer explicit `false` over `undefined` in predicate helpers.
- `get*Problem` returns one problem object or `undefined`; `get*Problems` returns/yields multiple problem objects.
- `report*` should call `context.report()` directly.
- Avoid `check*` for private helpers. Reserve `check*` for public boolean options, like `checkProperties`.
- Do not combine reporting/yielding with a predicate return. Split into a problem builder and a boolean at the call site.

## Rule languages

Every new rule should declare the official [`meta.languages`](https://eslint.org/docs/latest/extend/custom-rules#rule-languages) field, in `"plugin/language"` form, one per line: `['js/js']` for JavaScript/TypeScript-only rules (most), or the languages it supports (for example `['js/js', 'css/css']`, or `['*']` for any file type).

Available identifiers:

- `js/js` — JavaScript and TypeScript
- `css/css` — [`@eslint/css`](https://github.com/eslint/css)
- `json/json`, `json/jsonc`, `json/json5` — [`@eslint/json`](https://github.com/eslint/json)
- `markdown/commonmark`, `markdown/gfm` — [`@eslint/markdown`](https://github.com/eslint/markdown)
- `html/html` — [`@html-eslint/eslint-plugin`](https://github.com/yeonjuan/html-eslint)

Most rules visit JavaScript AST nodes, so `js/js` is all they can support. But when a rule's logic is language-agnostic (filename, raw text, comments, or disable directives), support as many languages as is feasible. Root node types differ per language (`Program` for JS/TS and HTML, `StyleSheet` for CSS, `Document` for JSON, `root` for Markdown), so use `onRoot(context, listener)` to run on every root and `getComments(context)` for cross-language comments (both from `rules/utils/`). For reference, see `prefer-https` (`['*']`, raw-text scan) and `no-empty-file` (per-language root handlers).

## Reusable utilities

Before writing helpers, check these directories:

- **`rules/ast/`** - AST node type checks: `isMethodCall`, `isCallExpression`, `isMemberExpression`, `isFunction`, `isStringLiteral`, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sindresorhus/eslint-plugin-unicorn](https://github.com/sindresorhus/eslint-plugin-unicorn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
