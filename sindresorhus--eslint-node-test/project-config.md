---
trigger: always_on
description: This plugin lints tests written with the Node.js built-in test runner (`node:test`). The infrastructure (rule adapter, snapshot test harness, doc generation) is adapted from [`eslint-plugin-unicorn`](https://github.com/sindresorhus/eslint-plugin-unicorn); the rules are ported and adapted from [`eslint-plugin-ava`](https://github.com/avajs/eslint-plugin-ava).
---

# Agents

## Philosophy

This plugin lints tests written with the Node.js built-in test runner (`node:test`). The infrastructure (rule adapter, snapshot test harness, doc generation) is adapted from [`eslint-plugin-unicorn`](https://github.com/sindresorhus/eslint-plugin-unicorn); the rules are ported and adapted from [`eslint-plugin-ava`](https://github.com/avajs/eslint-plugin-ava).

Keep rules simple. Target common patterns, skip rare edge cases rather than overcomplicating the rule.

## Detecting `node:test`

`node:test` is import-based. Every rule first resolves the file's imports and then matches calls against the resolved local names. Use the shared helper `rules/utils/node-test.js`:

- `resolveImports(context)` — scans the file's top-level `import` declarations and returns `{locals, namespace, assertNamespace, assertNamed}`. `locals` maps a local identifier to its canonical `node:test` export (`test`, `it`, `describe`, `suite`, `before`, `after`, `beforeEach`, `afterEach`, `mock`). Handles default import (`import test from 'node:test'`), named/renamed imports, and namespace import. CommonJS `require` is not supported. Most rules bail early when the file does not import `node:test`: `if (imports.locals.size === 0 && !imports.namespace) { return; }`.
- `parseTestCall(callExpression, imports)` — classifies a call as a test/suite/hook: returns `{name, kind, modifiers}` where `kind` is `'test'` (`test`/`it`), `'suite'` (`describe`/`suite`), or `'hook'`, and `modifiers` are the chained `.only`/`.skip`/`.todo` identifier nodes.
- `findModifier`, `getTestOptions`, `findOptionsProperty` — for the two ways a modifier is applied: chained (`test.only(…)`) and via the options object (`test('t', {only: true}, fn)`).
- `getTestTitle(call, context)`, `getStaticString(node, context)` — resolve static string titles.
- `getTestCallback(call)` — the inline implementation function (the last function argument).
- `parseAssertionCall(call, imports)` — classifies a `node:assert` assertion: `assert.strictEqual(…)`, bare `assert(…)`, named-import `strictEqual(…)`, or `t.assert.strictEqual(…)`. Returns `{method}`. Assertion rules activate on a `node:assert` import (not necessarily `node:test`), since the advice is correct wherever `node:assert` is used.

Note: subtests created via the test context (`t.test(…)`) are method calls, not imported bindings, so they are intentionally not matched by `parseTestCall`.

## Rule anatomy

Rules export a default config object with `create` and `meta`. The `create` function uses `context.on(NodeType, listener)` to register visitors (this is a plugin-specific adapter API, not standard ESLint). See the [ESLint custom rules guide](https://eslint.org/docs/latest/extend/custom-rules) for the underlying API.

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


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sindresorhus/eslint-node-test](https://github.com/sindresorhus/eslint-node-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
