---
trigger: always_on
description: This plugin only has rules for CSS. Rules that target CSS together with other languages, belong in `eslint-plugin-unicorn`, not here.
---

# Agents

This plugin only has rules for CSS. Rules that target CSS together with other languages, belong in `eslint-plugin-unicorn`, not here.

## Philosophy

Keep rules simple. Target common patterns, skip rare edge cases rather than overcomplicating the rule.

Avoid duplicating rules that already exist in [`@eslint/css`](https://github.com/eslint/css/tree/main/docs/rules) or [Stylelint](https://stylelint.io/user-guide/rules). Before adding a rule, check both. Only add an overlapping rule if it is clearly better, for example simpler, more accurate, or with an autofix.

## Rule anatomy

Rules export a default config object with `create` and `meta`. The `create` function uses `context.on(NodeType, listener)` to register visitors (this is a custom API, not standard ESLint). See the [ESLint custom rules guide](https://eslint.org/docs/latest/extend/custom-rules) for the underlying API.

Key differences from standard ESLint:

- Use `context.on('NodeType', listener)` and `context.onExit('NodeType', listener)` instead of returning a visitor object.
- Listeners return or yield problem objects (`{node, messageId, fix, suggest, data}`) directly. The adapter calls `context.report()` for you.
- Fix functions receive `(fixer, {abort})`. Call `abort()` to bail out of an unfixable case.

The node types are [CSSTree](https://github.com/csstree/csstree/blob/master/docs/ast.md) node types from [`@eslint/css`](https://github.com/eslint/css) (`StyleSheet`, `Rule`, `Atrule`, `Declaration`, `Identifier`, and others).

```js
const MESSAGE_ID = 'rule-name';

const messages = {
	[MESSAGE_ID]: 'Error message with {{placeholder}}.',
};

/** @param {import('eslint').Rule.RuleContext} context */
const create = context => {
	context.on('Declaration', node => {
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
		languages: ['css/css'],
		messages,
	},
};
export default config;
```

Options are accessed via `context.options[0]`. Use `meta.defaultOptions` for defaults (no manual merging).

### `recommended` config level

`meta.docs.recommended` picks the preset that enables the rule. `'unopinionated'` does NOT mean "too opinionated" — it means the opposite:

- **`'unopinionated'`** — Catches bugs: code that does not do what the author intended, such as a typo, a deprecated feature, or a declaration that browsers ignore. In both `unopinionated` and `recommended` (the former is a subset). The default for new rules.
- **`true`** — Style or modernization, or a bug check that also reports intentional patterns (such as fallback declarations). In `recommended` only.
- **`false`** — Off by default, only in `all`. Only for rules that cannot work well in normal projects, such as `no-unknown-animations`, which only sees `@keyframes` in the same file, and `no-descending-specificity`, which reports intentional "specific before general" ordering.

| `recommended` | `unopinionated` | `recommended` config | `all` |
|---|---|---|---|
| `'unopinionated'` | on | on | on |
| `true` | off | on | on |
| `false` (or omitted) | off | off | on |

Every rule should be in `recommended` unless it cannot work well in normal projects. If unsure which level fits, share your recommendation and ask.

The preset configs apply to `**/*.css` files, set `language: 'css/css'`, and add the `@eslint/css` plugin.

Name boolean options in the positive `check*` form (for example, `checkProperties`), never the negated `ignore*`/`skip*` form, so option naming stays consistent across rules. This does not apply to array/pattern options like `ignore` (a list of patterns to ignore), which follow ESLint's own conventions.

### Helper naming

Name helpers after what they return or do:

- `is*`/`has*`/`should*`/`can*`/`needs*` must return booleans. Prefer explicit `false` over `undefined` in predicate helpers.
- `get*Problem` returns one problem object or `undefined`; `get*Problems` returns/yields multiple problem objects.
- `report*` should call `context.report()` directly.
- Avoid `check*` for private helpers. Reserve `check*` for public boolean options, like `checkProperties`.
- Do not combine reporting/yielding with a predicate return. Split into a problem builder and a boolean at the call site.

## Rule languages

Every rule must declare the official [`meta.languages`](https://eslint.org/docs/latest/extend/custom-rules#rule-languages) field as `['css/css']`. `test/package.js` enforces this.

## Reusable utilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sindresorhus/eslint-cssicorn](https://github.com/sindresorhus/eslint-cssicorn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
