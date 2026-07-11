---
trigger: always_on
description: Keep rules simple and high-signal. Target common, real `package.json` mistakes. Skip rare edge cases rather than overcomplicating a rule. Prefer few powerful parameterized rules over many near-identical ones (one `dependency-version-range` with a `dependencyTypes` option, not a `no-caret-*` rule per dependency group).
---

# Agents

## Philosophy

Keep rules simple and high-signal. Target common, real `package.json` mistakes. Skip rare edge cases rather than overcomplicating a rule. Prefer few powerful parameterized rules over many near-identical ones (one `dependency-version-range` with a `dependencyTypes` option, not a `no-caret-*` rule per dependency group).

This plugin lints `package.json` via [`@eslint/json`](https://github.com/eslint/json). Rules visit the JSON AST (momoa), not JavaScript.

## Rule anatomy

Each rule is a plain ESLint rule: a module default-exporting `{create, meta}`. `create(context)` returns a standard ESLint visitor object.

Most rules operate on the top-level object:

```js
import {getRootObject, findMember} from './utils/index.js';

const MESSAGE_ID = 'rule-name';

const messages = {
	[MESSAGE_ID]: 'Message with {{placeholder}}.',
};

/** @param {import('eslint').Rule.RuleContext} context */
const create = context => ({
	Document(node) {
		const root = getRootObject(node);

		if (!root) {
			return;
		}

		const member = findMember(root, 'name');

		if (member?.value.type !== 'String') {
			return;
		}

		context.report({
			node: member.value,
			messageId: MESSAGE_ID,
			data: {placeholder: member.value.value},
		});
	},
});

/** @type {import('eslint').Rule.RuleModule} */
const config = {
	create,
	meta: {
		type: 'problem', // or 'suggestion'
		languages: ['json/json'],
		docs: {
			description: 'Enforce ….', // Ends with a period.
			recommended: true, // boolean — see below
		},
		fixable: 'code', // omit if no autofix; add `hasSuggestions: true` for suggestions
		schema: [],
		messages,
	},
};

export default config;
```

`meta.docs.url` is injected centrally in `index.js`; do not set it per rule.

### momoa JSON AST

- `Document.body` is the root value node. `getRootObject(document)` returns the top-level `Object` (or `undefined`).
- `Object` has `members: Member[]`. `Member` has `name` (a `String` node in strict JSON) and `value` (any value node). `getKey(member)` returns the key string.
- `Array` has `elements: Element[]`; `Element` has `value`. Note: `Element` nodes do not carry a `range` — use `element.value` for token/range lookups.
- Value nodes: `String{value}`, `Number{value}`, `Boolean{value}`, `Null`, `Object`, `Array`.
- `context.sourceCode`: `getText(node[, -1, -1] to strip quotes)`, `getRange`, `getLoc`, `getParent`, `getTokenBefore`/`getTokenAfter` (tokens include `{type: 'Comma'}`).
- There is no `eslint-utils` equivalent (JSON has no scopes or expressions).

### `recommended` config level

`meta.docs.recommended` is a boolean. `true` puts the rule in the `recommended` config (reserve for uncontroversial correctness rules). `false` keeps it out (opinionated/stylistic/opt-in rules); it is still in `all`. When unsure, default to `false` and ask.

### Option naming

Name boolean options in the positive `check*` form, never the negated `ignore*`/`skip*`. This does not apply to array/pattern options like `ignore` (a list to ignore), which follow ESLint's conventions. Read options defensively: `const {ignore = []} = context.options[0] ?? {};`.

### Helper naming

- `is*`/`has*`/`should*` return booleans (prefer explicit `false`).
- `get*` returns a value or `undefined`.
- Keep simple/rule-specific helpers local to the rule file. Only promote to `rules/utils/` when clearly general.

## Reusable utilities

`rules/utils/index.js` provides:

- `getRootObject(document)`, `findMember(object, key)`, `getKey(member)` — AST navigation.
- `iterateDependencies(root, types?)` — yields `{groupName, group, member, name}` across dependency groups.
- `dependencyTypes` — the four standard dependency group names.
- `removeMember`/`removeElement` — comma-aware removal (generators yielding fixes).
- `buildReorderedObject(sourceCode, object, orderedMembers, fallbackIndent)` + `isSameOrder` — for sorting fixes that preserve indentation.
- `getIndentString`/`getNewline` — detect the file's formatting.
- `optionsSchema(properties)` + `stringArraySchema` — build a rule's options schema without boilerplate.

Import from `'./utils/index.js'`.

External: `semver`, `validate-npm-package-name`, `spdx-expression-parse`, `detect-indent`.

## Autofix

Provide an autofix only if it cannot change install/runtime behavior. If it could (e.g. changing a version range, reordering `exports` conditions, moving a dependency between groups), provide a `suggest` instead and set `hasSuggestions: true`.

- Build replacement strings with `JSON.stringify(value)` so quoting/escaping is correct.
- Strict JSON has no trailing commas — handle comma tokens manually when adding/removing members or array elements (`removeMember` does this for object members).
- Whole-object reordering fixes must preserve the file's real indentation and newline (read them from the source, or use `getIndentString`/`getNewline`).
- Strict JSON has no comments, so fixes never need to preserve them.

## Rule naming

- `no-` — disallow something (`no-empty-fields`, `no-git-dependencies`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sindresorhus/eslint-package-json](https://github.com/sindresorhus/eslint-package-json) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
